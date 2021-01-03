---
layout: post
title: 02 Redis 多级缓存方案
categories: [Cache]
description: Redis 多级缓存方案 
keywords: Redis, Cache
topmost: false
---

三级缓存 和 分布式数据一致性解决方案 Redis

#### 1.解决  热点数据   redis 与 mysql **数据双写不一致**的问题

- 客户端**读取**数据，先从redis拿，reids没有从mysql拿，拿完缓存到redis

- 客户端**写**数据，先从redis清除，再写入mysql

  

##### 问题：**并发**量非常高的时候 

- 1.update 执行到删除缓存后，还没更新DB的时候， query 发现现缓存没有，就去DB查询，然后写入了缓存然后 update 把DB中这条数据删除了 ==> DB数据删了，Cache数据还在

- 2.query 查到旧数据准备写入 Cache, update 删除了缓存中的数据 (其实是空数据)，然后 query 把**旧数据**写入缓存了，上述两例，让Cache 和 DB出现了，数据不一致问题。

##### 方案 **堆队列**的方案

堆内存中开若干**有界阻塞队列**，使数据按照某 id 路由均分到所有队列中，实现读写互斥的先后顺序执行

- [ ] 不明白？

注意：

```
1.堆内存大小由业务量决定
2.思考内存泄漏问题
3.服务分布式部署时商品ID路由问题，需考虑 nginx 路由服务问题
```

[Blog](https://blog.csdn.net/weixin_40682142/article/details/104878162)



#### 2.解决 时效性低 的数据 **redis** 的异步解决方案

- 1）当更新了商品时效性低的信息时，如促销信息、商品介绍等

当商品服务更新商品信息后，发送信息到消息队列，如kafka或MQ，让队列通知缓存服务，缓存服务会查询DB，并更新本地缓存 和 reids 中。

- 2）当nginx缓存失效后，客户端服务请求服务查询商品信息，缓存服务发现redis和本地缓存中都没有该信息，回去查询DB，并跟新到本地缓存和Redis中。

##### 1.方案  三级缓存架构：

- nginx(10分钟) + redis cluster + JVM堆内存(数量限制)  
  **JVM堆内存** 可作为 mysql最后一道防线  
  采用 ehcasche 搭建一个缓存服务把商品信息放在堆内存中,作为redis集群和mysql 的中间层

```
redis cluster 的LRU算法
maxmemory 0 #对于64 bit的机器，如果maxmemory设置为0就默认不限制内存的使用，直到耗尽机器中所有的内存;对于32bit的机器，有一个隐式的闲置就是3GB
maxmemory-policy allkeys-lru
noeviction: 如果内存使用达到了maxmemory，client还要继续写入数据，那么就直接报错给客户端

allkeys-lru: 就是我们常说的LRU算法，移除掉最近最少使用的那些keys对应的数据
volatile-lru: 也是采取LRU算法，但是仅仅针对那些设置了指定存活时间（TTL）的key才会清理掉

allkeys-random: 随机选择一些key来删除掉
volatile-random: 随机选择一些设置了TTL的key来删除掉

volatile-ttl: 移除掉部分keys，选择那些TTL时间比较短的keys
```

- nginx缓存架构搭建： OpenResty （nginx + lua）

```
问题：当同一商品多次请求没有落在一台机器上时还是需要去 redis 集群里面查询高并发时会给 redis cluster 带来压力，且可能多台 nginx 上持有同一个商品的缓存数据造成资源浪费。
```

- 解决方案

```
分发层 + 应用层 双层 nginx
分发层 nginx，负责流量分发的逻辑和策略，根据商品Id去进行hash
然后对后端的nginx数量取模将某一商品的访问的请求固定路由到一个 nginx 后端服务器（应用层）上
保证只会从redis中获取一次缓存数据，后面全都是走 nginx 本地缓存
```

##### 2.三级缓存（nginx + redis cluster + JVM 堆内存）架构下，分布式缓存**重建并发冲突问题**

```
场景一：nginx 应用层没有根据 id 进行hash定向路由到分布式缓存服务，
	导致同一nginx 应用层前后两次请求打到不同服务的并发冲突

场景二：nginx 应用层服务和消息队列被动重建引起的并发冲突。
```

解决方案：基于zookeeper 的分布式锁解决方案：

- 1）执行**重建操作**前，获取对应商品id的分布式锁，创建一个临时节点 
- 2）拿到分布式锁之后，需要根据时间版本去比较一下，如果自己的版本新于redis中的版本，那么就更新，否则就不更新
- 3）如果拿不到分布式锁，那么就等待，不断轮询等待，直到自己获取到分布式的锁












## 参考：

