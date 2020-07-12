---
layout: post
title: Golang之map初学
date: 2020-07-12
author: tux
tags: golang map
---

### 简介

hash table是设计精巧，使用灵活的数据结构之一。它是一个key/value对组成的无序集合。value可以通过其绑定的key进行更新，查找，删除操作。
在golang中，map是对hash table的引用。map中的key要有相同的类型，同样value也要是相同的类型。其中key的类型必须是可以通过`==`进行比较的类型，虽然
浮点类型也可以比较，但浮点类型不适合作为key。value的类型没有任何限制。

### map需要注意的点

- 1 在map中使用一个key进行查找时，如果key不存在，则返回value类型的零值。所以在map中删除一个不存的key时也是安全的。
- 2 map中的元素不是变量，不能对其进行取地址操作，如： `_ = &ages["bob"] // compile error: cannot take address of map element`因为map增长时会导致元素的rehash到新的存储位置

因为key不存在时会返回value类型的零值，有时想区分元素是不存在，还是其已经存在，但值恰好是value的零值，可使用如下代码：

```go
if age, ok := ages["bob"]; !ok { /* ... */ }
```

### map的零值

map类型的零值时nil，此时没有引用任何hash table。对nil map reference进行查找，删除(delete)，求长度(len),遍历(range loops)是安全的，其行为与空map相同，如：

```go
var ages map[string]int
fmt.Println(ages == nil) // "true"
fmt.Println(len(ages) == 0) // "true"
```
但向nil map存储元素是会引发panic：

```go
ages["carol"] = 21 // panic: assignment to entry in nil map
```
因此map进行存储元素前必须初始化，所以map类型的变量声明，推荐使用内置函数make来完成，如`**ages := make(map[string]int)**`

### map按key排序

map的迭代是无序的，如果需要按key的一定顺序进行访问，则需要对key进行排序，如果key的类型是string，可使用如下代码：

```go
import "sort"

var names []string
for name := range ages {
names = append(names, name)
}
sort.Strings(names)
for _, name := range names {
    fmt.Printf("%s\t%d\n", name, ages[name])
}
```
如果提前知道map的长度，指定names的cap则更加高效。

### map实现set

golang没有set类型，因为map的key的唯一性，利用这个特性可实现集合：
```go
func main() {
seen := make(map[string]bool) // a set of strings
input := bufio.NewScanner()
for input.Scan() {
    line := input.Text
    if !seen[line] {
        seen[line] = true
        fmt.Println(line)
    }
}
if err := input.Err() err != nil {
    fmt.Fprintf(os.Stderr, "dedup: %v\n", err)
    os.Exit(1)
}
}

