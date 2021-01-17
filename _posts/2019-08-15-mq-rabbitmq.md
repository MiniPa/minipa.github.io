---
layout: post
title: MQ 08 RabbitMQ
categories: [MQ]
description: 
keywords: MQ, MessageQueue
topmost: false
---

#### 

#### 1.RabbitMQ

- **RabbitMQ** 实现AMQP消息中间件的一种，服务器端用Erlang语言编写,支持多种客户端，易用性、扩展性、高可用性。 

- **AMQP**: Advanced Message Queue Protocol (高级消息队列协议)，是应用层协议的一个开放标准,为面向消息的中间件设计  
  特征: 面向消息、队列、路由（包括点对点和发布/订阅）可靠性、安全。

##### 1.可靠性(**Reliablity**)：  

使用了一些机制来保证可靠性，比如**持久化**、**传输确认**、**发布确认**。 

##### 2.灵活的路由(**Flexible Routing**)：   

在消息进入队列之前，通过 **Exchange** 来路由消息。  
对于典型的路由功能，Rabbit 已经提供了一些内置的Exchange来实现。  
针对更复杂的路由功能，可以将多个Exchange绑定在一起，也通过插件机制实现自己的Exchange。 

##### 3.消息集群(**Clustering**)：

多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker。   
高可用 (Highly **Avaliable** Queues)：队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。   
多种协议(**Multi**-**protocol**)：支持多种消息队列协议，如STOMP、MQTT等。 

##### 4.多种语言客户端(Many **Clients**)：    

几乎支持所有常用语言，比如 Java、.NET、Ruby 等。 

##### 5.管理界面(**Management UI**)：

提供了易用的用户界面，使得用户可以监控和管理消息Broker的许多方面 

##### 6.跟踪机制(**Tracing**)：

如果消息异常，RabbitMQ提供了消息的跟踪机制，使用者可以找出发生了什么 

##### 7.插件机制(**Plugin** System)：

提供了许多插件，来从多方面进行扩展，也可以编辑自己的插件。



#### 2.整体架构

![arch](/images/posts/mq/arch.png)

#### 3.消息流转

![arch2](/images/posts/mq/arch2.png)

#### 4.组件功能

![comp](/images/posts/mq/comp.png)

- **Broker**：标识消息队列**服务器实体** 

- **Virtual Host**：虚拟主机   
  标识一批EBQ 交换机、消息队列和相关对象  
  虚拟主机是共享相同的**身份认证**和**加密环境**的**独立服务器域**。  
  每个vhost本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制  
  vhost 是AMQP概念的基础，必须在链接时指定，RabbitMQ 默认的 vhost 是 /

- **Exchange**：交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列 

- **Queue**：消息队列，用来保存消息直到发送给消费者  
  它是消息的容器，也是消息的终点，一个消息可投入一个或多个队列   
  消息一直在队列里面，等待消费者连接到这个队列将其取走 

- **Banding**：绑定，用于消息队列和交换机之间的关联   
  一个绑定就是基于路由键将交换机和消息队列连接起来的路由规则   
  所以可以将交换器理解成一个由绑定构成的路由表

- **Connection**：网络连接，比如一个TCP连接

- **Channel**：信道，**多路复用**连接中的一条**独立的双向数据流通道**  
  信道是建立在真实的TCP连接内地**虚拟链接**，AMQP命令都是通过信道发出去的   
  不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成  
  因为对于操作系统来说，建立和销毁TCP都是非常昂贵的开销，  
  所以引入了信道的概念，以复用一条TCP连接

- **Publisher**：生产者

- **Consumer**：消费者

- **Message**：消息，消息是不具名的，它是由消息头和消息体组成  
  消息体是不透明的，消息头则是由一系列的可选属性组成，这些属性包括

  - **routing-key**(路由键)、
  - **priority**(优先级)、
  - **delivery-mode**(消息可能需要持久性存储[消息的路由模式])等。

  

#### 5.Exchange

##### 1.**Direct**：

- 路由键(**routing** key of message) 和 

- 绑定键(**binding** key of Binding) 一致，则消息发送给Binding对应Queue

![et1](/images/posts/mq/et1.png)

##### 2.**Fanout**：

分到 Exchange 所有绑定的队列上去，Exchange 子网广播  
不处理该路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上-- 转发消息最快

![et2](/images/posts/mq/et2.png)

##### 3.**Topic**：

通过模**式匹配**分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。  
路由键(**routing**-key)和绑定键(**bingding**-key)的字符串切分成单词，这些单词之间用点隔开。 

通配符：

- **#** 匹配 0个或多个单词

- *****  匹配 一个单词 



- 1.路由键必须是一串字符，用句号（.） 隔开，比如说 company.us,或者 company.cn.shanghai 等

- 2.路由模式必须包含一个 星号（*），用于匹配路由键指定位置的一个单词  
  e.g.**shanghai..b.\***: 第一单词shanghai,第四单词b,       **shanghai.a.#** 以shanghai.a.开头的都匹配

##### 4.发送代码

```java
rabbitTemplate.convertAndSend("testTopicExchange","key.a.b.xx", " message of RabbitMQ!");
// 参数：交换机, routing key, 消息
```

![et3](/images/posts/mq/et3.png)

##### 5.**headers** 

(headers匹配AMQP消息的**header**而不是路由键(Routing-key)   
headers交换器和direct交换器完全一致，但是性能差了很多，目前几乎用不到了 ==> 不做解释

根据 arguments 来 routing，arguments为一组key-value对，任意设置。  
“x-match”是一个特殊的key，值为“all”时必须匹配所有argument，值为“any”时只需匹配任意一个argument，不设置默认为“all”。

#### 6.TTL Time to Live 生存时间

RabbitMQ支持消息的过期时间

- 1.消息发送时指定，配置消息体的**properties**，指定当前消息的过期时间

- 2.创建Exchange时指定

从进入消息队列开始计算，超过了队列的超时时间配置，那么消息会自动清除



#### 7.Confirm Return

##### 1.生产者**Confirm**机制：

**消息的确认**：生产者投递消息后，Broker收到消息，会给生产者一个应答，生产者进行接受应答，用来确认这条消息是否正常的发送到了Broker，这种方式也是消息的**可靠性**投递的核心保障！

![confirm](/images/posts/mq/confirm.png)

- 1、在channel上开启确认模式：**channel.confirmSelect()**

- 2、在channel上开启监听：**addConfirmListener**，监听成功和失败的处理结果，根据具体的结果对消息进行重新发送或记录日志处理等后续操作。



##### 2.Return 消息机制

Return Listener用于处理一些**不可路由**的消息

- [正常]消息生产者，通过指定一个Exchange和Routing，把消息送达到一个队列中去，消费者监听队列进行消息的消费处理操作。

- [异常]发送消息的时候，当前的exchange不存在或者指定的路由key路由不到，这个时候我们需要监听这种**不可达消息**，就需要使用到Returrn Listener。

基础API 中有个关键的配置项 **Mandatory**：如果为true，监听器会收到路由不可达的消息，然后进行处理。如果为false，broker端会自动删除该消息。

通过chennel.**addReturnListener**(ReturnListener rl)传入，已经重写过handleReturn方法的ReturnListener。



##### 3.消费端自定义监听

推模式和拉模式pull/push

- 1.一般通过while循环进行**consumer**.**nextDelivery**()方法进行获取下一条消息进行那个消费，(通过while将拉模式模拟成推模式，但是死循环会耗费CPU资源)

- 2.通过自定义Consumer，实现更加方便、可读性更强、解耦性更强的方式，(现默认使用的模式，直接订阅到queue上，如果有数据，就等待mq推送过来)

**Basic**.**Consume**将信道(Channel)置为接收模式，直到取消队列的订阅为止，在接受模式期间，RabbitMQ会不断的推送消息给消费者。

当然推送消息的个数还是受Basic.**Qos**的限制，如果只想从队列获得单条消息而不是持续订阅，建议还是使用**Basic**.**Get**进行消费。

但是不能将 Basic.Get 放在一个循环里来代替 Basic.Consume，这样会严重影响 RabbitMQ 的性能。 

如果要实现高吞吐量，消费者理应使用 Basic.Consume 方法。



#### 8.DLX 死信队列 Dead-Letter-Exchange

- 1.当消息在一个队列中变成死信(dead message)之后，它能被重新publish到另一个Exchange，这个Exchange就是DLX。
- 2.DLX是一个正常的Exchange，能在任何的队列上被指定，实际上就是设置某个队列的属性 

- 3.当这个队列中有**死信**时，RabbitMQ就会自动的将这个消息重新发布到设置的Exchange上去，进而被路由到另一个队列。

- 4.可以监听这个队列中消息做相应的处理，这个特性可以弥补RabbitMQ3.0之前支持的**immediate**参数的功能。

##### 1.消息变成死信情况

- 消息被**拒绝**(basic.reject/basic.nack)并且requeue=false

- 消息**TTL**过期

- 队列达到最大**长度**

##### 2.DLX 设置

需要设置死信队列的**exchange**和**queue**，然后通过routing key进行绑定，我们需要在队列加上一个参数即可

```java
Map<String, Object> arguments = Maps.newHashMapWithExpectedSize(3);

arguments.put("x-message-ttl", dlx-ttl);
arguments.put("x-dead-letter-exchange","exchange-name");
arguments.put("x-dead-letter-routing-key", "routing-key");
Queue ret = QueueBuilder.durable(
"queue-name".withArguments(arguments).build();
```

只需要通过**监听**该死信队列即可处理死信消息，还可以通过死信队列完成**延时队列**



#### 9.clusters 集群

##### 1.内建集群：高可靠、线性拓展

#####  2.Rabbit始终记录以下四种内部元数据

- **Queue** **队列**元数据：包括队列名称和他们的属性，比如是否可持久化，是否可持久化，是否自动删除。

- **Exchange** **交换器**元数据：交换器名称、类型、属性。

- **Binding** **绑定**元数据：内部是一张表格，记录如何将消息路由到队列。

- **vhost** 元数据：为vhost内部的队列、交换器、绑定提供命名空间和安全属性

 **单节点**：息存储在内存中，标记为可持久化的队列、交换器、 绑定存储在硬盘上
 **集群**：存到硬盘上(独立节点的默认配置) / 存在内存中

如果在集群中创建队列

集群只会在**单个节点**而不是所有节点上创建**完整的队列信息**，（元数据、状态、内容）。   
结果是只有队列的所有者节点知道有关队列的所有信息，当集群节点崩溃时，该节点的队列和绑定就消失了   。
并且任何匹配该队列的绑定的新消息也丢失了。   
还好RabbitMQ 2.6.0之后提供了**镜像队列**以避免集群节点故障导致的队列内容不可用。

##### 3.RabbitMQ 集群

1. 可以共享 **user**、**vhost**、**exchange** 等
2. 所有的数据和状态都是必须在所有节点上**复制的**，例外就是上面所说的消息队列
3. RabbitMQ 节点可以**动态的加入**到集群中

当在集群中声明队列、交换器、绑定的时候，这些操作会直到所有集群节点都成功提交元数据变更后才返回 

##### 4.集群中有内存节点和磁盘节点两种类型 

- **内存节点**虽然不写入磁盘，但是它的执行比磁盘节点要好

1.内存节点可以提供出色的性能  
2.将队列，交换器，绑定关系，用户，权限，和vhost的元数据信息保存在内存 

- **磁盘节点**能保障配置信息在节点重启后仍然可用，那集群中如何平衡这两者呢？

```
1.RabbitMQ 只要求集群中至少有 两个磁盘节点，保证高可用
2.所有其他节点可以是内存节点，当节点加入或离开集群时，它们必须要将该变更通知到至少一个磁盘节点
3.如果只有一个磁盘节点，刚好又是该节点崩溃了，那么集群可以继续路由消息，但不能创建队列、创建交换器、创建绑定、添加用户、更改权限、添加或删除集群节点

换句话说集群中的唯一磁盘节点崩溃的话，集群仍然可以运行，但直到该节点恢复，否则无法更改任何东西。
```



#### 10.集群模式

##### 1.**Master - Slave** 主备模式 == Warren (兔子窝) 模式

并发和数据量不高的情况下，MS 好用且简单，主节点提供读写，备用节点不提供读写。   
主节点挂了，就切换到备用节点，原来的备用节点升级为主节点提供读写服务，当原来的主节点恢复运行后，原来的主节点就变成备用节点。  
和 activeMQ 利用 zookeeper 做主/备一样，也可以一主多备

![ms](/images/posts/mq/ms.png)

##### 2.**Shovel** 远程模式 

把消息进行不同数据中心的复制工作，可以跨地域的让两个 MQ 集群互联，远距离通信和复制。  
Shovel 就是我们可以把消息进行数据中心的复制工作，我们可以跨地域的让两个 **MQ 集群互联**

![shovel](/images/posts/mq/shovel.png)

##### 3.mirror 镜像模式

保证 100% 数据不丢失。在实际工作中也是**用得最多的**，并且实现非常的简单，一般互联网大厂都会构建这种镜像集群模式。

保证 rabbitMQ 数据的高可靠性解决方案，主要就是实现数据的同步，一般来讲是 2 - 3 个节点实现数据同步，对于 100% 数据可靠性解决方案，一般是采用 3 个节点。

![mirror](/images/posts/mq/mirror.png)

用 KeepAlived 做了 HA-Proxy 的高可用，然后有 3 个节点的 MQ服务，消息发送到主节点上，主节点通过 mirror 队列把数据同步到其他的 MQ 节点，这样来实现其高可靠

##### 4.多活模式：**异地数据复制的主流模式**

实现异地集群采用这种**双活** 或者 **多活**模型实现，依赖 rabbitMQ 的 **federation** 插件，可以实现持续的，可靠的 AMQP 数据通信，多活模式在实际配置与应用非常的简单。

- rabbitMQ 部署架构采用双中心模式(**多中心**)，

那么在两套(或多套)数据中心各部署一套 rabbitMQ 集群，各中心的rabbitMQ 服务除了需要为业务提供正常的消息服务外，中心之间还需要实**现部分队列消息共享**

![many](/images/posts/mq/many.png)

##### 5.**federation** 插件

一个不需要构建 cluster，在 brokers 之间传输消息的高性能插件

- 1.federation 插件可以在 brokers 或者 cluster 之间传输消息，连接的双方可以使用不同的 users 和 virtual hosts，双方也可以使用不同版本的 rabbitMQ 和 erlang。

- 2.federation 插件使用 AMQP 协议通信，可以接受不连续的传输。

- 3.federation 不是建立在集群上的，而是建立在单个节点上的，如图上黄色的 rabbit node 3 可以与绿色的 node1、node2、node3 中的任意一个利用 federation 插件进行数据同步。



#### 11.幂等、一致、可靠、有序

##### 1.幂等性

消费端实现幂等性：消息不会消费多次，即使我们收到了多条一样的信息

###### 1.唯一ID + 指纹码机制，利用数据库主键去重

```
select count(1) from table where id = id + 指纹码

优点：实现简单
缺点：高并发下有数据库写入的性能瓶颈
解决：跟进ID进行分库分表进行算法路由
```

###### 2.利用redis的原子性去实现

```
问题1：是否需要落库。如果落库，如何保证数据的一致性和原子性？

问题2：如果不进行落库，缓存种的数据如果设置定时同步的策略？
```

[BLog](https://www.jianshu.com/p/78847c203b76?utm_campaign=hugo&utm_medium=reader_share&utm_content=note&utm_source=weixin-friends)

##### 2.一致性、可靠性

- 问题

P-sendMessage,默认情况下 P不知道Broker接收到没 

==> P-M 与 C-M 不一致问题

```
1.Transaction 事务机制
	会降低吞吐量，适用于不需要大批量消息场景
2.Confirm M 确认机制
	Channel设定为confirm模式，M会加个唯一ID
	M 被投送到所有匹配queue，broker会发个confirm
	==> 异步回调处理确认M，性能较好
```

- 生产端的可靠性投递

```
保证消息的成功发出
保障MQ节点的成功接受
发送端收到MQ节点(Broker)确认应答
完善消息的补偿机制
```

- 解决方案

###### 1.**消息落库**，对消息状态进行变更，缺点：对数据库有多次操作,不适用于高并发业务

![kekao](/images/posts/mq/kekao.png)

###### 2.**消息的延迟投递，做二次确认，回调检查**

```
拆出一个回调服务，将落库、检查等操作安排至回调服务上

1：发送者发送信息至MQ，消费者为下游业务方
      1.1：成功后，作为发送者发送信息至MQ，消费者为回调服务
              1.1.1 回调服务接受数据后，落库
      1.2：失败，等待发送者的延时投递信息

2、发送者发送延迟投递信息至MQ，消费者为回调服务
      1.1：查库，确认下游服务方消费已成功
      1.2：查库，确认下游服务方消费失败，通过rpc调用发送者的接口重新发送

消息发送者发送的两条信息是同时发送的
减少了对库的操作，同时解耦，保证了性能，不能百分百保证可靠性
```

![yanchitoudi](/images/posts/mq/yanchitoudi.png)



3.有序性

- [ ] 待补充

#### 12.消费端限流

海量消息瞬间推送过来，单个客户端无法同时处理那么多数据，严重会导致系统宕机，需要削峰。

QOS Quality Of Service 服务质量保证

RabbitMQ 提供了一种**qos**(服务质量保证)功能

```
在非自动确认消息的前提下(非ACK)
如果一定数目的消息(通过基于consume或者channel设置qos的值)未被确认前，不进行消费新的消息
// prefetchSize：消息体大小限制；0为不限制
// prefetchCount：RabbitMQ同时给一个消费者推送的消息个数

即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack，默认是1

// global：限流策略的应用级别
consumer[false]、channel[true]
void BasicQos(unit prefetchSize, unshort prefetchCount, bool global);
channel.basicQos(...);
```

#### 12.Serialize 序列化

- 1.**默认**情况下，RabbitMQ突然崩溃，消息/队列/交换器都**不具有**持久化的性质 

- 2.**Exchange**和 **Queue**在申明时候可通过一个durable参数实现 

```
(exchange='first', type='fanout', durable=True)
中断重启 Exchange 和 Queue 都会恢复，但 Queue 中 Message不会
```

3.**Message** 过**properties**属性，做成持久化

```
(
exchange='first', 
routing_key='', 
body='Hello World!', 
properties=pika.BasicProperties( delivery_mode = 2, )
)
```

 消息的持久化并不是一个很强的约束，涉及数据落地的时机, 及系统层面的 fsync 等问题，不要认为消息完全不会丢。需要配置其它的一些机制, 比如后面会谈到的 状态反馈 中的 confirm mode，来**共同提高消息持久化**















## 参考：

[RabbitMQ 官网](https://www.rabbitmq.com/ )
