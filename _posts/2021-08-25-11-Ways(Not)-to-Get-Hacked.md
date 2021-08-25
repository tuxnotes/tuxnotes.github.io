---
layout: post
title: 11 Ways (Not) to Get Hacked
author: tux
date: 2021-08-25
tags: kubernetes
---

**Author**: Andrew Martin (ControlPlane)

kubernetes从项目开始就注重安全，但任然有一些可圈可点的地方。安全问题覆盖到从控制平面到workerload和网络安全以及未来的安全问题。下面列出了一些建议，用于加固集群，增加集群遇到危险时快速恢复的能力。

- [Part One: The Control Plane](#Part One: The Control Plane)
  - [1. TLS Everywhere](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#1-tls-everywhere)
  - [2. Enable RBAC with Least Privilege, Disable ABAC, and Monitor Logs](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#2-enable-rbac-with-least-privilege-disable-abac-and-monitor-logs)
  - [3. Use Third Party Auth for API Server](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#3-use-third-party-auth-for-api-server)
  - [4. Separate and Firewall your etcd Cluster](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#4-separate-and-firewall-your-etcd-cluster)
  - [5. Rotate Encryption Keys](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#5-rotate-encryption-keys)
- Part Two: Workloads
  - [6. Use Linux Security Features and PodSecurityPolicies](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#6-use-linux-security-features-and-podsecuritypolicies)
  - [7. Statically Analyse YAML](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#7-statically-analyse-yaml)
  - [8. Run Containers as a Non-Root User](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#8-run-containers-as-a-non-root-user)
  - [9. Use Network Policies](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#9-use-network-policies)
  - [10. Scan Images and Run IDS](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#10-scan-images-and-run-ids)
- Part Three: The Future
  - [11. Run a Service Mesh](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#11-run-a-service-mesh)
- [Conclusion](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#conclusion)

# Part One: The Control Plane

控制平面是kubernetes的大脑，它是集群中每个容器和pod的全局总览，可以调度新的pod，含可以访问宿主机的pod，能读取存储在集群中的secret。这些漏洞需要防护，防止恶意入侵。

## 1 TLS Everywhere

TLS应该用到每个组件上，防止网络嗅探，验证服务器身份，并且双向验证客户端身份。

下面的网络图展示了放置TLS的理想位置：master节点的组件之间，还有kubelet与api server之间。[Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/1.9.0/docs/04-certificate-authority.md) 提供了手动创建证书的详细命令。

![](https://d33wubrfki0l68.cloudfront.net/d35c2b375b43b4fa374ae834f95224975418e33f/6b47b/images/blog/2018-06-05-11-ways-not-to-get-hacked/kubernetes-control-plane.png)

从前kubernetes node节点自动扩容是比较困难的，因为每个节点需要TLS key来连接到master节点。将秘钥注入到基础镜像中并不是一个好的办法。 [Kubelet TLS bootstrapping](https://medium.com/@toddrosner/kubernetes-tls-bootstrapping-cf203776abc7) 提供为新的kubelet创建证书签名请求的能力，因此证书会在启动的时候生成。

如下图所示

![](https://d33wubrfki0l68.cloudfront.net/dcff3b9b32534fc0b81c5316e1d08ad10e6fa302/682d5/images/blog/2018-06-05-11-ways-not-to-get-hacked/node-tls-bootstrap.png)

## 2 Enable RBAC with Least Privilege, Disable ABAC, and Monitor Logs启用最小权限的RBAC，关闭ABAC，监控日志

RBAC提供了细粒度的管理策略，用于用户访问资源，如对namespace的访问。

![](https://d33wubrfki0l68.cloudfront.net/5ffaca47a78a1e02787d105772eb7a72e28c281a/0a9d3/images/blog/2018-06-05-11-ways-not-to-get-hacked/rbac2.png)

kubernetes从1.6版开始就推荐使用RBAC了，并应该关闭ABAC，使用下面的参数使用RBAC

```
--authorization-mode=RBAC
```

GKE上使用下面的命令关闭ABAC

```
--no-enable-legacy-authorization
```

There are plenty of [good examples](https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/) of [RBAC policies for cluster services](https://github.com/uruddarraju/kubernetes-rbac-policies), as well as [the docs](https://kubernetes.io/docs/admin/authorization/rbac/#role-binding-examples). And it doesn't have to stop there - fine-grained RBAC policies can be extracted from audit logs with [audit2rbac](https://github.com/liggitt/audit2rbac).

