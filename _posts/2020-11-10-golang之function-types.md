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
https://golang.org/doc/codewalk/functions/中提到如下内容：
Go supports first class functions, higher-order functions, user-defined function types, function literals, closures, and multiple return values.

**User-defined function types**

In Go, functions can be passed around just like any other value. A function's type signature describes the types of its arguments and return values.

The action type is a function that takes a score and returns the resulting score and whether the current turn is over.

If the turn is over, the player and opponent fields in the resulting score should be swapped, as it is now the other player's turn.

```go
// An action transitions stochastically to a resulting score.
type action func(current score) (result score, turnIsOver bool)
```
首先看下面的代码：

```go
package main

import "fmt"

// Greeting function types
type Greeting func(name string) string

func say(g Greeting, n string) {
    fmt.Println(g(n))
}

func english(name string) string {
    return "Hello, " + name
}

func main() {
    say(english, "World")
}
```
输出`Hello, World`。
say()函数要求传入一个Greeting类型，因为english函数的参数和返回值与Greeting一样，参考接口的概念这里可以做类型转换，换个方式实现上面的功能，代码如下：

```bash
package main

import "fmt"

// Greeting function types
type Greeting func(name string) string

func (g Greeting) say(n string) {
    fmt.Println(g(n))
}

func english(name string) string {
    return "Hello, " + name
}

func main() {
    g := Greeting(english)
    g.say("World")
}
```
同样输出`Hello, World`，只是给Greeting类型添加了`say()`方法。前面提到函数类型表示所有包含相同参数和返回类型的函数集合。开始先把`func(name string) string`这样的函数声明成Greeting类型，接着通过Greeting(english)将english函数转换成Greeting类型。通过这个转换以后，就可以借助变量g调用Greeting类型的say()方法。上面两段代码的差异就是go的类型系统添加方法和类C++语言添加类型方法的差异。具体可参考《Go语言编程》第3章为类型添加方法这一节。

既然是函数集合，那只有一个函数显然是不足以说明问题的。
```go
package main

import "fmt"

// Greeting function types
type Greeting func(name string) string

func (g Greeting) say(n string) {
    fmt.Println(g(n))
}

func english(name string) string {
    return "Hello, " + name
}

func french(name string) string {
    return "Bonjour, " + name
}

func main() {
    g := Greeting(english)
    g.say("World")
    g = Greeting(french)
    g.say("World")
}
```

输出如下：

```
Hello, World
Bonjour, World
```
在其他语言里，有些函数可以直接作为参数传递，有些是以函数指针进行传递，但是都没有办法像go这样**可以给函数类型"增加"新方法**.

回到net/http包中的HandlerFunc类型，任何函数只要遵循文档中`type HandlerFunc func(ResponseWriter, *Request)`的要求，就可以转换成HandlerFunc类型，也就可以调用`func(HandlerFunc) ServeHTTP`函数。

net/http包中HandlerFunc类型定义如下：

```go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```
可见HandlerFunc是用type定义的函数，而函数的类型就是开始传入的类型`func(ResponseWriter, *Request)`. ServeHTTP是HandlerFunc的一个方法(注意，golang中的方法和函数不是一回事)。并且HandlerFunc实现了Handler接口，Handler接口定义如下：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```
回到HandleFunc方法中，`mux.Handle(pattern, HandlerFunc(handler))`的第二个参数是把传入的函数handler强转成HandlerFunc类型，这样handler就实现了Handler接口。到此我们明白`HandlerFunc(handler)`是把普通函数强转成type定义的函数。

### 总结

函数在golang类型系统中作为一等公民，可以像使用变量一样来使用它。
如：
- 将函数作为函数的参数：函数回调
- 将函数作为函数的返回值：闭包
- 将函数视为一种类型，可以进行类型转换，为其添加新方法

### Reference

- https://www.jianshu.com/p/fc4902159cf5
- https://studygolang.com/articles/23072
