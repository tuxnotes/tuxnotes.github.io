---
layout: post
title: Garbage Collection:Young vs Old Generation译
date: 2021-01-29
author: tux
tags: java
---

# 垃圾回收：Young vs Old Generations

## 垃圾回收

垃圾回收运行与heap内存，用于移除没有任何引用的对象。首先来看Oracle官方对垃圾回收的定义：

自动垃圾回收是一个过程，查找堆内存，标记哪些对象正在使用，哪些对象没有被使用，然后删除没有使用的对象。使用中的对象或被引用的对象意味着你的程序任然有指针指向那个对象。不在使用或没有引用的对象，意思是对象不被程序的任何部分引用。因此没有引用对象的内存就可以清除。

## Young Generation and Old Generation新生代和老年代

谈到内存使用的时候会涉及栈和堆内存。新创建的对象会分配到堆内存中。heap内存的使用如下图所示：

![](https://miro.medium.com/max/875/1*XXqPGJCxQfri7qE_GrSB8Q.png)

HotSpot Heap Structure by [Oracle](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)

Young Generation包括：Eden,Survivor0(S0),Survivor1(S1)

- Eden: 对象第一次初始化的时候存储的位置。占用空间比S0和S1大
- 当Young Generation填满的时候，minor garbage collection就会允许，仅移除unused objects和objects that are not referenced
- minor garbage collection运行后存活下来的对象会被转移到其他的Survivor Space
- 每次minor garbage collection后都存活下来的可以认为是old.young generation到old generation的转变通常会设置一个门槛值。成为老年代会后进入堆内存的Tenured的部分
- 老年代用于存储long-lived对象
- 最后老年代也会被垃圾回收，这成为major garbage collection
- Permanent generation包含了JVM用于描述类和方法的元数据。主要存放的是应用程序使用的JVM运行时的类。Java SE 库中的类和方法就存储在这个位置。

# 参考

- https://codeahoy.com/2017/08/06/basics-of-java-garbage-collection/
- https://stackoverflow.com/questions/2129044/java-heap-terminology-young-old-and-permanent-generations
- https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html

