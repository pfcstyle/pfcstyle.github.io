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

# 查看部署错误日志
```
kubectl describe deploy [deployname]
kubectl get deploy [deployname] -o yaml
```

## journalctl
journalctl用法：

- 查看所有日志（默认情况下 ，只保存本次启动的日志）： journalctl 
- 查看内核日志（不显示应用日志）： journalctl -k 
- 查看系统本次启动的日志： journalctl -b

- 查看上一次启动的日志（需更改设置）：
 在该[Journal]部分下，将该Storage=选项设置为“persistent”以启用持久记录：
 ```
        vim    /etc/systemd/journald.conf
    . . .
    [Journal]
    Storage=persistent
```

追踪日志

要主动追踪当前正在编写的日志，大家可以使用-f标记。同样功能类似为tail -f，只要不终止，会一直监控`journalctl -f`

也许最有用的过滤方式是你感兴趣的单位。我们可以使用这个-u选项来过滤我们可以使用这个-u选项来过滤`journalctl -u`


### journalctl example
```
# 查看某个服务
journalctl -f -u etcd
```

#### 如果某个节点突然挂掉（not ready)

一般重启docker服务即可

```
journalctl -f -u kubelet
```


#### 重启某个节点的kubelet
```
# 重启docker
systemctl daemon-reload
systemctl restart docker
# 重启kubelet
systemctl restart kubelet.service
```
