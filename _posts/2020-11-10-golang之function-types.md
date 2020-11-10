---
layout: post
title: golang之function types
date: 2020-11-10
author: tux
tags: golang
---

### 1 官方对Function types的解释

链接：https://golang.org/ref/spec的Function types部分

A function type denotes the set of all functions with the same parameter and result types. The value of an uninitialized variable of function type is nil.

说明function type是函数的集合，这些函数有相同的参数类型和返回值类型。当然这里也包括参数的个数。

参数列表或返回值列表中，参数名称要么全部出现要么全不出现。如果出现了，每个参数名代表一项指定类型的参数。且函数签名中每个非空参数名称必须唯一。参数和返回值列表必须使用括号括起来，除了仅有一个非命名的返回值的情况。函数签名中的列表可以包含variadic类型，如:

```go
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```



