---
layout: post
title: kubernetes-api-swagger-ui
date: 2021-07-27
author: tux
tags: Kubernetes
---

将Kubernetes的API通过web浏览器进行可视化的展示。使用swagger-ui实现。具体操作步骤如下：

```bash
# kubectl proxy --port=8080
# curl localhost:8080/openapi/v2 > k8s-swagger.json
# docker run -d --restart=always -p 7070:8080 -e SWAGGER_JSON=/k8s-swagger.json -v $(pwd)/k8s-swagger.json:/k8s-swagger.json swaggerapi/swagger-ui:latest
```

Reference

- https://jonnylangefeld.com/blog/kubernetes-how-to-view-swagger-ui 

