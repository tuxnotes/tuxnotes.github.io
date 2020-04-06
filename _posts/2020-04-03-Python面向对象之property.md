---
layout: post
title: 'Python面向对象之property'
date: 2020-04-03
author: tux
tags: Python
---

### 背景

调用一个实例或类的属性时，会返回其存储的值。在不使用property的情况下，通常要使用方法来实现对属性的修改或约束。这样带来的问题是，当类添加或修改了属性的方法，调用方就要修改其代码，来满足变更。
```python
class Celsius:
    def __init__(self, temperature = 0):
        self.temperature = temperature

    def to_fahrenheit(self):
        return (self.temperature * 1.8) + 32
```
此时的调用方式是`obj.temperature`.但调用方发现，稳定不能低于-273.15.要实现这个约束，将代码修改为如下：
```python
class Celsius:
    def __init__(self, temperature = 0):
        self.set_temperature(temperature)

    def to_fahrenheit(self):
        return (self.get_temperature() * 1.8) + 32

    # new update
    def get_temperature(self):
        return self._temperature

    def set_temperature(self, value):
        if value < -273:
            raise ValueError("Temperature below -273 is not possible")
        self._temperature = value
```
此时所有调用这个类的代码都要从`obj.temperature`的形式改成`obj.get_temperature()`.且所有的类似`obj.temperature=val`的语句都要重构成`obj.set_temperature(val)`的样式。如果量大的话，这对调用方来说可能是个灾难。
因此这里引出property的使用场景，就是像访问属性一样实现对方法的调用。还有一种常用场景是，对属性的保护。

### 使用场景

#### 1 装饰方法，使方法像属性一样访问

使用property装饰器修改背景中的代码，如下：
```python
class Celsius:
    def __init__(self, temperature = 0):
        self._temperature = temperature

    def to_fahrenheit(self):
        return (self._temperature * 1.8) + 32

    @property
    def temperature(self):
        print("Getting value")
        return self._temperature

    @temperature.setter
    def temperature(self, value):
        if value < -273:
            raise ValueError("Temperature below -273 is not possible")
        print("Setting value")
        self._temperature = value
```
```bash
>>> c.temperature
Getting value
0

>>> c.temperature = 37
Setting value

>>> c.to_fahrenheit()
Getting value
98.60000000000001
```

#### 2 对属性做限定

使用property可以对属性做只读限制，防止被修改。

```python
class Student(object):

    @property # 把一个getter方法变成属性，只需要加上@property即可
    def birth(self):
        return self._birth

    @birth.setter #  @birth.setter负责把一个setter方法编程一个属性赋值
    def birth(self, value):
        self._birth = value

    @property
    def age(self):
        return 2015 - self._birth
```
只定义getter方法，不定义setter方法就是一个只读属性，如上面的`age`。因为`age`可以根据`birth`和当前的时间计算出来。

### Reference

1 https://www.programiz.com/python-programming/property

2 https://www.liaoxuefeng.com/wiki/1016959663602400/1017502538658208




