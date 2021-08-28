---
layout: post
title: gitlab reset root password
author: tux
date: 2021-08-27
tags: gitlab
---

参考链接：https://docs.gitlab.com/ee/security/reset_user_password.html

Rake Task是gitlab13.9引入的，通过rake task重置root密码

命令行执行如下命令，需要有root权限

```bash
sudo gitlab-rake "gitlab:password:reset"
```

然后gitlab会提示要求写入用户名，密码和确认密码。在填写正确的值后，此指定用户的密码就会更新

rake task还可以接受用户名作为参数，示例如下：

```bash
sudo gitlab-rake "gitlab:password:reset[johndoe]"
```

要重置默认管理员用户的密码，运行rake task，用户名为root，root是默认的管理员账户名称。