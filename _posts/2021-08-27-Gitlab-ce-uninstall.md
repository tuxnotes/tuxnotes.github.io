---
layout: post
title: Gitlab-ce uninstall
author: tux
date: 2021-08-27
tags: gitlab
---

Gitlab-ce功能强大，组件众多，配置与数据目录也较多。在使用过程中通过deb安装包安装后，通过apt卸载后，再次安装发现依然存在之前的数据，所以下面是完全卸载gitlab-ce的方法。

参考链接：https://askubuntu.com/questions/824696/is-it-fine-to-remove-the-opt-gitlab-directory-manually-after-removing-the-gitl

`/opt/gitlab`目录下的数据是安装gitlab后运行`gitlab-cet reconfigure`命令后生成的，主要用于存储变化的数据，以及`gitlab-ce`包的配置相关的数据。

推荐的卸载流程如下：

1. 删除服务:删除之前可能要先停止服务`gitlab-ctl stop`

   ```bash
   sudo gitlab-ctl uninstall
   ```

   

2. 删除数据

   ```bash
   sudo gitlab-ctl cleanse
   ```

   

3. 移除配置的账户信息

   ```bash
   sudo gitlab-ctl remove-accounts
   ```

   

4. 卸载安装包

   ```bash
   sudo dpkg -P gitlab-ce
   ```

   

