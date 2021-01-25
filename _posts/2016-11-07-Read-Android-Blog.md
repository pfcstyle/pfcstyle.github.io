---
layout:		post
title:		"Read-Android-Blog"
description	"the heart"
date:		2016-11-07
author:		"PfCStyle"
header-img:	"img/post/2016-11-07/head.jpg"
categories: "Android"
keywords
    - android
    - blog
---

> 站在巨人的肩膀上

最近在[掘金](http://gold.xitu.io/)上读了一些好的android blog,有些心得，总结一下。

# [［译］开发安卓Apps，我所努力学习到的三十多条宝贵经验](http://yifeng.studio/2016/10/27/android-develop-30-things-that-experience-made-me-learn-the-hard-way/)

其中的某些推荐博客需要翻墙才可浏览。

### 选择与使用第三方库

1. 你应该选择大众的选择，一是因为经得起考验，二是因为用的人多，总会用贡献者加入，作者维护起来也会更加用心。
2. 选择可信的作者或者组织。
3. 选择文档比较全面的。
4. 选择专精某一功能的库，而不是大而全的库。否则这个库一旦停止维护，你可能要面临全部重构的危险。
5. 使用第三方库时，你应该对其再次封装，防止在你的工程需要替换这个库的时候导致的大范围代码修改。
6. 一定要理解第三方库的原理，但不到万不得已，一定不要修改第三方库！
7. 应该养成积累自己的轮子的习惯。

### 避免过度绘制

首先你要学会使用android的一个检测工具：*android手机=>开发者选项=>调试GPU强制渲染(Debug GPU Overdraw)* 打开此选项之后，你会发现自己的手机屏幕充满了各种色块,如下图：

![](/img/post/2016-11-07/overdraw.png)
本图来源于https://riggaroo.co.za/optimizing-layouts-in-android-reducing-overdraw/

各种颜色的含义：
* **原色**：没有过度绘制
* **蓝色**：*1xOverdraw*-绘制了2次
* **绿色**：*2xOverdraw*-绘制了3次
* **粉色**：*3xOverdraw*-绘制了4次
* **红色**：*4xOverdraw*-绘制了5次

ok,现在已经知道了如何绘制，那么接下来就是如何fix

这个要根据具体的项目来解决，其实无非是一个原则，重复绘制的地方看如何避免绘制。一些常见的问题，比如你设置的背景色是否真的需要？你设置的背景图片，你的某些空间是否真的需要一直显示？还有，很重要的一点，你得有一个好的产品设计。
你可以从[这里](https://riggaroo.co.za/portfolio/book-dash-android-app/)得到更多关于过度绘制的信息

### 如何分包(按功能)
这个，怎么说呢，各执一词，我从android转到ios,又从ios转回android,就我的经验来说，我觉得应该按照功能来分包更加的科学，如何分包，解决的无非是维护时如何快速寻找到目标的问题。显然，如果你按照功能模块进行分包，可以快速的定位一个区域，然后，在功能模块的包内，你再根据你选用的设计模式来分包，这样就能很快的找到你的目标。如果你是刚开始就按照设计模式分包，甚至按照控件分包，那么，等待你的，就是眼花缭乱。

### 如何加快gradle的编译速度
现在的android的开发者大多都已经转到android studio阵营了，而且不得不承认，android studio比eclipse要更加专业，功能也更强大。但是，当你的项目大到一定程度时，每次要启动一次应用都会变得异常煎熬。

在项目根目录下的gradle.properties中加入：


```gradle
//# The Gradle daemon aims to improve the startup and execution time of Gradle.
//# When set to true the Gradle daemon is to run the build.
org.gradle.daemon=true

//# Specifies the JVM arguments used for the daemon process.
//# The setting is particularly useful for tweaking memory settings.
//# Default value: -Xmx10248m -XX:MaxPermSize=256m
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

//# When configured, Gradle will run in incubating parallel mode.
//# This option should only be used with decoupled projects. More details, visit
//# http://www.gradle.org/docs/current/userguide/multi_project_builds.html
//#sec:decoupled_projects
org.gradle.parallel=true

//# Enables new incubating mode that makes Gradle selective when configuring projects.
//# Only relevant projects are configured which results in faster builds for large multi-projects.
//# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:configuration_on_demand
org.gradle.configureondemand=true

```

具体的原理我就不再记录了，不是我想关注的，如果你想要了解，请看[这里](https://medium.com/@cesarmcferreira/speeding-up-gradle-builds-619c442113cb#.n6fmf1pv6)（需要梯子）

### SVG 代替 PNG

当年xcode6开始支持pdf作为矢量图时，着实给我我们的UI省了一大把力，UI只需要确定一个屏幕尺寸，出一套pdf图给我们就可以了，xcode会自动生成3个尺寸的png给我们，而我们的调用仍然想以前一样简单。

现在我看到android也可以这样搞了，给我们一套svg或者psd的图，我们也可以自己生成其他尺寸的图，同样也是不会有任何锯齿的。不过遗憾的是4.4及以下的系统是不支持的，所以恐怕现在还是不能大范围使用的，毕竟5.0以下系统还不能直接抛弃。

# [Android App优化之内存优化(序)](http://www.jianshu.com/p/48475df838d9)

至于说GC机制以及系统切换APP的内存管理机制，这里都不会解释，不懂得可以参考上述博文，写的很详细生动。

我要总结的是：*为了不要让系统kill掉我们的App, 可以从进程级别, 内存消耗量等几个方面进行优化。*

### 内存监测工具Memory Monitor

![一图看懂Memory Monitor](/img/post/2016-11-07/memorymonitor.jpg)
此图来源于:http://blog.lmj.wiki/2016/10/25/app-opti/app_opt_mat/

* 		① GC按钮, 点击执行一次GC操作.
* 		② Dump Java Heap按钮, 点击会在该调试工程的captures目录生成一个类似这样”com.anly.githubapp_2016.09.21_23.42.hprof”命名的hprof文件, 并打开Android Studio的HPROF Viewer显示该文件内容.
* 		③ Allocation Traking按钮, 点击一次开始, 再次点击结束, 同样会在captrures目录生成一个文件, 类似”com.anly.githubapp_2016.09.21_23.48.alloc”, alloc后缀的文件, 并打开Allocation Tracker视图展示该文件内容.

这里只简单提一下android studio自带的监测工具，具体使用，可以参照上面的博文系列。

### 内存泄漏的常见可能

1. **Context泄漏**。某些全局对象没有使用Application级别的对象，而是使用的指定的activity的Context，导致activity难以回收。
2. **内部类泄漏**。当内部类持有当前类的引用，并且内部类的生命周期长与当前类，就会导致当前类的内存泄漏。如Handler泄露, Thread泄露等。其实，当我们发生这种错误时，一般lint会进行提示，并且建议我们使用使用Static + WeakReference的方式，这正是正确的解决方式，而不是使用@SuppressLint("HandlerLeak")来逃避提示。
3. **Register泄漏**。对于观察者，广播，Listener等，添加与删除没有成对出现，从而导致的内存泄露，这个只能希望程序员们多长点心了。
4. **资源泄漏**。当你操作文件或者数据库等等时，打开资源之后，请一定记得关闭资源。
5. **Bitmap泄漏**。Bitmap没有及时的调用recycle()回收导致内存泄漏。

### 有效使用内存的建议

* 	合理使用ServiceService的及时关闭可以让我们节省内存消耗, 对于一次性的任务, 建议使用IntentService.
* 	使用优化后的数据容器使用Android提供的SparseArray, SparseBooleanArray, LongSparseArray来代替HashMap的使用.关于HashMap，ArrayMap，SparseArray, [这篇文章](http://www.jianshu.com/p/7b9a1b386265)有个比较直观的比较, 可以看下.
* 	少用枚举enum结构相比于静态常量(static final), enum会耗费双倍的内存.
* 	避免创建不必要的对象诸如一些临时对象, 特别是循环中的.
* 	考虑实现onTrimMemory(), 在此根据当前的内存状态做些处理.
* 	Bitmap的合理有效使用.对于Bitmap的使用, 建议直接查看官方开发文档中的[高效显示Bitmap](https://developer.android.com/training/displaying-bitmaps/index.html)(需翻墙).





