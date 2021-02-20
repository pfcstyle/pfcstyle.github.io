---
layout:		post
title:		"kubernetes 日志管理"
description:	""
date:		2021-02-16
author:		"Yawei"
categories: "kubernetes"
keywords:
    - k8s
    - kubernetes
    - log
---

# 日志管理特征及方案

k8s日志分为两块儿：
* stdout, stderr
  这部分是容器自身的日志输出，通常会存储在`/var/lib/docker/containers/[cname]/[cname]-json.log`中
* 服务日志文件
  默认会写在容器中，服务重启会丢失

## 如何解决丢失问题
* 远端存储
* Sidecar
* LogAgent

## 远端存储

## SideCar
## LogAgent