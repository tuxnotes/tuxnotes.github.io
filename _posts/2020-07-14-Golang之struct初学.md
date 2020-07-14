---
layout: post
title: Golang之struct初学
date: 2020-07-14
author: tux
tags: golang struct
---

### golang结构体

#### 结构体

结构体是将零个或多个**任意类型**的**命名变量**组合在一起的聚合数据类型。每个变量叫做结构体的成员，示例如下：

```go

type Employee struct {
    ID int
    Name string
    Address string
    DoB time.Time
    Position string
    Salary int
    ManagerID int
}
```

结构体成员变量通常一行写一个，变量的名称在类型的前面。但是相同类型的连续成员变量可以写在一行，如Name和Address：
```go
type Employee struct {
    ID int
    Name, Address string
    DoB time.Time
    Position string
    Salary int
    ManagerID int
}
```
**成员变量的顺序对结构体的同一性很重要**，如果将也是字符串类型的Position和Name，Address组合在了一起，或交换了Name和Address的顺序，那么就定义了一个不同的结构体。一般来讲，只组合相关的成员变量。

##### 需要注意的点

**命名结构体类型，假设名称为s，不可以定义一个拥有相同结构体类型s的成员变量**,也就是一个聚合类型不可以包含它自己(同样的限制对数组也使用).但是s中可以定义一个s的指针类型，即`*s`。这样就可以创建一些递归数据结构，如链表和树。

#### 空结构体

没有任何成员变量的结构体称为空结构体，写做`struct{}`.它没有长度，也不携带任何信息，但有时会很有用。有一些go程序员使用它来代替被当做集合使用的map中的布尔值，来强调只有键是有用的，**但这种方式节约的内存很少且语法复杂，所以一般尽量避免这样使用**。

```go
seen := make(map[string]struct{}) // 字符串集合
// ...
if _, ok := seen[s];!ok {
    seen[s] = struct{}{}
    // ...首次出现s...
}
```

#### JSON

JSON是一种发送和接受格式化信息的标准，其并不是唯一的标准，xml，ASN.1和google的Protocol Buffer都是相似的标准，各自有使用的场景。但因为JSON简单，可读性强，并且支持广泛，所以用的最多。

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Movie struct {
	Title  string
	Year   int  `json:"released"` // field tags
	Color  bool `json:"color,omitempty"`
	Actors []string
}

var movies = []Movie{
	{
		Title: "Casablanca",
		Year:  1942,
		Color: false,
		Actors: []string{
			"Humphrey Bogart",
			"Ingrid Bergman",
		},
	},
	{
		Title: "Cool Hand Luke",
		Year:  1967,
		Color: true,
		Actors: []string{
			"Paul Newman",
		},
	},
	{
		Title: "Bullit",
		Year:  1968,
		Color: true,
		Actors: []string{
			"Steve McQueen",
			"Jacqueline Bisset",
		},
	},
}

func main() {
	data, err := json.MarshalIndent(movies, "", "    ")
	if err != nil {
		fmt.Printf("JSON marshaling failed: %s\n", err)
	}
	fmt.Printf("%s\n", data)

	// var antidata []struct{
	// 	Title string
	// 	Actors []string
	// }
	// var titles []string{Title string}
	var years []struct {
		Released int `json:"released"` // JSON字符串中的key为小写，所以定义可导出成员名，且还要使用field tag
	}
	if err := json.Unmarshal(data, &years); err != nil {
		fmt.Printf("%v\n", err)
	}
	fmt.Println(years)
}
```
成员标签(field tag)定义可以是任意字符串，但按照习惯，是由一串空格分开的`key： "value"`对。tag的值使用双引号括起来，所以标签都是原生的字符串字面量。tag的键 `json` 控制着`encoding/json`包的行为。`encoding/...`等包也遵循这样的规则。tag的值的第一部分指定了Go结构体成员对应JSON中字段的名字。成员标签通常这样使用，比如`total_count`对应Go里面的`TotalCount`。`Color`的标签还有一个额外的选项`omitempty`，表示如果成员的值是零值或为空，则不输出这个成员到JSON中。

marshal的逆操作将JSON字符串解码为go数据结构，这个过程为unmarshal，由json.Unmarshal实现。通过合理定义Go的数据结构，可以选择将哪部分json数据解码到结构体对象中，哪些数据可以丢弃。

很多web服务都提供JSON接口，通过发送HTTP请求来获取想要得到的JSON信息。