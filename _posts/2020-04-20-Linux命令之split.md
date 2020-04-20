---
layout: post
title: Linux命令之split
date: 2020-04-20
author: tux
tags: Linux Command
---

### split命令的简单用法记录

由于邮箱对附件大小的限制，需要将文件进行分割，在逐个发送邮件。具体操作如下：

```bash
$ split -b 30M prometheus.tar.gz -d -a 2 prometheus_
```
参数说明：

`-b 指定被分割后单个文件的大小`
`-d 指定使用数字作为单个文件名后缀`
`-a 后缀的长度，结合前面的-d，-a 2,表示使用2位数字`

最后的`prometheus_`是文件的前缀。更详细的参数可参考split的帮助文档，使用格式如下：
```bash
$split [选项] [被分割文件名称] [分割后文件名称的前缀]
```

### 将分割后的文件合并

可采用简单的方式，如下：
```bash
# cat prometheus_* > prometheus.tar.gz
```
