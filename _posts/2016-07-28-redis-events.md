---
layout: post
title: Redis 10 server 事件模型 
comments: true,
categories: [Redis]
description: Redis server 事件模型
keywords: Redis, events
topmost: false
---

#### Event 事件驱动

##### 1.Redis Server 是 event 驱动的程序

- file event 文件事件 -- io ，底层 select/epoll 模型驱动的

  select/epoll是啥？ 

- time event 时间事件 -- 定时任务，底层维护一个定时任务列表

##### 2.redis 事件模型 是经典NIO模型

- 底层通过select/epoll等机制实现异步NIO

- 通过检测event到来后，for循环实现串行处理

[Blog](https://www.jianshu.com/p/0e414a70874d)

#### Event 注册

##### 1.server启动流程

redis server的**main**函数在启动过程中按照以下三个步骤进行初始化并进入运行状态：

- 1.初始化服务器**配置**

- 2.载入**配置文件**初始化配置（redis的监听端口在这一步完成配置）

- 3.创建并初始化服务器的数据结构**监听端口**等（完成端口的监听，事件注册等）循环处理事件（循环处理**file** event 和 **time** event）

```java
int main(int argc, char **argv) {
    
    // 初始化服务器
    initServerConfig();

    // 载入配置文件， options 是前面分析出的给定选项
    loadServerConfig(configfile,options);

    // 创建并初始化服务器数据结构
    initServer();

    // 运行事件处理器，一直到服务器关闭为止
    aeSetBeforeSleepProc(server.el,beforeSleep);
    aeMain(server.el);

    // 服务器关闭，停止事件循环
    aeDeleteEventLoop(server.el);

    return 0;
}
```

##### 2.server 初始化流程

- 1.创建监听的**socket**用于accept连接

- 2.注册监听socket到底层时间驱动如select/epoll当中（aeCreateFileEvent），创建时间事件（aeCreateTimeEvent）

```
void initServer() {
    // 打开 TCP 监听端口，用于等待客户端的命令请求
    if (server.port != 0 &&
        listenToPort(server.port,server.ipfd,&server.ipfd_count) == REDIS_ERR)
        exit(1);


    // 为 serverCron() 创建时间事件
    if(aeCreateTimeEvent(server.el, 1, serverCron, NULL, NULL) == AE_ERR) {
        redisPanic("Can't create the serverCron time event.");
        exit(1);
    }

    // 为 TCP 连接关联连接应答（accept）处理器，用于接受并应答客户端的 connect() 调用
    for (j = 0; j < server.ipfd_count; j++) {
        if (aeCreateFileEvent(server.el, server.ipfd[j], AE_READABLE,
            acceptTcpHandler,NULL) == AE_ERR)
            {
                redisPanic(
                    "Unrecoverable error creating server.ipfd file event.");
            }
    }
}
```














## 参考：

