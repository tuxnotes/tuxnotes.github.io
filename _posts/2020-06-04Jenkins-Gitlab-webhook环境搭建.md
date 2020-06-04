---
layout: post
date: 2020-06-04
title: Jenkins-Gitlab-webhook环境搭建
author: tux
tags: Jenkins Gitlab 
---

### 环境说明

由于环境处于内网，还有各种限制，所以这里只是在各种限定的情况下的一种安装方式。这里暂不表其他方式。

#### Jenkins安装

最初打算用容器的方式，发现镜像获取困难。最后决定使用`jenkins.war`的方式部署。首先确保环境中配置了JDK，JDK安装方式不做说明。
下载war包：
```bash
# mkdir -p /opt/jenkins
# cp jenkins.war /opt/jenkins
# cat start.sh

#!/bin/bash

export JENKINS_HOME=/opt/jenkins
java -jar jenkins.war -httpPort=8080 --daemon

# 启动
# bash start.sh
```
这里由于没有网络环境，可采用一台能上网的机器，也安装jenkins，同时将需要的插件全部安装。最后在JENKINS_HOME目录下的plugins目录即是安装的插件，将这个目录打包复制到无网环境的JENKINS_HOME下，重启jenkins。

#### Gitlab安装

与jenkins在同一台机器上。由于按组件安装非常繁琐，这里采用容器的方式。镜像名称`gitlab/gitlab-ce:latest`.镜像有些大。启动命令如下：
```bash
docker run --detach \
    --hostname gitlab.tux.com \
    --env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.tux.com'; gitlab_rails['lfs_enabled'] = true;" \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab
    --restart always
    --volume /home/gitlab/config:/etc/gitlab \
    --volume /home/gitlab/logs:/var/log/gitlab \
    --volume /home/gitlab/data:/var/opt/gitlab \
    --privileged=true \
    gitlab/gitlab-ce:latest
```

容器启动的时间较长，通过docker ps观察，当status为health的时候就是启动完成了。然后通过域名登录(自己修改hosts文件即可)，第一次登录会要求设置root的密码。

#### webhook集成

由于网络问题，让人头疼的问题就是插件的安装以及插件依赖的安装。插件主要是gitlab-hook插件。
webhook集成主要是jenkins和Gitlab的设置

##### jenkins设置

在创建项目的过程中，Build Triggers部分，勾选`Build when a change is pushed to Gitlab.GitLab webhook URL:http://192.168.107.129:8080/project/test-hook` 这个URL是项目创建是jenkins生成的。点击下面的`Advanced`，在`Secret token`右边的空白框下有一个`Generate`按钮。点击它，就会生成一个字符串，这个字符串是要配置到Gitlab中的。其他根据具体环境填写即可。

##### Gitlab配置

在Gitlab上打开项目，如`gitlab.tux.com/sre/test-hook` . 找到左侧的`Settings`---> `Webhooks`后，点击进入配置界面：

URL：即是上一步jenkins配置的时候显示的GitLab webhook URL: http://192.168.107.129:8080/project/test-hook
Secret Token: 上一步jenkins配置过程中的Secret Token

勾选`Push event`,其他根据自己的情况配置，最后点击`Add webhook`.令人吃惊的是居然报错了：
```
url is blocked requests to the local network are not allowed
```
搜索一下，很好解决。[官方解释][https://docs.gitlab.com/ee/security/webhooks.html]

部分内容如下：
```
To prevent this type of exploitation from happening, starting with GitLab 10.6, all Webhook requests to the current GitLab instance server address and/or in a private network will be forbidden by default. That means that all requests made to 127.0.0.1, ::1 and 0.0.0.0, as well as IPv4 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 and IPv6 site-local (ffc0::/10) addresses won't be allowed.

This behavior can be overridden by enabling the option "Allow requests to the local network from hooks and services" in the "Outbound requests" section inside the Admin area under Settings (/admin/application_settings):
————————————————
版权声明：本文为CSDN博主「xukangkang1hao」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/xukangkang1hao/java/article/details/80756085
```
大概意思是从Gitlab10.6开始，为了防止这种类型的漏洞，默认情况下所有的对当前Gitlab实例服务器地址或者私有网络地址的webhook请求都会被禁止。但可通过开启`Outbound requests`中的`Allow requests to the local network from hooks and services`来关闭这种禁止行为。开启方法：

`Admin area` ---> `Settings` --- > `Network` --- > `Outbound requests` --- > `Expand` --- > 勾选`Allow requests to the local network from web hooks and services`
