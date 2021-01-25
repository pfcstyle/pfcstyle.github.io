---
layout:		post
title:		"Android Studio for Beginer(一)"
description	"The Best Android Tool"
date:		2016-03-26
author:		"PfCStyle"
header-img:	"img/post/2016-03-26/head.jpg"
categories: "Android"
keywords
    - Android
    - Tool
    - Android studio
---

> 工欲善其事，必先利其器

相信使用过eclipse的朋友们都体验过eclipse每次装插件都装不上的痛苦，尤其是配置adt，最是让人头痛，因为还面临着adt和sdk版本不匹配的问题，特别是每次google发布新的sdk，adt就必须跟着升级才行，实在是苦不堪言。终于，历时两年，google终于推出了android studio,完善的插件体统，以及对eclipse等工程的兼容，还有方便的sdk管理，最后再集成了Gradle项目管理，真是处处体现了android studio的强大与方便。

android studio的安装我相信不用多说，我们直接从hello world开始。

# Hello World

![](/img/post/2016-03-26/hello_world.jpg)

如果你是刚刚装好了Android Studio,你应该是在欢迎界面，点击Create New Project,或者你已经打开或者导入过项目了，那就选择File>New>New Project,然后你会看到下图的界面。在Application name中填上Hello World，这里建议是以大写字母开头。Company Domain就是公司域名，Package name是反转的Company Domain加上Application Name。最后修改你的工程路径，本次修改后，路径会记录，下次如果不想修改就不用管了，感觉这个比设置默认的工作路径要方便很多。

![](/img/post/2016-03-26/newproject.png)

next之后是硬件选择界面，Phone and Tablet(手机和平板)是默认被选中的，下面依次是Wear(手表)、TV(电视)、Android Auto(车载应用)、Glass(眼镜),他们每一项都要求设定最低支持的SDK版本，你们可以根据自己的需求来设定，如果自己无法确定，下面还有一个Help me choose,他可以向你展示android各个版本的市场份额，可以帮助你确定你的需求。

![](/img/post/2016-03-26/choose_devices.png)

next之后是模板选择界面，我们这里选择empty Activity

![](/img/post/2016-03-26/template.png)

next之后是设置activity的名称这些，我们就使用默认的就好了。

![](/img/post/2016-03-26/activity.png)

点击完成，我们的hello world就创建成功了！

### 使用虚拟机运行Hello World

Android虚拟设备管理器允许你创建Android虚拟设备（AVDs），然后你可以在你的电脑上运行**模拟器**。模拟和仿真有一个很重要但是微妙的区别。模拟意味着虚拟设备只有一个外形，模拟实际的物理设备如何运作，但是不针对特定的操作系统。IOS开发环境使用模拟器，对于有限数量的设备的平台的IOS来说可能是一个不错的选择。
 
然而对于**仿真器**而言，你的电脑留出一块内存去复制基于仿真器正在仿真设备上的环境。Android Studio使用仿真器，这意味着Android虚拟设备管理器启动一个 Linux内核的大沙箱和整个Android栈为了仿真基于Android物理设备的环境。尽管仿真器提供了一个比模拟器更可靠的环境来测试你的应用程序，但是启动一个AVD需要数分钟，这取决于你电脑的速度。好消息是你的仿真器仍然活跃在内存中，它仍然是有响应的。然而，如果你有Android手机或者平板电脑，我们建议使用物理设备来测试你的应用程序，而不是使用AVD。也就是说，我们首先使用Android虚拟设备管理器创建一个AVD，在后来的章节我们将想你展示如何连接你的物理设备,当然如果你有的话。

下面我们将创建一个仿真器，选择工具栏中的avd manager
 
![](/img/post/2016-03-26/avd_manage.png)
 
打开之后，点击左下角的Create Virtual Device,选择Galaxy Nexus，然后点击Next。下一个界面允许你选择一个系统镜像。选择Lollopop（或最新的API）和x86_64的API,如果你没有，那么点击download下载就好了，android studio会自动为你配置好的。点击Next
 
![](/img/post/2016-03-26/select_virtual.png)
  
![](/img/post/2016-03-26/choose_image.png)

接下来是虚拟机的具体的一些配置参数，点击show Advanced Settings会显示出更多的高级选项。下图中会详细标出，点击finish，恭喜你，你的第一个虚拟机已经创建成功了。

![](/img/post/2016-03-26/virtual_config1.png)

![](/img/post/2016-03-26/virtual_config2.png)

点击绿色按钮运行，选择你刚刚创建的虚拟机，你将会看到hello world。

![](/img/post/2016-03-26/run.png)

![](/img/post/2016-03-26/ok.png)

### 使用真机运行Hello World

使用真机调试的关键是要让你的电脑连接上你的手机，你可能需要安装与你的手机匹配的USB驱动，你可以自己去找一下，也可以让360之类的手机助手帮你安装，现在已经不是问题了。此外，你还需要打开开发人员选项并确保USB调试框被选中。当你成功连接真机之后，你可以在android device monitor中查看你的真机是否出现，并且状态为online，如下图：

![](/img/post/2016-03-26/device_manager.png)

接下来，直接点击运行就好了，Hello World应该成功出现在你的手机上了。
 
 
 
 
 
 
 
 
 
 
 
 