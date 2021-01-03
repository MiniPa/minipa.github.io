---
layout: post
title: Redis 04 Pub / Sub
categories: [Redis]
description: Redis 订阅发布机制
keywords: Redis, pub, sub 
topmost: false
---

#### Pub/Sub MQ

一种消息通信模式  可用来实现 MQ Message Queue

redis客户端可以订阅任意数量的频道，无法序列化

![ps1](/images/posts/2016-07-19-redis/ps1.png) 

![ps2](/images/posts/2016-07-19-redis/ps2.png) 



#### Redis 5.0 版本新增加的数据结构

主要用于消息队列（MQ，Message Queue）

提供消息持久化、主备复制功能，

可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失

```
XADD 向队列添加消息
XTRIM 对流进行修剪，限制长度
XDEL 删除消息

XLEN 获取流包含的元素数量，即消息长度
XRANGE 获取消息列表，会自动过滤已经删除的消息
XREVRANGE 获取消息列表，会自动过滤已经删除的消息

XREAD 以阻塞或非阻塞方式获取消息列表
XGROUP CREATE 创建消费者组
XREADGROUP GROUP 读取消费组中的消息
```

[Blog](https://www.runoob.com/redis/redis-stream.html)

![ps3](/images/posts/2016-07-19-redis/ps3.png)




## 参考：

