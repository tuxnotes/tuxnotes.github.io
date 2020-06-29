---
layout: post
title: docker images命令使用0--format参数
date: 2020-06-29
author: tux
tags: docker
---

### 说明

使用`docker images`命令查看镜像时，如果不加参数，输出很多列。如果镜像很多，可采用`--format`选项，并加上其他的shell命令来提高工作效率。
例如有一些tag为`nocloudwise`的镜像，需要push到镜像仓库，可采用如下命令：

```shell
$ docker images --format "{{.Repository}}:{{.Tag}}" | grep nocloudwise | xargs -I {} docker push {}
```

`--format`后面其实是`golang`模板的语法。

`--format`后有效的模板变量：

| Placeholder | Description |
|-------------|-------------|
| .ID         | Image ID    |
| .Repository | Image Repository |
| .Tag        | Image tag   |
| .Digest     | Image digest|
| .CreateSince| Elapsed time since the image was created |
| .CreateAt   | Time when the image was created |
| .Size       | Image disk size |