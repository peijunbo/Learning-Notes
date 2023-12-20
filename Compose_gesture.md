# Compose Pointer Input 实现滚动控制

在某些情况下，我们可能希望在禁用 `scrollable` 的同时，利用 `ScrollableState` 的种种方法来自己控制滚动，这篇文章将利用 `Modifier.pointerInput` 实现这个功能。

首先在 `PointerInputScope` 中有 `awaitEachGesture` 方法。

从源码中我们可以看到，每个 `Gesture` 是以所有触摸均抬起作为结束标志。所以这个函数的逻辑是每个手势调用一次我们提供的 `block` 代码块。只要有一个手指没有抬起，就不会进行下一次调用。


```kotlin
suspend fun PointerInputScope.awaitEachGesture(block: suspend AwaitPointerEventScope.() -> Unit) {
    val currentContext = currentCoroutineContext()
    awaitPointerEventScope {
        while (currentContext.isActive) {
            try {
                block()

                // Wait for all pointers to be up. Gestures start when a finger goes down.
                awaitAllPointersUp()
            } catch (e: CancellationException) {
                if (currentContext.isActive) {
                    // The current gesture was canceled. Wait for all fingers to be "up" before
                    // looping again.
                    awaitAllPointersUp()
                } else {
                    // detectGesture was cancelled externally. Rethrow the cancellation exception to
                    // propagate it upwards.
                    throw e
                }
            }
        }
    }
}
```

接下来我们尝试实现基础的滚动。首先定义一个自己的修饰器。对于每个手势，先检测落下事件，然后进入循环。在循环中我们要将每个事件的偏移应用到 `scrollState` 上。取消的条件就是有事件被消耗了，或者是手指全部抬起。在最后添加 `pointerInput` 修饰器来处理事件。同时加上 `verticalScroll` 修饰器进行滚动。因为我们的 `scrollState` 单独并不具有影响布局的功能，它只是一个状态，而 `verticalScroll` 修饰器会根据这个状态修改布局。所以为了实现滚动的效果，我们仍然需要添加这个修饰器，但是为了实现自己控制，我们将 `enabled` 设置为 `false`.

这里需要注意方向的问题。对于安卓而言，坐标原点在屏幕左上角，如下图所示。然后当我们手指上滑的时候，事件的 y 偏移是负数，但是对于我们的布局而言应该是向下滑动，所以要对方向进行反转。

![A grid showing the coordinate system showing the top left [0, 0] and bottom right [width, height]](./imgs/android_coordinates.png)

下面是目前的代码：

```kotlin
fun Modifier.myScroll(
    scrollState: ScrollState
): Modifier = this.composed { // composed 修饰器可以提供 Composable 上下文。传入的 block 的返回值是需要被应用到modifier
    val coroutineScope = rememberCoroutineScope()
    val block: suspend PointerInputScope.() -> Unit = {
        scrollState.scroll { // 提供滚动的上下文
            awaitEachGesture {
                try {
                    awaitFirstDown(requireUnconsumed = false)
                    do {
                        val event = awaitPointerEvent()
                        val canceled = event.changes.any { it.isConsumed }
                        val pan = event.calculatePan()
                        scrollBy(-pan.y) // 进行滚动
                        XLog.d("myScrollable: scrollBy ${-pan}")
                    } while (!canceled && event.changes.any { it.pressed })
                } catch (e: CancellationException) {
                    XLog.e("myScrollable: CancellationException")
                } finally {
                    XLog.d("myScrollable: finally")
                }
            }
        }

    }
  	// composed 会将返回的修饰器与 this 进行组合，所以这里只需要返回 Modifier 就行
    Modifier
        .pointerInput(Unit, block) //添加点击事件处理逻辑
        .verticalScroll(scrollState, enabled = false) //采用默认的滚动，但因为要自己控制，所以 enabled = false
}
```

现在我们已经实现了滚动的控制了，但是你很快就会发现不对。在我们松手之后，滚动就停止了，并没有平时的那种惯性的感觉，在 Compose 中这种动效由 `FlingBehavior` 定义。

要实现这种动效，我们可以使用 Compose 提供的 `ScrollableDefaults.flingBehavior()`，当然也可以自己实现。这里采用默认的动效。

我们先看看 `FlingBehavior` 接口的定义：

```kotlin
interface FlingBehavior {
    /**
     * Perform settling via fling animation with given velocity and suspend until fling has
     * finished.
     *
     * This functions is called with [ScrollScope] to drive the state change of the
     * [androidx.compose.foundation.gestures.ScrollableState] via [ScrollScope.scrollBy].
     *
     * This function must return correct velocity left after it is finished flinging in order to
     * guarantee proper nested scroll support.
     *
     * @param initialVelocity velocity available for fling in the orientation specified in
     * [androidx.compose.foundation.gestures.scrollable] that invoked this method.
     *
     * @return remaining velocity after fling operation has ended
     */
    suspend fun ScrollScope.performFling(initialVelocity: Float): Float
}
```

它只有一个方法，就是 `performFling`。接收的参数是初速度，返回执行动画后剩余的速度。

现在开始着手实现。在上面的代码中加入下面的部分。

```kotlin

fun Modifier.myScroll(
    scrollState: ScrollState
): Modifier = this.composed {
    val flingBehavior = ScrollableDefaults.flingBehavior()
	  val coroutineScope = rememberCoroutineScope()
    /* other code */
    val block: suspend PointerInputScope.() -> Unit = {
        scrollState.scroll {
            awaitEachGesture {
                try {
                 		/* other code */
                } catch (e: CancellationException) {
                    XLog.e("myScrollable: CancellationException")
                } finally {
                  	val velocity = velocityTracker.calculateVelocity()
                  	coroutineScope.launch { 
                        with(flingBehavior) {
                            performFling(-velocity.y) // 滑动与手势方向相反
                        }
                    }
                }
            }
        }
    }
    ScrollableDefaults.flingBehavior()
    Modifier
        .pointerInput(Unit, block)
        .verticalScroll(scrollState, enabled = false)
}
```

现在我们已经能够播放 `Fling` 动画了，但是我们该如何获取合适的速度呢，难道又要去自己实现吗。其实并不需要，如果去查看 `verticalScroll` 的源码就会发现，速度计算是通过 `VelocityTracker` 类实现的。通过 `VelocityTracker.addPosition(timeMillis: Long, position: Offset)` 函数添加位置，最后调用 `VelocityTracker.calculateVelocity(): Velocity` 计算速度就行了。

在这里需要注意速度方向的问题。对于安卓而言，

加上计算速度的功能后，我们的 `Fling` 也就正式实现了。

```kotlin
fun Modifier.myScroll(
    scrollState: ScrollState
): Modifier = this.composed {
    /* other code */
    val block: suspend PointerInputScope.() -> Unit = {
        scrollState.scroll {
            awaitEachGesture {
                try {
                 		/* other code */
                  	event.changes.firstOrNull()?.let { 
                    		velocityTracker.addPosition(it.uptimeMillis, it.position)
                    }
                } catch (e: CancellationException) {
                    XLog.e("myScrollable: CancellationException")
                } finally {
                  	coroutineScope.launch { 
                        with(flingBehavior) { 
                            performFling(-velocityTracker.calculateVelocity().y)
                        }
                    }
                  	velocityTracker.resetTracking()
                }
            }
        }
    }
    ScrollableDefaults.flingBehavior()
    Modifier
        .pointerInput(Unit, block)
        .verticalScroll(scrollState, enabled = false)
}
```


