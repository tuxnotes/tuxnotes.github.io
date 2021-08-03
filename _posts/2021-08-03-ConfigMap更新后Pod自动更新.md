---
layout: post
title: ConfigMap更新后Pod也自动更新
date: 2021-08-03
author: tux
tags: kubernetes
---

# 1 需求描述

在更新ConfigMap后，如何保证引用这个ConfigMap的Pod也自动更新。

# 2 解决方案

Pod是否随ConfigMap自动更新，取决于Pod使用ConfigMap的方式。如果ConfigMap是以文件系统的方式挂载的，则在更改后的短时间内进行更新(有一个TTL值).如果是以环境变量的形式使用ConfigMap，则就没有那么幸运了。环境变量的建立是在进程启动的时候就确定的，这种情况下，你需要重启进程。

If the ConfigMap is mounted onto the file system of the pod, the file will be updated on-disk. So, in the case the ConfigMap is a volume mount in a pod, the file contents will be updated after a small delay but no signal is sent to any processes at that point. You could listen for filesystem events (inotify, epoll, etc) and react accordingly, or even just read from the FS every so often. If you need to consume values as environment variables, there’s no real way to update them. Put simply, that’s not how environment variables work. A process is started in a particular environment, and that process can muck with the environment it’s running in, but an outside process has no way to change it. It would be the responsibility of your process to reflect changes in a ConfigMap (probably by consuming events from the kubernetes API) into its own environment which would be kind of silly—at that point, why use environment variables at all.

In either case, you can always run a ConfigMap watching sidecar that can send your process a signal or something. There exist a number of those floating around, just search for “ConfigMap reloader.”

