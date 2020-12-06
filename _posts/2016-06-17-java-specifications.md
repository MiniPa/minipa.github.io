---
layout: post
title: 《Java 开发手册》 五、安全规约
categories: [编程规范, Java 开发手册, Security]
description: Alibaba 对外开放的 Java 编程手册添加些自己总结的
keywords: 编程规范, 阿里巴巴 JAVA 开发手册, 
topmost: false
---

## 《Java 开发手册》 五、安全规约

##### 背景：

1. Alibaba Java 开发手册（有对外开放的），在地铁里看过很多遍了。
2. 这次梳理下，加上些自己补充的内容，拿出来溜溜，也夯实下自己的编程基础:smile:。
3. 另会将实际生成中碰到的不规范，不合适的情景备注进来，警告小辈:wink:

##### 级别：强制、推荐、参考
------

1. 【强制】可被用户直接访问的功能必须进行权限控制校验。 

   说明：防止没有做权限控制就可随意访问、操作别人的数据，比如查看、修改别人的订单。   
   
   

1. 【强制】用户敏感数据禁止直接展示，必须对展示数据**脱敏**。    

   说明：支付宝中查看个人手机号码会显示成:158****9119，隐藏中间 4 位，防止隐私泄露。
   
   

1. 【强制】用户输入的 SQL 参数严格使用**参数绑定**或者 **METADATA** 字段值限定，防止 SQL 注入，禁止字符串拼接 SQL 访问数据库。    

   
   
1. 【强制】用户请求传入的任何参数必须做**有效性验证**。    

   说明：忽略参数校验可能导致：

   - page size 过大导致内存溢出

   - 恶意 order by 导致数据库慢查询

   - 正则输入源串拒绝服务 ReDOS

   - 任意重定向

   - SQL 注入

   - Shell 注入

   - 反序列化注入



1. 【强制】禁止向 HTML 页面输出**未经安全过滤**或**未正确转义**的用户数据。    

   
   
1. 【强制】表单、AJAX 提交必须执行 **CSRF** 安全过滤。    

   说明：**CSRF**(**Cross-site request forgery**) 跨站请求伪造是一类常见编程漏洞。  
   对于存在 CSRF 漏洞的应用/网站，攻击者可以事先构造好 URL，只要受害者用户一访问，后台便在用户不知
   情情况下对数据库中用户参数进行相应修改。

   

1. 【强制】URL 外部重定向传入的目标地址必须执行白名单过滤。    

   ```java
   // 正例：
   try {
   	if (com.alibaba.fasttext.sec.url.CheckSafeUrl
   	       .getDefaultInstance().inWhiteList(targetUrl)){
   		response.sendRedirect(targetUrl);
   	}
   } catch (IOException e) {
       logger.error("Check returnURL error! targetURL=" + targetURL, e);
   	throw e;
   }
   ```

   

1. 【强制】Web 应用必须正确配置 **Robots** 文件，非 SEO URL 必须配置为禁止爬虫访问。    

   ```
   User-agent: * Disallow: / 
   ```

1. 【强制】 在使用平台资源，譬如短信、邮件、电话、下单、支付，必须实现正确的**防重放限制**，
   如数量限制、疲劳度控制、验证码校验，避免被**滥刷、资损**。  
   说明：如注册时发送验证码到手机，如果没有限制次数和频率，那么可以利用此功能骚扰到其它用户，并造成短信平台资源浪费。   

   

1. 【推荐】发贴、评论、发送即时消息等用户生成内容的场景必须实现防刷、文本内容违禁词过滤等风控策略。    

   





## 参考：

