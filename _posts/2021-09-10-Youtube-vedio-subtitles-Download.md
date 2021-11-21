---
layout: post
title: Youtube Vedio Subtiles Download
author: tux
date: 2021-09-08
tags: Linux Command
---

# 简介

介绍YouTube视频的命令行下载工具youtube-dl.前提条件是需要安装了ffmpeg，且ffmpeg工具加入了PATH。

# 使用

## 安装

```bash
sudo pip3 install you-get # 用于从B站下载视频
sudo pip3 install youtube-dl # 用于从youtube网站下载视频
sudo pip3 install bilili # you-get目前有问题，bilili也可以从B站下载视频，但线程数不要太大，线程数设置为2是比较稳定的
```

## 使用

`you-get`的使用,下载B站视频

```bash
you-get -i URL # 获取视频信息
you-get -l --format=dash-flv720 URL # 当链接包含多个视频时(有多集),需要使用-l.--format=dash-flv720用于下载720p的mp4格式的视频
```

`youtube-dl`下载YouTube视频

```bash
youtube-dl -f best --proxy socks5://127.0.0.1:1080 URL # URL可以使用单引号括起来，因为URL中可能包含&符号
youtube-dl -f best --yes-playlist --proxy socks5://127.0.0.1:1080 URL
youtube-dl -f best --yes-playlist --playlist-start 2 --proxy socks5://127.0.0.1:1080 URL # 从第2个视频开始，第一个视频编号为1.--playlist-end指定结束下载第几个视频，默认是最后一个
```
`bilili`下载B站视频

```bash
bilili -n 2 -q 74 URL # 74表示视频质量为720p
```
具体使用方式可以参考文档：

- https://bilili.nyakku.moe/cli/
- https://pypi.org/project/bilili/
  
附YouTube的网站下载方式：

- https://savesubs.com/zh # 下载字幕
- http://en.savefrom.net/ # 下载视频

mp4视频与字幕合并方法：

```bash
ffmpeg -i infile.mp4 -f srt -i infile.srt -c:v copy -c:a copy -c:s mov_text outfile.mp4
```

