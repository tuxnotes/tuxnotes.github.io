---
layout: post
title: Fibonacci-recursive-with-cache
date: 2020-11-02
author: tux
tags: algorithms
---

Fibonacci数列直接采用递归的方法计算时会造成大量的重复计算，如在计算fib(6)时，计算步骤如下图所示：

![](/assets/img/fibonacci.png)

从上图可以看到，颜色相同的都是重复计算，当n越大，重复的越多，所以我们可以使用一个map把计算过的值存起来，每次计算的时候先看看map中有没有，如果有就表示计算过，直接从map中取，如果没有就先计算，计算完之后再把结果存到map中。

