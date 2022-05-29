---
layout: post
title: Java 线程 十一、ThreadLocal
comments: true,
categories: [Thread 线程, 高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发, ThreadLocal
topmost: false
---



#### ThreadLocal 本地线程变量

![threadlocal](/images/posts/2016-07-08-thread-context/threadlocal.png)  

- 和本地 Thread 无关，也不是特殊的 Thread

- 线程的局部变量

[数据结构]：Map<key,value>

```
key: Thread对象
value: 线程变量副本,为每个Thread都提供变量副本
```

 [多线程资源共享]

```
synchronized: 时间换空间, Threads共享一份变量
ThreadLocal:  空间换时间, Threads每人一份变量
```

 ThreadLocal 实例通常是类中的 private static 字段，希望与某一个Thread关联

```
1、每个线程都有自己的局部变量
2、独立于变量的初始化副本
3、状态与某一个线程相关联
```

==> 方便Thread处理自己的状态





## 参考：

[Blog1](https://www.cnblogs.com/moonandstar08/p/4912673.html)

[Blog2](https://www.cnblogs.com/fsmly/p/11020641.html)
