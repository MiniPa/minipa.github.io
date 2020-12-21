---
layout: post
title: JVM 01 编译和执行机制
categories: [JVM]
description: Java 虚拟机 编译和加载机制
keywords: JVM, JDK, Java Compile
topmost: false
---

#### Java编译/执行

编译流程:    
源代码->词法分析器->Token流->语法分析器->语法树/抽象语法树->词义分析器->字节码生成器->JVM字节码   
(符号表)

![compile](/images/posts/2016-07-10-jvm-compile/compile.png)

执行流程:  
JVM字节码->字节码解释器  
JVM字节码->机器无关优化->中间代码->机器相关化->中间代码->寄存器分配器->中间代码->目标代码生成器->目标代码    
(符号表)

![compile2](/images/posts/2016-07-10-jvm-compile/compile2.png)



#### Java源码编译机制

- 分析&输入到符号表 Parse&Enter
- **注解**处理 Annotation      Processing
- 语义分析和生成class文件，Analyse and Generate

##### class文件

结构信息：class文件版本号&数量大小信息  

元数据：对应.java文件  
常量申明信息  
包含类/继承类/接口的申明信息  
域&方法申明信息&常量池

方法信息：.java中语句和表达式对应信息  
	字节码 异常处理器表 求值栈 局部变量区  
	大小 求值栈类型记录 调试符号信息

##### 类加载机制

```java
通过ClassLoader&子类实现
ClassLoader Architecture
>>> Bootstrap ClassLoader : Load JRE\lib\rt.jar
	或者Xbootclasspath选项指定的jar包
	----加载$JAVA_HOME所有class C++实现
>>>Extension ClassLoader: Load JRE\lib\ext\*.jar
	或者Djava.ext.dirs指定目录下的jar包
	----加载$JAVA_HOME…
>>>App ClassLoader: Load ClassPath/Djava.class.path
	指定目录下的类&jar包
	----加载classpath中指定jar&目录中class
>>>Custom ClassLoader:通过java.lang.ClassLoader的
	子类的自定义加载class
	应用程序根据自身需要定义的ClassLoader
	如：tomcat jboss根据j2ee规规范自行实现CL
自上向下尝试加载类 --- 自下向上尝试是否已经加载
```




## 参考：

