title: tensorFlow运行android demo
date: 2015-11-17 09:59:55
tags: 机器学习
categories: 机器学习
comments: true
---

本月初，google开源了第二代深度机器学习系统tensorFlow。

[https://github.com/tensorflow/tensorflow](https://github.com/tensorflow/tensorflow)

代码中提供了一个android的example，下面我们就来一步步把它运行起来。

------
在代码的这层目录下，我们能看到android的example：

```
tensorflow/tensorflow/examples/android/ 
```

目测是ADT结构的，README中也明确指出了运行demo需要的一些环境：

> ## To build/install/run
As a prerequisite, Bazel, the Android NDK, and the Android SDK must all be installed on your system. The Android build tools may be obtained from: https://developer.android.com/tools/revisions/build-tools.html

我们需要这三个东东：Bazel、Android NDK、Android SDK

其中NDK和SDK的安装就不废话了，网上有一大堆的安装教程，接下来介绍一些Bazel的安装.

那么什么是Bazel呢？具体可以参考这篇博文[Google软件构建工具Bazel FAQ](http://www.cnblogs.com/Jack47/p/bazel-faq.html)

Bazel是一个软件构建工具，tensorFlow就是基于这个构建工具进行编译的。而且如果想要运行Bazel，必须要有java1.8以及支持c++11的编译器才可以。接下来介绍如何安装Bazel：

**下载代码：**

```
$ git clone https://github.com/google/bazel/
```

**编译**

直接执行 ./compile.sh 脚本来编译Bazel

```
$ ./compile.sh
```

如果安装过程中，出现问题，可以参考[这篇文章](http://www.tuicool.com/articles/rMbMRbU)(适用于mac和linux)

安装成功后，将bazel配置到环境变量中，然后按照README继续走：

执行：

```
$ bazel build //tensorflow/examples/android:tensorflow_demo -c opt --copt=-mfpu=neon
```

不出意外的话，会出现错误：

```
"The external label '//external:android/sdk' is not bound to anything" will be reported.
```

因为在tensorflow根目录下，有一个WORKSPACE的文件，我们要更改这个文件中的sdk和ndk的路径。并且将注释打开，例如：

```
android_sdk_repository(
    name = "androidsdk",
    api_level = 22,
    build_tools_version = "23.0.2",
    # Replace with path to Android SDK on your system
    path = "/Users/lijianan/Documents/Android/sdk",
)

android_ndk_repository(
    name="androidndk",
    path="/Users/lijianan/Documents/Android/ndk/android-ndk-r10d",
    api_level=21)
```

继续重新执行构建命令，中间可能会出现api或者build_tools的版本不存在，那么建议去下载相应版本的api和build_tools。

继续执行，应该就会成功了。

生成的apk文件会放在这里：

```
bazel-bin/tensorflow/examples/android/tensorflow_demo_incremental
```

安装到一个5.0以上的android机器里就可以成功运行了：

```
$ adb install -r -g bazel-bin/tensorflow/examples/android/tensorflow_demo_incremental.apk
```

