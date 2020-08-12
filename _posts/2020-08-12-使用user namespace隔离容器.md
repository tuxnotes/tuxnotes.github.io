---
layout: post
title: 使用user namespace隔离容器
date: 2020-08-12
author: tux
tags: docker, user
---

摘译与docker文档，原文链接：https://docs.docker.com/engine/security/userns-remap/

### 使用user namespace隔离容器

Linux namespace提供了对进程的隔离，使进程在无感知的情况下限制其对系统资源的访问。防止容器特权逃逸攻击的最好方法就是设置容器中的应用以一个非特权用户运行。对于容器，如果进程必须以
root用户运行的话，可以将这个用户re-map到Docker主机上的一个less-privileged的用户。被映射的user赋予了一个UID的范围，就像主机上的UID从0到65535，但此用户在主机上没有特权。

#### remapping and subordinate user and group IDs

remapping通过两个文件完成：`/etc/subuid`和`/etc/subgid`。两个文件的功能相同，只不过一个关注用户ID的范围，一个关注用户组ID的范围。假定`/etc/subuid`有如下内容：
```
testuser:231072:65536
```
上面内容的意思是`testuser`用户赋予了一个subordinate user ID的 范围，从231072当接下来的按顺序的65536个整数。UID `231072`被映射为namespace(这里只容器)中的UID 0
即(root)。UID `231073`被映射为命名空间中的UID 1，以此类推。如果进程试图逃出namespace，则进程将会在主机上以一个无特权的较大数字的UID运行，这甚至根本不会映射到一个真正
的用户。这意味着这个进程在主机系统上根本没有任何特权。

>Multiple ranges:也可以赋予多个subordinate范围，对于一个给定的用户或用户组。通过添加给同一个用户或用户组在`/etc/subuid`和`/etc/subgid`中间中添加多个非重叠的映射范围。
这种情况下，Docker仅使用前5个mapping，这与内核仅限制`/proc/self/uid_map`和`/proc/self/gid_map`中的前5条是一致的。

当设置Docker使用`userns-map`特性是，你可以有选择的指定一个存在的用户或用户组。也可以指定为`default`,如果指定为`default`，将会创建用于此目的的`dockermap`用户和用户组。

>警告：一些发行版，如RHEL和CentOS7.3并不会自动的在`/etc/subuid`和`/etc/subgid`文件中添加新用户，你应该编辑这些文件，并添加非重叠的范围。详细步骤键下文

range范围不能重叠是非常重要的，以便进程在不同的namespace中不能获取访问权限。大多数发行版，在你添加和删除用户是，系统工具会为你管理ranges。

re-mapping对于容器是透明的，但这引入了一些配置上的复杂度，当容器需要访问Docker host上的一些资源的时候，如bind mounts到系统用户无法写入的文件系统。从安全的角度，
应尽量避免这样的场景。

#### Prerequistes

1. subordinate UID 和GID range必须关联到一个已存在的用户上。 用户拥有`/var/lib/docker`下的namespaced storeage directories.如果你不想使用已存在用户，Docker会
为你创建一个用户。 如果你想使用一个已存在的用户和用户ID，那就必须得存在。已存在用户在`/etc/passwd`和`/etc/group`文件中。使用`id`命令来验证：`$ id testuser`

2. namespace remapping是通过`/etc/subuid`和`/etc/subgid`文件完成的。这些文件在添加和移除用户时由系统自动管理，但RHEL和CentOS7.3需要你手动管理。

#### Enable userns-remap on the daemon

命令行方式：
```bash
$ dockerd --userns-remap="testuser:testuser"
```
推荐使用`daemon.json`配置的方式，编辑`/etc/docker/daemon.json`文件，添加如下内容：

```json
{
  "userns-remap": "testuser"
}
```
如果想使用`dockermap`用户，让Docker创建，应该将上面的值设置为`default`而不是`testuser`.
保存并重启docker.

