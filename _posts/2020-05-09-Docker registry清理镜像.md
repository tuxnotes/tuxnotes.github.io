---
layout: post
title: Docker registry清理镜像
date: 2020-05-09
author: tux
tags: docker registry
---

### Docker Registry清理镜像的步骤

通过docker registry的HTTP API来完成。这里使用`curl`命令与docker registry进行交互。

#### 1 查看镜像列表

```bash
# curl http://localhost:5000/v2/_catalog?n=1000
```
这里`n`参数用来分页，如果镜像很多，没有指定`n`的话可能输出结果并不包含所有的镜像。所以这里指定了一个比较大的数字。

#### 2 查看镜像tag列表
```bash
curl http://localhost:5000/v2/vodaka/busybox/tags/list
```

#### 3 查看镜像的digest

镜像的删除必须指定digest，所以删除指定的镜像前，先要获取其digest
```bash
# curl -v -u "user@example.com:passw0rd" -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X HEAD https://registry.example.com/v2/vodaka/busybox/manifests/latest
```
docker registry 2.3及以后版本，需要头部信息中使用`"Accept: application/vnd.docker.distribution.manifest.v2+json"`.如果使用Python中的requests库的get方法，则返回的digest在响应的头部信息中，而不是响应的body中。头部信息中的'Docker-Content-Digest'的value就是digest.如`sha256:e45f25b1760f616e65f106b424f4ef29185fbd80822255d79dabc73b8eb715ad`

#### 4 调用API删除镜像

```bash
# curl -u "user@example.com:passw0rd" -X DELETE https://registry.example.com/v2/vodaka/busybox/manifests/sha256:e45f25b1760f616e65f106b424f4ef29185fbd80822255d79dabc73b8eb715ad
```
如果删除成功，则返回的状态码是202.

#### 5 在registry的容器中执行操作

```bash
# docker exec -it CONTAINER_ID /bin/sh
# registry garbage-collect --dry-run=false /etc/docker/registry/config.yml
```
如果不是真正的删除，只是执行一下，则使用`--dry-run=true`

#### 脚本
```bash
registry='localhost:5000'
name='my-image'
curl -v -sSL -X DELETE "http://${registry}/v2/${name}/manifests/$(
    curl -sSL -I \
        -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
        "http://${registry}/v2/${name}/manifests/$(
            curl -sSL "http://${registry}/v2/${name}/tags/list" | jq -r '.tags[0]'
        )" \
    | awk '$1 == "Docker-Content-Digest:" { print $2 }' \
    | tr -d $'\r' \
)"
```

#### Reference
[参考链接][https://stackoverflow.com/questions/37033055/how-can-i-use-the-docker-registry-api-v2-to-delete-an-image-from-a-private-regis]