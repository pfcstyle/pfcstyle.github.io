---
layout:		post
title:		"android source download on windows"
description	"see through the appearance to perceive the essence"
date:		2016-03-29
author:		"PfCStyle"
header-img:	"img/post/2016-03-29/head.jpg"
categories: "Android"
keywords
    - Android源码
    - windows
    - Android系统
---

> 透过现象看本质

先跟大家推荐一个[网站](http://laod.cn/hosts/2016-google-hosts.html),这里提供了可以翻墙的hosts，毕竟google,大家都懂得。

最近想看看android的源码，于是就去google下载，google提供了具体的环境需求和下载方式，大家可以参考[这里](https://source.android.com/source/initializing.html),但是，google官方提供的这种方式只能用于linux,因为它提供的repo是一个python脚本，里面一些模块是linux特有的，windows无法安装。

一种变通的方式是在windows上安装Cygwin，这是一个模拟linux环境的软件，安装好后再按照google官网说的搭建环境，下载源码即可。

但是我觉得上面的过程都太复杂了，用起来很不方便，后来我看了下google官方提供的repo文件，发现其本质就是先使用git clone下来android源码的[清单文件](https://android.googlesource.com/platform/manifest/),所以，你需要先安装git,git的安装就不说了，我之前的博客已经有介绍过了。假设你已经装好了git,找到你想要放android源码的目录，执行：

{% raw %}

```git
git clone https://android.googlesource.com/platform/manifest
cd manifest
```

{% endraw %}

接下来执行:

{% raw %}

```git
//列出android各个分支版本
git tag 
//使用git checkout 切换到你想要的源码的分支，名称就是git tag列出的名称，比如android4.4.2
git checkout android-4.4.2_r1
```

{% endraw %}

这里所谓的切换分支只是切换到了对应分支的manifest清单文件，接下来，我们将使用清单文件进行源码下来，下面，有请python出场。[下载](https://www.python.org/downloads/)安装python，具体过程我就不说了，很简单。建议安装python2.7，比较稳定。

这里提供一个根据manifest清单文件下载的python脚本，我在网上找到了下载的基础代码，自己添加了断点续传，方便大家使用。

{% raw %}

```python
import xml.dom.minidom
import os
from subprocess import call
import stat

#downloaded source path
rootdir = "F:/Documents/android_src/AndroidCode"

#git program path
git = "D:/Git/bin/git.exe"

dom = xml.dom.minidom.parse("F:/Documents/android_src/AndroidCode/manifest/default.xml")
root = dom.documentElement

prefix = git + " clone https://aosp.tuna.tsinghua.edu.cn/"
suffix = ".git"

if not os.path.exists(rootdir):
    os.mkdir(rootdir)


def rmtree(top):
    for root, dirs, files in os.walk(top, topdown=False):
        for name in files:
            filename = os.path.join(root, name)
            os.chmod(filename, stat.S_IWUSR)
            os.remove(filename)
        for name in dirs:
            os.rmdir(os.path.join(root, name))
    os.rmdir(top)


lastPath_pre = None
lastName = None
lastPath_all = None

for node in root.getElementsByTagName("project"):
    os.chdir(rootdir)
    d_all = node.getAttribute("path")
    last = d_all.rfind("/")
    if last != -1:
        d_per = rootdir + "/" + d_all[:last]
        d_all = rootdir + "/" + d_all
    else:
        d_per = rootdir + "/"
        d_all = rootdir + "/" + d_all
    print d_per
    if os.path.exists(d_all):
        lastPath_all = d_all
        lastPath_pre = d_per
        lastName = node.getAttribute("name")
    else:
        if not lastPath_all == None:
            rmtree(lastPath_all)
            os.chdir(lastPath_pre)
            cmd = prefix + lastName + suffix
            call(cmd)
            lastPath_all = None
        if not os.path.exists(d_per):
            os.makedirs(d_per)
        os.chdir(d_per)
        cmd = prefix + node.getAttribute("name") + suffix
        call(cmd)


```

{% endraw %}

需要注意的是要将里面的git和存放源代码路径都替换为你自己的。上面我使用的是清华的镜像，速度很快，推荐使用，如果不放心，可以自己替换为google的https://android.googlesource.com/，但是表示速度难以忍受。。。