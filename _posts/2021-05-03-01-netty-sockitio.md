---
layout: post
title: Netty-SocketIO 源码阅读和使用分析
comments: true,
categories: [socket, netty, socketIO, netty-socketIO, 高并发]
description: 
keywords: netty, socketIO, netty-socketIO, concurrency
topmost: false
---

#### Socket Netty 思维脑图
<iframe id="embed_dom" name="embed_dom" frameborder="0" style="display:block;width:525px; height:245px;" src="https://www.processon.com/embed/61d9967ae0b34d1be7e3bb31"></iframe>

## Netty-SocketIO 
[netty-socketIO Github](https://github.com/mrniko/netty-socketio)  
[netty-socketIO-demo Github](https://github.com/mrniko/netty-socketio-demo)

- netty-SocketIO 代码分析图例(建议下载到本地查看，作为代码分析参考，个人源码分析学习图稿)
![netty-SocketIO](/images/posts/socket/nettySocketIO.png)

### 性能
CentOS，1 个 CPU，4GB RAM 在 VM 上运行：CPU 10%，内存 15%  
- 6000个 xhr 长的轮询会话   
或   
- 15000 个 websockets 会话
每秒 4000 条消息

2014年客户反馈：  
	同时运行了 3W 个 websocket 客户端，
	并设法达到了每秒 14W/s 条消息的峰值，
平均延迟不到 1秒

百万连接需要服务器量估计： 不考虑其它损耗/优化下，4C16G 机器 100台服务器左右

### 功能
- 1.监听前台事件
- 2.向指定客户端发送事件
- 3.将指定客户端加入指定房间 
- 4.向指定房间广播事件
- 5.客户端从指定房间退出

### 核心功能介绍

#### 源码结构
![netty-socketIO-code](/images/posts/socket/netty-socketIO-code.png)

#### 核心抽象

###### SocketIOServer
封装 netty 服务，对外提供一个 socket基础服务

####### SocketIOClient
封装每一个与 Server 连接的客户端为一个 SocketIOClient 

###### Room 房间概念
- Namespace 命名空间： 包含多个房间、事件监听器、所有SocketIOClient， 以及基本的加入房间、离开房间，获取Clients、增加监听等操作。一般一个 SocketIOServer 一个 Namespace；已有连接监听、断开连接监听、心跳监听、
  - SocketIONamespace： 包含了 Namespace 和 相关的操作 BroadcastOperations
  - NamespaceHub 配置和命名空间的集线器

- Room 房间： 一个网络教室(教室内经常互通信息)，一个聊天群(群里经常互通信息) 等经常存在广播、单播操作的整体
![room](/images/posts/socket/room.png)

- SocketIOClient 客户端连接： 指一个 SocketIOClient， 一个房间内包含多个 SocketIOClient，每个有自己的 session，
一个 SocketIOClient 可以存在于多个 Room 中，如一个学生客户端的 Socket，可以同时在多个 Room，课堂1、课堂2、聊天群1、聊天群2中
  - NamespaceClient: 命名空间和Socket集合体，并提供一个 ClientHead 存储 Client 额外信息和操作，本身提供发送事件、发送信息包、监听连接、断开连接等功能
  - ClientHead: 提供连接的 命名空间对象 Namespace~NamespaceClient，以及传输对象 Transport~TransportState。
    - HandshakeData 握手信息处理
    - AckManager Ack管理：注册、触发、缓存，给每个操作存一个 ack 任务， AckEntry 以及 AckCallback。
    - CancelableScheduler 超时断开任务处理
  - ClientsBox: 提供 session~ClientHead、channel~ClientHead 的映射 缓存
  - Transport: 提供一个 Packet 的队列，以及绑定的 处理通道 Channel。
  里面会进行 Authorize 认证， ssl 安全， handshake 握手，channel 读取等处理， 底层会使用 netty 的 ChannelInboundHandlerAdapter。
    - WebSocketTransport
    - PollingTransport  
  - SocketIOChannelInitializer: 继承 netty ChannelInitializer，进行 ack、packet、handler 等处理，并 start() SocketChannel。
  
- BoradcastOperations: 房间内操作的抽象：广播、单播、排除自己广播等
  - MultiRoomOperations: 多房间操作
  - SingleRoomOperations: 单房间操作

###### Store 客户端存储
存储 SocketIOClient信息，采用发布和订阅模式，可以监听不同 SocketIOClient 事件，并可以提供持久化机制，保证高可用。
  - PubSubType: 连接、取消连接、加入房间、离开房间、广播事件
  - DataListener: 监听事件并做处理 
  - StoreType: 内存、Redis、Hazelcast 等
![pubsubStore](/images/posts/socket/pubsubStore.png)

###### netty 对接
- Handler Channel 处理
  - InPacketHandler： 读取、解码、获取Namespace、执行监听，
  - AuthorizeHandler: 认证，一般会在 Handshake 信息里处理认证，然后进行连接、激活通道、缓存、ack准备等系列操作

- JsonSupport JSON序列化处理，包编码解码中使用
![jsonsupport](/images/posts/socket/jsonsupport.png)


#### 总结
基于 netty Socket通信，定制一些 Channel Handler，以房间，连接为核心对象进行一些结构化数据存储，提供基本的 ping、ack、join、leave、dispatch 等业务操作


















































