---
layout: post
title: Services without selectors[译]
date: 2020-11-02
author: tux
tags: kubernetes
---

注:本文摘译自kubernetes文档:https://kubernetes.io/docs/concepts/services-networking/service/#services-without-selectors

Service多数情况下是对kubernetes的Pod资源访问的一种抽象，但也可以抽象其他类型的后端服务。如下面的场景：

- 比如你想在生成环境中有一个外部的数据库集群，但在你的test环境中，你使用自己的数据库
- 比如你想将你的Service指向其他namespace的中的Service或者是其他集群中的Service
- 比如你正在将负载迁移到kubernetes集群中，在评估迁移方法时，将部分后端负载迁移到kubernetes中

上面的这几种情况，你可以定义一个没有Pod selector的Service，如：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

因为上面这种类型的Service没有selector,所以对应的Endpoint对象就不会自动创建。你可以手动将Service映射到服务运行的网络地址和端口，这种映射方法可以使用手动添加Endpoint对象的方式来实现。Endpoint对象的YAML配置示例如下：

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 102.0.2.42
    ports:
      - port: 9376
```

Endpoint对象的名称必须是一个有效的DNS subdomain name

>NOTE: endpoint IP不能是：loopback (127.0.0.0/8 for IPv4, ::1/128 for IPv6), or link-local (169.254.0.0/16 and 224.0.0.0/24 for IPv4, fe80::/64 for IPv6).
Endpoint IP地址列表表也不能是其他kubernetes集群中service的cluster IP，因为kube-proxy不支持将虚拟IP作为destination.


endpoints与service的名称相同，都是my-service。对没有selector的service进行访问时与有selector的service的访问方式相同，如上示例，流量会被路由到YAML文件中定义的单个endpoint:`192.0.2.42：9376`(TCP).
