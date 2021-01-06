---
layout: post
title: MQ 04 sync/async 同步异步消息 
categories: [MQ]
description: MQ sync/async 同步异步消息 优劣分析
keywords: MQ, sync MQ, async MQ
topmost: false
---

#### 1.同步、异步

- 异步：关心结果，但不是立马关心，用轮询或者回调等方式处理结果

- 同步：   当时就关心的结果的

- oneway：发出去就不闻不问

#### **RPC** C-S 同步异步 

RPC **客户端异步** 与 **服务端异步** 的，而且是可以任意组合的

- C = Client 客户端

- S = Server 服务端

##### 1.C--同步 ~ 服-异步

```java
Future<Result> future = request(server); //server立刻返回future
synchronized(future){ 
    while(!future.isDone()){
       future.wait(); //server 处理结束后会 notify 这个 future，并修改 isdone 标志
    }
}
return future.get();
```

##### 2.C-异步 ~ S-异步

```java
Future<Result> future = executor.submit(
    new Callable(){
        public void call<Result>(){
    		result = request(server);
		}
    }
)
return future;
```

##### 3.C-同步 ~ S-同步

```java
Result result = request(server);
```

##### 4.C-异步 ~ S-同步

```java
Future<Result> future = request(server); // server 立刻返回 future
return future
```

- 1）对于客户端 C 来说，同步与异步主要是拿到一个 **Result**，还是 **Future**(Listenable) 的区别 

- 2）服务端异步：需要 RPC 协议支持的。  
  参考servlet 3.0规范，服务端可以吐一个future 给客户端，并且在future done的时候通知客户端

#### 3.2个误区

- RPC 只有客户端能做异步，服务端不能

- 异步只能通过线程池

##### 1.服务端使用异步 

==> 解放了线程和I/O ，如服务端有大量I/O消耗，每个请求需同步响应，立刻返回，几乎没法做I/O合并（当然接口可以设计成batch的，但可能batch发过来的仍然数量较少）。 

==> 异步的方式返回给客户端 future，就可以有机会进行I/O的合并，把几个批次发过来的消息一起落地（对于MySQL等允许batch insert的数据库效果明显），并且彻底释放了线程。（当然也会产生事务、可靠性等系列问题） 

==> S 异步有效提高并发量

##### 2.返回 future 的方式不一定只有线程池。

- ThreadPool 内可进行 同步/异步 操作。
- 异步操作除 ThreadPool 可使用 NIO、Event 等方案

MQ 发送消息可能会有网络延迟等问题，解耦 C 发送，保持主流程畅通比较好。

==> C 半同步 半异步

- ThreadPool 不阻塞主流程

- ThreadPool 中 Task 要等待 server反回

==> S 纯异步 

C 的线程池 wait，等 server 端吐回的future上，处理完毕，除阻塞继续进行。

#### 4.同步能够保证结果**，**异步能够保证效率，合理结合。























## 参考：
