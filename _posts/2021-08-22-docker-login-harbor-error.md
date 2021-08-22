---
layout: post
title: docker login harbor报错
author: tux
date: 2021-08-18
tags: harbor
---

# 使用harbor的一些错误排查

## 1 docker login报错

root@dev01:~/harbor# docker login 192.168.254.139:1121
Username: admin
Password: 
Error response from daemon: Get https://192.168.254.139:1121/v2/: http: server gase to HTTPS client

但harbor的配置文件harbor.yml已经将https部分的配置，端口，证书等全部注释了。仍然报错。从报错提示来看，连的还是https，所以大胆猜测docker默认将其认为了安全仓库，所以需要配置docker

```bash
# vim /etc/docker/daemon.json

# 加入下面的内容
{
	"insecure-registries": ["192.168.254.139:1121"]
}
```
保存上述文件后重启docker即可。

如果将insecure-registries错误的写成了insecure-registry，则报错信息如下：

```
root@dev01:~/harbor# docker login 192.168.254.139:1121
Username: admin
Password: 
Error response from daemon: login attempt to http://192.168.254.139:1121/v2/ fail: 502 Bad Gateway
```
## 2 harbor web页面登录报错

打开浏览器，输入用户名admin，和密码后，提示密码不正确，但密码确实是按harbor.yml文件中的配置的密码输入的。

首先检查harbor启动的各个组件的docker 容器是否正常
```bash
root@dev01:~/harbor# docker-compose ps
      Name                  Command                    State                    P
---------------------------------------------------------------------------------
harbor-core         /harbor/entrypoint.sh      Up (health: starting)             
harbor-db           /docker-entrypoint.sh      Exit 137                          
harbor-jobservice   /harbor/entrypoint.sh      Exit 137                          
harbor-log          /bin/sh -c                 Up (healthy)            127.0.0.1:
                    /usr/local/bin/ ...                                p         
harbor-portal       nginx -g daemon off;       Up (healthy)            8080/tcp  
nginx               nginx -g daemon off;       Up (healthy)            0.0.0.0:11
redis               redis-server               Up (healthy)            6379/tcp  
                    /etc/redis.conf                                              
registry            /home/harbor/entrypoint.   Up (healthy)            5000/tcp  
                    sh                                                           
registryctl         /home/harbor/start.sh      Up (healthy)
```
发现harbor-db和harbor-jobservice两个容器退出了。所以重启harbor
```bash
# cd /root/harbor # 此目录下是包含docker-compose的配置文件
# docker-compose stop
# docker-compose start
```
在各个容器正常启动后，在此登录页面就成功了。
