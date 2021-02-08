---
layout:		post
title:		"kubernetes 教程（一）"
description: "核心概念介绍"
date:		2021-02-08
author:		"Yawei"
categories: "kubernetes"
keywords:
    - k8s
    - kubernetes
    - 教程
---

# Kubernetes能提供什么
* 服务发现和负载均衡
* 存储编排

    Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。
* 自动部署和回滚
* 自动完成装箱计算

    Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。
* 自我修复

    Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的 运行状况检查的容器
* 密钥与配置管理
    
    Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

# Kubernetes集群部署图

![k8s-cluster-deploy](/img/post/2021-02-08/k8s-cluster-deploy.jpg)

* Pod：最小的调度分配单元，可包含一组容器和卷组成，同一个Pod的容器组共享同一个网络命名空间;
* ReplicationController/ReplicateSet: 用于控制Pod副本数量，保证Pod始终运行;
* StatefulSet：类似ReplicateSet，但StatefulSet提供Pod状态保持，在重新调度后保留他们的标识和状态，适合数据库等有状态应用场景;
* DeamonSet： 保证在所有群集节点上运行一个Pod;
* Service: 为一组功能相同的Pods提供单一不变的接入点，解耦Pods伸缩能力;
* Deployment：类似ReplicationController/ReplicateSet， 增加滚动升降级策略和能力;
* ConfigMap：配置选项，支持配置从Pod解耦分离;
* Secret：类似ConfigMap，用于敏感数据保存;

# 重要组件

集群中包含多个节点，分为主节点和工作节点。

工作节点上用于负载应用的单元称为Pod,主节点用来管理工作节点和Pod.

## 控制面板组件（主节点）
对集群做出全局决策，以及检测和响应集群事件。每一个主节点同时也是node
### kube-apiserver
Kubernetes的api服务，可以用来动态操作kubernetes，支持伸缩扩展。
### etcd
保存Kubernetes集群数据的后台键值数据库
### kube-scheduler
用来调度Pod及Pod的资源
### kube-controller-manager
每一个控制器应该是一个进程，但为了降低复杂性，他们被编译到一个应用中，并运行在一个进程中

* 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应。
* 副本控制器（Replication Controller）: 负责为系统中的每个副本控制器对象维护正确数量的 Pod。
* 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)。
* 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌
### cloud-controller-manager
用于链接管理云提供商的应用编程接口，本地集群不需要
* 节点控制器（Node Controller）: 用于在节点终止响应后检查云提供商以确定节点是否已被删除
* 路由控制器（Route Controller）: 用于在底层云基础架构中设置路由
* 服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器

## Node组件
### Kubelet
每个工作节点上的代理，通过接收PodSpecs来管理容器，并保证安全的运行在Pod中
### Kube-proxy
每个工作节点上运行的网络代理，维护节点网络规则
### Container Runtime
负责运行容器的软件，如Docker、Containerd等任何实现Kubernetes CRI（容器运行环境接口）的容器软件

## 插件
使用Kubernetes资源提供集群级别功能，其命名空间域属于kubu-system命名空间。常用插件：
### DNS
几乎所有Kubernetes集群都应该有集群DNS用于做域名服务
### 仪表盘
Kubernetes集群通用的、基于WEb的用户界面，可以管理集群
### 容器资源监控
### 集群级别日志