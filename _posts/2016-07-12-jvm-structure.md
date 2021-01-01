---
layout: post
title: JVM 03 JMM 结构 + JVM 调优
categories: [JVM]
description: JMM 结构 + JVM 调优
keywords: JVM, JDK, structure
topmost: false
---

#### 1.JMM Java Memory Model  Java 内存模型

局部变量区: 组织为一个数组   
操作数栈: 同为数组 

- JVM 字 只能通过标准的栈操作 push & pop 操作 

- CPU & ALU 逻辑上识别操作数的唯一来源

帧数据区域 - 保存一些数据 支持常量池解析    
正常方法返回 异常 派发机制   
a=b+c, 栈机制操作数流程

 默认虚拟机内存比例

```
 Eden:From:To = 8:1:1  ==> -XX:SurvivorRatio = 8:1
 Yong:Old = 1:2        ==> -XX:NewSize = 1 修改为 1:1
```

[官方文档查询](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)

#### 2.JMM 基本结构

![JVM1](/images/posts/2016-07-12-jvm-structure/JVM1.png)

![JVM2](/images/posts/2016-07-12-jvm-structure/JVM2.png)

![JVM3](/images/posts/2016-07-12-jvm-structure/JVM3.png)

```
-XX:NewSize和-XX:MaxNewSize YongG 建议设为整个堆大小的1/3或者1/4
-XX:SurvivorRatio Eden和其中一个Survivor的比值
-XX:+PrintTenuringDistribution 显示每次Minor GC时Survivor区中各个年龄段的对象的大小
-XX:InitialTenuringThreshol和-XX:MaxTenuringThreshold 晋升到老年代的对象年龄的最小值和最大值
```

![JVM4](/images/posts/2016-07-12-jvm-structure/JVM4.jpg)

![JVM5](/images/posts/2016-07-12-jvm-structure/JVM5.png)

![JVM6](/images/posts/2016-07-12-jvm-structure/JVM6.png)

![JVM7](/images/posts/2016-07-12-jvm-structure/JVM7.png)

##### 案例代码

![JVM8](/images/posts/2016-07-12-jvm-structure/JVM8.png)

```
Heap：
	Object：HelloWord、SimpleDateFormat、String、LOGGER
Method Area：
	Class：SimpleDateFormat、Logger、HelloWord ...
		method：sayHello()...
Thread 1：
	Parameter reference： "message" to String object
	Variable reference："formatter" SimpleDateFormat、 "today" String
local primitive："lineNo"
```

![JVM9](/images/posts/2016-07-12-jvm-structure/JVM9.png)



#### 3.JVM 调优

JDK内存查看：**JConsole** / **JavaVisualVM**

目的：减少 **GC** 频率和 **FullGC** 次数

- Java在没有授权的情况下不能访问内存  
  **堆**：运行时动态分配内存 存取速度慢些  
  **栈**：基本类型变量 存取快 可共享 大小生存期确定 不灵活

```
-X print help on non-standard options
-Xms<size> 设置初始Java堆大小
-Xmx<size> 设置最大Java堆大小
-Xss<size> 设置Java线程堆栈大小
-Xmn 设置新生代内存大小
XX:+PrintGCDetails
```

##### 3.1 内存控制效率优化

```
String & StringBuffer
二维数组 & 一维数组
HashMap & 普通数组 
arrayCopy() 提高数组截取速度
System.gc() finalize()
protected void finalize(){};
```

##### 3.2 FullGC原因：

1. 旧生代空间不足：多方面避免思考  
2. PemanentGeneartion 空间不足 - 静态对象太多
3. 统计 GC 晋升到旧的平均大小比旧剩余空间大-控制比例
4. System.gc() 显式调用

```
NewGeneration & Survivor 比例问题
NG-小：GC多 大对象放不下
NG-大：OG小 FG 比例1/3合适
Survivor-小：对象直接 eden 到 old 降低存活时间
S-大：eden大 GC多
-XX:MaxTenuringThreshold=n 控制 NG 存活时间
```

##### 3.3 调优设置

1. 吞吐量优先 -XX:**GCTimeRatio**=n;
2. 暂停时间优先 -XX:**GCPauseRatio**=n;

- 堆设置

```
-Xms：初始堆大小 -Xmx:最大堆大小 
-XX:NewSize=n:年轻代大小 -XX:NewRatio=n:老/轻比例
-XX;SurvivorRatio=n;年轻中Eden与2Survivor比值
-XX;MaxPermSize=n;持久代大小
```

- 收集器设置

```
-XX：+UseSerialGC--串行收集器
-XX：+UseParallelGC--并行收集器
-XX：+UseParalledlOldGC--并行老年代收集器
-XX：+UseConcMarkSweepGC--并发收集器
```

- 垃圾回收统计信息

```
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:filename;
```

- 并行收集器设置

```
-XX:ParallelGCThreads=n;设置并行用CPU数 线程数
-XX:MaxGCPauseMillis=n;设置并行最大暂停时间
-XX:GCTimeRatio=n;设置GC占程序运行时间%1/(1+n)
```

- 并发收集器设置

```
-XX:+CMSincrementalMode;设增量模式-单CPU
-XX:ParallelGCThreads=n;设置并发年轻代收集方式为并行使用的CPU数 并行收集线程数
```



#### 4.JVM Tools 工具

##### jps 查看 java 进程

![jps](/images/posts/2016-07-12-jvm-structure/jps.png)  

##### jstat -gc 173344  

#查看java 进程GC情况

```
S0C     S1C      S0U    S1U     EC         EU        OC          OU         
75776.0 77312.0  0.0   25955.4  2357760.0  2188549.0 1043968.0   130895.9  

MC       MU        CCSC    CCSU 
111872.0 104881.0 14336.0 13114.3     

YGC     YGCT    FGC    FGCT     GCT
11    0.356   4      1.231    1.587
```

上述案例中   
S0:S1:EC = 1:1:30 比正常的 1:8:8 大  
Yong:Old = 2.5:1  比正常的 1:2 大

![gc](/images/posts/2016-07-12-jvm-structure/gc.png)

```
解释如下：
S0C 是指：Survivor0区的分配空间
S0U 是指：Survivor1区的已经使用的空间

EC是指:Eden区所使用的空间
EU是指：Eden区当前使用的空间

OC是指：老年代分配的空间
OU是指：老年代当前使用的空间

PC是指：持久待分配的空间
PU是指：持久待当前使用的空间

YGC是指：年轻代发生的次数，这里是354次
YGCT是指：年轻代发送的总时长，这里是54.272秒，因此每次年轻代发生GC，即平均每次stop-the-world的时长为54.272/354=0.153秒。

FGC是指：年老代回收的次数，或者成为FullGC的次数。
FGCT是指：年老代发生回收的总时长。
GCT是指：包括年轻代YGC及年老代FGC的总时间长。
```



[调优命令](https://mp.weixin.qq.com/s/QNr8somjodyvU9dRAQG2oA?)

##### jmap: JVM memory map

生成 heap dump   还以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件    

jmap -dump:live,format=b,file=dump.hprof 9186

root用户

![root](/images/posts/2016-07-12-jvm-structure/root.png)

erp用户

![erp](/images/posts/2016-07-12-jvm-structure/erp.png)

dump.hprof 可以用 [MAT](http://www.eclipse.org/mat/) 打开 

##### jhat:  JVM Heap Analysis Tool 

与 jmap 搭配使用，用来分析jmap生成的dump    
jhat生成了一个微型的HTTP/HTML服务

![jhat](/images/posts/2016-07-12-jvm-structure/jhat.png)

##### jinfo: JVM Configuration info 

实时查看和分析jvm运行参数，jps -v 只能查看到显示指定参数

![jinfo](/images/posts/2016-07-12-jvm-structure/jinfo.png)

##### jps: JVM Process Status Tool 

显示指定系统内所有HotSpot虚拟机进程

![jps2](/images/posts/2016-07-12-jvm-structure/jps2.png)

##### jstat: JVM statistics Monitoring 

监视JVM运行时状态命令

![jstat1](/images/posts/2016-07-12-jvm-structure/jstat1.png)

![jstat2](/images/posts/2016-07-12-jvm-structure/jstat2.png)

![jstat3](/images/posts/2016-07-12-jvm-structure/jstat3.png)

![jstat4](/images/posts/2016-07-12-jvm-structure/jstat4.png)

MAT Memory Analyzer Tool: 内存分析工具

http://www.eclipse.org/mat/downloads.php
https://www.cnblogs.com/liuchuanfeng/p/8484641.html



#### 5.调优实操

##### 5.1 调优目标  、核心

目标：尽量避免 **YongGC** 和 **FullGC**，或其时间足够短 

核心：调整 **Yong**、**Old** 代内存空间大小，及使用 **GC发生器** 的类型等

![jvm](/images/posts/2016-07-12-jvm-structure/jvm.png)

##### 5.2 准备工作

######  1）配置 jstatd 的远程 RMI 服务
将 jstatd.all.policy 文件放在 JAVA_HOME/bin 中，如下为文件内容

```
grant codebase "file:${java.home}/../lib/tools.jar" {
permission java.security.AllPermission;

};
```

执行命令 ==> 开启远程VisualVM

```
jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=10.27.20.xx &
```
10.27.20.xx为你服务器的ip地址,&表示用守护线程的方式运行

###### 2）执行 jvisualvm 
C:\glassfish4\jdk7\bin\jvisualvm.exe 打开JVM控制台

![jvm2](/images/posts/2016-07-12-jvm-structure/jvm2-1609471150356.png)

工具--插件--中找到 Visual GC 插件进行安装

###### 3）对要执行java程序进行调优
以 c1000k.jar 为例，在该 jar 包所在目录下建立一个 start.sh 文件，文件内容如下

```
java -server -Xms4G -Xmx4G -Xmn2G -XX:SurvivorRatio=1 -XX:+UseConcMarkSweepGC -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1100 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -jar c1000k.jar&
```

###### 4）配置JVM远程连接

[如下是jvm远程配置]

```
-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port=1100 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false
```

###### 5）正式调优 jvisualvm.exe

![jvm3](/images/posts/2016-07-12-jvm-structure/jvm3-1609471280714.png)

![jvm4](/images/posts/2016-07-12-jvm-structure/jvm4.png)

做好如上工作，你就有了观察 JVM 的手段了，后面依据 JVM 实际情况参数进行相关调优

##### 5.3 调优实操

###### 1.调优核心：调整Yong、Old代内存空间大小，及使用GC发生器的类型等

如下配置了 **JVM 常用参数**，并开启了**远程调控**方便 JVM 观察

```
java -server -Xms4G -Xmx4G -Xmn2G -XX:SurvivorRatio=1 -XX:+UseConcMarkSweepGC -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1100 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -jar c1000k.jar&
```

-Xms4G:  启动时 heap 占用大小(Y+O)，大点程序会启动快点
-Xmx4G:  运行期间 heap 最大大小
**-Xmn2G**: 年轻代的空间大小，剩下的是年老代的空间

-XX:SurvivorRatio=1:  年轻代空间中2个Survivor空间与Eden空间的大小比例，此处 1:1:1

-XX:+UseConcMarkSweepGC: 使用 GC的回收类型 Serial ParNew CMS G1

Xss 是指设定每个线程的堆栈大小



###### 2.当EU = EC时候，发生YGC YGC+1 YGCT时长增加

当发生YGC的时候，如果S0U或S1U区如果有任意一个区域为0的时候，此时YGC的速度很快，相反如果S0U或者S1U中都有数据，或相对满的时候，此时YGC的时间边长，这就是因为S0/S1及Eden区的比例问题导致



###### 3.调优最优结果：

YGC次数少，时间快  
FGC时间长，好多天都不发生

![jvm5](/images/posts/2016-07-12-jvm-structure/jvm5-1609471810792.png)



###### 4.tomcat 内存调优

[Blog](https://www.cnblogs.com/littlehb/archive/2013/04/02/2994785.html)

```
JAVA_OPTS=-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms1024m -Xmx1024m -Xss1m -XX:NewSize=256m -XX:MaxNewSize=512m -XX:PermSize=256M  -XX:MaxPermSize=512m 

-XX:+DisableExplicitGC
```

然后查看 tomcat 的管理平台，查看内存情况



#### 6.调优总结

##### 1.Yong 年轻代大小选

- **响应时间优先**：

Y**尽可能设**大，直到接近系统的最低响应时间限制 （根据实际情况选择）、 在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象 

- **吞吐量优先**的应用 ：

**尽可能的大**，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用

##### 2.Old 大小选择

- **响应时间优先**的应用：

年老代使用**并发收集器**，所以其大小需要小心设置，一般要考虑并发会话率 和会话持续时间 等一些参数。  
如果堆设置小了，可以会造成内存碎片、高回收频率，以及应用暂停而使用传统的标记清除方式。  
如果堆大了，则需要较长的收集时间，最优化的方案，一般需要参考以下数据获得：

```
并发垃圾收集信息
持久代并发收集次数
传统GC信息
花在年轻代和年老代回收上的时间比例
```

减少年轻代和年老代花费的时间，一般会提高应用的效率

- **吞吐量优先**的应用 ：Y很大 O较小

可以尽可能回收掉大部分短期对象，减少中期的对象   
而年老代尽存放长期存活对象

##### 3.较小堆引起的碎片问题

因为年老代的并发收集器使用标记、清除算法，所以不会对堆进行压缩。  
当收集器回收时，他会把相邻的空间进行合并，这样可以分配给较大的对象。  
但是，当堆空间较小时，运行一段时间以后，就会出现“碎片”，  
如果并发收集器找不到足够的空间，那么并发收集器将会停止，然后使用传统的标记、清除方式进行回收。

```
如果出现“碎片”，可能需要进行如下配置：
-XX:+UseCMSCompactAtFullCollection ：用并发收集器时，开启对年老代的压缩
-XX:CMSFullGCsBeforeCompaction=0 ：  上面配置开启的情况下，这里设置多少次Full GC后，对年老代进行压缩
```




## 参考：

