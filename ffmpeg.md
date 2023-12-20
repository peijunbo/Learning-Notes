[【FFmpeg 分P教学】转码、压制、录屏、裁切、合并、提取 … 统统不是问题。_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Ft411s7Xa?p=1)

ffmpeg -h 打开帮助菜单

## 1.转换格式

ffmpeg -i input.mov output.mp4

-i 指定输入文件

## 2.提取mp4视频

​	ffmpeg -i in_file_name.mp4 -vcodec copy -an outfile.mp4

- -an 静音

## 3.提取音频

​	ffmpeg -i in.mp4 -vn -acodec copy a.m4a

## 4. 合并音视频

​	ffmpeg -i a.m4a -i v.mp4 -c copy out.mp4

## 5.音频转码

ffmpeg -i in.flac -acodec libmp3lame -ar 44100 -ab 320k -ac 2 out.mp3

- -acodec 指定音频编码器(可不输入，自动识别)

- libmp3lame mp3音频编码器

- -ar 44100 设置音频采样率(可不输入使用默认值原音频的采样率)
- -ab 320k 指定音频比特率(可不输入使用默认值180k)

- -ac 2 设置声道数为2声道(可不输入使用默认值原音频的声道数)

## 6.视频压制

ffmpeg -i in.webm -s 1920x1080 -pix_fmt yuv420p -vcodec libx264 -preset medium profile:v high -level:v 4.l -crf 23 -acodec aac -ar 44100 -ac 2 -b:a 128k out.mp4

- -s 缩放视频尺寸
- -pix_fmt yuv420p 设置视频颜色空间 ffmpeg -pix_fmts查看具体参数
- -vcodec libx264 设置视频流的编码器
- -preset 编码器预设。影响编码算法精度与运行速度。参数:ultrafast superfast veryfast faster fast medium slow slower veryslow placebo
- -profile:v 指定编码器配置
- -level:v 对编码器配置的限制(一般1080p视频就用4.1)
- -crf 码率控制模式。crf为恒定速率因子模式。参数可从0~51中任选，数字越小，质量越高，0为无损画质，一般在18~28内选
- -r 30 设置视频帧率每秒三十帧 

# 编译

## ffmpeg编译过程

下载ffmpeg源码

使用下面的编译脚本

```sh
#!/bin/bash
set -x
# 目标Android版本
API=30
ARCH=arm64
CPU=armv8-a
TOOL_CPU_NAME=aarch64
#so库输出目录
OUTPUT=/home/peijunbo/software/ffmpeg-5.1.2/android/$CPU
# NDK的路径，根据自己的NDK位置进行设置
NDK=/home/peijunbo/Android/Sdk/ndk-bundle
# 编译工具链路径
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64
# 编译环境
SYSROOT=$TOOLCHAIN/sysroot

TOOL_PREFIX="$TOOLCHAIN/bin/$TOOL_CPU_NAME-linux-android"
 
CC="$TOOL_PREFIX$API-clang"
CXX="$TOOL_PREFIX$API-clang++"
OPTIMIZE_CFLAGS="-march=$CPU"
function build
{
  ./configure \
  --prefix=$OUTPUT \
  --target-os=android \
  --arch=$ARCH  \
  --cpu=$CPU \
  --disable-asm \
  --enable-neon \
  --enable-cross-compile \
  --enable-shared \
  --disable-static \
  --disable-doc \
  --disable-ffplay \
  --disable-ffprobe \
  --disable-symver \
  --disable-ffmpeg \
  --cc=$CC \
  --cxx=$CXX \
  --sysroot=$SYSROOT \
  --extra-cflags="-Os -fpic $OPTIMIZE_CFLAGS" \

  make clean all
  # 这里是定义用几个CPU编译
  make -j8
  make install
}
build
```

报错

```
STRIP   install-libavdevice-shared
strip: Unable to recognise the format of the input file `/home/peijunbo/software/ffmpeg-5.1.2/android/armv8-a/lib/libavdevice.so'
make: *** [ffbuild/library.mak:120: install-libavdevice-shared] Error 1
```

去到NDK的目录下找到对应的strip

修改源码ffbuild目录下的config.mak

```makefile
STRIP=strip
# 改为
STRIP=/home/peijunbo/Android/Sdk/ndk-bundle/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android-strip
```

## 再次尝试编译

脚本

```sh
#!/bin/sh
# NDK 所在的路径
NDK=/home/peijunbo/Android/Sdk/ndk/21.4.7075529
# 需要编译出的平台，这里是 arm64-v8a
ARCH=aarch64
# 支持的最低 Android API
API=21
# 编译后输出目录，在 ffmpeg 源码目录下的 /android/arm64-v8a
OUTPUT=$(pwd)/android/arm64-v8a
# NDK 交叉编译工具链所在路径
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64
SYSROOT_L=$TOOLCHAIN/sysroot/usr/lib/aarch64-linux-android
GCC_L=$NDK/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/lib/gcc/aarch64-linux-android/4.9.x
# 作者：咩咩熊
# 链接：https://juejin.cn/post/6990246430682120223
# 来源：稀土掘金
# 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
build() {
   ./configure \
   --target-os=android \
   --prefix=$OUTPUT \
   --arch=$ARCH \
   --sysroot=$TOOLCHAIN/sysroot \
   --disable-ffplay \
   --pkg-config="pkg-config" \
   --disable-debug \
   --disable-doc \
    --enable-cross-compile \
    --enable-gpl \
  --disable-shared \
   --enable-static \
   --extra-cflags="-fpic -I$OUTPUT/include" \
   --extra-ldflags="-lc -ldl -lm -lz -llog -lgcc -L$OUTPUT/lib" \
   --cross-prefix=$TOOLCHAIN/bin/aarch64-linux-android- \
 --cc=$TOOLCHAIN/bin/aarch64-linux-android$API-clang \
     --cxx=$TOOLCHAIN/bin/aarch64-linux-android$API-clang++ \

   make clean all
   make -j12
   make install
}

package_library() {
   $TOOLCHAIN/bin/aarch64-linux-android-ld -L$OUTPUT/lib -L$GCC_L \
    -rpath-link=$SYSROOT_L/$API -L$SYSROOT_L/$API -soname libffmpeg.so \
    -shared -nostdlib -Bsymbolic --whole-archive --no-undefined -o $OUTPUT/libffmpeg.so \
    -lavcodec -lpostproc -lavfilter -lswresample -lavformat -lavdevice -lavutil -lswscale -lgcc \
      -lc -ldl -lm -lz -llog \
      --dynamic-linker=/system/bin/linker
    # 设置动态链接器，不同平台的不同，android 使用的是/system/bin/linker
}

build
package_library
```

生成的.so库放在项目中后多次报错

```
ld: error: undefined symbol: avdevice_register_all
```

查看报错信息，多个含有avdevice的变量名均报错为定义。

返回查看脚本，发现没有-lavdevice，于是尝试加上。加上后成功编译。

# 后记

再次编译，再次报错

### file ......./aarch64-linux-android-nm not found

不知道为什么Andoird把ndk里的这几个文件命名改了。可以参考[将 NDK 与其他构建系统配合使用  | Android NDK  | Android Developers](https://developer.android.com/ndk/guides/other_build_systems?hl=zh-cn)。修改的有下边这几个，都是把前缀直接改成了llvm。

```sh
AR=$TOOLCHAIN/bin/llvm-ar

STRIP=$TOOLCHAIN/bin/llvm-strip

NM=$TOOLCHAIN/bin/llvm-nm

RANLIB=$TOOLCHAIN/bin/llvm-ranlib
##然后./configure 后加入参数 --nm=$NM --ar=$AR...
```

经过测试，似乎也可以更改`--cross-prefix=$TOOLCHAIN/bin/llvm-`

### fatal error: 'vulkan_beta.h' file not found

方法1：配置configure脚本时禁用vulkan:` --disable-vulkan`

方法2：配置`extra-cflags`时添加 `-DVK_ENABLE_BETA_EXTENSIONS=0`，防止引用`vulkan_beta.h`头文件

### 过滤ABI

在build.gradle中

```groovy
android {
  ...
    defaultConfig {
      externalNativeBuild {
            cmake {
                cppFlags ""
                abiFilters "arm64-v8a"
            }
       }
    }
}
```

### 添加库路径

```
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/cpp/lib']
        }
    }
}
```
