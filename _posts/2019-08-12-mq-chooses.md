---
layout: post
title: MQ 03 选型
categories: [MQ]
description: ActiveMQ、RabbitMQ、RocketMQ、Kafka 选型
keywords: MQ, MessageQueue
topmost: false
---



#### 1.MQ 选型考虑哪些因素

- 单机**吞吐量**         -- 万/十万

- **topic** 数量影响      -- 功能灵活性

- **时效**               -- 延迟 

- **可用**性             -- 主从/分布式

- 消息**可靠性**         -- 基本不丢/0丢失

- **功能**支持           -- 完善不? 并发好不?

- **社区**活跃度         -- 支持

2.MQ 4种类型选择

- **Kafka** 10万级 高吞吐、**毫秒**ms延迟、可用性非常高、分布式、可做到**0丢失**、

  功能较为**简单**

  在大数据实时计算，以及日志采集被大规模使用

  高社区活跃度

- **RoketMQ** 10万级别、同等机器可支持**更多topic**、ms延迟、分布式、

  参数优化有可达到**0丢失**、

  功能较为**完善**

  高社区活跃度

- RabbitMQ **万级** **微妙**级延迟，**主从**架构、基本不丢、Erlang并发极好，但不懂的话排查不易、

  中社区活跃度

- ActiveMQ 不考虑

![choose](/images/posts/mq/choose.jpg)




























## 参考：

