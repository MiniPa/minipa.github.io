---
layout: post
title: Java 线程 二、AVO 线程3大核心特征
comments: true,
categories: [Thread 线程--高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发, Atomicity, Visiable, Order
topmost: false
---

#### Atomicity 原子性

同 DataBase 数据库的原子性一样：所有操作同生共死
JMM(Java Memory Model)：保证了基本的原子性

- i++  三步操作，非原子，只是个语法糖 

```
获取 i 的值
自增。
再赋值给 i
```

- 基础类原子操作 atomic，如 AtomicInteger,本质是 CAS+CPU

```java
public final long incrementAndGet() {
    for (;;) {
        long current = get();
        long next = current + 1;
        if (compareAndSet(current, next)) {
            return next;
    	}
	}
}
public final boolean compareAndSet(long expect, long update) {
    return unsafe.compareAndSwapLong(this, valueOffset, expect, update);
}
```



#### Visiable 可见性

##### 1.cpu **高速缓存**

CPU 直接从主内存中读取数据的效率不高，所以都会对应的 CPU 高速缓存。  
先将主内存中的数据读取到缓存中，线程修改数据之后首先更新到缓存，之后才会更新到主内存。   
如果此时还没有将数据更新到主内存其他的线程此时来读取就是修改之前的数据，Thread计算后的数据还在高速缓存，另外线程从主内存拿数据，拿到的就是修改前的数据。

==> 我还没从高速缓存出来，你别拿内存数据啊

##### 2.volatile

保证内存(高速内存缓存)可见性(所有缓存可见) ，刷高速缓存到主存，清空线程的缓存。  
线程A更新了 volatile 修饰的变量时，它会立即刷新到主线程。  
并且将其余B、C、D缓存中该变量值清空，其余线程只能去主内存读取最新值。  
清空前，缓存中该值已经被用了，被另一个cpu计算了，如何应对？

**synchronized** 和 **加锁**也能能保证可见性  
实现原理就是在释放锁之前其余线程是访问不到这个共享变量的，但是和 volatile 相比开销较大

![1607908217712](/images/posts/2016-07-01-thread-avo/1607908217712.png)



#### Order 顺序性

```java
int a = 100 ; //1
int b = 200 ; //2
int c = a + b ; //3// 正常执行顺序  1>>2>>3
// JVM提高效率会重排, 可能时 2>>1>>3
// JVM保证最终结果和顺序执行结果一致情况下才进行重排
```

- **volatile**：保证顺序性  
  针对于 volatile 关键字的写操作肯定是在读操作之前，也就是说读取的值肯定是最新的

- **synchronized** 和 **lock** 也可以来保证有序性，道理和 Atomicity 一样

- JVM 还通过 **happen**-**before** 原则来隐式的保证顺序性














## 参考：

