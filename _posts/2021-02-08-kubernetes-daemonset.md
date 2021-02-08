---
layout:		post
title:		"kubernetes Daemonset"
description: ""
date:		2021-02-08
author:		"Yawei"
categories: "kubernetes"
keywords:
    - k8s
    - kubernetes
    - daemonset
---

守护进程集包含存储进程、日志收集进程、节点监控进程、和Pods管理（创建与回收）进程。

DaemonSet可以只有一个，也可以根据不同的daemon划分为多个。这些进程都以Pods的形式运行

## Spec
### 必需字段
apiVersion, kind, metadata等通用字段

template, 这是一个pod template,其配置与pod几乎完全一样，除了没有apiVersion和kind

selector,1.8开始，必须指定来匹配template的label，否则会报错
* matchLabels - 匹配template的label
* matchExpressions - 通过指定key, values列表和关联key-value的操作，来构建更复杂的匹配

### 在指定的node上运行Pods
如果你指定一个.spec.template.spec.nodeSelector, 那么DaemonSet controller会在匹配的node上创建相关的Pods。如果指定spec.template.spec.affinity，那么会在亲和度匹配的node上创建Pods。如果什么都没有指定，那么将会在所有的node上创建。

```yaml
Spec Example
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Daemon Pods如何调度
一般情况下，pods由Kubernetes调度器调度，但是Daemon pods被DaemonSet controller创建并调度。这会导致两个问题：
* 不一致的Pod表现：一般的Pods会以Pending的状态创建，但是DaemonSet pods不会
* Pod preemption被默认的调度器控制。当抢占启用时，DaemonSet controller会做出调度决定，而不考虑pod的优先级和抢占逻辑
可以通过添加NodeAffinity项到DaemonSet Pods可以让其使用默认的调度器

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchFields:
      - key: metadata.name
        operator: In
        values:
        - target-host-name
```
另外，`node.kubernetes.io/unschedulable:NoSchedule` **toleration**会自动添加到DaemonSet Pods上，从而使默认的调度器忽略unschedulable的Nodes


