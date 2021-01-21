---
layout:		post
title:		"史上最简单的VPN搭建-softether"
subtitle:	"在VPS上搭建自己的VPN"
date:		2016-04-25
author:		"PfCStyle"
header-img:	"img/post/2016-04-25/head.png"
tags:
    - VPN
    - VPS
    - SoftEhter
    - IPV6
    - AWS
    - CE2
---

> 所有的墙都是纸老虎-PfCStyle

我真是觉得很悲催，眼望着马上要毕业了，我们学校网络中心发了一个通告，从此，每月20元不限流量的舒服日子结束了，以后要根据流量算每月的网费了，再也不能随心所欲的看想看的片了。。。

然而，作为一个崇尚自由的程序猿，面对一切封锁，都要打破！打破！打破！于是我伸展一下那过膝的双臂，准备翻墙了。

![](/img/post/2016-04-25/fanqiang.jpg)

大家都知道，大学的ipv6的流量一直是免费的，而我，将要利用的就是这个。下面就是我的翻墙历程。


# 从VPS开始

首先，我得选择一个免费的VPS进行试验，于是我在网上找呀找，找到了[亚马逊的aws]( http://aws.amazon.com/cn/), 只要你有一张信用卡，那么，你可以很轻松的获得12个月的免费试用。具体免费套餐内容请浏览[这里](https://aws.amazon.com/cn/free/). 没有信用卡？没有关系，经网友们测试，从淘宝买的虚拟信用卡可以用作激活Amazon AWS，关键词：“虚拟信用卡 amazon”.

### 注册aws

好的，都准备好了，现在，去[亚马逊的aws]( http://aws.amazon.com/cn/) 注册一个账号。看图：

![](/img/post/2016-04-25/awsres1.png)

![](/img/post/2016-04-25/awsres2.png)

![](/img/post/2016-04-25/awsres3.png)

![](/img/post/2016-04-25/awsres4.png)

![](/img/post/2016-04-25/awsres5.png)

![](/img/post/2016-04-25/awsres6.png)

![](/img/post/2016-04-25/awsres7.png)

![](/img/post/2016-04-25/awsres8.png)

在这一步一定要注意，因为默认选择的是开发人员选项，要扣40美金的偶，立刻就会扣，没有反应时间。。。好吧，被看出来了，我被扣了--

这一步之后注册就算是完成了，等待几分钟账号就会被激活了，有些人的会等很久才能激活，可能跟前面填写的信息有关系。

### 开始创建VPS

这个描述起来也是相当麻烦，大家看图吧。

![](/img/post/2016-04-25/awslogin1.png)

![](/img/post/2016-04-25/awslogin2.png)

![](/img/post/2016-04-25/awsbuild7.png)

![](/img/post/2016-04-25/awsbuild1.png)

![](/img/post/2016-04-25/awsbuild2.png)

![](/img/post/2016-04-25/awsbuild3.png)

![](/img/post/2016-04-25/awsbuild4.png)

![](/img/post/2016-04-25/awsbuild5.png)

![](/img/post/2016-04-25/awsbuild6.png)

到了这里，已经算是把一个VPS建好了，需要什么服务都可以自己配置了，接下来说如何本地连接到远程VPS,有多种方式，大家可以详细参看[官网教程](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AccessingInstances.html?icmpid=docs_ec2_console), 我这里只介绍我使用的方式，putty连接。

### 连接VPS

#### 1.软件准备

从[putty下载页面](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 下载putty和puttygen.

putty是用来连接VPS的，但是VPS的连接需要提供秘钥，就是上文要你下载的pem文件。但是putty并不直接支持pem文件，你需要把pem文件转换为ppk文件，这就需要puttygen来完成了，下面，我们来转换。

#### 2.转换秘钥格式

![](/img/post/2016-04-25/VPSconnect1.png)

![](/img/post/2016-04-25/VPSconnect2.png)

![](/img/post/2016-04-25/VPSconnect3.png)

这样就转换好了，接下来我们开始正式连接VPS了：

#### 3.获取连接VPS需要的信息

实例的获取公有DNS信息，如图：

![](/img/post/2016-04-25/VPSconnect4.png)

填写信息：Host Name的格式是user_name@public_dns_name，其中user_name是root 或 ec2-user，但是经过测试，root是禁止直接登录的，所以只能使用ec2-user。注意不同的linux系统user_name是不同的，这里说得是Redhat，请到上文提到的官网教程查看具体内容。public_dns_name就是上步获取的公有dns

为了避免每次都重复填写，可以点击save按钮进行配置保存。

![](/img/post/2016-04-25/VPSconnect5.png)

配置上文转换过得秘钥

![](/img/post/2016-04-25/VPSconnect6.png)

#### 4.连接

点击open启动

![](/img/post/2016-04-25/VPSconnect7.png)

现在已经连上了VPS了，大家想干什么就随意吧。

# 开始配置VPN 使用SoftEther

### 软件准备

在VPS上[下载softEther VPN](http://www.softether-download.com/cn.aspx?product=softether):

![](/img/post/2016-04-25/vpndownload1.png)

你可以选择下载最新版本的，也可以选择较为稳定的版本。

{% raw %}

```shell
//首先安装wget
sudo yum install wget
sudo wget http://www.softether-download.com/files/softether/v4.20-9608-rtm-2016.04.17-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.20-9608-rtm-2016.04.17-linux-x64-64bit.tar.gz
```

{% endraw  %}

![](/img/post/2016-04-25/vpndownload2.png)

这个网上有些教程说下载这个是需要翻墙的，对于国内用户确实是这样，但是，但aws可是在美国呢，嘿嘿，不用担心，直接下就好了，速度杠杠的。还是提供一下[百度云下载](http://pan.baidu.com/s/1jH9Vqpo)

插一下，putty经常死机，准确来说，如果你隔了两三分钟没有操作，aws就会把你的连接踢掉，但是如果你一直在使用就不会有什么问题的。死机了就重连好了。

### 开始安装

好的，下载完成了，解压：

{% raw %}

```shell
tar -zxvf softether-vpnserver-v4.20-9608-rtm-2016.04.17-linux-x64-64bit.tar.gz
```

{% endraw  %}

接下来开始配置VPN了，是不是想想都要头疼了？但是，史上最简单，可不是白说的，看看一键配置！

{% raw %}

```shell
cd vpnserver/
./.install.sh
```

{% endraw  %}

接下来，它会让你阅读用户协议，然后让你同意，你就连输入3个‘1’就ok了，进入正式安装。oh!出错了，提示缺少gcc，不早说。。。

![](/img/post/2016-04-25/vpnconfig1.png)

{% raw %}

```shell
//安装gcc
sudo yum install gcc
```

{% endraw  %}

安装完毕，再去执行上面的命令，输入3个‘1’，安装成功啦（如果你使用的其他系统，获取其他VPS，可能会缺少其他的依赖，按照错误提示一个个安装即可），好了，试试：

{% raw %}

```shell
./vpnserver start
./vpnserver stop
```

{% endraw  %}

so easy!

### 在本地管理VPN

#### 下载管理软件

好吧，还得需要[下载](http://www.softether-download.com/cn.aspx?product=softether) 一个管理软件，这个要翻墙了。如图：

![](/img/post/2016-04-25/vpnconfig2.png)

这里提供[百度云下载](http://pan.baidu.com/s/1jH9Vqpo)。 下载之后解压就可以使用了，打开vpnsmgr.exe。

#### 配置连接

![](/img/post/2016-04-25/vpnconfig3.png)

#### 连接VPN server

![](/img/post/2016-04-25/vpnconfig4.png)

欧，遇到问题了，为什么会连接不上呢？现在要来说说aws的安全组是怎么一回事了。这个安全组类似于防火墙，你可以配置允许出入的协议和流量，看图：

![](/img/post/2016-04-25/vpnconfig5.png)

配置允许访问的端口和协议，这些端口都是softether监听的端口号，在官网可以找到。如果你实在是觉得烦，可以开放所有流量，这个你自己决定吧，不安全偶~

![](/img/post/2016-04-25/vpnconfig6.png)

好的，接下来成功连接

![](/img/post/2016-04-25/vpnconfig7.png)

你会进入下面这个界面（刚刚进入，会提示你进行各种配置，全X掉吧，我一步步的展示）

![](/img/post/2016-04-25/vpnconfig8.png)

#### 配置VPN

配置L2TPVPN

![](/img/post/2016-04-25/vpnconfig9.png)

管理VPN用户配置

![](/img/post/2016-04-25/vpnconfig10.png)

![](/img/post/2016-04-25/vpnconfig11.png)

这里创建的用户以及密码会在连接vpn时使用。

配置NAT

![](/img/post/2016-04-25/vpnconfig12.png)

![](/img/post/2016-04-25/vpnconfig13.png)

这个不多解释，按照我的配置来吧。点击确定之后，VPN的所有配置算是完成了。怎么样，全部界面化，是不是非常简单。

# 连接VPN

接下来就是最激动人心的时刻了，试着连接VPN吧。直接使用windows自带的VPN连接就可以了：

![](/img/post/2016-04-25/vpnconnect6.png)

![](/img/post/2016-04-25/vpnconnect1.png)

![](/img/post/2016-04-25/vpnconnect2.png)

![](/img/post/2016-04-25/vpnconnect3.png)

![](/img/post/2016-04-25/vpnconnect4.png)

![](/img/post/2016-04-25/vpnconnect5.png)

![](/img/post/2016-04-25/vpnconnect7.png)

![](/img/post/2016-04-25/vpnconnect8.png)

![](/img/post/2016-04-25/vpnconnect9.png)

![](/img/post/2016-04-25/vpnconnect10.png)

连接成功啦！！！ 

# IPV6配置

好的，现在万事俱备，只欠东风啦。配置一个公网的Ipv6就可以了。到哪里找呢，我只推荐[HE](https://ipv6.he.net/), 就是好用，下面看操作。

### 先注册账号

![](/img/post/2016-04-25/ipv6config7.png)

注册就不多说了，简单。接下来看配置。

### 申请ipv6地址

![](/img/post/2016-04-25/ipv6config1.png)

![](/img/post/2016-04-25/ipv6config2.png)

申请成功之后就会显示如下界面

![](/img/post/2016-04-25/ipv6config3.png)

### Vps配置ipv6地址

转到下图界面，选择自己的系统，通用的linux就是选择下图的linux-net-tools,然后就会出现下面的配置命令了。但是你要记得在添加sudo权限才可以执行

![](/img/post/2016-04-25/ipv6config4.png)

配置ipv6的网络命令

{% raw %}

```shell
sudo ifconfig sit0 up
sudo ifconfig sit0 inet6 tunnel ::【he中Server IPv4 Address】
sudo ifconfig sit1 up
sudo ifconfig sit1 inet6 add 【he中Server IPv6 Address】
sudo route -A inet6 add ::/0 dev sit1
```

{% endraw %}

配置完毕之后，进行测试，如下图：

![](/img/post/2016-04-25/ipv6config5.png)

{% raw %}

```shell
ping ipv6.baidu.com
```

{% endraw %}

好了，接下来就可以尝试使用ipv6进行vpnconnect了：

![](/img/post/2016-04-25/ipv6config6.png)

这个除了主机地址是使用ipv6外，其他都是和ipv4配置相同的。连接就好了，如果你也是在校学生，那么，嘿嘿，我要飞得更高~~

结束！

PS:说明一下啊，这个aws的流量并不是无限制的，一个月只有15g（上行和下行分别为15g）的，超过要扣费的。其他具体免费套餐内容参见[这里](https://aws.amazon.com/cn/free/). 但是你们可以用aws练手，然后再去网上租VPS,也挺便宜的。





