---
layout: post
title: Debian Repository Format
author: tux
date: 2021-12-31
tags: Debian
---

这里主要对Debian源的配置做个简单的说明，参考连接：https://wiki.debian.org/DebianRepository/Format

Debian package archive主要用于自动的存储和分发软件包。

几乎所有的包管理者都是libapt用于从外部介质和互联网获取安装包。

**sources.list**文件指定了包安装源的格式：

```
deb uri distribution [component1] [component2] [...]
```

例如

```
deb https://deb.debian.org/debian stable main contrib non-free
```
这里的`deb`指定了这个源是二进制安装包，`deb-src`是源码包。

一个archive可以有二进制安装包，也可以有源码包，当然也可以两者均有。

`uri`，如这里的`https://deb.debian.org/debian`指定了这个archive的根root。一遍Debian archive在`debian/`目录，但是也可以是其他的目录如，`pub/linux/debian`

`distribution`部分(如示例中的`stable`)是**$ARCHIVE_ROOT/dists**下的子目录。它还会包含子目录，如`stable/updates`.distribution通常对应于Release文件中的Suite或Codename.

从apt仓库中下载包会从$ARCHIVE_ROOT/dists/$DISTRIBUTION目录下载一个InRelease或Release文件。

Release文件列出了此distribution的索引文件以及其hash值(列出的索引文件是相对于Release文件的位置)

要下载main component的索引，apt会扫描Release文件中对应main目录下文件的hash值，比如https://deb.debian.org/debian/dists/testing/main/binary-i386/Packages.gz应该在 https://deb.debian.org/debian/dists/testing/Release文件中为main/binary-i386/Packages.gz

二进制包索引在component目录下的binary-$arch子目录，source索引在source子目录。

package索引列出了source或二进制相对于archive root的位置。

为了避免文件重复，二级制或源码包通常会保存在archive root下的pool子目录。尽管Packages或Sources所有可以列出相对于archive root的任何路径，但还是建议将包放单archive root下除dists之外的子目录，而不是直接放到archive root下。

例如，清华大学开源软件镜像站kali的根为：https://mirrors.tuna.tsinghua.edu.cn/kali/

根下有如下目录

![](/assets/img/tuna-kali.png)

根下的dists目录下有如下子目录

![](/assets/img/tuna-kali-dists.png)

kali-rolling下的目录如下

![](/assets/img/tuna-kali-comp.png)

根下的pool目录下如如下子目录,pool目录下才是真正存放deb包的位置

![](/assets/img/tuna-kali-pool.png)

所有为自己的kali配置清华的源如下：

```bash
$ cat /etc/apt/sources.list.d/tuna.list
deb https://mirrors.tuna.tsinghua.edu.cn/kali/ kali-rolling main contrib non-free
```

同理，清华源没有为kali提供docker-ce的安装源，但是kali是基于Debian的，清华源提供了Debian的docker-ce安装源，所以kali就可以使用为Debian配置的docker-ce的源了
```bash
$ cat /etc/apt/sources.list
# See https://www.kali.org/docs/general-use/kali-linux-sources-list-repositories/
# deb http://http.kali.org/kali kali-rolling main contrib non-free

# Additional line for source packages
# deb-src http://http.kali.org/kali kali-rolling main contrib non-free
#deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian kali-rolling stable
deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian bullseye stable
# deb-src [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian kali-rolling stable
```

可以看到配置中没有体现任何关于kali的内容，安装清华源上的提示distribution部分是通过`lsb_release -cs`命令得到，但这里仍然不能使用kila-rolling作为distribution的值，一切都遵循镜像源网站上的目录结构，上面有啥就配置啥，截图如下

首先是docker-ce源的根

![](/assets/img/tuna-docker-root.png)

其次是dists子目录

![](/assets/img/tuna-docker-dists.png)

可以上面的dists目录下并没有kali-rolling目录

由于安装的kali是基于bullseye的Debian的，所以查看一下bullseye目录

![](/assets/img/tuna-docker-bullseye.png)

上图中stable,test,nightly对应于component部分，而pool下也会有这三个目录，是真正存放deb包的位置。

