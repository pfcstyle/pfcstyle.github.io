---
layout:		post
title:		"iOS项目总结(八)"
description: "解决4s启动图黑屏"
date:		2016-06-28
author:		"PfCStyle"
header-img:	"img/post/2016-06-28/head.jpg"
categories: "iOS"
keywords:
    - iOS
    - Xcode
    - LaunchScreen
    - launchimage
    - 启动界面适配
    - Black Screen
---

> 不积跬步，无以至千里；不积小流，无以成江海；

今天被分了一个bug,说是iphone4s上的启动画面黑屏，本来想着应该是小菜一碟，因为凭本大侠的水平，怎么可能会搞不定一个静态的启动画面，结果，一整天就这样过去了==！不过好在在下班前1小时搞定了，来做一下总结，日了狗~

# 我的问题

首先说我的问题，非常的amazing。我们的应用是要求同时支持横竖屏的，但是刚开始我们只要求支持横屏，后来又不得不添加竖屏。配置就是我们在target>general>device orientation中又多勾选了一个竖屏(当然，同时支持横竖屏只打个勾是远远不够的，需要你精确控制，这个网上很多，跟本文无关)，然后，打印4s的屏幕尺寸，你会发现这货是横屏的尺寸，因此，根本无法显示启动图(配置的启动图里没有4s的横屏)。解决方式很简单，把device orientation全取消，按顺序，先选择Portrait,然后选择Landscape Left和Landscape Right,记住，顺序很重要！！！你会发现你的启动图出来了！！（从这也可以看出来，苹果的配置文件大部分是字典存储，但是遇到这种多选的，显然是按照队列的方式进行存取的，默认是选择第一个选项）

# 启动画面的适配总结

### iOS7之前(use asset catalog)

iOS8之前我们使用use asset catalog，什么意思呢，简单来说就是最传统的UI做了一堆不同屏幕的图，然后按照苹果官方命名好，添加到你的资源文件夹中，然后Xcode里这样配置：

![](/img/post/2016-06-28/assetcatalog1.png)

那么你启动的时候，就会发现启动界面已经弄好了。但是现在不能用了，因为你会发现，当你在plus运行的时候，你的界面整体放大了！所以，现在这种方式一般是和Launch Screen.xib或者Launch Screen.storyboard 一起使用，下面说。

### iOS7以后(xib或者storyboard 与 use asset catalog一起(兼容iOS7以下))

如果你需要兼容iOS7以下，那么你还需要把上述的也弄一下，去适配相应的硬件。但是如果你不需要，那么你只需要使用Launch Screen.storyboard 或者 Lauch Screen.xib即可，使用autolayout和sizeclass来进行约束适配即可。配置如下：

![](/img/post/2016-06-28/xib.png)

### 都可以使用的(images.xcassts的launch images)

我们项目就是使用本方式，在images.xcassts中右键选择App Icons & Launch Images>new iOS launch image新建Launch Images,创建好了效果如下：

![](/img/post/2016-06-28/imagesassets.png)

在右侧选择你需要适配的所有系统版本，然后填充上对应大小的启动界面即可。注意我的上述问题。

# 其他问题总结(针对方式3)

在我解决我的问题的时候，也是走了不少弯路，总结一下其他可能问题。

1.png图的格式问题

判断方法：打开图片(双击，默认就是使用preview.app打开的),ctrl+i，查看信息，如果你的图片有问题，是这样的：

![](/img/post/2016-06-28/wrong.png)

没有问题的：

![](/img/post/2016-06-28/right.png)

有问题就重换喽。

2.Launch Screen File没有置空

![](/img/post/2016-06-28/launchfile.png)






