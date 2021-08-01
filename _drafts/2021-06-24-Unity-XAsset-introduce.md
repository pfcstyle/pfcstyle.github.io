---
layout:		post
title:		"Unity - XAsset4.0入门"
description: ""
date:		2021-06-24
author:		"Yawei"
categories: ["Unity"]
keywords:
    - XAsset
---


XAsset是资源管理利器，详细功能看官方的XAsset图：
![](/img/post/2021-06-24/xasset-function.png)


源码地址： https://github.com/xasset/xasset （内含Demo)

# 从Demo开始

下载源码，导入Unity，打开init场景，运行。
![](/img/post/2021-06-24/xasset-demo1.png)

建议开启VFS。

## VFS-虚拟文件系统

VFS使用文件流读取资源，初始化的时候会把索引表全部加载到内存，但不是把所有内容全部读取到内存。

VFS与物理文件性能安全对比：
![](/img/post/2021-06-24/vfs-performance.jpg)

后文源码解读详细解释原理

## 热更

经过VFS选择之后，不管是否打开，都会进入下一个界面：
![](/img/post/2021-06-24/xasset-demo2.png)

点击下载开始下载热更资源：
![](/img/post/2021-06-24/xasset-demo3.png)

这一步，XAsset作者已经配置好了远程的资源服务器地址：
![](/img/post/2021-06-24/xasset-updater.png)

下载完成后，进入资源加载界面：
![](/img/post/2021-06-24/xasset-demo4.png)
下拉框选择任意资源，点击加载

# 自己从头配置一次
接下来，我们按照下面的步骤从头自己走一遍demo, 同时分析下源码。
1. 打包
   1. Apply Rules
   2. Build bundles
2. 部署资源
   1. HFS
   2. 修改updater资源服务器地址
3. 热更