# Eventlet详解

## Eventlet简介

Eventlet是一个基于greenthread（协程）实现的高性能并发框架，可用于高并发网络编程。

通过开启多个greenthread并对其进行管理，实现非阻塞式I/O操作。

Eventlet主要依赖两个库实现：

1. greenlet：Eventlet封装了greenlet，作为并发基础
2. python-epoll：Eventlet封装了epoll，作为网络通信模型



## 线程与协程

**线程**是程序执行中一个单一的顺序控制流程，是程序执行流的最小单元，是处理器调度和分派的基本单位。 一个进程可以有一个或多个线程，各个线程之间共享程序的内存空间。



### 并发与并行



## 基本使用

### 生成绿色线程

- eventlet.**spawn**(*func*, **args*, ***kw*)

  启动一个绿色线程来执行函数`func`，`spawn()`方法的返回值是一个`greenthread.GreenThread`对象，用于获取函数`func`的返回值。

- eventlet.**spawn_n**(*func*, **args*, ***kw*)

  与`eventlet.spawn`类似，但不返回结果或异常，因此执行速度更快。

- eventlet.**spawn_after**(*seconds*, *func*, **args*, ***kw*)

  与`eventlet.spawn`类似，在给定时间间隔后开始运行。



### 控制绿色线程

- eventlet.**sleep**(*seconds=0*)

- *class* eventlet.**GreenPool**

  绿色线程池。

- *class* eventlet.**GreenPile**

  表示一系列的任务，本质上GreenPile是一个返回任务结果的迭代器。

- *class* eventlet.**Queue**

  用于绿色线程间的通信。

- *class* eventlet.**Timeout**



### Patch函数

- eventlet.**import_patched**(*modulename*, **additional_modules*, ***kw_additional_modules*)
- eventlet.**monkey_patch**(*all=True*, *os=False*, *select=False*, *socket=False*, *thread=False*, *time=False*)



### 网络函数

- eventlet.**connect**(*addr*, *family=AddressFamily.AF_INET*, *bind=None*)
- eventlet.**listen**(*addr*, *family=AddressFamily.AF_INET*, *backlog=50*, *reuse_addr=True*, *reuse_port=None*)
- eventlet.**wrap_ssl**(*sock*, **a*, ***kw*)
- eventlet.**serve**(*sock*, *handle*, *concurrency=1000*)
- *class* eventlet.**StopServe**



## 设计模式

### 客户端模式

### 服务器模式

### 分发模式
