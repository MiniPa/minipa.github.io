---
layout: post
title: Redis 06 Redis 本身事务 与 分布式锁 
categories: [Redis]
description: Redis 本身事务 与 分布式锁
keywords: Redis, transaction 
topmost: false
---



#### 1.Redis  multi exec  自身事务

1.MULTI 批量操作进入队列缓存  
2.EXEC 依次执行，但任意命令失败，**其他命令仍然执行** **不保证原子性**  
3.事务执行过程中，其他客户端命令不会执行 **不会插队**

Redis命令是**原子**的  
Redis事务**没有**任何原子性（批量执行脚本），且**不回滚**

```
开始事务 MULTI
命令入队
执行事务 EXEC
```

[Blog1](http://www.runoob.com/redis/redis-transactions.html)

[Blog2](http://www.redis.cn/topics/transactions.html)

- 1.Redis事务 一次性、顺序性、排他性的执行一个队列中的一系列命令
- 2.Redis事务没有隔离级别的概念，不存在事务内的查询要看到事务里的更新，事务外查询不能看到

##### 1.相关命令

```
watch key1 key2 ... : 监视一或多个key,如果在事务执行之前，被监视的key被其他命令改动，则事务被打断 （ 类似乐观锁 ）
multi : 标记一个事务块的开始（ queued ）
exec : 执行所有事务块的命令 （ 一旦执行exec后，之前加的监控锁都会被取消掉 ）
discard : 取消事务，放弃事务块中的所有命令
unwatch : 取消watch对所有key的监控
```

##### 2.AOF

AOF持久化时候，会使用单个 write(2) 依次将事务写入磁盘  
某些原因，事务只写了一半，重启时加载AOF会退出，并报错误  
==> redis-check-aof 修复下  

##### 3.Redis不支持回滚

Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现）  
或是命令用在了错误类型的键上面，失败的命令是由编程错误造成的，应该在开发中清除，而非生产中无回滚，Redis内部简单快速  
通常情况下回滚不能解决编程带来的错误

##### 4.案例 [Blog](https://www.cnblogs.com/DeepInThought/p/10720132.html)

![multi1](/images/posts/2016-07-19-redis/multi1.png) 

![multi2](/images/posts/2016-07-19-redis/multi2.png) 

- **命令性**错误 所有命令不执行

![multi3](/images/posts/2016-07-19-redis/multi3.png) 

- **语法性**错误 其它命令执行，错误不执行抛出异常

![multi4](/images/posts/2016-07-19-redis/multi4.png) 

##### 5.watch: 使得 exec 执行有了条件

exec 开启事务执行，无论是否成功，watch监控都将取消  
事务执行失败后，需重新执行 WATCH 命令对变量进行监控，并开启新的事务进行操作 

===============================

1.实现类似**乐观锁**：通过 **check-and-set** 实现乐观锁，事务提交时，watch 监控的 **key**值变更，则事务队列不执行，返回Nullmulti-bulk

```shell
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
// exec前，其它客户端修改了 mykey，当前事务失败

// 乐观锁 程序不断重试，直到没有碰撞
```

================================

2.创建Redis 非内置**原子操作** 实现 **ZPOP**，原子地弹出 zset 中 score 最小 element

```shell
WATCH zset
element = ZRANGE zset 0 0
MULTI
ZREM zset element
EXEC
// 程序执行这段代码，只得到exec返回值不是 **nil-replay** 即可
```

![multi5](/images/posts/2016-07-19-redis/multi5.png) 



2.Redis 分布式锁 

分布式锁在很多场景中是非常有用的**原语**  

不同的进程必须以独占资源的方式实现资源共享

Redlock算法 http://www.redis.cn/topics/distlock.html

- [ ] TODO













## 参考：

