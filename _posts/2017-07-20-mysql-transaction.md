---
layout: post
title: Mysql 03 ACID 事务特性、SRCU 隔离级别
comments: true,
categories: [Mysql]
description: Mysql ACID 事务特性、SRCU 隔离级别
keywords: mysql, ACID, SRCU
topmost: false
---



#### 1.ACID 事务4个特征  AID ==> C

- 内部：**Atomicity** 原子性 - 不可分割性 一起死一起活

- 外部：**Consistency** 一致性 (事务处理后数据库数据一致)

- 事务之间：**Isolation**  分隔性 - **并发**操作的事务有自己的操作**空间**

- 状态：**Durability**  持久性 - 事务产生的新状态一定会被存储

![acid](/images/posts/2017-07-20-mysql-transaction/acid.png)

DB事务、Java事务、Jdbc事务、分布式事务 etc



#### 2.SCRU 事务隔离级别  Consistency

##### 1.隔离级别 (高 => 低)

- **serialiizable**  **串行**化顺序执行,不能并发,效率低下

- **repeatable** read **可重复读** ==> Mysql 默认事物级别  
  -- "可能还有幻读问题"

```
事务A中，1时刻读a=**10**; 之后2时刻，B事务提交a=**12**; A事务 3时刻读 **a=10**  
(一个事务内相同的查询返回的都是事务开始时的相同数据)  
[幻读 -- 因为期间 INSERT 提交了]  ==> Mysql 用MVCC 方式进行的控制
```

- **Read commited** 授权读、读提交 -- **不可重复读**  ==> Oracle 默认事务级别

```
 事务A中，1时刻读a=**10**; 之后2时刻，B事务提交a=**12**; A事务 3时刻读 a=**12**  
 (一个事务内相同的查询返回了不同数据)  
[不可重复读 -- 因为期间 DML提交了]
```

- **Read uncommited** 可**读未提交**  -- **脏读**

![srcu](/images/posts/2017-07-20-mysql-transaction/srcu.png)  

##### 2.事务隔离要达到的效果

- 1.**可靠性**：  
  当DML:update,insert,remove或DB crash了  
  保障数据Consistency一致，用的A、I、D来进行处理的

- 2.**并发**处理：  
  多个并发请求来，其中有请求修改了数据，为不读到不合适的数据  
  进行Isolation事务隔离，保证事务独立性

- 3.**数据完整，不丢失**：Durability 持久性，保证宕机等情况，数据正常



##### 3.脏独

一个事务读取到了另外一个事务没有提交的数据



##### 4.幻读

一个事务第二次读取到了，另一个事务在这期间内提交的数据

- 定义：事务A 按照一定条件进行数据读取， 期间事务B 插入了相同搜索条件的新数据，事务A再次按照原先条件进行读取时，发现了事务B 新插入的数据 称为幻读

- mysql 在 repeatable 隔离级别下解决了幻读问题   
  方式：Next-Key 行锁 + 间隙锁  解决并发和幻读问题



1.表中有两条记录，并且 age 字段已经添加了索引，两条记录 age 的值分别为 10 和 30

![huan1](/images/posts/2017-07-20-mysql-transaction/huan1.png)

2.数据库中会为索引维护一套B+树，用来快速定位行记录，B+索引树是有序的，所以会把这张表的索引分割成几个区间

![huan2](/images/posts/2017-07-20-mysql-transaction/huan2.jpg)

3.如图所示，分成了3 个区间，(负无穷,10]、(10,30]、(30,正无穷]，在这3个区间是可以加**间隙锁**的

![huan3](/images/posts/2017-07-20-mysql-transaction/huan3.jpg)

4.当事务A执行update user set name='风筝2号’ where age = 10; 的时候，  
由于条件 where age = 10 ，数据库不仅在 age =10 的行上添加了行锁，   
而且在这条记录的两边，也就是(负无穷,10]、(10,30]这两个区间加了间隙锁，从而导致事务B插入操作无法完成，只能等待事务A提交

不仅插入 age = 10 的记录需要等待事务A提交，age<10、10<age<30 的记录页无法完成，而大于等于30的记录则不受影响，这足以解决幻读问题了




























## 参考：
