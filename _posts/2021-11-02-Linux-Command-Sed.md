---
layout: post
title: Linux命令之sed
author: tux
date: 2021-11-02
tags: Linux
---

# 1 简介

有时文本存在大量的需要删除或替换的内容，如果手动则工作量非常大。使用Linux的sed命令可以完成大量的替换和删除操作，功能非常强大。这里仅列举实际中遇到的场景进行示例。

就ES应用本身来讲，问题的排查思路可以从两个大方向开展：

# 2 示例
## 2.1 删除匹配到行的下一行内容

需求来源：使用smplayer播放器播放视频，不自动加载字幕，字幕配置没有问题。经检查发现是字幕文件有问题。问题字幕如下：
```
1
00:00:00,460 --> 00:00:05,190
Welcome. You must be excited about how to create and write your first go program!
1

2
00:00:05,430 --> 00:00:10,110
So fasten your seat belts and get your favorite drink wine is coffee.
2

3
00:00:10,110 --> 00:00:16,200
We're starting. Now I'm going to show you how to create and run a very simple Go program. You will print
3
```
正常字幕如下：
```
1
00:00:00,460 --> 00:00:05,190
Welcome. You must be excited about how to create and write your first go program!

2
00:00:05,430 --> 00:00:10,110
So fasten your seat belts and get your favorite drink wine is coffee.

3
00:00:10,110 --> 00:00:16,200
We're starting. Now I'm going to show you how to create and run a very simple Go program. You will print
```
从上面的对比可以看出，需要将问题字幕中每行英文字母下面的数字删掉。如果数量少可以手动删除，但如果有上千行手动就非常累了。

根据正常字幕的格式，可以使用sed，匹配所有字母开头的行，然后删除匹配到的行的后面一行。命令如下：
```bash
sed -i '/^[a-zA-Z]/{n;d}' file.srt
```
