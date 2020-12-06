---
layout: post
title: UML 介绍 和 8 种类关系
categories: [系统设计, UML]
excerpt: United Modeling Language 统一建模语言
description: 用来设计软件蓝图的可视化建模语言
keywords: UML, Diagram, Relationships, 系统设计
topmost: false
---

### 为啥要学 UML 和 类关系?

了解和熟悉 Java 类之间可能存在的关系，有助于更好的理解类如何组织，如何进行业务模型编排。

学习 UML 有助于使用这种工具来辅助工作，特别是遇到复杂系统设计与构建时候，尤其必要。


#### 一、UML *Unified Modeling Language*：统一建模语言  

用来设计软件蓝图的可视化建模语言。  
描述了一个系统的静态结构和动态行为。  
它支持面向对象系统的分析、设计、实现和 交付等各个环节，可以用于系统的理解、设计、浏览 、维护和信息控制。

包含如下项  
**事物**是对模型中最具有代表性的成分的抽象；**关系**把事物结合到了一起；**图**聚集了相关的事物。

```
事务：    结构 行为 分组 注释
	构成模型图的一些基本图示符号，它们表示一些面向对象的基本概念

关系：    依赖 关联-聚合组合 泛化 实现
	表示基本图示符号之间的关系

图：      用例 交互 类 活动 状态
	特定的视角对系统所作的抽象描述

扩展机制： Stereotype Tagged Value Constraint
工具：    用例 逻辑 组件 部署
```

**用途**  

```
用例 - 领域OOA业务 - 交互OOD动态 - 设计类图
草图 - 蓝图 - 编程语言
图形 - 过程 - 工具

UP  
SDP software development process - 软件开发过程
USD Unified Softawre Develpment - 统一软件开发
RUP Rational Unified Process - 统一软件开发

构架 - 用例驱动 - 迭代&增量 -> 不同于瀑布模型
XP Extreme Programing 测试/重构/持续集成
```


#### 二、从下面这张图里，找找总共有多少种关系

![uml-relationships](/images/posts/2016-05-31-uml-relationship/uml-relationships.png)



#### 三、Java 类 8 种关系介绍 -- Class Diagram

![1607154170885](/images/posts/2016-05-31-uml-relationship/1607154170885.png)

1. Association 关联  
   解析：两个类以任何形式互相连接，称互相有联系 （实线连接）
   备注：从某种角度说，世间万物都是关联的，但在设计系统的设定边界内，定义恰当的关联关系，有助于对系统的理解和规划。  
   案例：Bank 和 Account 存在 Register 的关联关系  
   代码体现：成员变量
   ![1607145911427](/images/posts/2016-05-31-uml-relationship/1607145911427.png)
2. Directed Association / Dependency 定向关联  
   解析：类之间的关联是双向的，可以用箭头表示包含关系  
   案例：Bank 里面有 Account，称为 B 依赖于 A  
   代码体现：局部变量、方法参数、静态方法调用  
   ![1607146081726](/images/posts/2016-05-31-uml-relationship/1607146081726.png)
3. Reflexive Association 反身关联  
   解析：当一个类有许多不同类型的职责时，就会形成反身关联。  
   案例：公司的员工可以是执行官、助理经理或首席执行官。但是，这里的关系是不能用同一个符号的。
4. Multiplicity 1~n / n~1 关联关系
   解析：一对多、多对一关联关系，注意它和组合 / 聚合是有区别的  
   案例：Bank 注册了 很多 Account，一般用*标注多的一方  
   ![1607147675600](/images/posts/2016-05-31-uml-relationship/1607147675600.png)
5. Aggregation 聚合  
   解析：一群大雁聚合成雁群，放到一起  
   代码体现：成员变量  
   ![1607148148928](/images/posts/2016-05-31-uml-relationship/1607148148928.png)
6. Composition 组合  
   解析：翅膀 + 腿 +  其它 组合成鸟  
   备注：组合 和 聚合 以及 1对多关联比较难区分，日常开发种可以多留心   
   代码体现：成员变量    
   ![1607148963736](/images/posts/2016-05-31-uml-relationship/1607148963736.png)
7. Generalization / Inheritance 泛化 / 继承  
   解析：父子关系，一般是为了元素可重用  
   ![1607151047159](/images/posts/2016-05-31-uml-relationship/1607151047159.png)
8. Realization 实现  
   解析：定义一组共功能、合约，其它类来进行实现，一般说的就是接口的实现啦  
   ![1607151155251](/images/posts/2016-05-31-uml-relationship/1607151155251.png)



#### 推荐使用的软件 StartUML、Rose 等

- UML  
  ![未命名图片](/images/posts/2016-06-06-uml-diagram/未命名图片.png)



## 参考：

[Class Diagram 8种类关系解析](https://www.linkedin.com/pulse/guide-uml-class-diagram-relationships-amanda-athuraliya/)

