---
layout: post
title: Mysql 04 Lock 锁、RWLock 读写锁
categories: [Mysql]
description: Mysql Lock 锁、RWLock 读写锁
keywords: mysql, Lock
topmost: false
---



#### 1.Lock 锁  -- Isolation 

##### 实用：Mysql 死锁后如何解决

```
show status like 'table%';
show processlist;
show full processlist;
找到锁进程，kill id ;
```

##### 1.读写锁

- **LOCK TABLES** products **WRITE**；  写锁  
  [排他锁 exclusive lock]  
  锁定之后，只有当前线程可以进行读操作和写操作，其他线程读操作和写操作均被堵塞.....

-  **LOCK TABLES** products **READ**；读锁   
  [共享锁 shared lock]    
  锁定之后，无论是当前线程还是其他线程均只能读操作，写操作全部被堵塞....
- **UNLOCK** TABLES;  解锁：

![lock](/images/posts/2017-07-21-mysql-lock/lock.png)

##### 2.锁机制

- 引擎锁类型  
  MyISAM Memory -- table  
  BDB           -- table + page   
  Innodb        -- table + row

- 锁类型 比对 （开销、加锁、死锁、粒度、并发）

  - **table**: 开销小，加锁快；不会出现死锁；  
    锁定粒度大，发生锁冲突的概率最高，**并发度最低** 

  - **page**: 开销和加锁时间界于表锁和行锁之间；会出现死锁；  
    锁定粒度界于表锁和行锁之间，并发度一般 

  - **row**: 开销大，加锁慢；会出现死锁；  
    锁定粒度最小，发生锁冲突的概率最低，**并发度也最高**

  ```
  Lock tables orders read local, order_detail read local;
  
  Select sum(total) from orders;
  Select sum(subtotal) from order_detail;
  
  Unlock tables;
  ```

  [Blog](https://www.cnblogs.com/leedaily/p/8378779.html)



#### 3.MVCC 

MultiVersion Concurrent Control 多版本并发控制 ==> 可重复读，Mysql 依靠此解决了幻读问题

##### 1.Innodb - MVCC: 每行记录的后面保存两个隐藏的列实现

- 列1：行创建时间

- 列2：行过期时间

存储的并非时间值，而是系统版本号  
通过数据多版本来做到**读写分离**,从而实现不加锁读进而做到读写并行

![lock2](/images/posts/2017-07-21-mysql-lock/lock2.png)

##### 2.[快照] 一致性视图

可重复读：事务开始时生成一个全局事务快照  
读提交：  每次执行语句一个快照  
快照可以读到 当前 transaction 内提交的，或快照创建前提交的版本数据

##### 3.依赖 undo log 与 read view 实现

undolog:     记录某行数据的多个版本的数据
read view:   判断当前版本数据的可见性

![lock3](/images/posts/2017-07-21-mysql-lock/lock3.jpg)






## 参考：

[Blog](https://www.cnblogs.com/wanghuaijun/p/5949934.html)
