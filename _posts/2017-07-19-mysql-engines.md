---
layout: post
title: Mysql 02 数据库引擎
categories: [Mysql]
description: Mysql 数据库引擎分析
keywords: mysql, 数据库引擎
topmost: false
---



#### Mysql Engines 引擎

```
# 查询引擎
$show engines
# 修改引擎
$alter table mytest ENGINE = MyIsam
```

#### 1.MyISAM： 默认存储引擎

```
CREATE TABLE t (i INT) ENGINE = MYISAM;
```

```
适合：对事务完整没有要求 或以 select insert为主的应用，如log
不支持事务、也不支持外键
访问速度快
```

#####  3 种存储格式：静态表、动态表、压缩表

- **静态表**：字段都是**非变长字段**，这样每个记录都是固定长度的   
  优点：非常迅速，容易缓存，出现故障容易恢复 -- **定长速度快**  
  缺点：占用的空间通常比动态表多 -- 宽度补足原因

- **动态表**：记录非定长  
  优点：空间占用较少  
  缺点：频繁更新、删除数据容易产生碎片

需定期期执行**OPTIMIZE TABLE**或者**myisamchk-r**命令来改善性能

- **压缩表**：每个记录是被单独压缩的，所以只有非常小的访问开支



#### 2.InnoDB

```
比MyISAM，写效率差些，需更多空间存储索引
适合：大量 select、update 场景
```

提供提交、回滚、崩溃恢复能力的**事务****支持**（ACID兼容）  
支持**外键**约束  
支持**自动增长序列**



#### 3.memory

使用在内存中的内容来创建表

```
适合：内容变化不频繁的代码表，或者作为统计操作的中间结果表，
便于高效地对中间结果进行分析并得到最终的统计结
```

==> 好像用redis更多了，目前看来这个比较鸡肋

每个memory表，实际对应一个磁盘文件 格式 **.frm**

因为在内存中：访问非常快，默认用Hash索引，一旦服务关闭，表中数据会丢失 ---- 谨慎
可选 BTREE 或 Hash 索引

```
[**Hash**]检索效率 =  O(1) 效率高 
不支持like等范围查找
[**B-Tree**] 检索效率 = O(logn)
```



#### 4.merge

一组MyISAM表组合，merge本身没有数据，对merge类型表，可以进行查询、更新、删除操作

实际是对内部 MyISAM 的操作



5.其他：

- **BDB** (BerkeleyDB)      存储引擎
- EXAMPLE               存储引擎
- FEDERATED             存储引擎
- **ARCHIVE**               存储引擎
- **CSV**                   存储引擎
- **BLACKHOLE**             存储引擎














## 参考：
