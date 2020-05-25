---
layout: post
date: 2020-04-06
title: Python面向对象之staticmethod与classmethod
author: tux
tags: Python
---

### 定义方式的不同

- staticmethod: 既不需要`self`也不要`cls`作为其第一个参数。staticmethod装饰的方法，其行为与普通函数一样，只是碰巧在类中定义了，而不是在模块层面。除了其调用方式是源自于一个实例或类来调用。
- classmethod：定义用于操作类的方法，而不是操作实例的方法。classmethod改变了方法的调用方式，因此类方法的第一个参数是类本身，而不是实例。classmethod最常见的用途是定义备选构造方法。

### 使用目的的不同

- staticmethod： 与普通函数没什么区别，完全可以定义在模块层面。定义在类中，是出于功能分组的目的，将逻辑上与类紧密相关的函数定义在类中。
- classmethod：定义备选构造方法。类方法可以操作并访问于类的属性。

### 示例

假设有一个时间的类：
```python
class Date:
    #构造函数
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    def tomorrow(self):
        self.day += 1
```
接下来，遇到一个问题，如果实例化这个类时参数的格式是('dd-mm-yyyy') 这样的，要怎么办呢？
这个时候classmethod就非常好用：
```python
@classmethod
    def from_string(cls, date_str):
        year, month, day = tuple(date_str.split("-"))
        return cls(int(year), int(month), int(day))
```
下面的写法便于理解：
```python
@classmethod
def from_string(cls, date_as_string):
    day, month, year = map(int, date_as_string.split('-'))
    date1 = cls(day, month, year)
    return date1

date2 = Date.from_string('11-09-2012')
```

那么staticmethod可以解决什么问题呢？这里假设需要对时间进行校验，就可以用下面这个staticmethod来完成：
```python
@staticmethod
def is_date_valid(date_as_string):
    day, month, year = map(int, date_as_string.split('-'))
    return day <= 31 and month <= 12 and year <= 3999

# usage
is_date = Date.is_date_valid('11-09-2012')
```
可以看出来，staticmethod定义的方法在逻辑上是属于这个类的，但在调用的时候却不需要任何类的实例化.