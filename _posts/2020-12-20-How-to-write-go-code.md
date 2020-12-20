---
layout: post
title: How to write go code[译]
date: 2020-12-20
author: tux
tags: golang
---

# 1 How to write go code如何编写go代码

## 1.1 Introduction介绍

本文档演示了使用module开发一个简单的go的package。引入go tool，这是获取，构建，安装go模块，包和命令的标准方式。

本文假定使用go 1.13，且GO111MODULE环境变量没有设置。

## 1.2 代码的组织方式

Go程序以包的方式进行组织。一个包是在相同目录下的go源文件的集合，会一起进行编译。在一个源码文件中定义的函数，类型，变量和常量对同一个包内的其他源码文件都是可见的。

一个仓库包含一个或多个模块。一个模块是一些相关的go package的集合，且一起发行。一个仓库一般只包含一个模块，位于仓库的根目录。`go.mod`文件声明了模块的路径。模块包含的包会在包的目录中包含自己的`go.mod`文件。

需要注意的是在构建之前，你并不需要将代码发布到远程仓库。应为模块可以在本地定义，而不属于任何远程仓库。但将代码以一种将来要远程发布的方式进行组织是一个非常好的习惯。

每个模块的路径不仅仅是其包含的包的导入路径前缀，它还提示了go命令从哪里查找病下载它。比如，为了下载模块`golang.org/x/tools`，go命令会访问https://golang.org/x/tools提示的仓库。

导入路径是一个字符串，用于导入包。一个包的导入路径是其模块路径加上其子目录。比如，模块`github.com/google/go-cmp`在`cmp/`目录中包含一个包，则这个包的导入路径就是`github.com/google/go-cmp/cmp`.标准库的包没有模块路径的前缀。

## 1.3 Your first program

