---
layout: post
title: Stack and Heap: In a Nutshell-译
date: 2021-02-02
author: tux
tags: java
---

# Stack与Heap简明教程

原文链接：https://blog.usejournal.com/til-9-stack-and-heap-in-a-nutshell-48e61fe5d140

## Stack栈

Stack栈是RAM中的一块区域，用于分配由函数创建的临时变量。栈主要用作静态内存分配。

栈是线程相关的，因此栈中存储的变量的作用域是线程级别。栈主要是有CPU来管理。以先进先出的方式存储由函数创建的变量，函数退出后释放分配的内存。

与heap相比，栈的大小比较小，但是有更快的读写速度，并且优点是内存是CPU帮你管理的。栈更加快速的原因是内存的分配和释放与堆相比更加轻量。

何时使用栈？

在编译前你就知道你的数据需要分配多少内存，并且你需要存储一些占用空间小的局部变量，并且需要快速的访问分配和释放的时候就可以考虑使用栈。

## Heap堆

堆是RAM中占据的较大的比例的空间。堆使用动态内存分配。

堆的作用域是整个应用的范围，它可以从应用的任何部分进行访问。堆中的存储单元是相互独立的，并可以在任何时候进行随机访问。在多线程的环境中，每个线程有自己的栈空间，但堆是共享的。

堆的大小受限于虚拟空间的大小。堆内存的管理不是由CPU处理的，因此其速度较慢，且需要人工分配和释放，并释放使用过后的空间。如果管理不当，则可能发生内存泄露。这导致内存对其他进程也是不用的。

何时使用堆？当你不知道在运行时，你的数据需要占用多少内存是，且需要存储较大的变量，能全局访问的时候使用堆。

## 两者的异同

栈和堆的异同如下图所示：

![](https://miro.medium.com/max/875/1*OGHcuzHaFL8Rz7gL17PriQ.png)

## Java关于栈和堆的规范注释

- Java的堆空间用于对象和JRE类。任何时候新建的对象都存储在堆中
- 堆空间的管理通过两种方式：Garbage collection and young-generation, old-generation
- GC运行在堆内存中，开释放没有任何引用的对象
- 使用新生代和老年代将Java堆内存划分，有助于GC的优先级顺序
- Java的栈内存用于线程的执行
- 如果栈中没有内存用于函数的调用和局部变量，JVM将抛出 java.lang.StackOverFlowError；如果是没有足够的堆来创建新的对象，JVM将抛出java.lang.OutOfMemoryError: Java Heap Space
- -Xss:用于定义线程栈内存的起始大小
- -Xms和-Xmx用于定义起始堆内存大小和最大堆内存大小

## Reference

- https://stackify.com/java-heap-vs-stack/
- https://dzone.com/articles/stack-vs-heap-understanding-java-memory-allocation
- https://www.geeksforgeeks.org/stack-vs-heap-memory-allocation/
- http://net-informations.com/faq/net/stack-heap.htm