---
layout:		post
title:		"iOS项目总结(六)"
description: "iOS被拒"
date:		2016-06-23
author:		"PfCStyle"
header-img:	"img/post/2016-06-23/head.jpg"
categories: "iOS"
keywords:
    - iOS
    - OC
    - app store
    - swiftsupport
---

> 不积跬步，无以至千里；不积小流，无以成江海；

今天项目提交app store被拒了，总结一下原因。邮件说的原因很明白：

> invalid Swift Support - The SwiftSupport folder is missing. Rebuild your app using the current public (GM) version of Xcode and resubmit it.

这里先总结一下OC项目使用swift代码如何配置：

1. 导入swift文件，xcode会自动提示生成一个oc桥接头文件，点击创建，会自动创建一个`项目名-Bridging-Header.h`
2. 使用时直接引入这个头文件即可，可以像OC一样直接调用swift函数。
3. 在buildsettings中，设置EMBEDDED_CONTENT_CONTAINS_SWIFT = yes

一般来说，设置好上述的3点，使用以及到最后打包都不会有问题了，但是我们还就是出了问题，总结一下。

1. archive之后export时选择`Export as an Xcode Archive`就会在archive包中自动包含swiftsupport文件夹，上面说这个文件夹miss了，就是因为我们选择了`Save Build Products`(我们的问题有点傻，因为第一次提交吗，也算是提醒一下新手吧。当然，如果你们是使用脚本打的包，也会出现类似的问题，手动放进去即可。)
2. swiftsupport中不可以包含libswiftXCTest.dylib，这是app store不允许的，因此，你应该确保你的swiftsupport中没有这个东西。如果有的话，参考下面的解决方案：

   * Open your Xcode project
	* Select Product > Scheme > Edit Scheme
	* Click Build in the left sidebar
	* For your test target, uncheck the 	Archive checkbox
	* Click the Close button
	* Select Product > Archive
	* Submit the latest archive to the App 	Store

