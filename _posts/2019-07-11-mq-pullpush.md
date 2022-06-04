---
layout: post
title: MQ 03 Pull、Push 模式
comments: true,
categories: [MQ]
description: Pull 和 Push MQ 模式优劣分析
keywords: MQ, Pull、Push
topmost: false
---

#### Pull、Push

Pull、Push 是从 **broker** 视角来定义的  

- **Push** broker 将消息推送到消费者    
  **RoketMQ RabitMQ** 等大多数   没有消息**延迟与忙**，消费快的时候，不会等待太久

- **Pull** 消费者轮询从 broker 拉去消息，  
  **Kafka**、**MetaQ**   全局**有序**，没有**慢消费**问题，消费慢的时候不会被 push 打扰

![pushpull](/images/posts/mq/pushpull.png)

#### 1.慢消费 push 模式大弊病

##### 1.Push模式 

消费 Message 速率 <<< **小于很多** <<< Message 注册速率 ==> broker **消息堆积**

- 1.如 Message 无法拒绝，则全部 broker 保存

- 2.如 boker push Message 给消费者，大多情况下消费者处于"忙"，会 reject/error，broker 再发送消息，形成**踢皮球**。

##### 2.Pull 模式 ==> 适合慢消费场景

- 消费者按需拉取，避免处理不了的 Message 骚扰，没有踢皮球。
- broker 堆积，无序记录 Message 被 push 的状态，维护 MQ 队列和偏移即可



#### 2.全局顺序消息实现

##### 1.Push 模式

假设 PushMQ，支持分区，单分区一个消费者，当且仅当此消费者确认前 Message 消费了，才 push 下一个 Message。另发送者保证全局顺序唯一 ==> 实现顺序，成本太高。

##### 2.Pull 模式 ==> 比较适合顺序消息

- producer 对应 partition，并且单线程。

- consumer 对应 partition，消费确认（或批量确认），继续消费即可

##### 场景 Kafka

对于日志**push**送这种最好全局有序，但允许出现小误差的场景，pull模式非常合适，如 Kafka 是 Pull 模式的。



#### 3.Message 延迟与忙

##### 1.Push 模式











##### 2.Pull 模式 

何时拉取消息决定权再消费方，无法准确决定何时去取消息。

- 一次 pull 取到消息了还可以继续去 pull

- 没有 pull 取到则需要等待一段时间重新 pull

==> **等多久**，我还要再等多久。。。

消息来的分布律如果稳定，比较好应用，如果前一分钟 1000M，后半小时 0M，就不好办了，结合业务处理。

方案：从短时间开始（不会对broker有太大负担），然后**指数级增长等待** 。比如开始等5ms，然后10ms，然后20ms，然后40ms……直到有消息到来，然后再回到5ms。

==> **延迟问题**

**长轮询**：**RocketMq** **长轮询** ==> 平衡PP各自优缺点

```
基本思路是:
消费者如果尝试拉取失败，不是直接 return,而是把连接挂在那里 wait,服务端如果有新的消息到来，把连接notify起来，这也是不错的思路。
但海量的长连接 block 对系统的开销还是不容小觑的，还是要合理的评估时间间隔，给 wait 加一个时间上限比较好
```







## 参考：
