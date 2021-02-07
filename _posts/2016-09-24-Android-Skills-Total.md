---
layout:		post
title:		"Android一些小技巧及小知识点总结(持续更新)"
description: "小技巧，大功能！"
date:		2016-09-24
author:		"PfCStyle"
header-img:	"img/post/2016-09-24/head.jpg"
categories: "Android"
keywords:
    - Android
    - Java
    - jNI
---

> 不积跬步，无以至千里；不积小流，无以成江海；

其实早就想开一个这样的blog,这样每次写代码有感的时候就可以来记录一下。由于拖延等等问题，今天才补上这么个开始，那么，就开始吧，万事，总得有个开始才好继续。

# Android隐藏标题栏的区别

Android隐藏标题栏可以在清单文件里设置：

{% raw %}

```android
<!--这种方式是直接移除了标题栏，不占位-->
<activity android:name=".MainActivity"
            android:theme="@android:style/Theme.Black.NoTitleBar"/>

```

{% endraw %}

也可以在onCreate方法中设置

{% raw %}

```android
//这种方式只是隐藏了  但是还会占位
requestWindowFeature(Window.FEATURE_NO_TITLE);

```

{% endraw %}

### 举例

我们应该都遇到过适配华为等带有虚拟按键的屏幕，这些按键占据了屏幕的底部，使我们的底部布局被遮挡。按照网上大部分的说法是因为沉浸式布局导致的，只需要设置

{% raw %}

```android
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:id="@+id/fl_main"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
             android:fitsSystemWindows="true">//设置这一句

</FrameLayout>

```

{% endraw %}

但是这会导致虚拟按键虚拟按键背景色会变成透明色。必须关闭沉浸式布局才行（可能吧，没有研究）。

可是突然发现你有可能根本没有配置沉浸式布局，也出现了这种情况。这时候，就是我上面说的代码设置隐藏标题栏的占位的原因了。你改为使用xml设置即可更正。

#编译出的gradle问题

### 编译时出现Error:No service of type Factory available in ProjectScopeServices.
	
在根目录的build.gradle中，直接将' classpath com.github.dcendents:android-maven-gradle-plugin:1.3'更新到1.4.1就可以解决问题了。

其实gradle出问题解决方式相对固定：
1. 查看gradle版本号
2. 查看build_tools版本号
3. 查看gradle/gradle-wrapper.properties是否是互联网路径(有些公司可能为了保持统一，将gradle的zip包下到本地，并在此文件中配置为本地路径)
4. 在settings的gradle中查看是否选择了*Use local gradle distribution*  一般改为*Use default gradle wrapper*即可

# viewPager的数据更新

更新的函数就是`mPager.getAdapter().notifyDataSetChanged();`。但是，并非任何情况都会生效。

* Override getItemPosition in your PagerAdapter like this:

```
public int getItemPosition(Object object) {
    return POSITION_NONE;
}
```
这是为了通知viewpager所有的view失效，此时调用`notifyDataSetChanged()`函数就会触发更新。如果你需要使用到这个函数，那么，你应该再去写一个public函数去实现相同的功能。

* to setTag()method in instantiateItem() when instantiating a new view. Then instead of using notifyDataSetChanged(), you can use findViewWithTag() to find the view you want to update.

这种方式相当于你自己管理viewpager的view,然后手动去更新，但是显然不适合用于删除或者添加数据的情况。

# AS2.2及以后版本，Failed to crunch file ！

这个问题的根本原因是文件名（包括路径）太长了，AndroidStudio里路径名不能超过240个字符，所以，尝试把工程放到根目录试试。

# UnsatisfiedLinkError

你可以在[这里](https://docs.oracle.com/javase/7/docs/api/java/lang/UnsatisfiedLinkError.html) 看到这个错误的解释，简单来说，就是虚拟机找不到native函数的声明。这时候，你可以去看看你的library有没有load对，然后看看jni里面的命名是不是正确。

# System.Load(library)找不到

首先，可能是笔误，名字弄错了。

其他隐蔽的可能就是，cpu平台的问题。

arm-v8 => arm64-v8a

arm-v7 => armeabi-v7a

arm-v5 => armeabi

x86 => x86

x86-64 => x86_64

mips => mips

mips-64 => mips64

在编译so时，有时候需要你能选择正确的cpu平台。

或者去过滤你的cpu平台，下面的是删除了64bit平台，防止apk只去64bit中寻找so,导致一些so找不到。当然，这个需要看你具体需要的so包。

```
android {
    ....
    defaultConfig {
        ....
        ndk {
            abiFilters "armeabi", "armeabi-v7a", "x86", "mips"
        }
    }
}
```




