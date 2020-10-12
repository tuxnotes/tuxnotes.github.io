---
layout: post
title: Debian9升级到Debian10的post install
date: 2020-10-12
author: tux
tags: Debian
---

# 1 升级的大概流程

参考链接：https://www.cyberciti.biz/faq/update-upgrade-debian-9-to-debian-10-buster/

1. 根据需要备份系统
2. update installed packages

```bash
sudo atp update
sudo apt upgrade
sudo apt full-upgrade
sudo apt --purge autoremove
sudo reboot
```

3. update /etc/apt/sources.list file

```bash
sudo cp -v /etc/apt/sources.list /root/
sudo cp -rv /etc/apt/sources.list.d/ /root/
sudo sed -i 's/stretch/buster/g' /etc/apt/sources.list
sudo sed -i 's/stretch/buster/g' /etc/apt/sources.list.d/*
```

then update the package list

```bash
sudo apt update
```
4. minimal system upgrade

```bash
sudo apt upgrade
```

5. upgrading debian 9 to debian 10

```bash
sudo apt full-upgrade
```

then reboot

```bash
sudo reboot
```

# 2 post install

1. sudo apt install nautilus

2. disable tracker related because of 100% CPU usage and more memory

参考链接：https://www.linuxuprising.com/2019/07/how-to-completely-disable-tracker.html

```bash
systemctl --user mask tracker-store.service tracker-miner-fs.service tracker-miner-rss.service tracker-extract.service tracker-miner-apps.service tracker-writeback.service
tracker reset --hard
```

3. 重新安装dash to dock,因为升级后gnome-shell版本改变了

```bash
rm -rf ~/.local/share/gnome-shell/extensions/dash-to-dock@micxgx.gmail.com/*
wget https://extensions.gnome.org/extension-data/dash-to-dock%40micxgx.gmail.com.v64.shell-extension.zip
unzip dash-to-dock@micxgx.gmail.com.v64.shell-extension.zip -d ~/.local/share/gnome-shell/extensions/dash-to-dock@micxgx.gmail.com/
sudo reboot
```
