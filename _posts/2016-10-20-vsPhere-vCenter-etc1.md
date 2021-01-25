---
layout:		post
title:		"ESXi+vCenter6.0虚拟化集群配置(一)"
description	"ESXi的安装与配置管理"
date:		2016-10-20
author:		"PfCStyle"
header-img:	"img/post/2016-10-20/head.jpg"
categories: "Virtulized"
keywords
    - ESXi6.0
    - vCenter6.0
    - vsPhere
---

> 如果你在现实中遇到难以解决的问题，不妨尝试把问题虚拟化一下！

# ESXI安装前准备

首先说下本次实验的物理的结构,画张图吧：

![](/img/post/2016-10-20/v-structure.png)

解释一下，图中的master和client是准备部署puppet的server和agent的，但是本系列文章不会记录这些，只是纯粹介绍vsphere系列的部署，我会抽时间再写些文章记录puppet的使用。图中的Source是记录部署自己的yum仓库。

### 下载vsphere系列软件

方法一，直接去[官网](https://my.vmware.com/cn/web/vmware/info/slug/datacenter_cloud_infrastructure/vmware_vsphere_with_operations_management/6_0)下载，但是比较麻烦，要注册账号什么的，你们可以自己搞搞。

方法二，提供[百度云下载](http://pan.baidu.com/s/1bQ8YeU)

下载ESXI和vCenter即可，其他的本系列文章不会介绍。另外vCenter本次实验使用的是linux版本，即VCSA(Vmware vCenter Server Appliance
),但我也会介绍下windows版本的安装。

### 安装硬件说明

ESXI并不是兼容所有的硬件的，因此，如果你不是在使用虚拟机做实验，那么在配置硬件时，必须要选购ESXI支持的硬件，这里的硬件主要说的是cpu,网卡和存储设备。你可以在[这里](http://www.vmware.com/resources/compatibility/search.php)去查询你将要选购的硬件是否被ESXI兼容。如果你懒得麻烦，那么，英特尔系列的产品是确定兼容的。

# 安装ESXI

安装ESXI跟安装普通系统是没有什么区别的，制作一个启动U盘，然后U盘启动即可。下面直接看图吧，很简单：

![](/img/post/2016-10-20/esxi_install1.png)

![](/img/post/2016-10-20/esxi_install2.png)

![](/img/post/2016-10-20/esxi_install3.png)

![](/img/post/2016-10-20/esxi_install4.png)

![](/img/post/2016-10-20/esxi_install5.png)

![](/img/post/2016-10-20/esxi_install6.png)

![](/img/post/2016-10-20/esxi_install7.png)

![](/img/post/2016-10-20/esxi_install8.png)

![](/img/post/2016-10-20/esxi_install9.png)

![](/img/post/2016-10-20/esxi_install10.png)

![](/img/post/2016-10-20/esxi_install11.png)

# 配置ESXI主机

配置ESXI主机主要是配置网络，按照规划好的来

ESXI主机开机后是这样的，这里下面的ip等网络信息都是已经配置好的。
![](/img/post/2016-10-20/esxi_home.png)

F2进入设置界面
![](/img/post/2016-10-20/esxi_login.png)
进入界面后，选择**Configure Management Network**,然后设置ipv4的地址
![](/img/post/2016-10-20/toIPv4.png)
![](/img/post/2016-10-20/IPv4-configure.png)
配置dns,这里dns我们还没有配置，按照规划先设置好就行，后续还会讲配置域控与dns
![](/img/post/2016-10-20/DNS-configure.png)
设置一下域
![](/img/post/2016-10-20/domain-name.png)

设置shell和ssh选项
![](/img/post/2016-10-20/troubleshoot.png)

![](/img/post/2016-10-20/troubleshoot-configure.png)

![](/img/post/2016-10-20/esxi-shell.png)

# 连接ESXI主机

### Web Client连接

web页面管理还是最方便的，直接在浏览器输入ESXI主机的ip或者域名即可：
![](/img/post/2016-10-20/esxi_web_home.png)

![](/img/post/2016-10-20/esxi_web_login.png)

![](/img/post/2016-10-20/esxi_manage.png)

具体怎么使用下节说。

### desktop Client连接

![](/img/post/2016-10-20/esxi_desktop.png)

![](/img/post/2016-10-20/desktop_manager.png)

具体使用，下节介绍。





