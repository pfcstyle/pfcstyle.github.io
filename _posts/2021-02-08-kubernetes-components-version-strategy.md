---
layout:		post
title:		"kubernetes 组件版本偏差策略"
description: ""
date:		2021-02-08
author:		"Yawei"
categories: "kubernetes"
keywords:
    - k8s
    - kubernetes
---

# 版本偏差

* 多个 kube-apiserver 实例小版本号最多差1
* kubelet 版本号不能高于 kube-apiserver，最多可以比 kube-apiserver 低两个小版本。
* kube-controller-manager、kube-scheduler 和 cloud-controller-manager 版本不能高于 kube-apiserver 版本号
* kubectl 可以比 kube-apiserver 高一个小版本，也可以低一个小版本。

# 组件升级顺序

1. kube-apiserver
2. kube-controller-manager、kube-scheduler 和 cloud-controller-manager
3. Kubelet
4. Kube-proxy

