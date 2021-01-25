---
layout:		post
title:		"iOS项目总结(五)"
description	"教你怎么设置Architectures"
date:		2016-06-19
author:		"PfCStyle"
header-img:	"img/post/2016-06-19/head.jpg"
categories: "iOS"
keywords
    - iOS
    - OC
    - architectures
---

> 不积跬步，无以至千里；不积小流，无以成江海；

在做项目的时候，经常会遇到duplicate symbols for architecture armvxx, Undefined symbols for architecture armxx等的问题，尤其是在添加第三方库的时候，给人一种措手不及的感觉。今天我将总结一下为什么会出现这样的问题，以及如何解决。

# armxx这些都是什么？

这些其实都是iOS设备的指令集，每一种指令集其实都是对应一类处理器硬件，这些所谓的指令，其实就是汇编指令，只是不同指令集会略有不同，导致各个指令集无法完全兼容，相互混用。但是，一般来说，这些指令集是自上而下能够完整兼容的。下面是指令集的介绍：
- armv6
-- iPhone
-- iPhone2
-- iPhone3G
-- 第一代和第二代iPod Touch
- armv7
-- iPhone4
-- iPhone4S
- armv7s
-- iPhone5
-- iPhone5C
-- arm64
-- iPhone5S

# Xcode的相关配置

在build settings中的第一个配置项就是arm相关的，解释如下：

Architecture ： 指你想支持的指令集。
Valid architectures : 指即将编译的指令集。
Build Active Architecture Only : 只是否只编译当前适用的指令集。

只有在目标设备上，才会执行设备对应的指令集。
如果在工程Build Setting的Architectures 中的“Build Active Architecture Only”选择为YES，则即使你设置成armv7 , armv7s同时支持，也只会编译对应指令集的包；若选择NO，则编译器会整合两个指令集到一起，此时的包比较大，但是能在iPhone5上使用armv7s的优化，同时也能适配老的设备。一般都是Debug时“Build Active Architecture Only”选择YES，用当前的架构看代码逻辑是否有问题；而在Release时选择NO，来适配不同的设备。

此外，模拟器并不运行arm代码，软件会被编译成x86可以运行的指令。所以生成静态库时都是会先生成两个.a，一个是i386的用于在模拟器运行，另一个是在真实设备上运行的，然后再用命令将两个.a合并成一个。

# 可能出现的问题

那么，可能出现的问题及原因也就很容易明白了。

- Undefined symbols for architecture arm64

1.你的静态库不支持arm64,但是你的工程支持ram64。
	这样你只能选择替换支持arm64的静态库了，因为现在苹果上架也是要求必须支持64位，所以你没得选择。
	
2.OC与C++混编的时候，调用C++的OC类的实现文件没有更改后缀名为.mm

[更多可能与解决方案](http://stackoverflow.com/questions/19213782/undefined-symbols-for-architecture-arm64)

- duplicate symbols for architecture armv64

1.文件名重复

2.检查是否在#import头文件的时候，不小心把.h写成了.m。

3.build settings中，other linker flags 设置的重复，比如同时包含 -all_load和-ObjC，将-all_load删去即可

注意，这里第三种解决方式并不完美，这样做的话，会使一些外部的静态库，使用objc扩展函数(catagory)的方法失效。例如BaiduMapApi。
如果是有些库使用到了扩展函数(catagory)可以分别对这个库进行加载
使用：-force_load
-force_load BaiduMapApi/libs/Release-iphoneos/libbaidumapapi.a
(BaiduMapApi是添加到当前目录下的)
或
-force_load $(BUILT_PRODUCTS_DIR)/libxxx.a
(这里是直接添加静态库项目源码的做法)

参考博客：

[armv6, armv7, armv7s的区别](http://blog.csdn.net/liangliang103377/article/details/38586485)

[duplicate symbols for architecture armv7解决办法](http://blog.sina.com.cn/s/blog_6f72ff900102v6ai.html)

[iOS解决两个静态库的冲突 duplicate symbol](http://blog.csdn.net/slowfei/article/details/9137811)


