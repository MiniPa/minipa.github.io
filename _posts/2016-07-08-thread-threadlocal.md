---
layout: post
title: Java 线程 十、context 上下文切换
categories: [Thread 线程--高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发, context, 上下文切换
topmost: false
---

#### 1.多线程：

单个CPU，可通过给线程分配时间片，通常几十ms，达到多线程效果



#### 2.上下文切换

线程切换，需要保存本次执行信息，下次获取cpu再次执行会恢复执行信息  
一出一入，就是上线程上下文切换，非常消耗效率  
running -> waiting -> running



#### 3.解决方案

- 无锁编程，数据按照 Hash(id) 分段取模，每个线程处理各自数据，避免切换

- CAS(compare and swap) 如 Atomic 包采用的 CAS 

- 合理创建线程，避免大部分线程处于 waiting 状态



#### 4.线程切换类型

##### 自发性上下文切换

```java
Thread.sleep()
Object.wait()
Thread.yeild()
Thread.join()
LockSupport.park()
```

非自发性上下文切换

```java
切出线程的时间片用完
有一个比切出线程优先级更高的线程需要被运行
虚拟机的垃圾回收动作
```



#### 5.上下文开销

1.直接开销，操作系统**保存回复**上下文所需的开销，线程调度器**调度线程**的开销
2.间接开销， 处理器**高速缓存重新加载**的开销  
上下文切换可能导致整个一级**高速缓存中的内容被冲刷**，即被写入到下一级高速缓存或主存














## 参考：

