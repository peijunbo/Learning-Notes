# 编译运行

##### 生成头文件

使用下面的命令生成头文件(记得代码里要有native方法，否则是无法生成的)

```shell
javac -h <directory> <source_files>
# example
javac -h . Student.java
```

##### C/C++实现函数并编译出动态链接库

略

##### 执行.class文件

在`-Djava.library.path`参数中指明lib*.so文件的路径

```shell
java -Djava.library.path=./lib -classpath ./src/classes com.peijunbo.jnidemo.Main
```

# 类型介绍及转换

##### 基本类型

下面是一个类型对应表

| Java 类型 | Native 类型 | 描述             |
| --------- | ----------- | ---------------- |
| boolean   | jboolean    | unsigned 8 bits  |
| byte      | jbyte       | signed 8 bits    |
| char      | jchar       | unsigned 16 bits |
| short     | jshort      | signed 16 bits   |
| int       | jint        | signed 32 bits   |
| long      | jlong       | signed 64 bits   |
| float     | jfloat      | 32 bits          |
| double    | jdouble     | 64 bits          |
| void      | void        | not applicable   |

##### jstring -- const char *

```c
// jstring 转为 const char *
// 参数说明：JNIEnv *env, jstring jstr
const char *str = env->GetStringUTFChars(/*str:*/jstr, /*isCopy:*/NULL);

// const char * 转为 jstring
jstring jstr = env->NewStringUTF(str);
```

##### j\<Type\>Array -- \<Type\> *

```c
// jintArray to jint *
// 参数说明：jintArray jarr
jboolean b;
jint *arr = env->GetIntArrayElements(jarr, /*isCopy:*/&b);
int len = env->GetArrayLength(jarr);

// 创建 jintArray
jintArray jarr = env->NewIntArray(2);
jint *arr = env->GetIntArrayElements(jarr, NULL);
arr[0] = 2;
arr[1] = 3;
env->ReleaseIntArrayElements(jarr, arr, /*mode:*/0);
```

###### isCopy参数

官方：因为返回的`elems`数组可能会是Java数组的拷贝，所以对返回数组的更改不一定会反映到原数组中，直到`Release<PrimitiveType>ArrayElements()`被调用。

大致来说就是如果返回的`elems`是拷贝，会将传给`isCopy`的`jboolean`赋值为`JNI_TRUE`否则赋值为`JNI_FALSE`。当返回的是拷贝时，对数组的修改是无法直接体现在原java数组中的，只有调用release函数时才会处理你所进行的修改。

我只遇到过TRUE的情况。个人猜测可能是在内存不充足时为了节省内存会放弃复制并返回原数组。

###### mode参数

官方：`mode`参数决定这个数组应该如何被释放。当`elems`不是原数组的复制时，`mode`参数不会起作用。否则`mode`参数的作用参考下表：

如果`isCopy`为假，说明`elems`就是原数组，修改会直接体现出来，自然而然`mode`参数没有意义。

| 模式         | 行为                              |
| ------------ | --------------------------------- |
| `0`          | 将内容复制回去并释放`elems`       |
| `JNI_COMMIT` | 将内容复制回去但不释放`elems`     |
| `JNI_ABORT`  | 释放`elems`且不将修改复制回原数组 |

# Java类型签名

JNI使用Java虚拟机的类型签名表示。下表显示了这些类型签名。

| Type Signature            | Java Type             |
| ------------------------- | --------------------- |
| Z                         | boolean               |
| B                         | byte                  |
| C                         | char                  |
| S                         | short                 |
| I                         | int                   |
| J                         | long                  |
| F                         | float                 |
| D                         | double                |
| L fully-qualified-class ; | fully-qualified-class |
| [ type                    | type[]                |
| ( arg-types ) ret-type    | method type           |
| V                         | void(return type)     |

例如`String`类的签名`Ljava/lang/String;`

例如下面的java方法:

```
long f (int n, String s, int[] arr);
```

有着下边这样的类型签名:

```
(ILjava/lang/String;[I)J
```

有时候包名中会有`_`或者一些奇怪的字符，所以还有一些转义字符用以识别。

| 转义字符序列 | 表示             |
| ------------ | ---------------- |
| _0XXXX       | Unicode字符XXXX  |
| _1           | 字符“_”          |
| _2           | 签名中的字符“；” |
| _3           | 签名中的字符“[”  |

# 访问Java类属性

##### 函数说明

- 获取jfieldID

`jfieldID GetFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig)`

`jfieldID GetStaticFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig)`

参数`const char *sig`是该属性对应的类型签名。

- 获取属性值

`NativeType Get<type>Field(JNIEnv *env, jobject obj, jfieldID fieldID)`

`NativeType GetStatic<type>Field(JNIEnv *env, jclass clazz, jfieldID fieldID)`

- 修改属性值

`void SetStatic<type>Field(JNIEnv *env, jclass clazz, jfieldID fieldID, NativeType value)`

`void Set<type>Field(JNIEnv *env, jobject obj, jfieldID fieldID, NativeType value)`

##### 使用方法

获取`jclass`->获取`jfieldID`->根据`jfieldID`获取属性值和修改属性值

- 获取属性值

```c
jclass cls = env->GetObjectClass(parent); // jclass
jfieldID fieldID = env->GetFieldID(cls, "name", "Ljava/lang/String;"); // 获取jfieldID
jstring jstr = (jstring)env->GetObjectField(parent, fieldID);
const char *str = env->GetStringUTFChars(jstr, NULL);//获取到属性
```

- 修改属性值

```c
// 延续上一个代码块
char cname[] = "这是新的名字";
jstring jname = env->NewStringUTF(cname);// 创建新的jstring
env->SetObjectField(parent, fieldID, jname);// 修改属性
```



# 访问方法

##### 函数说明

- 调用<静态/实例>方法

下面函数中的[A/V]可以没有，分别是三个函数，什么都不加，加A和加V。三者参数略有不同，可以参考其他参数部分。

`NativeType Call<type>Method[A/V](JNIEnv *env, jobject obj, jmethodID methodID, ...)`

`NativeType CallStatic<type>Method(JNIEnv *env, jclass clazz, jmethodID methodID, ...);`

二者的使用很类似，根据需要传入参数即可。

- 一个很神奇的调用函数（主要是我没有搞懂）

`CallNonvirtual<type>Method[A/V]`

与上一个函数的区别：`Call<type>Method` 例程根据对象的类或接口调用方法，而 `CallNonvirtual<type>Method` 例程根据 `jclass` 参数指定的类调用方法，从中获取方法 ID。 方法 ID 必须从对象的实际类或其超类型之一获得。

> 以上为官方所说，但我测试的结果是只与`jmethodID`参数有关，甚至可以调用这个类不存在的方法。

##### 其他参数

以`Call<type>Method[A/V]`为例

- `Call<type>Method` 直接写参数
- `Call<type>MethodA` 传入参数数组
- `Call<type>MethodV` 传入`va_list`

##### 使用方法

获取`jclass`->获取`jmethodID`->根据`jmethodID`调用方法;

```c
// jobject obj
jclass cls = env->GetObjectClass(obj);// 获取jclass
jmethodID jMethod = env->GetMethodID(cls, "javaMethod", "(I)V");// 通过jclass获取实例方法的ID
jmethodID jStatic = env->GetStaticMethodID(cls, "staticJavaMethod", "(I)I");// 通过jclass获取静态方法的ID
env->CallVoidMethod(obj, jMethod, 2);// 调用实例方法
jint ret = env->CallStaticIntMethod(cls, jStatic, 2); // 调用静态方法
```

# 构造Java对象

##### 函数说明

`jobject NewObject[A/V](JNIEnv *env, jclass clazz, jmethodID methodID, ...)`

[A/V]的参数与方法调用时的相同。

##### 使用方法

获取`jclass`->获取`<init>`方法的`jmethodID`->调用`NewObject`

```java
// Parent类构造函数
public Parent(int i) {
    this.id = i;
}
```

```c
jmethodID initMethod = env->GetMethodID(cls, "<init>", "(I)V");
jint id = 233;
jobject parent = env->NewObject(cls, initMethod, id);
```

