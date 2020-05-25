---
layout: post
title: Data Structure之Bit Array
date: 2020-04-11
author: tux
tags: Data Structure, Algorithms
---

### Bit Array位数组

Bit Array (aka bit map, bit set, bit string or bit vector)是一种用于存储比特位的数组。位数组中元素的类型是bit(概念上)，元素的值只可能是0或1.计算机处理信息是的最小单位其实就是一个二进制单位，即**比特(Bit)**。而每8个二进制比特位可组成另一个单位，即**字节(Byte)**,字节是内存空间中的一个基本单位。

一般在高级语言中是无法直接声明一个以比特为单位的基本类型的，因此在高级语言中是无法直接声明一个以比特为基本单位的数组。那可以声明一个以int为单位的数组，这个数组的值规定只能为"0"或"1"，来表示比特位的"0"或"1"。这种方法有一个明显的缺点，就是消耗了过多的存储空间。无论是32位还是64位的机器上，int这种基本类型在Java中的大小都是占4个字节空间，即总共占4X8=32个比特位。理论上讲我们只是需要其中的一个比特位来记录状态，所以在这里这个数组浪费了21/32=96.875%的内存空间。

### 解决办法

既然一个int类型是32个比特位，那我们就可以用数组中一个int类型的元素表达32个比特位元素。如果数组中有个两个int类型的元素，则总共可以表达64个状态为。声明拥有3个元素的int数组的内存模型如下图所示：

！[](http://www.mathcs.emory.edu/~cheung/Courses/255/Syllabus/1-C-intro/FIGS/bit-array1.gif)

这个数组总共可以表达3X32=96个状态。从上图可得知位数组在内存中的结构即这个数组索引的分布。
当操作位数组中索引为i的这个比特位时，该如何操作？使用下面的公式来定位：
```
所在数组中的元素为： i / data_size (整数除法，商数为数组元素的索引)
比特位在元素中的位置：i % data_size (整数除法，余数为比特位在元素中的索引)
```
以定位索引为68的比特位为例，使用上面的公式：
```
所在数组中的元素为： 68 / 32 = 2
比特位在元素中的位置： 68 % 32 = 4
```
所以这个比特位位于A[2]这个元素上索引为4的位置。下图的示意更清晰：
![](http://www.mathcs.emory.edu/~cheung/Courses/255/Syllabus/1-C-intro/FIGS/bit-array1a.gif)

### 位数组的实现

位数的实现重点实现3个核心操作。

1. GetBit

声明GetBit的方法签名为：
```
boolean GetBit(int[] array, long index);
```
这个方法用于获取array位数组中index位上的比特位是什么状态，如果为"1"，则返回true,如果为"0"则返回false.

2. SetBit

声明SetBit的方法签名为：
```
void SetBit(int[] array, long index);
```
该方法用于将array位数组中index位上比特位设置为1。

3. ClearBit

该方法的签名为：
```
void ClearBit(int[] array, long index);
```
该方法用于将array位数组中index位上比特位设置为0。

### 位数组的使用

位数组这种数据结构可极大地优化内存空间，当我们要表达的状态只有true和false时，便可以考虑使用这种数据结构。假设如下场景：
先考虑一个关于用户行为分析的问题，假设要统计《xxx》专栏每个月的用户活跃度。在每个月中，只要用户登录并学习了这个专栏，都会将用户的ID写入MySQL表中。如果想知道在11月和12月两个月内都在学习这个专栏的用户有多少该如何做？

SQL的直观做法：
```sql
SELECT COUNT(*) FROM nov_stats INNER JOIN dec_stats ON nov_stats.user_id=dec_stats 
WHERE nov_stats.logged_in_month = "2019-11" AND dec_stats.logged_in_month = "2019-12"
```
不过这种做法需要进入到数据库中读取数据并做内连接，效率并不是很高。

位数组解决方法：
由于原理都一样，这里直接使用redis中的bitmap来实现。
首先针对11月学习的用户和12月学习的用户，为它们创建独立的位数组。例如，logins:2019:11和longins:2019:12。在11月，每当有用户登录学习时，程序会自动调用"SETBIT logins:2019:11 user_id 1"，同理，对于12月登录学习的用户，也可调用"SETBIT logins:2019:12 user_id 1".SETBIT命令可以将user_id在数组中的相应的位设置为"1".

当要获得在这两个月内同时学习了这个专栏的用户数时，可以调用"BITOP AND logins:2019:11-12 logins:2019:11 logins:2019:12".将logins:2019:11和logins:2019:12这两个位数组做位运算中的与操作，将结果存放到"logins:2019:11-12"这个位数组中，最后调用"BITCOUNT logins:2019:11-12"来得到结果。

为了便于理解，可以将user_id想象为从0开始的自增整数，这样user_id的值就直接是位数组中比特位的索引。

如果一个对象有多个属性(数据)，每个属性有两种状态，则可以考虑使用bitarray进行相关从存储和操作。位操作是在内存中操作，所以极大的提高了效率，并节省大量存储空间。

### Reference

- http://www.mathcs.emory.edu/~cheung/Courses/255/Syllabus/1-C-intro/bit-array.html
- 数据结构精讲，蔡元楠