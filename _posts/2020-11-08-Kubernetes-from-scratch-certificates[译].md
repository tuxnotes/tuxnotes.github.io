---
layout: post
title: Kubernetes from scratch:certificates[译]
date: 2020-11-08
author: tuxnotes
tags: kubernetes, certificate
---

原文链接：https://medium.com/@oleg.pershin/kubernetes-from-scratch-certificates-53a1a16b5f03

理想情况，下kubernetes的所有组件之间的相互交互都要通过TLS。并且没有证书就没有K8S认证和RBAC。这其中有大量的客户端，服务端，parent(CA)和child证书需要提前创建好，在部署kubernetes集群之前。首先需要明确的是我们需要什么样的证书，以及如何生成这些证书。

### Authentication认证

首先看一下在kubernetes集群中基于证书的认证是如何工作的，如下图所示：

![](/assets/img/k8s-auth.jpeg)

上图展示了apiserver为什么使用一个特殊的client certificate对kubectl进行认证；kubectl为什么信任apiserver的certificate。因为apiserver和kubectl都信任这个相同的intermediate kubernetes CA,此CA用来签署client和server的证书。