---
layout: post
title: Redis 05 持久化 RDB / AOF
comments: true,
categories: [Redis]
description: Redis 
keywords: Redis, 
topmost: false
---

#### 1.RDB Redis DataBase 冷备

- 1.**快照**式的持久化方法，将**某一时刻**的数据持久化到磁盘中
- 2.有**独立线程**进行持久化，不影响主线程

##### 适用

1）进行**大规模数据**的恢复，且对于数据恢复的完整性不是非常敏感  
2）**紧凑**的单一文件,方便传送，适用于**灾难恢复**  
3）单独子进程，最大化redis的**性能**  
4）恢复大数据时**更快**

##### 劣势

1）宕机,可能会丢失**几分钟**的数据  
2）保存珍整个数据集，需要设置5分钟/更久做一次完整的保存 --**耗时** -- ms毫秒级内不能响应客户端请求   
3）经常 **fork** 子进程来保存数据集到硬盘上   

```
当数据集比较大的时候,fork的过程是非常耗时的,
可能会导致Redis在一些毫秒级内不能响应客户端的请求.
如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续更久
```

##### 配置

redis.conf默认配置

```==&gt; 用 save 10 1 ## 10s内超过1个key被修改，发起快照保存 进行测试
save 900 1   # 900秒内，如果超过1个key被修改，则发起快照保存
save 300 10  # 300秒内，如果超过10个key被修改，则发起快照保存
save 60 10000  # 60秒内，如果1万个key被修改，则发起快照保存
```

用 save 10 1 ==> 10s内超过1个key被修改，发起快照保存 进行测试



#### 2.AOF Append Of File 热备

- 1.记录每次对 server 的 write 操作
- 2.恢复方式：重新执行一遍 AOF 中 write 操作
- 3.能对 AOF 文件进行后台**重写**,使得 AOF 文件的体积不至于过大

##### 策略

每秒钟 **fsync**一次（fsync：把缓存中的写指令记录到磁盘中）,保持很好的处理性能  
即使redis故障，也只会丢失**最近1秒钟**的数据 

磁盘空间满、inode满或断电等 ==> redis提供了redis-check-aof工具，可以用来进行日志修复

AOF重写：先写临时文件，全部完成后再替换的流程，断电、磁盘满等问题都不会影响AOF文件的可用性

##### 适用

- 1）数据更加**耐久**  
  fsync策略：无fsync,每秒fsync(default),每次写的时候fsync，最多丢失**1秒的数据**
- 2）日志**追加**文件，不需要写入**seek**，用redis-check-aof工具修复问题.
- 3）**体积**过大，可安全的**重写**，不会丢失  
  重写：100次incr,100条记录,重写为set xx, 1条记录 BGREWRITEAOF
- 4）**可读**：有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存，   
  因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析也很轻松

```
不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 
那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 
并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态
```

##### 配置

redis.conf默认配置：

```
appendonly no  // 修改为 yes
```

开启后，启动redis服务端，发现多了一个appendonly.aof文件

AOF持久化并不会立即将命令写入到硬盘文件中，而是写入到**硬盘缓存**，  
在接下来的策略中，配置多久来从硬盘缓存写入到硬盘文件  
所以在一定程度一定条件下，还是会有**数据丢失**，不过你可以大大减少数据损失

```
# appendfsync always
appendfsync everysec
# appendfsync no

# always: 每次操作都会立即写入aof文件中
# everysec: 每秒持久化一次(默认配置)
# no: 不主动进行同步操作，默认30s一次
```

可用 flushall 再修改aof来恢复，进行测试



#### 数据安全

1.想达到媲美 PostgreSQL 的数据安全性，应两种同时使用，恢复时AOF优先  
2.redis-check-aof 可用来修复 AOF 文件 $ redis-check-aof –fix  
3.备份redis,无论何时，复制RDB都是绝对安全的












## 参考：

[Blog](https://www.jb51.net/article/121848.htm)

