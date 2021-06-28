---
layout:		post
title:		"Android Reverse - Dynamic Debug by Android studio and ideasmali"
description: ""
date:		2019-01-22
author:		"Yawei"
categories: "Android Reverse"
keywords:
    - Android Reverse
    - Android Studio
    - smaliidea
---

# 工具准备

1. Android Studio安装
2. ideasmali插件下载：https://github.com/JesusFreke/smalidea

# 前置操作

在使用Android Studio动态调试前，需要先获得smali代码工程，可以选择使用AndroidKiller(APK改之理)或者Jadx。

AndroidKiller生成smali项目默认路径：AndroidKiller安装目录/projects/[应用名]/Project

Jadx: 导出gradle工程： File -> Save as gradle project

记得先修改这些工程的manifest文件，添加上`android:debuggable="true"`
![](/img/post/2019-01-22/debuggable.png)

# 安装ideasmali插件

![](/img/post/2019-01-22/smaliidea.png)

需要注意的是，新版本的Android Studio已经内置了smali的插件，这会覆盖smaliidea，导致在动态调试时看不到变量值。
根据下图，删除上面这个smali的*.smali配置，为下面这个smali配置上*.smali。
![](/img/post/2019-01-22/smalitype.png)

# Android Studio导入smali工程并调试

选择File->New->Import Project，选择Gradle工程，导入smaili项目即可。

右键将根目录标记为`Source Root`
![](/img/post/2019-01-22/sources_root.png)

File->Project Structure设置Android SDK版本与手机系统版本相同
![](/img/post/2019-01-22/sdkversion.png)

## 配置调试

这里的核心关键是要使`Attach debugger`按钮能用，所以只需要添加一个Android app的配置项即可
![](/img/post/2019-01-22/attachdebugger.png)
![](/img/post/2019-01-22/adddebugandroid.png)

## 开始调试

1. 安装apk到手机或模拟器
```
 adb install output.apk
```
2. 启动调试
```
adb shell am start -D -n 包名/activity全名

# Example
adb shell am start -D -n com.test.example/com.test.example.SplashActivity
```
3. Attach到progress
![](/img/post/2019-01-22/progress.png)
4. 添加断点运行到指定位置就会自动断下来了