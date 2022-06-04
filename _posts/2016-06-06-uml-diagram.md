---
layout: post
title: UML = 关系(Relationships) + 关系图(Diagrams)
comments: true,
categories: [系统设计, UML]
excerpt: 用例图、时序图、类图 ...
description: 看图 ==> 会分析图 ==> 会画图 ==> 会讲图 ==> 会转化落地为项目
keywords: UML, Diagram, Relationships, 系统设计
topmost: false
---

### 为啥要学 UML 和 关系图?



了解和熟悉 Java 类之间可能存在的关系，有助于更好的理解类如何组织，如何进行业务模型编排。

学习 UML 有助于使用这种工具来辅助工作，特别是遇到复杂系统设计与构建时候，尤其必要。



####  一、关系图9种类型

![1607154278486](/images/posts/2016-06-06-uml-diagram/1607154278486.png)

####  二、来学图吧

##### 如何学会图：

先会看图 ==> 会分析图 ==> 会画图 ==> 会讲图 ==> 会转化落地为项目

- [ ] 后续如果可行期间内，将自己设计的系统的相关图稿共享出来，文末有共享的资源可查看


##### 1.用例图 Use Case Diagram

参与者+用例 = 职责功能  
用动词来表示 用户观点来描述 多用用户行业语言  
用例编号 参与者 用例描述 参与者 前置条件 后置条件    

基本规则：

- 描述参与者与系统间的**交互**  
- 参与者动作 数据的传输 系统响应--原则  
- 基本路径里不要有：如果。(放到扩展点)

![usecase1](/images/posts/2016-06-06-uml-diagram/usecase1.png)

![usecase2](/images/posts/2016-06-06-uml-diagram/usecase2.png)

如何发现UC:

- 选择**系统边界**--确定主要**参与者**

- 确定参与者**目标** -- 定义满足目标的用例

- 根据目的定UC-NAME UC密度(少于10-15步)

思维习惯：-- 发现用例的方法

- 多少部门 多少岗位 

- 组织架构 权利关系 职责划分 ？ 

- 问平时都做什么 谁交代的 需要通知或者传达给谁 填写什么样的表格？等等的需求问题。  

用例间联系：

-   **包含**关系-依赖 避免用例重复编写


-   **扩展**关系--可选路径一般不必要


-   **泛化**关系--用例间行为 结构 目的 相似过多

编写用例文本比关注用例关系更重要



##### 2.设计类 

###### 2.1 类图 Class Diagram  -- 

- 设计--**低耦合** **高内聚**--减少依赖不稳定对象  

- 依赖--使用或者包含的对象：传递性--包含

- 关联--一段时间内两对象会产生联系--引用-交集

关联名 导航 角色-位置？ 多重性 **关联<-聚合<-组合**  类中属性已经建模为关联了对应另一类就默认有了  
--关联建立树形结构

- 类分析：实体类、控制类、边界类 等
- 概念模型：领域模型 OO 分析 Domain Model

![classDiagram](/images/posts/2016-06-06-uml-diagram/classDiagram.png)



###### 2.2 对象图 Object Diagram

- 解析：在某一时刻类的对象静态结构和行为，以及对象间的关系  
  Object-D 可看作 Class-D 的特殊用例  

- 核心概念：对象、链接、多重性  
  ![1607158860954](/images/posts/2016-06-06-uml-diagram/1607158860954.png)

- 用途：系统原型、逆向工程、复杂数据结构、瞬态图、捕捉实例

##### 3.进程类 / 交互类

###### 3.1 时序图 Sequence Diagram

- 7种元素：

  - 角色(Actor)：人 / 子系统 等
  - 对象(Object)：类 / 对象
  - 生命线(LifeLine)：对象生命
  - 控制焦点(Activation)：对象某段生命种执行的操作
  - 消息(Message)：同步 / 异步 / 返回 的消息
  - 自关联消息：调用其它 / 自身调用
  - 组合片段

- 以下是 Oath2 场景的时序图

  ![1607159998166](/images/posts/2016-06-06-uml-diagram/1607159998166.png)

- 这里有另一个时序图 [Blog](https://blog.csdn.net/fly_zxy/article/details/80911942) 

  ![img](/images/posts/2016-06-06-uml-diagram/20180704155415694.jpg)

- 其它一些时序图，时序图非常重要，可以明确流程信息，避免扯淡

![1](/images/posts/2016-06-06-uml-diagram/1.png)

![2](/images/posts/2016-06-06-uml-diagram/2.png)

###### 3.2 协作图 Collaborative Diagram

- 交互图的一种，描述了收发消息的对象的组织关系，强调对象之间的合作关系。
- 时序图按照时间顺序布图，而协作图按照空间结构布图

![1607160575343](/images/posts/2016-06-06-uml-diagram/1607160575343.png)

###### 3.3 状态图 State Diagram

- 描述类的对象所有可能的状态以及时间发生时状态的转移条件
- 由**状态**、**变迁**、**事件**和**活动**组成的状态机
- 图符：状态、转移、起点和终点
  - 1）名称 name
  - 2）进入协作和退出动作 entry action/exit action
  - 3）内部转换 internal transition 
  - 4）子状态 substate
  - 5）延迟事件 deferred event

![StateChart1](/images/posts/2016-06-06-uml-diagram/StateChart1.png)

![StateChart2](/images/posts/2016-06-06-uml-diagram/StateChart2.png)

![1607160980165](/images/posts/2016-06-06-uml-diagram/1607160980165.png)

![1607161001421](/images/posts/2016-06-06-uml-diagram/1607161001421.png)

###### 3.4 活动图 Activity Diagram

- 同步条 -- 成对 泳道 对象流

- 描述了活动到活动的控制流，是一种表述过程基理、业务过程以及工作流的技术
- 交互图强调的是对象到对象的控制流，而活动图则强调的是从活动到活动的控制流

- 用来对业务过程、工作流建模，也可以对用例实现甚至是程序实现来建模

  [活动图 Blog](https://www.cnblogs.com/ywqu/archive/2009/12/14/1624082.html)

  ![1607161510372](/images/posts/2016-06-06-uml-diagram/1607161510372.png)

![1607161583608](/images/posts/2016-06-06-uml-diagram/1607161583608.png)

##### 4.构件图 Component 

- 表示系统中构件与构件之间，类或接口与构件之间的关系图
- 目的是在软件系统中遵从并实现一组接口的物理的、**可替换**的软件模块
- 构建图之间的关系表现为**依赖**关系，定义的类或接口与类之间的关系表现为依赖关系或实现关系。
- = 构件（**Component**）+接口（**Interface**）+关系（**Relationship**）+端口（**Port**）+连接器（**Connector**）

![1607162539519](/images/posts/2016-06-06-uml-diagram/1607162539519.png)

5. ##### Deployment 部署图

   - 描述系统运行时进行处理的**结点**以及在结点上活动的**构件的配置**。强调了物理设备以及之间的**连接关系**。

![1607162781771](/images/posts/2016-06-06-uml-diagram/1607162781771.png)

## 参考

[UML各种图总结](https://www.cnblogs.com/wuyuxin/p/7001561.html)

[UMK系统学习](https://www.w3cschool.cn/uml_tutorial/uml_tutorial-mi5w28ur.html)

[UML学习资源 百度网盘 dv36](https://pan.baidu.com/s/1adLJGDXmrYFBfW11izZGZw)
