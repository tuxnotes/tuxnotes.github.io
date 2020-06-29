---
layout: post
title: [译]Python Asyncio 教程
author: Fahmida Yesmin
date: 2020-06-29
tags: Python Asyncio
---

**原文链接：https://linuxhint.com/python_asyncio_tutorial/ , by Fahmida Yesmin**

python 3.4引入了Asyncio库，用来执行单线程的并发程序。由于其速度和使用多样性的优势，比其他的库和框架更加流行。Python使用Asyncio这个库用来创建，执行和组织协程，在不使用并行处理多任务的情况下，来并发处理多个任务。Asyncio库主要由以下几个部分组成：

**Corotine**: 多线程的脚本能能暂停和恢复的代码称为协程。多线程中协程以合作的方式进行工作，当一个协程暂停时，其他的协程就可以执行。

**Event loop**: 事件循环用于启动协程的执行，并处理I/O操作。使用多任务完成。

**Task**: task定义了协程的执行和结果。你可以使用Asyncio库指定多个任务进行异步运行。

**Future**: 一个未来结果的存储，协程最终执行完成的结果将存储在future中。当任何一些协程需要等待其他协程的结果时，这变得非常有用。

### 示例一：Create Single coroutine with a single task

创建一个名为`async1.py`的文件并添加下面的代码。`add`函数用于计算执行范围内数字的和。

```python
import asyncio

async def add(start,end,wait):

    #Initialize sum variable
    sum = 0

    #Calculate the sum of all numbers
    for n in range(start,end):
        sum += n

    #Wait for assigned seconds
    await asyncio.sleep(wait) # After calculating the value, the function will wait for one second and print the result
    #Print the result
    print(f'Sum from {start} to {end} is {sum}')


async def main():
    #Assign a single task
    task=loop.create_task(add(1,101,1)) # The number range from 1 to 101 is assigned by the task with one second delay
    #Run the task asynchronously
    await asyncio.wait([task])

if __name__ == '__main__':
    #Declare event loop
    loop = asyncio.get_event_loop()
    #Run the code until completing all task
    loop.run_until_complete(main()) # The event loop is declared that it will run until all the tasks of main method complete
    #Close the loop
    loop.close()
```

### 示例二： Create Multiple coroutines

创建文件`async2.py`文件，并添加如下代码。在main()函数中生成三个不同数字范围和等待时间的任务。等待时间最短的任务最先完成，等待时间最长的任务最后完成。

```python
import asyncio

async def add(start,end,wait):
    #Initialize sum variable
    sum = 0

    #Calculate the sum of all numbers
    for n in range(start,end):
        sum += n
    #Wait for assigned seconds
    await asyncio.sleep(wait)
    #Print the result
    print(f'Sum from {start} to {end} is {sum}')

async def main():
    #Assign first task
    task1=loop.create_task(add(5,500000,3))
    #Assign second task
    task2=loop.create_task(add(2,300000,2))
        #Assign third task
    task3=loop.create_task(add(10,1000,1))
    #Run the tasks asynchronously
    await asyncio.wait([task1,task2,task3])

if __name__ == '__main__':
    #Declare event loop
    loop = asyncio.get_event_loop()
    #Run the code until completing all task
    loop.run_until_complete(main())
    #Close the loop
    loop.close()
```

### 示例三：coroutines with future

创建文件`async3.py` 并添加如下代码。本例中future指定了两个任务。`show_mesage`函数用于打印在协程执行前后打印消息。第一个任务会等待2秒最后完成，第二个任务等待1秒最先完成。

```python
import asyncio

async def show_message(number,wait):
    #Print the message
    print(f'Task {number} is running')
    #Wait for assigned seconds
    await asyncio.sleep(wait)
    print(f'Task {number} is completed')

async def stop_after(when):
    await asyncio.sleep(when)
    loop.stop()

async def main():
    #Assign first task
    task1=asyncio.ensure_future(show_message(1,2))
    print('Schedule 1')
    #Assign second task
    task2=asyncio.ensure_future(show_message(2,1))
    print('Schedule 2')

        #Run the tasks asynchronously
    await asyncio.wait([task1,task2])



if __name__ == '__main__':
    #Declare event loop
    loop = asyncio.get_event_loop()
    #Run the code of main method until completing all task
    loop.run_until_complete(main())
```
