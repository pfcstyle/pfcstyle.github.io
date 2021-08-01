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
![remote store](/img/post/2021-02-15/remote-store.png)
* 简单
* 对应用有侵染

## SideCar
为每一个Pod配置一个SideCar, SideCar与主容器共享Volume,然后由SideCar转发日志到远端。

这会对每一个Pod都有侵染，官方不建议使用。
![remote store](/img/post/2021-02-15/sidecar.png)


## LogAgent
为每一个Node配置一个Agent, 通过一个Agent采集所有的日志。一般以DaemonSet方式运行。

需要为应用约定好日志路径。
![remote store](/img/post/2021-02-15/log-agent.png)

## LogAgent方案实践
![remote store](/img/post/2021-02-15/agent-practice.png)

[LogPilot](https://github.com/AliyunContainerService/log-pilot)是阿里开源日志采集工具，支持动态配置，对容器更加友好。本质上是对静态日志采集工具的包装，目前支持[Fluentd Plugin](https://github.com/AliyunContainerService/log-pilot/blob/master/docs/fluentd/docs.md)和[Filebeat Plugin](https://github.com/AliyunContainerService/log-pilot/blob/master/docs/filebeat/docs.md)
* 智能的容器日志采集工具
* 自动发现

### Elasticsearch

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
        - containerPort: 9200 #外部resetfutl接口
          protocol: TCP
        - containerPort: 9300 #内部通信接口
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

### 部署es

```bash
kubectl apply -f elasticsearch.yaml

# 检查部署
kubectl get svc -n kube-system
kubectl get statefulset -n kube-system
```
![es检查](/img/post/2021-02-15/es-check.png)

### log-pilot
```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-pilot
  namespace: kube-system
  labels:
    k8s-app: log-pilot
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      k8s-app: log-es # must specify a pod selector (spec.selector) that matches the labels of the (.spec.template.metadata.labels)
  template:
    metadata:
      labels:
        k8s-app: log-es # <------ matches this one
        kubernetes.io/cluster-service: "true"
        version: v1.22
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      serviceAccountName: kubernetes-dashboard
      containers:
      - name: log-pilot
        image: log-pilot:0.9-filebeat
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        env:
          - name: "FILEBEAT_OUTPUT"
            value: "elasticsearch"
          - name: "ELASTICSEARCH_HOST"
            value: "elasticsearch-api"
          - name: "ELASTICSEARCH_PORT"
            value: "9200"
          - name: "ELASTICSEARCH_USER"
            value: "elastic"
          - name: "ELASTICSEARCH_PASSWORD"
            value: "changeme"
        volumeMounts:
        - name: sock
          mountPath: /var/run/docker.sock
        - name: root
          mountPath: /host
          readOnly: true
        - name: varlib
          mountPath: /var/lib/filebeat
        - name: varlog
          mountPath: /var/log/filebeat
        securityContext:
          capabilities:
            add:
            - SYS_ADMIN
      terminationGracePeriodSeconds: 30
      volumes:
      - name: sock
        hostPath:
          path: /var/run/docker.sock
      - name: root
        hostPath:
          path: /
      - name: varlib
        hostPath:
          path: /var/lib/filebeat
          type: DirectoryOrCreate
      - name: varlog
        hostPath:
          path: /var/log/filebeat
          type: DirectoryOrCreate


```

```bash
kubectl apply -f log-pilot.yaml

# 检查
kubectl get ds -n kube-system
```
> 第一次需要下载镜像，因此会比较慢，可以使用`kubectl describe ds log-pilot -n kube-system`查看状态
![logpilot检查](/img/post/2021-02-15/logpilot-check.png)

### kibana 部署

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: kube-system
  labels:
    component: kibana
spec:
  selector:
    component: kibana
  ports:
  - name: http
    port: 80
    targetPort: http
---
#ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kibana
  namespace: kube-system
spec:
  rules:
  - host: kibana.esribj.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kibana
          servicePort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: kube-system
  labels:
    component: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
     component: kibana
  template:
    metadata:
      labels:
        component: kibana
    spec:
      containers:
      - name: kibana
        image: kibana:5.5.1
        env:
        - name: CLUSTER_NAME
          value: docker-cluster
        - name: ELASTICSEARCH_URL
          value: http://elasticsearch-api:9200/
        resources:
          limits:
            cpu: 1000m
          requests:
            cpu: 100m
        ports:
        - containerPort: 5601
          name: http


```

```bash
kubectl apply -f kibana.yaml
kubectl get deploy -n kube-system
```
配置kibana.esribj.com到hosts, 然后访问http://kibana.esribj.com:30080

## 测试logpilog日志 
启动一个web demo来测试
```yaml
#deploy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-log-demo
spec:
  selector:
    matchLabels:
      app: web-log-demo
  replicas: 1
  template:
    metadata:
      labels:
        app: web-log-demo
    spec:
      containers:
      - name: web-log-demo
        image: yw-worker1.esri.com/kubernetes/web-demo:1.0
        ports:
        - containerPort: 8080
        env: # 配置采集目录
        # 这里注意命名格式 aliyun_logs_[自定义名称]
        # 对接es，就是索引；对接kafka，就是topic
        - name: aliyun_logs_catalina
          value: "stdout"
        - name: aliyun_logs_access
          value: "/usr/local/tomcat/logs/*"
        volumeMounts:
        - mountPath: /usr/local/tomcat/logs
          name: accesslogs
      volumes:
      - name: accesslogs
        emptyDir: {}
---
#service
apiVersion: v1
kind: Service
metadata:
  name: web-log-demo
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: web-log-demo
  type: ClusterIP

---
#ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: web-log-demo
spec:
  rules:
  - host: web-log.esribj.com
    http:
      paths:
      - path: /
        backend:
          serviceName: web-log-demo
          servicePort: 80

```
配置kibana索引

https://github.com/AliyunContainerService/log-pilot