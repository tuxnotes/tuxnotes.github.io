---
layout: post
title: VScode server在Linux中离线安装
date: 2020-06-24
author: tux
tags: vscode
---

### VScode server在Linux中的离线安装方式

参考链接：https://stackoverflow.com/questions/56671520/how-can-i-install-vscode-server-in-linux-offline

在使用VScode remote-ssh extension连接远程主机的时候，远程主机需要连接网络下载vscode-server，然后进行安装，但远程主机不具备联网条件。此时可根据安装过程中的输出信息，进行离线安装。首次安装时，由于不能联网，会安装失败，但在终端会输出一些信息。其中有一项commit id，是一个十六进制的字符的字符串。

使用这个commit id 拼接vscode-server的下载地址：

```
https://update.code.visualstudio.com/commit:${commit_id}/server-linux-x64/stable
```

然后解压下载后的`vscode-server-linux-x64.tar.gz`,解压开始是一个目录`vscode-server-linux-x64`.将此目录中的所有内容拷贝到`~/.vscode-server/bin/${commit id}/`下，注意不是将`vscode-server-linux-x64`拷贝到`~/.vscode-server/bin/${commit id}/`下。

最后在`~/.vscode-server/bin/${commit id}/`下创建一个文件名为`0`的文件：

```bash
$ touch 0
```

然后再次远程连接即可。
