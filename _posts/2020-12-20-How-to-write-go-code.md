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

为了编译和运行一个简单的程序，第一步就是选择一个模块路径(这里使用example.com/user/hello)，并创建一个go.mod文件来声明这个模块路径。

```bash
$ mkdir hello # Alternatively, clone it if it already exists in version control.
$ cd hello
$ go mod init example.com/user/hello
go: creating new go.mod: module example.com/user/hello
$ cat go.mod
module example.com/user/hello

go 1.14
$
```
go源码文件的第一个语句必须是`package name`。可执行命令必须使用`package main`.接下来创建一个名为hello.go的文件，并写入如下代码：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, world.")
}
```
现在可以使用go工具对程序进行编译，安装：

```bash
$ go install example.com/user/hello
$
```
上面的命令将会构建hello命令，并产生一个二进制文件。接着会将二进制文件安装到`$HOME/go/bin/hello`(or, under Windows, %USERPROFILE%\go\bin\hello.exe)

安装目录由GOPATH和GOBIN环境变量控制。如果设置了GOBIN，则二进制文件会安装到GOBIN目录。如果设置了GOPATH，则二进制文件将会安装到GOPATH列表的第一个目录的bin子目录中。否则二进制文件将会安装到默认GOPATH目录的bin子目录中($HOME/go or %USERPROFILE%\go).

可以使用go env命令可移植的方式设置环境变量的默认值。

```bash
$ go env -w GOBIN=/somewhere/else/bin
$
```
使用go env -u来取消前面go env -w对环境变量的设置：

```bash
$ go env -u GOBIN
$
```
go install命令的使用，需要模块的上下文包含当前工作目录，如果当前工作目录不在模块example.com/user/hello中，则go install命令会失败。

为了使用方便，够命令接受当前工作目录的相对路径，如果没有给定路径，则默认只想当前工作目录中的package。因此在我们的当前工作目录中，下面几个命令是等同的：

```bash
$ go install example.com/user/hello
$ go install .
$ go install
```
接下来我们需要运行一下程序，以确保其能正常工作。为了使用方便，我们添加安装目录到PATH变量中，以便能方便的运行生成的二进制文件。

```bash
# Windows users should consult https://github.com/golang/go/wiki/SettingGOPATH
# for setting %PATH%.
$ export PATH=$PATH:$(dirname $(go list -f '{{.Target}}' .))
$ hello
Hello, world.
$
```

如果你使用代码版本控制系统，现在是初始化仓库很好的时候。添加文件，并提交你的第一次变化。但这步是可选的，编写go代码并不需要代码版本控制系统。

```bash
$ git init
Initialized empty Git repository in /home/user/hello/.git/
$ git add go.mod hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 7 insertion(+)
 create mode 100644 go.mod hello.go
$
```
go命令根据给定的module path请求对应的HTTPS URL，并读取返回的HTML中的元数据来定位仓库。很多服务已经提供了仓库中go源代码的数据，所以使你的module能被其他人访问的最简单的办法是让module path能匹配到仓库的地址。

## 1.4 Importing packages from your module 从你的模块中导入包

接下来编写一个名为morestrings的package，并在hello程序中使用它。首先为包创建一个目录$HOME/hello/morestrings。接着在此目录中创建名为reverse.go文件，并写入如下内容：

```go
// Package morestrings implements additional functions to manipulate UTF-8
// encoded strings, beyond what is provided in the standard "strings" package.
package morestrings

// ReverseRunes returns its argument string reversed rune-wise left to right.
func ReverseRunes(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```
因为我们的ReverseRunes函数是已大写字母开头，因此是可导出的，即可以被其他的包通过import morestrings来使用。

接下来通过使用go build来测试包的编译

```bash
$ cd $HOME/hello/morestrings
$ go build
$
```

上面的命令不会产生文件，取而代之的是将编译后的包保存在本地的build cache中。

在确认morestrings包编译成功后，让我们在hello程序中使用它。为了使用它，我们首先需要修改原来的$HOME/hello/hello.go文件来使用morestrings包。

```go
package main

import (
	"fmt"

	"example.com/user/hello/morestrings"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
}
```

接着安装hello程序

```bash
$ go install example.com/user/hello
```

运行新版本的程序，你将看到一个新的反序的信息：

```bash
$ hello
Hello, Go!
```
## 1.5 Importing packages from remote modules从远程模块中导入包

一个导入路径描述了如何通过版本控制系统如git或Mercurial来获取包中的源码。go工具使用这个特性来获取远程仓库的包。比如在你的程序中使用github.com/google/go-cmp/cmp

```bash
package main

import (
	"fmt"

	"example.com/user/hello/morestrings"
	"github.com/google/go-cmp/cmp"
)

func main() {
	fmt.Println(morestrings.ReverseRunes("!oG ,olleH"))
	fmt.Println(cmp.Diff("Hello World", "Hello Go"))
}
```
当你运行命令如go install, go build, go run时，go命令会自动下载远程模块并在你的go.mod文件中记录它的版本

```bash
$ go install example.com/user/hello
go: finding module for package github.com/google/go-cmp/cmp
go: downloading github.com/google/go-cmp v0.4.0
go: found github.com/google/go-cmp/cmp in github.com/google/go-cmp v0.4.0
$ hello
Hello, Go!
  string(
- 	"Hello World",
+ 	"Hello Go",
  )
$ cat go.mod
module example.com/user/hello

go 1.14

require github.com/google/go-cmp v0.4.0
$
```
模块以来会自动下载到GOPATH目录中的pkg/mod子目录中。对于给定版本的模块的下载内容，在所有依赖这个版本模块的其他模块是共享的，所以go命令会将这些文件和目录标记为只读。如果要清楚所有下载的模块，需要给go clean命令指定-modcahce选项：

```bash
$ go clean -modcache
$
```

## 1.6 test测试

Go有一个有go test命令和testing包组成的轻量级测试框架。你可以创建一个已_test.go结尾的文件来写一个测试，这个文件需要包含名为`TestXXX`的函数，函数签名为`func (t *test.T)`。测试框架会运行每个测试函数。如果测试函数调用了失败函数如t.Error或t.Fail，则认为测试没有通过。

为morestrings包创建一个测试，首选需要创建文件$HOME/hello/morestrings/reverse_test.go，并写入如下代码：

```go
package morestrings

import "testing"

func TestReverseRunes(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := ReverseRunes(c.in)
		if got != c.want {
			t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```
接着使用go test运行测试：

```bash
$ go test
PASS
ok  	example.com/user/morestrings 0.165s
$
```
