---
layout: post
title: JVM 02 classload 类加载机制
categories: [JVM]
description: Java 虚拟机 类加载机制
keywords: JVM, JDK, classload
topmost: false
---

#### 1.类加载

将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的**方法区**内，然后在堆区创建一个 **java.lang.Class**对象，用来封装类在方法区内的数据结构。

![classload2](/images/posts/2016-07-11-jvm-classload/classload2-1609205624867.png)


##### 2.生命周期

加载loading：

- 1.通过类全限定名获取二进制流

- 2.将字节流代表的静态存储结构变为方法区运行时的数据结构

- 3.在Java堆中生成代表类的java.lang.Class对象，作为对方法区中这些数据的访问入口

 

###### [连接]

验证**verification**：确保被加载类的正确性，并不会危害jvm自身安全

- 1.文件格式验证

- 2.元数据验证，分析语义保证内容符合Java语言规范

- 3.字节码验证，分析数据流和控制流

- 4.符号引用验证

"**-****XverifyNobe**" 可关闭大部分的类验证措施，缩短虚拟机的类加载时间

 

准备**preparation**：为类静态变量分配内存，并初始化为默认值

- 1.类变量（static）的内存分配，而不包括实例变量

- 2.设置的初始值通常是数据类型的默认零值，如0、0L、null、false

public static int value = 3; value = 3 操作存放在类构造器 <clinit>中的

- 3.ConstantValue属性，即同时被final和static修饰的属性，在准备阶段value就被初始化为需要的值了

 

解析**resolution**：把类中的符号引用转化为直接引用

- 1.解析动作针对 类/接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号进行引用

- 2.符号引用就是一组符号来描述目标，可以是任何字面量

- 3.直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄

 

初始化**initialization**：为类的静态变量赋予初始值

- 1.申明类变量是指定初始值

- 2.使用静态代码块为类变量指定初始值

 

##### 补充：JVM初始化步骤：

1）假设类还没有被加载和连接，则程序先加载和连接该类       
2）假设该类父类还没被初始化，则先初始化其直接父类     
3）假设类中有初始化语句，则系统依次执行这些初始化语句

只有对类的主动使用才会导致类的初始化，主动使用如下：

使用using-卸载unloading

结束生命周期：

- 1）System.exit() 方法

- 2）程序正常执行结束

- 3）程序在执行中遇到异常或错误而异常终止

- 4）操作系统导致的JVM的进程终止


 3.".class"文件存在缺失或错误，类加载器必须在程序首次使用该类时候，才报告错误。

 

#### 2.类生命周期

1.从本地系统直接加载

2.通过网络加载.class

3.从zip，jar等归档文件中加载.class文件

4.从专有数据库中提取.class文件

5.将Java源文件编译为.class文件



#### 3.JVM 类加载机制

- 1.**全盘负责**，一个类加载器负责加载一个类，则该类所依赖和引用的Class也由该类加载器负责载入，除非显示的用其他类加载器 

- 2.**父类委托**：先让父加载器尝试加载该类，父加载器无法加载再尝试用子加载器

- 3.**缓存机制**：保证所有被加载过的Class都被缓存，当程序需要使用某个Class时，类加载器先从缓存区找该Class，

缓存区不存在，系统才重新读取该Class，这就意味着更改Class必须重启JVM，修改的程序才有效



类加载3种方式：

- 1.命令行启动由JVM负责初始化加载

- 2.Class.forName()方法动态加载：将.class文件加载到JVM中，对类进行解释，执行类中的static块

Class.forName(name, initialize, loader)可控制是否加载static块

- 3.ClassLoader.loadClass()方法动态加载：将.class文件加载到JVM，不执行static，只有newinstance才执行

![classload](/images/posts/2016-07-11-jvm-classload/classload.png)

![classload2](/images/posts/2016-07-11-jvm-classload/classload2.png)



##### **双亲委派模型**：类加载器会优先让父加载器尝试加载类

AppClassLoader 加载类都失败的化，会抛出异常 ClassNotFoundException

1.防止系统中出现多份同样的字节码

2.保证Java程序安全稳定运行

![classload3](/images/posts/2016-07-11-jvm-classload/classload3.png)

##### **自定义类加载器**

如：网络类加载器，会需要加密处理

- 1.传递的文件名需要是类的全限定名称

- 2.最好不要重写loadClass()方法，容易破坏双亲委托模式

- 3.如下Test类本身可以被 AppCL加载，故不能把类放在常规路径下，会由于双亲委托模式导致Test

不被自定义类加载器加载

![classload4](/images/posts/2016-07-11-jvm-classload/classload4.png)

![classload5](/images/posts/2016-07-11-jvm-classload/classload5.png)










## 参考：

1.[JVM类加载机制](http://mp.weixin.qq.com/s/rLooaTOU_NQTJdn28KAUFw) 

2.[JVM内存结构](http://mp.weixin.qq.com/s/li3ISdodGu2EK_Fo_4NJPA) 

3.[GC算法](http://mp.weixin.qq.com/s/olNXcRAT3PTK-hV_ehtmtw) 

4.[JVM调优--命令](http://mp.weixin.qq.com/s/QNr8somjodyvU9dRAQG2oA) 

5.[Java GC分析](http://mp.weixin.qq.com/s/S3PcA2KIzCVB2hJmsbVzyQ) 

6.[Java GC调优案例](http://mp.weixin.qq.com/s/oMZVwg6ypW9QOWal7ioFVA) 

7.[JVM调优--工具](http://mp.weixin.qq.com/s/SsJeaWz4EvZvQkYjc6J6jg) 

8.[JVM知识点总览](http://mp.weixin.qq.com/s/ebg0bT_xBahGV7OAKorBAw) 

9.[如何优化JavaGC ](http://mp.weixin.qq.com/s/ydkEkh_Uc1paftJLKIsm0w)

10.[OOM Kille](http://mp.weixin.qq.com/s/34GVlaYDOdY1OQ9eZs-iXg)r 