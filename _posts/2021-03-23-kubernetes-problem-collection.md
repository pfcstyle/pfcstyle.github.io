---
layout:		post
title:		"kubernetes 问题汇总"
description: "遇到的各种坑啊"
date:		2021-03-23
author:		"Yawei"
categories: "kubernetes"
keywords:
    - k8s
    - kubernetes
    - 坑
---

# 突然所有服务无法访问
因为是使用的公司虚拟机集群搭建的k8s集群，发现公司虚拟机即使配置了静态ip，仍然会发生改变！所以重新apply一下静态ip配置，恢复原ip即可。
```bash
# ubuntu18
sudo netplan apply
```