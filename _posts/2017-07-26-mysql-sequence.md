---
layout: post
title: Mysql 08 Mysql 自增序列探讨
categories: [Mysql]
description: Mysql 自增序列如何实现
keywords: mysql, sequence
topmost: false
---

#### Mysql 中实现自增序列

案例：

接口中要求发送一个int类型的流水号，由于多线程模式，如果用时间戳，可能会有重复的情况(当然概率很小)  
所以想到了利用一个独立的自增的sequence来解决该问题。

- ==> Mybatis-Plus 中有事先分布式系统全局自增sequence，可以满足要求

- ==> java 纯代码自增序列 支持集群 [Blog](https://blog.csdn.net/qq1170993239/article/details/105764744)

- ==> [Blog](https://www.cnblogs.com/dirgo/p/9566442.html)

#### Mysql Java 字段映射表

![mysqljvaa](/images/posts/2017-07-26-mysql-sequence/mysqljvaa.png)




## 参考：
