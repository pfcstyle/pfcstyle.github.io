---
layout:		post
title:		"记mapbox-android-sdk的修改编译过程"
subtitle:	"填坑"
date:		2016-12-28
author:		"PfCStyle"
header-img:	"img/post/2016-12-28/head.jpg"
tags:
    - Android
    - mapbox
    - libmapbox-gl
---

> 好记性不如烂笔头

接到一个android地图项目，准备使用[mapbox](https://www.mapbox.com/)作为底图。但是有些地方需要修改一下，主要是读取一些自己的数据等，所以这里得重新编译so及sdk。本篇不会记录如何修改，更不会去讲C++相关的任何知识，只是记录编译过程。
如果你是需要编译其他平台的sdk,看完本篇，也是完全没有问题的。

# 准备工作

* 环境

1. mac OS X EI Capitan(Command)
2. or Linux(Command)
3. or windows Cygwin

本人使用的是环境1。

另外，一个建议是，mac的硬盘比较小，我的是128的固态，编译完成可能有10多G，所以，我最后是把工程放到了移动硬盘里，也建议大家这样做(在编译开始前你就该放到移动硬盘里，否则会因为路径的问题，导致下载的好的包及配置文件失效)。

* 源码下载

在[github](https://github.com/mapbox/mapbox-gl-native)下载源码。仓库tag和branch很多，clone的话很费时间，我个人的建议是选择一个tag或者分支就可以了，直接download zip比较快。
我这里选择的是[android-v4.2.1](https://github.com/mapbox/mapbox-gl-native/archive/release-android-v4.2.1.zip),这是当前最新的android分支。

* 配置必要的环境

既然是要编译android sdk,那么，基础的android的环境配置是肯定要配置好的。主要是两个环境变量:

在主要是两个环境变量:`ANDROID_HOME`和`NDK`的路径。

在*~/bash_profile*的末尾添加：

```shell
//当然  这里是我的路径  大家如果和我的不同  记得换啊
export ANDROID_HOME=/Users/developer/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
export PATH=$PATH:/Users/developer/Library/Android/sdk/ndk-bundle
```

# 开始编译

可以开始编译了。使用终端进入*mapbox-gl-native*(这个名字不是固定的，你下载源码的方式和版本不同，名称都是不一样的，新手不必纠结)的根目录。

```shell
cd ~/Downloads/mapbox-gl-native-release-android-v4.2.1
make android
```

不同make命令对应的cpu平台不同，如下：


```
android=>armeabi-v7

android-lib-$1:

$1=>(arm-v5 arm-v7 arm-v8 x86 x86-64 mips)

arm-v8 => arm64-v8a

arm-v7 => armeabi-v7a

arm-v5 => armeabi

x86 => x86

x86-64 => x86_64

mips => mips

如：make android-lib-x86

如果想编译全部平台:

make apackage (编译android所有cpu架构,建议使用这个)
```

这些都可以从根目录的**makefile**文件里得到，这里粘贴makefile的一部分文件。


```makefile
#### Android targets ###########################################################

ANDROID_ENV = platform/android/scripts/toolchain.sh
ANDROID_ABIS = arm-v5 arm-v7 arm-v8 x86 x86-64 mips

.PHONY: style-code-android
style-code-android: $(BUILD_DEPS)
	node platform/android/scripts/generate-style-code.js

define ANDROID_RULES

build/android-$1/$(BUILDTYPE): style-code-android
	mkdir -p build/android-$1/$(BUILDTYPE)

build/android-$1/$(BUILDTYPE)/toolchain.cmake: platform/android/scripts/toolchain.sh build/android-$1/$(BUILDTYPE)
	$(ANDROID_ENV) $1 > build/android-$1/$(BUILDTYPE)/toolchain.cmake

build/android-$1/$(BUILDTYPE)/Makefile: build/android-$1/$(BUILDTYPE)/toolchain.cmake platform/android/config.cmake
	cd build/android-$1/$(BUILDTYPE) && cmake ../../.. -G Ninja \
		-DCMAKE_TOOLCHAIN_FILE=build/android-$1/$(BUILDTYPE)/toolchain.cmake \
		-DCMAKE_BUILD_TYPE=$(BUILDTYPE) \
		-DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
		-DMBGL_PLATFORM=android

.PHONY: android-lib-$1
android-lib-$1: build/android-$1/$(BUILDTYPE)/Makefile
	$(NINJA) $(NINJA_ARGS) -j$(JOBS) -C build/android-$1/$(BUILDTYPE) all

.PHONY: android-$1
android-$1: android-lib-$1
	cd platform/android && ./gradlew --parallel --max-workers=$(JOBS) assemble$(BUILDTYPE)

apackage: android-lib-$1
endef

$(foreach abi,$(ANDROID_ABIS),$(eval $(call ANDROID_RULES,$(abi))))

.PHONY: android
android: android-arm-v7

.PHONY: android-test
android-test:
	cd platform/android && ./gradlew testDebugUnitTest --continue

.PHONY: android-test-apk
android-test-apk:
	cd platform/android && ./gradlew assembleDebug --continue && ./gradlew assembleAndroidTest --continue

.PHONY: apackage
apackage:
	cd platform/android && ./gradlew --parallel-threads=$(JOBS) assemble$(BUILDTYPE)

.PHONY: android-generate-test
android-generate-test:
	node platform/android/scripts/generate-test-code.js

#### Miscellaneous targets #####################################################

.PHONY: style-code
style-code:
	node scripts/generate-style-code.js

.PHONY: clean
clean:
	-rm -rf ./build \
	        ./platform/android/MapboxGLAndroidSDK/build \
	        ./platform/android/MapboxGLAndroidSDKTestApp/build \
	        ./platform/android/MapboxGLAndroidSDK/src/main/jniLibs \
	        ./platform/android/MapboxGLAndroidSDKTestApp/src/main/jniLibs \
	        ./platform/android/MapboxGLAndroidSDK/src/main/assets

.PHONY: distclean
distclean: clean
	-rm -rf ./mason_packages
	-rm -rf ./node_modules

```

可以看到`.PHONY:`后面的就是各种命令。其他平台类似。

先打针强心剂，在编译过程中，需要下载很多依赖，我在编译完成之后，整个文件夹有10.58G...我用了一整晚。

这样如果顺利的话，编译so就完成了。但是可能会有问题，看末尾问题解释。

另外说下，编译完成后，so文件在`mapbox-gl-native/platform/android/MapboxGLAndroidSDK/src/main/jniLibs`下

![](/img/post/2016-12-28/jniLibs.png)

# mapbox-android-sdk 的打包和使用

* 打包

使用android studio打开platform下的android工程

![](/img/post/2016-12-28/open-android.png)

然后直接运行就好了。

![](/img/post/2016-12-28/open-android-after.png)

在build的output下就可以找到aar文件了。

![](/img/post/2016-12-28/cp-after.png)

* 使用aar

这里我直接还以上述android工程里的测试工程为例。

** 拷贝aar到目标工程的libs(没有自己创建)下

这里我将aar重命名了一下，大家随意，后有图。

** 配置builid.gradle


```gradle
android {

    ···

    repositories {
        flatDir {
            dirs 'libs'
        }
    }
}

dependencies {
//    compile(project(':MapboxGLAndroidSDK')) {
//        transitive = true
//    }

    compile(name: 'mapbox-android-sdk-4.2.1', ext: 'aar')

    ···

    //mapbox aar dependences
    compile 'com.squareup.okhttp3:okhttp:3.4.1'

    // Exclude Guava to avoid an unnecessary transitive dependency
    // See: https://github.com/mapbox/mapbox-gl-native/issues/7129
    compile ('com.mapzen.android:lost:1.1.1') {
        exclude group: 'com.google.guava'
    }

    // Mapbox Android Services
    compile('com.mapbox.mapboxsdk:mapbox-java-services:1.3.1@jar') {
        transitive = true
    }
    //mapbox aar dependences
    ···
}
```

![](/img/post/2016-12-28/cp-config.png)

需要说下的是，打包的aar不会自动打包依赖，**除非是你自己手动添加对应的依赖aar或者jar到目录下**，只是用gradle管理依赖的话，是不会自动打包到aar中的。

** 记得添加上access_token

![](/img/post/2016-12-28/access_token.png)

# 问题解释

* 某个包总是下载失败

我想说的是，多试几次，有代理，vpn什么的都连上，多试几次。从终端输出里可以看出来，这些包都是aws上的，但是亚马逊的服务器对我们来说太不稳定了，所以，试试vpn什么的，会好很多。

* 在x86等手机上提示找不到so

如果你是用`make android`命令来编译的，那么就只有arm-v7的so,64bit的运行没有问题(没有全部测试，用三星S7 edge测试没有问题)，但其他32bit平台就不行了，所以建议你编译上其他平台的包。使用`make apackage`编译。

很多问题在前面步骤中已经自动避过了，现在反而没什么需要多说了，祝好运吧。

