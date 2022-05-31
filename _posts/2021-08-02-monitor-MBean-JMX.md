---
layout: post
title: JMX MXBean
comments: true,
categories: [jmx, MXBean, Monitor]
description: Java 监控 MXBean
keywords: JMX, MXBean, 监控, monitor
topmost: false
---

## Java核心技术 节点（Monitor）
<iframe id="embed_dom" name="embed_dom" frameborder="0" style="display:block;width:525px; height:245px;" src="https://www.processon.com/embed/623b22e5e401fd070bbe3acd"></iframe>


## 一、JMX Java Management Extensions
1）一个为应用程序植入管理功能的框架,一套标准的代理和服务  
2）主要用于对JAVA应用程序和JVM进行监控和管理  
JConsole和JVisualVM中能够监控到JAVA应用程序和JVM的相关信息都是通过JMX实现的

### 1.JMX 架构
```html
1.分布层（Distributed layer）：包含可以使管理应用与JMX Agents交互的组件
	一旦通过交互组件与JMX Agents建立连接，用户可以用管理工具来和注册在Agents中的MBeans进行交互

2.代理层（Agent layer）：包含JMX Agent以及它们包含的MBean Servers
	主要组件是MBean server，作为JMX Agents的核心，它充当MBeans的注册中心
	提供了4个Agent 服务来使对MBean的管理更容易：
		计时器（Timer）
		监控（monitoring）
		动态加载MBean（dynamic MBean loading ）
		关系服务（relationship services ）

3.指示层（Instrumentation layer）：包含代表可管理资源的MBeans
	最接近管理资源的，它由注册在Agents中的MBeans组成，这个MBean允许通过JMX Agent来管理
每个MBean都暴露出来针对底层资源的操作和访问
```

![jmx 架构图](/images/types/jmx/jmx-arch.png)

![jmx 监控图](/images/types/jmx/jmx-monitor.png)


### 2.JMX 角色
- 1.管理资源（Manageable resource）: 所有影响程序运行的资源
- 2.管理组件（MBean，managed bean）: 直接面向MBean
- 3.管理组件服务器（MBean Server）: 盛装和管理一组MBeans的容器


### 3.JMX 机制
#### 3.1.注册机制
- 每个添加到MBean Server的MBean在注册的时候都要提供一个ObjectName来区分彼此
- MBean 注册到 MBeanServer，MBean 注册ID ObjectName
- ObjectName
  - 1）域名:通常是和想要注册到的MBean Server的名称标识相同
  - 2）键值对列表:唯一的标识MBean，如：HelloAgent:name=helloWorld

#### 3.2.代理机制: 代理服务来管理 MBeans
- 创建MBean间关系
- 动态加载类
- 简单监视服务
- 计时器


### 4.JMXServiceURL格式说明
```html
service:jmx:rmi://localhost:0/jndi/rmi://localhost:1099/jmxrmi，localhost:0 部分可以省略掉

service:jmx: 这个是JMX URL的标准前缀，所有的JMX URL都必须以该字符串开头，否则会抛MalformedURLException

rmi: 这个是jmx connector server的传输协议，在这个url中是使用rmi来进行传输的

localhost:0 这个是jmx connector server的IP和端口，也就是真正提供服务的host和端口，可以忽略，那么会在运行期间随意绑定一个端口提供服务

jndi/rmi://localhost:1099/jmxrmi 这个是jmx connector server的路径，
具体含义取决于前面的传输协议。
比如该URL中这串字符串就代表着该jmx connector server的stub是使用 jndi api 绑定在 rmi://localhost:1099/jmxrmi 这个地址
```

### 5.JMX 的使用步骤
- 步骤一
```html
1. 定义接口与资源实体类，接口中定义属性与操作；get表示可读，set表示可写

2. 创建Agent类
  a. 创建MBeanServer，相当于管理MBean的容器，创建方式有两种一获取JVM中默认启动的Mbean server 或者 自己创建。
  b. 创建ObjectName，用于标识唯一的资源MBean，格式为："域名：name=MBean名称"。
  c. 绑定MBean与对应的ObjectName并注册到MBeanServer
至此即可通过JConsole管理本地的MBean的先关信息了，同事也可以提供其他的连接方式，如：

3. rmi连接方式
  a. 注册监听端口号
  b. 创建JMXServiceURL，格式为：service:jmx:rmi://localhost:0/jndi/rmi://localhost:1099/jmxrmi（完整版）  JMXServiceURL格式说明 。
  c. 创建jmxConnectorServer，绑定MBserver与Url

4. HtmlAdaptor连接管理方式（需要引入jmxtools.jar）
  a. 创建Html适配器，并设置监听端口
  b. 创建适配器ObjectName，绑定Html适配器并注册到 MBeanServer
  c. 开启适配器
```
```java
    public static void init(LoggerContext loggerContext) throws Exception {
        mBeanServer = MBeanServerFactory.createMBeanServer(DOMAIN_NAME); // 创建 MBeeanServer管理服务
        // 注册服务
        ObjectName objectName = new ObjectName(DOMAIN_NAME + ":name=" + RELOAD_CONFIG_NAME);
        jmxConfigurator = new JMXConfigurator(loggerContext, mBeanServer, objectName);
        mBeanServer.registerMBean(jmxConfigurator, objectName);  // 注册MBean JMI

        // htmlAdaptor 注册连接
        htmlAdaptorServer = new HtmlAdaptorServer();
        htmlAdaptorServer.setPort(HTML_PORT);
        objectName = new ObjectName(DOMAIN_NAME + ":name=" + CONNECTOR_NAME);
        mBeanServer.registerMBean(htmlAdaptorServer, objectName);  // 注册MBean HTMLAdaptor
        htmlAdaptorServer.start();

        // rmi方式
        //这句话非常重要，不能缺少！注册一个端口，绑定url后，客户端就可以使用rmi通过url方式来连接JMXConnectorServer
        LocateRegistry.createRegistry(RMI_PORT);
        JMXServiceURL url = new JMXServiceURL("service:jmx:rmi:///jndi/rmi://localhost:" + RMI_PORT + "/logback_config");
        jmxConnectorServer = JMXConnectorServerFactory.newJMXConnectorServer(url, null, mBeanServer);
        jmxConnectorServer.start();
    }
```
- 步骤二 创建MBeanServer和JMX Agent，MBeanServer是在JMX Agent 中存在的
```java
public class HelloAgent implements NotificationListener {
    private MBeanServer mbs;

    public HelloAgent() {
        this.mbs = MBeanServerFactory.createMBeanServer("HelloAgent");

        HelloWorld hw = new HelloWorld();
        ObjectName helloWorldName = null;
        try{
            helloWorldName = new ObjectName("HelloAgent:name=helloWorld");
            mbs.registerMBean(hw, helloWorldName);
        } catch (Exception e) {
            e.printStackTrace();
        }
        startHtmlAdaptorServer();
    }

    public void startHtmlAdaptorServer(){
        HtmlAdaptorServer htmlAdaptorServer = new HtmlAdaptorServer();
        ObjectName adapterName = null;
        try {
            // 多个属性使用,分隔
            adapterName = new ObjectName("HelloAgent:name=htmladapter,port=9092");
            htmlAdaptorServer.setPort(9092);
            mbs.registerMBean(htmlAdaptorServer, adapterName);
            htmlAdaptorServer.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String args[]){
        System.out.println(" hello agent is running");
        HelloAgent agent = new HelloAgent();
    }
}
```


## 二、MXBean
- 一种引用预定义数据类型的MBean
```html
1）通过这种方式，您可以确保任何客户机（包括远程客户机）都可以使用您的MBean，而不需要客户机访问代表MBean类型的特定的类

2）提供一种方便的方法来绑定数据，而不需要客户端进行特殊的绑定操作
```
- @MXBean: 可以用于注解Java接口，这样接口的名称就不必以MXBean为结尾了

### MBean 和 MXBean
```html
1）MXBean并不要求 Java类的名称 必须与 接口名称前部分相同  
2）MXBean中还可以正常使用 自定义数据类型

如果在MBean中使用自定义数据类型的话，通过JConsole查看时会显示不可用
当MXBean中使用被转换成 CompositeDataSupport 类
```
- 标准MBean与MXBean的区别
```html
1.在标准MBean中使用自定义数据类型时，JConsole中会显示不可用；而在MXBean中可以正常所使用

2.标准的MBean接口与Class的命名必须遵守是一定规范，而MXBean的接口与Class的命名没有约束条件
```




## 三、MBean
### 1.MBean 定义
- 1.JavaBean的一种，在JMX中代表一种可以被管理的资源
```html
一组可读和可写属性，或者两者兼而有之
一组调用方法
自我描述
```
- 2.在MBean实例的生命周期中，管理接口都不会发生变化
- 3.MBeans可以在某种预定义的事件发生时发送通知

### 2.MBean 接口
属性（可读的，可能也是可写的）+ 操作（可以由应用程序调用）组成
```html
standard MBean   这种类型的MBean最简单，一个标准的MBean由一个
	            1）MBean 接口(该MBean接口列出了所有被暴露的属性和操作对应的方法)
                  2）和一个 class(这 个class实现了这个MBean接口并提供被监测资源的功能)组成（接口和class必须放在同一个包下，不然会出错）

                 它的命名也必须遵循一定的规范，例如我们的MBean为Hello，则接口必须为HelloMBean。标准MBean只能操作基本数据类型，
                 如 int、dubbo、lang 等

dynamic MBean   必须实现javax.management.DynamicMBean接口，所有的属性，方法都在运行时定义

open MBean      此MBean的规范还不完善，正在改进中

model MBean     与标准和动态MBean相比，你可以不用写MBean类，只需使用javax.management.modelmbean.RequiredModelMBean即可。
                RequiredModelMBean实现了ModelMBean接口，而ModelMBean扩展了DynamicMBean接口，因此与DynamicMBean相似，
                Model MBean的管理资源也是在运行时定义的

	                      与DynamicMBean不同的是，DynamicMBean管理的资源一般定义，在DynamicMBean中（运行时才决定管理那些资源），而model MBean管理的资源并不在MBean中，
		         而是在外部（通常是一个类），只有在运行时，才通过set方法将其加入到model MBean中。后面的例子会有详细介绍
MXBeans
```

### 3.MBean Server
MBean注册的仓库 JMX代理层的核心

### 4.MBean 案例
- MBean 接口：包括一些可读或者可写的属性,还有定义好的方法，可以被MBean管理应用程序调用
```java
public interface HelloMBean {  
    public void sayHello();  
    public int add(int x,int y);  
    public String getName();  
    public void setCacheSize(int size);  
    public int getCacheSize();  
}
```
- MBean 实现类：类名必须为suffixMBean 中的suffix
```java
public class Hello implements HelloMBean {  
    //可读属性 不可写  
    private final String name = "gittudou";  
    private int  DEFUALT_CACHE_SIZE = 200;  
    //可读写属性  
    private int cacheSize = DEFUALT_CACHE_SIZE;  
  
    public void sayHello() {  
        System.out.print("hello jmx");  
    }  
  
    public int add(int x, int y) {  
        return x+y;  
    }  
  
    public String getName() {  
        return this.name;  
    }  
    //使用synchronized 同步控制 防止多个线程同时调用set方法  
    public synchronized void setCacheSize(int size) {  
       this.cacheSize = size;  
        System.out.println("Cache size now "+this.cacheSize);  
    }  
    public int getCacheSize() {  
        return this.cacheSize;  
    }  
}
```
- MBean 在 MBean Server 上注册
```java
public class Main {  
    public static  void  main(String args[]) throws  Exception{  
        //获取MBeanServer  如果没有MBean server存在那么下面会自动调用ManagementFactory.createMBeanServer()  
        MBeanServer mBeanServer = ManagementFactory.getPlatformMBeanServer();  

        //包名加 类名 创建一个ObjectName  
        ObjectName name = new ObjectName("git.tudou.manage.jmx.test.essential:type=Hello");  

        //创建一个Hello实例  
        Hello mbean = new Hello();  

        //在MBean server上注册MBean  
        mBeanServer.registerMBean(mbean,name);  
        System.out.println("Waiting forever...");  

        //线程等待 management 操作   
        Thread.sleep(Long.MAX_VALUE);  
    }
}
```
- 通过jdk jconsole去连接这个本地进程 打开Mbean 点击hello，可使用RMI进行远程连接MBean server, 进行管理和操作
![MBean JConsole](/images/types/jmx/mbean-example.png)

### 5.MBeanUtils
```java
public class JMXUtil {

    /**
     * 注册一个MBean
     */
    public static ObjectName register(String name, Object mbean) {
        try {
            ObjectName objectName = new ObjectName(name);
            MBeanServer mbeanServer = getMBeanServer();
            try {
                mbeanServer.registerMBean(mbean, objectName);
            } catch (InstanceAlreadyExistsException ex) {
                mbeanServer.unregisterMBean(objectName);
                mbeanServer.registerMBean(mbean, objectName);
            }
            return objectName;
        } catch (JMException e) {
            throw new IllegalArgumentException(name, e);
        }
    }

    /**
     * 取消一个MBean
     */
    public static void unregister(String name) {
        try {
            MBeanServer mbeanServer = getMBeanServer();
            mbeanServer.unregisterMBean(new ObjectName(name));
        } catch (JMException e) {
            throw new IllegalArgumentException(name, e);
        }
    }

    public static MBeanServer getMBeanServer() {
        MBeanServer mBeanServer = null;
        if (MBeanServerFactory.findMBeanServer(null).size() > 0) {
            mBeanServer = MBeanServerFactory.findMBeanServer(null).get(0);
        } else {
            mBeanServer = ManagementFactory.getPlatformMBeanServer();
        }
        return mBeanServer;
    }


    /**
     * 生成一个ObjectName，如果出错，返回null
     */
    public static ObjectName createObjectName(String pattern) {
        try {
            return new ObjectName(pattern);
        } catch (Exception ex) {
            // Ignore.
        }

        return null;
    }

    /**
     * 获取类型Pattern列表
     */
    public static ObjectName[] getObjectNames(ObjectName pattern) {
        ObjectName[] result = new ObjectName[0];
        Set<ObjectName> objectNames = getMBeanServer().queryNames(pattern, null);
        if (objectNames != null && !objectNames.isEmpty()) {
            result = objectNames.toArray(new ObjectName[objectNames.size()]);
        }
        return result;
    }

}
```

### 6.MBean 项目实践
```java
// beacon-middleware-framework
@MXBean
public interface PersistMBean {

    String[] getPersistInfo();

}
```






## 四、Spring 与 JMX
### 1.核心功能
- 注册：将任何 Spring bean 自动注册为 JMX MBean
- 界面：一种用于控制 beanManagement 界面的灵活机制
- 暴漏：通过远程 JSR-160 连接器以声明方式公开 MBean
- 代理：本地和远程 MBean 资源的简单代理

### 2.注册行为
Registration behavior | Explanation
--- | ---
FAIL_ON_EXISTING | 这是默认的注册行为。如果MBean实例已经在同一ObjectName下注册，则不注册正在注册的MBean，并抛出InstanceAlreadyExistsException。现有的MBean不受影响。
IGNORE_EXISTING	| 如果MBean实例已经在同一ObjectName下注册，则正在注册的MBean不会被注册。现有的MBean不受影响，并且不会抛出Exception。这在多个应用程序要在共享MBeanServer中共享一个公共MBean的设置中很有用。
REPLACE_EXISTING |如果MBean实例已经在同一ObjectName下注册，那么先前已注册的现有MBean将被取消注册，而新的MBean会在其位置注册(新的MBean有效地替换先前的实例)。

### 3.源级元数据类型
Purpose |	Annotation |	Annotation Type
--- | --- | ---
将Class的所有实例标记为 JMX 托管资源	 | @ManagedResource |	Class
将方法标记为 JMX 操作	 | @ManagedOperation	 | Method
将一个 getter 或 setter 标记为 JMX 属性的一半	 | @ManagedAttribute	 | 方法(仅 getter 和 setter)
定义操作参数的描述 |	@ManagedOperationParameter和@ManagedOperationParameters	 | Method

### 4.Annotation
- @ManagedResource  : 将类的所有实例标识为JMX受控资源 -- Class 类
- @ManagedAttribute  : 将getter或者setter标识为部分JMX属性 -- Method (only getters and setters) 
- @ManagedOperation : 将方法标识为JMX操作 -- Method方法
- @ManagedOperationParameter 和 @ManagedOperationParameters -- Method方法
用途 Commons Attributes属性 JDK 5.0注解 属性/注解类型

### 5.MBeanExporter
Spring-JMX 核心类： 获取您的 Spring bean 并向 JMX MBeanServer注
```java
package org.springframework.jmx;

import org.springframework.jmx.export.annotation.ManagedResource;
import org.springframework.jmx.export.annotation.ManagedOperation;
import org.springframework.jmx.export.annotation.ManagedAttribute;

@ManagedResource(
        objectName="bean:name=testBean4",
        description="My Managed Bean",
        log=true,
        logFile="jmx.log",
        currencyTimeLimit=15,
        persistPolicy="OnUpdate",
        persistPeriod=200,
        persistLocation="foo",
        persistName="bar")
public class AnnotationTestBean implements IJmxTestBean {

    private String name;
    private int age;

    @ManagedAttribute(description="The Age Attribute", currencyTimeLimit=15)  // 仅标记get() 只读
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @ManagedAttribute(description="The Name Attribute",
            currencyTimeLimit=20,
            defaultValue="bar",
            persistPolicy="OnUpdate")
    public void setName(String name) {
        this.name = name;
    }

    @ManagedAttribute(defaultValue="foo", persistPeriod=300)
    public String getName() {
        return name;
    }

    @ManagedOperation(description="Add two numbers")   // 暴漏操作
    @ManagedOperationParameters({
        @ManagedOperationParameter(name = "x", description = "The first number"),
        @ManagedOperationParameter(name = "y", description = "The second number")})
    public int add(int x, int y) {
        return x + y;
    }

    public void dontExposeMe() {
        throw new RuntimeException();
    }

}
```




































