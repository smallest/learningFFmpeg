目录：
1. 前期准备
2. 修改config文件
3. 编写编译脚本
4. 执行编译

### 1. 前期准备
1. 准备linux或者mac环境
2. 下载NDK [NDK官方下载地址](https://developer.android.google.cn/ndk/downloads/index.html)
3. 下载FFmpeg的源码 [官方地址](https://ffmpeg.org/download.html)

### 2. 修改config文件
config文件在ffmpeg源码根目录下，修改config文件是为了指定打包后的名字，不修改的话，编译出来的版本如下libavcodec.so.56 在Android下面是不识别的，linux下面是可以的。因此需要如下修改：

```
SLIBNAME_WITH_VERSION='$(SLIBNAME).$(LIBVERSION)'
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
```
将上面的内容修改如下：
```
SLIBNAME_WITH_VERSION='$(SLIBNAME).$(LIBVERSION)'
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

### 3. 编写编译脚本
用于指定对编译器的配置等。
脚本bulid_android.sh如下：
```
#!/bin/bash
#这里修改为你的ndk的路径
NDK=/home/ndk/android-ndk-r12b
#注意android-9文件夹的版本号，替换好自己的版本号。下面的路径同理 
SYSROOT=$NDK/platforms/android-9/arch-arm/
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64
function build_one(){
 ./configure \
--prefix=$PREFIX \
--enable-shared \
--disable-static \
--disable-doc \
--disable-ffserver \
--enable-cross-compile \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--target-os=linux \
--arch=arm \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
}
CPU=arm
PREFIX=$(pwd)/android/$CPU
ADDI_CFLAGS="-marm"
build_one

```
将脚本放到ffmpeg源码的根目录下。

上面脚本指定了运行的平台和架构，指定了编译器、链接器的前缀，给出了sysroot和编译、链接参数。这里是以arm作为编译目标架构的。

### 4. 执行编译
在ffmpeg源码根目录路径下依次执行以下命令
```
./build_android.sh
make
make install

```
编译好后会在android目录下生成编译出来的so库和对应的头文件。这里编译的是armeabi平台的。

```
├── include
│?? ├── libavcodec
│?? ├── libavdevice
│?? ├── libavfilter
│?? ├── libavformat
│?? ├── libavutil
│?? ├── libswresample
│?? └── libswscale
├── lib
│?? ├── libavcodec.so 
│?? ├── libavdevice.so 
│?? ├── libavfilter.so
│?? ├── libavformat.so
│?? ├── libavutil.so
│?? ├── libswresample.so 
│?? └── libswscale.so

```
