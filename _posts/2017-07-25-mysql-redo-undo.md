---
layout: post
title: Mysql 07 redo undo log
categories: [Mysql]
description: Mysql 事务和可靠性实现方式
keywords: mysql, redo, undo
topmost: false
---



#### 1.redolog、binlog 持久化日志

##### 1.redolog **重做日志**：

- 1.Innodb 独有

- 2.作为内存和磁盘的缓冲，**批量写入**磁盘

- 3.物理日志，记录"在某数据页上做了什么修改"

- 4.循环写，空间固定会用完 

记录已成功提交的transaction的，用来**恢复数据**，保障事务的 **Durability**

##### 2.binlog 归档日志：

- 1.server层所有

- 2.逻辑日志：记录语句的原始逻辑，如"给ID=2行C字段值添加1"

- 3.追加写，满了开启下一个

##### 3.方案 [二阶段提交] 看下方流程图

- 先记录redo log，且状态为prepare

- 然后记录binlog

- 最后再把redo log状态改为 commit

- 其实这个就是两阶段提交，保证两份日志逻辑的一致性

#### 2.一条 UPDATE SQL 执行过程

```
update t_students set fname = 'tom' where fid = 1001;
```

![sql1](/images/posts/2017-07-25-mysql-redo-undo/sql1.png)

![sql2](/images/posts/2017-07-25-mysql-redo-undo/sql2.png)

#### 3.undo log **回滚日志** -- **Atomic** 原子性日志

undolog: 记录数据被修改前的信息，记录数据的逻辑变化，rollback用

redolog: --------------后------








## 参考：

[Blog1](https://blog.csdn.net/qinshi965273101/article/details/103543760)
[Blog2](https://www.cnblogs.com/wswgot/archive/2004/01/13/13373188.html)
[Blog3](https://www.cnblogs.com/f-ck-need-u/archive/2018/05/08/9010872.html)
