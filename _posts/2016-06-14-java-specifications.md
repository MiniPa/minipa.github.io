---
layout: post
title: 《Java 开发手册》 二、异常日志 
comments: true,
categories: [编程规范, Java 开发手册, Exception]
description: Alibaba 对外开放的 Java 编程手册添加些自己总结的
keywords: 编程规范, 阿里巴巴 JAVA 开发手册, 
topmost: false
---

## 《Java 开发手册》 二、异常日志 

##### 背景：

1. Alibaba Java 开发手册（有对外开放的），在地铁里看过很多遍了。
2. 这次梳理下，加上些自己补充的内容，拿出来溜溜，也夯实下自己的编程基础:smile:。
3. 另会将实际生成中碰到的不规范，不合适的情景备注进来，警告小辈:wink:

##### 级别：强制、推荐、参考
------



#### (一) 异常处理 

1. 【强制】不要捕获 Java 类库中定义的继承自 **RuntimeException** 的运行时异常类，  
如：IndexOutOfBoundsException / NullPointerException，这类异常由程序员预检查来规避，保
   证程序健壮性。    
   
   ```
   正例：if(obj != null) {...}
   反例：try { obj.method() } catch(NullPointerException e){…}
   ```
   
1. 【强制】异常不要用来做流程控制，条件控制，因为异常的处理效率比条件分支低。    

   
   
1. 【强制】对**大段代码进行 try-catch**，这是不负责任的表现。  
catch 时请分清稳定代码和非稳定代码，**稳定代码**指的是无论如何不会出错的代码。  
   对于**非稳定代码**的 catch 尽可能进行区分异常类型，再做对应的异常处理。    
   
   
   
1. 【强制】捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请
将该异常抛给它的调用者。  
   最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。    
   
   
   
1. 【强制】有 try 块放到了事务代码中，catch 异常后，如果需要回滚事务，一定要注意手动回
滚事务。    
   
   
   
1. 【强制】finally 块必须对资源对象、流对象进行关闭，有异常也要做 try-catch。  

   说明：如果 JDK7，可以使用 **try-with-resources** 方法。    
   
   
   
1. 【强制】不能在 finally 块中使用 return，**finally** 块中的 **return** 返回后方法结束执行，不
会再执行 try 块中的 return 语句。    
   
   
   
1. 【强制】捕获异常与抛异常，必须是完全匹配，或者捕获异常是抛异常的父类。    

   ```
   说明：如果预期抛的是绣球，实际接到的是铅球，就会产生意外情况。
   ```
   
1. 【推荐】方法的返回值可以为 **null**，不强制返回空集合，或者空对象等，必须添加注释充分
说明**什么情况下会返回 null 值**。调用方需要进行 null 判断防止 NPE 问题。    
   
   ```
   说明：本规约明确防止 NPE 是调用者的责任。
   即使被调用方法返回空集合或者空对象，对调用者来说，也并非高枕无忧，必须考虑到远程调用失败，运行时异常等场景返回 null 的情况。
   ```
   
1. 【推荐】防止 NPE，是程序员的基本修养，注意 NPE 产生的场景：    

   ```
   1） 返回类型为包装数据类型，有可能是 null，返回 int 值时注意判空。
    反例：public int f(){ return Integer 对象}，如果为 null，自动解箱抛 NPE。
   2） 数据库的查询结果可能为 null。
   3） 集合里的元素即使 isNotEmpty，取出的数据元素也可能为 null。
   4） 远程调用返回对象，一律要求进行 NPE 判断。
   5） 对于 Session 中获取的数据，建议 NPE 检查，避免空指针。
   6） 级联调用 obj.getA().getB().getC()；一连串调用，易产生 NPE。
   ```
   
1. 【推荐】在代码中使用“抛异常”还是“返回错误码”  
对于公司外的 **http/api 开放接口**必须使用“**错误码**”；  
   而应用**内部推荐异常抛**出；  
   跨应用间 RPC 调用优先考虑使用 **Result** 方式，封装 isSuccess、“错误码”、“错误简短信息”。    
   
   ```
   说明：关于 RPC 方法返回方式使用 Result 方式的理由：
   1）使用抛异常返回方式，调用方如果没有捕获到就会产生运行时错误。
   2）如果不加栈信息，只是 new 自定义异常，加入自己的理解的 error message，对于调用端解决问题的帮助不会太多。
   如果加了栈信息，在频繁调用出错的情况下，数据序列化和传输的性能损耗也是问题。
   ```
   
1. 【推荐】定义时区分 **unchecked** / **checked** 异常，避免直接使用 RuntimeException 抛出，更
不允许抛出 Exception 或者 Throwable，应使用有业务含义的自定义异常。推荐业界已定义过
   的自定义异常，如：**DaoException** / **ServiceException** 等。    
   
   
   
1. 【参考】避免出现重复的代码（Don’t Repeat Yourself），即 **DRY** 原则。    

   ```
   说明：随意复制和粘贴代码，必然会导致代码的重复，在以后需要修改时，需要修改所有的副本，容易遗漏。必要时抽取共性方法，或者抽象公共类，甚至是共用模块。
   
   正例：一个类中有多个 public 方法，都需要进行数行相同的参数校验操作，这个时候请抽取：
   private boolean checkParam(DTO dto){...} 
   ```



#### (二) 日志规约

1. 【强制】应用中不可直接使用日志系统（**Log4j**、**Logback**）中的 API，而应依赖使用日志框架SLF4J 中的 API，使用**门面模式**的日志框架，有利于维护和各个类的日志处理方式统一。    

   ```
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   private static final Logger logger = LoggerFactory.getLogger(Abc.class); 
   ```
   
1. 【强制】日志文件推荐至少保存 15 天，因为有些异常具备以“周”为频次发生的特点。    

   
   
1. 【强制】应用中的扩展日志（如打点、临时监控、访问日志等）命名方式：**appName_logType_logName.log** 

   ```
   logType:日志类型，推荐分类有 stats/desc/monitor/visit等；
   logName:日志描述。
   这种命名的好处：通过文件名就可知道日志文件属于什么应用，什么类型，什么目的，也有利于归类查找。
   
   正例：mppserver 应用中单独监控时区转换异常，如：mppserver_monitor_timeZoneConvert.log
   说明：推荐对日志进行分类，错误日志和业务日志尽量分开存放，便于开发人员查看，也便于通过日志对系统进行及时监控。
   ```
   
1. 【强制】对 trace/debug/info 级别的日志输出，必须使用条件输出形式或者使用占位符的方式。    

   ```
   说明：logger.debug("Processing trade with id: " + id + " symbol: " + symbol); 
   如果日志级别是 warn，上述日志不会打印，但是会执行字符串拼接操作，如果 symbol 是对象，会执行 toString()方法，浪费了系统资源，执行了上述操作，最终日志却没有打印。
   
   正例：（条件）
   if (logger.isDebugEnabled()) {
   	logger.debug("Processing trade with id: " + id + " symbol: " + symbol);
   }
   正例：（占位符）
   logger.debug("Processing trade with id: {} and symbol : {} ", id, symbol); 
   ```
   
1. 【强制】避免重复打印日志，浪费磁盘空间，务必在 log4j.xml 中设置 **additivity=false**。    

   ```
   正例：<logger name="com.taobao.ecrm.member.config" additivity="false">
   ```
   
1. 【强制】异常信息应该包括两类信息：**案发现场信息**和**异常堆栈信息**。如果不处理，那么往上抛。    

   ```
   正例：logger.error(各类参数或者对象 toString + "_" + e.getMessage(), e);
   ```
   
1. 【强制】输出的 **POJO** 类必须重写 **toString** 方法，否则只输出此对象的 hashCode 值（地址值），没啥参考意义。    

   
   
1. 【推荐】可以使用 **warn** 日志级别来记录用户输入参数错误的情况，避免用户投诉时，无所适从。  
注意日志输出的级别，error 级别只记录系统逻辑出错、异常、或者重要的错误信息。  
   如非必要，请不要在此场景打出 error 级别，避免频繁报警。    
   
   
   
1. 【推荐】谨慎地记录日志。  
生产环境禁止输出 debug 日志；  
   有选择地输出 info 日志；  
   如果使用 warn 来记录刚上线时的业务行为信息，一定要注意日志输出量的问题，避免把服务器磁盘撑爆，并记得及时删除这些观察日志。    
   
   ```
   说明：大量地输出无效日志，不利于系统性能提升，也不利于快速定位错误点。
   纪录日志时请思考：这些日志真的有人看吗？看到这条日志你能做什么？能不能给问题排查带来好处？
   ```
   
1. 【参考】如果日志用英文描述不清楚，推荐使用**中文注释**。  
对于中文 UTF-8 的日志，在 **secureCRT**中，**set encoding=utf-8；**  
   如果中文字符还乱码，  
   请设置：全局>默认的会话设置>外观>字体>      
   选择字符集 gb2312；  
   如果还不行，执行命令：set termencoding=gbk，并且直接使用中文来进行检索。  






## 参考：

