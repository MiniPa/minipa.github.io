---
layout: post
title: Redis 07 缓存雪崩、缓存穿透、memory 优化-4 释放机制
comments: true,
categories: [Redis]
description: Redis 缓存雪崩、缓存穿透、memory 优化-4 释放机制
keywords: Redis, 
topmost: false
---

#### 1.内存优化

##### 1.**小的聚合类型数据**的特殊编码处理

CPU 时间换空间   
Hashes, Lists, Sets 和 Sorted Sets，当这些集合中的所有数都小于一个给定的元素，并且集合中元素数量小于某个值时，存储的数据会被以一种非常节省内存的方式进行编码，使用这种编码理论上至少会节省10倍以上内存（**平均节省5倍**以上内存）

- redis.conf可配

```
hash-max-zipmap-entries 64 (2.6以上使用hash-max-ziplist-entries)
hash-max-zipmap-value 512  (2.6以上使用hash-max-ziplist-value)
list-max-ziplist-entries 512
list-max-ziplist-value 64
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
set-max-intset-entries 512
```

（集合中）如果某个值超过了配置文件中设置的最大值，redis将自动把把它（集合）转换为正常的散列表。这种操作对于比较小的数值是非常快的，但是，如果你为了使用这种编码技术而把配置进行了更改，你最好做一下基准测试（和正常的不采用编码做一下对比）



##### 2.**位级别操作**

Redis 2.2引入了**位级别和字级**别的操作:  GETRANGE, SETRANGE, GETBIT 和 SETBIT.使用这些命令，你可以把redis的字符串当做一个随机读取的（字节）数组。

例如你有一个应用，用来标志用户的ID是连续的整数，你可以使用一个位图标记用户的性别，使用1表示男性，0表示女性，或者其他的方式。  
这样的话，1亿个用户将仅使用12 M的内存。

你可以使用同样的方法，使用 GETRANGE 和 SETRANGE 命令为每个用户存储一个字节的信息。  
这仅是一个例子，实际上你可以使用这些原始数据类型解决更多问题。

##### 3.尽可能使用散列表（**hashes**）

小散列表（是说散列表里面存储的数少）使用的内存非常小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。比如你的web系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密码设置单独的key,而是应该把这个用户的所有信息存储到一张散列表里面.

[Blog](http://www.redis.cn/topics/memory-optimization.html)

##### 4.**内存分配**

为了存储用户数据,当设置了maxmemory后Redis会分配几乎和maxmemory一样大的内存（然而也有可能还会有其他方面的一些内存分配）.  

精确的值可以在配置文件中设置，或者在启动后通过 CONFIG SET 命令设置(see Using memory as an LRU cache for more info). Redis内存管理方面，你需要注意以下几点:

![memory1](/images/posts/2016-07-19-redis/memory1.png)

![memory2](/images/posts/2016-07-19-redis/memory2.png) 

##### 5.当key被删除时

Redis并不总是释放(返回)内存到操作系统  
这并不是Redis的特别之处，但大多数 malloc() 实现都是这样工作的 

- ==> MEMORY PURGE命令进行内存整理

- ==> 开启activedefrag，热碎片整理（会占用CPU，在主线程执行，可以设置CPU占用率）

- ==> 重启  








## 参考：

