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
使用如下命令确定视频提供的质量格式
```bash
sudo youtube-dl -F https://www.youtube.com/watch?v=ECIU3SQyUU4

 ECIU3SQyUU4: Downloading webpage
 ECIU3SQyUU4: Extracting video information
 ECIU3SQyUU4: Downloading js player do
 ECIU3SQyUU4: Downloading DASH manifest
[info] Available formats for ECIU3SQyUU4:
format code extension resolution  note
171         webm      audio only  DASH audio , audio@128k (worst)
140         m4a       audio only  DASH audio , audio@128k
139         m4a       audio only  DASH audio   49k , audio@ 48k (22050Hz), 1.19MiB
140         m4a       audio only  DASH audio  129k , audio@128k (44100Hz), 3.16MiB
171         webm      audio only  DASH audio  132k , audio@128k (44100Hz), 3.08MiB
172         webm      audio only  DASH audio  191k , audio@256k (44100Hz), 4.33MiB
141         m4a       audio only  DASH audio  255k , audio@256k (44100Hz), 6.27MiB
160         mp4       144p        DASH video , video only
278         webm      256x144     DASH video   96k , webm container, VP9, 1fps, video only, 2.14MiB
160         mp4       256x144     DASH video  111k , 13fps, video only, 2.70MiB
242         webm      240p        DASH video , video only
133         mp4       240p        DASH video , video only
242         webm      426x240     DASH video  223k , 1fps, video only, 4.62MiB
133         mp4       426x240     DASH video  253k , 25fps, video only, 6.00MiB
243         webm      360p        DASH video , video only
134         mp4       360p        DASH video , video only
243         webm      640x360     DASH video  397k , 1fps, video only, 8.30MiB
134         mp4       640x360     DASH video  620k , 25fps, video only, 13.78MiB
244         webm      480p        DASH video , video only
135         mp4       480p        DASH video , video only
244         webm      854x480     DASH video  798k , 1fps, video only, 16.47MiB
135         mp4       854x480     DASH video 1117k , 25fps, video only, 25.52MiB
247         webm      720p        DASH video , video only
136         mp4       720p        DASH video , video only
247         webm      1280x720    DASH video 1476k , 1fps, video only, 30.38MiB
136         mp4       1280x720    DASH video 2246k , 25fps, video only, 49.86MiB
248         webm      1080p       DASH video , video only
137         mp4       1080p       DASH video , video only
248         webm      1920x1080   DASH video 2427k , 1fps, video only, 50.28MiB
137         mp4       1920x1080   DASH video 4176k , 25fps, video only, 96.17MiB
17          3gp       176x144
36          3gp       320x240
5           flv       400x240
43          webm      640x360
18          mp4       640x360
22          mp4       1280x720    (best)
```
因为这是个音乐 MV，所以音质也可以选最好的，我们想下载 1080p 的 mp4 格式，注意 video 的 ID 是 137，audio 的 ID 是 141.使用如下命令下载
```bash
sudo youtube-dl -f 137+141 https://www.youtube.com/watch?v=ECIU3SQyUU4
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

