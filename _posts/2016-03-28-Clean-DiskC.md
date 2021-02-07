---
layout:		post
title:		"C盘清理小技巧"
description: "必须要减肥了"
date:		2016-03-28
author:		"PfCStyle"
header-img:	"img/post/2016-03-28/head.jpg"
categories: "杂记"
keywords:
    - 磁盘清理
    - 系统盘瘦身
---

> 小记

下面是C盘可以清理的路径：

{% raw %}

```path
- C盘搜索FileRepository,这是windows自动推送的驱动更新，全选，删除。如果有些删不掉就跳过。
- C:\Windows\SoftwareDistribution\Download
- C:\Users\**user**\AppData\Local\Temp
```

{% endraw %}