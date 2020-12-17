---
layout: post
title: Java 线程 五、悲观锁、乐观锁
categories: [Thread 线程--高并发]
description: Java Thread 并发处理
keywords: 线程, Thread, 高并发
topmost: false
---

#### OL Optimistic Lock 乐观锁

- 读多写少，避免幻读、RT过长

- CAS: atomic 使用CAS实现乐观锁， version: 数据添加version表示被修改次数，  
  更新时校验被更新的version与自己之前获取的version是否相同，相同更新、不同重新来过。



#### PCC Pesimistic Concurrency Control 悲观锁

- 写多读少，先上锁再写

  1.关系型数据库行锁，表锁，读锁，写锁等  
  2.synchronized 悲观锁(1.6进行了优化 增加了锁升级过程)

- **S锁** Shared locks:        共享锁、读锁，多事务只读不改

- **X锁** Exclusive locks:    排他锁、写锁



#### 案例

##### 1.Mysql Innodb 悲观锁

mysql 行锁基于索引，用不到索引会锁表

![1.MysqlInnodb悲观锁](/images/posts/2016-07-02-thread-pcc-ol/1.MysqlInnodb悲观锁.png)

##### 2.CAS + version 面对高并发，大量失败问题处理

==> 想办法减少乐观锁的粒度

![CASversion](/images/posts/2016-07-02-thread-pcc-ol/CASversion.png)

如上，用户下单数为1，则通过quantity-1 > 0 的方式进行乐观锁控制

- [ ] 不明白--why?

-- 高并发下，锁粒度把控非常重要，保证数据安全情况下，提升吞吐率



#### PCC OL 选择

- 需响应效率要高： OL，注意锁粒度不好，容易大量失败 

- 2.业务冲突频率高： PCC，冲突多 OL 重试太多，消耗反而大 

- 3.重试代价大：PCC 

- 4.OL有人在你前更新了，你的更新应当是被拒绝的，可让用户重新操作，PCC会等待前一个更新完成








## 参考：

