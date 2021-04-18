---
layout:		post
title:		"kubernetes 日志管理"
description:	""
date:		2021-02-15
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
直接为应用配置远端日志，如kafka, es等。

业内使用较多。
![remote store](../img/post/2021-02-15/remote-store.png)
* 简单
* 对应用有侵染

## SideCar
为每一个Pod配置一个SideCar, SideCar与主容器共享Volume,然后由SideCar转发日志到远端。

这会对每一个Pod都有侵染，官方不建议使用。
![remote store](../img/post/2021-02-15/sidecar.png)


## LogAgent
为每一个Node配置一个Agent, 通过一个Agent采集所有的日志。一般以DaemonSet方式运行。

需要为应用约定好日志路径。
![remote store](../img/post/2021-02-15/log-agent.png)

## LogAgent方案实践
![remote store](../img/post/2021-02-15/agent-practice.png)

### [LogPilot](https://github.com/AliyunContainerService/log-pilot)
阿里开源日志采集工具，支持动态配置，对容器更加友好。本质上是对静态日志采集工具的包装，目前支持[Fluentd Plugin](https://github.com/AliyunContainerService/log-pilot/blob/master/docs/fluentd/docs.md)和[Filebeat Plugin](https://github.com/AliyunContainerService/log-pilot/blob/master/docs/filebeat/docs.md)
* 智能的容器日志采集工具
* 自动发现

## Elasticsearch

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-api
  namespace: kube-system
  labels:
    name: elasticsearch
spec:
  selector:
    app: es
  ports:
  - name: transport
    port: 9200
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-discovery
  namespace: kube-system
  labels:
    name: elasticsearch
spec:
  selector:
    app: es
  ports:
  - name: transport
    port: 9300
    protocol: TCP
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
spec:
  replicas: 3
  serviceName: "elasticsearch-service"
  selector:
    matchLabels:
      app: es
  template:
    metadata:
      labels:
        app: es
    spec:
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
      serviceAccountName: kubernetes-dashboard
      initContainers:
      - name: init-sysctl
        image: busybox:1.27
        command:
        - sysctl
        - -w
        - vm.max_map_count=262144
        securityContext:
          privileged: true
      containers:
      - name: elasticsearch
        image: elasticsearch:5.5.1
        ports:
        - containerPort: 9200
          protocol: TCP
        - containerPort: 9300
          protocol: TCP
        securityContext:
          capabilities:
            add:
              - IPC_LOCK
              - SYS_RESOURCE
        resources:
          limits:
            memory: 4000Mi
          requests:
            cpu: 100m
            memory: 2000Mi
        env:
          - name: "http.host"
            value: "0.0.0.0"
          - name: "network.host"
            value: "_eth0_"
          - name: "cluster.name"
            value: "docker-cluster"
          - name: "bootstrap.memory_lock"
            value: "false"
          - name: "discovery.zen.ping.unicast.hosts"
            value: "elasticsearch-discovery"
          - name: "discovery.zen.ping.unicast.hosts.resolve_timeout"
            value: "10s"
          - name: "discovery.zen.ping_timeout"
            value: "6s"
          - name: "discovery.zen.minimum_master_nodes"
            value: "2"
          - name: "discovery.zen.fd.ping_interval"
            value: "2s"
          - name: "discovery.zen.no_master_block"
            value: "write"
          - name: "gateway.expected_nodes"
            value: "2"
          - name: "gateway.expected_master_nodes"
            value: "1"
          - name: "transport.tcp.connect_timeout"
            value: "60s"
          - name: "ES_JAVA_OPTS"
            value: "-Xms2g -Xmx2g"
        livenessProbe:
          tcpSocket:
            port: transport
          initialDelaySeconds: 20
          periodSeconds: 10
        volumeMounts:
        - name: es-data
          mountPath: /data
      terminationGracePeriodSeconds: 30
      volumes:
      - name: es-data
        hostPath:
          path: /es-data


```

# log-pilot
