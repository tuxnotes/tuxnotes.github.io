---
layout: post
title: kubernetes重启Pod的几种方式
date: 2020-09-24
author: tux
tags: kubernetes pod
---

kubernetes可以通过一下几种方式重启：


### 1 Rolling restart

```bash
kubectl rollout restart deployment <deployment_name>
```

>NOTE:根据kubernetes的版本不同，其可用的子命令可能也不同


### 2 Using Environment Variables

另一种方法是通过设置或改变环境变量的方式来轻质pod重启或同步修改后的状态

```bash
kubctl set env deployment <deployment_name> DEPLOY_DATE="$(date)"
```

### 3 Scaling the Number of Replicas

场景：没有yaml文件，但是使用的是Deployment对象

通过使用scale命令，更改replicaset的数量2次来达到重启的目的

```bash
kubectl scale deployment <deployment_name> --replicas=0
kubectl scale deployment <deployment_name> --replicas=1
```

### 4 kubectl replace

如果有最新的yaml文件，可能强制替换Pod的API对象，从而达到重启的目的

```bash
kubectl replace --force -f xxx.yaml
```

### 5 kubectl delete删除Pod

使用场景：没有yaml文件，但使用的是Deployment对象

```bash
kubectl delete pod <pod_name> -n <namespace>
```

### 6 直接使用Pod对象

使用场景：没有yaml文件，直接使用Pod对象

```bash
kubectl get pod <pod_name> -n <namespace> -o yaml | kubectl replace --force -f -
```

### Reference

- https://phoenixnap.com/kb/how-to-restart-kubernetes-pods
- segmentfault.com/a/1190000020675199
