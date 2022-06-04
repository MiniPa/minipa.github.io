---
layout: post
title: 04 CDN 和 方向代理缓存
comments: true,
categories: [Cache]
description: CDN 和 方向代理缓存 
keywords: Redis, Cache
topmost: false
---

#### CDN 缓存

Content Delivery Network 内容分发网络
广泛采用各种缓存服务器，将这些缓存服务器分布到用户访问相对集中的地区或网络中

![cdn1](/images/posts/distribute-system-cache/cdn1.png)

![cdn2](/images/posts/distribute-system-cache/cdn2.png)



#### 反向代理缓存

反向代理位于应用服务器机房，处理所有对 Web 服务器的请求
如果用户请求的页面在代理服务器上有缓冲的话，代理服务器直接将缓冲内容发送给用户 

一般只缓存体积较小静态文件资源，如 css、js、图片

![proxy](/images/posts/distribute-system-cache/proxy.png)












## 参考：

