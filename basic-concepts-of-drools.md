---
title: basic concepts of drools
tags:
  - Drools
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-06-02 23:29:18
thumbnail:
---



##### 什么是 KIE
KIE 是 jboss 的一些通用组件的集合，包括 drools， jbpm 等。

##### KieServices
KieServices 是程序入口，通过 KieServices 来获取规则构建，管理，执行需要的对象。通过 KieServices 创建 KieContainer ，由 KieContainer 创建 KieSession ， 

##### KieContainer
由 KieServices 获得，是存放 KieBase 的容器。

##### KieBase 
KieBase 是知识库，包含规则和方法。创建成本很高。

##### KieSession
由 KieBase 创建，KieBase 只有规则和方法，并不包含运行数据，KieSession 包含运行数据和事实（fact），负责数据按照规则进行运算。KieSession 是程序和规则引擎的会话，创建成本低，可反复创建。

##### KieRepository
KieRepository 是存放 KieModule 的仓库，单例对象。

##### KnowledgeBuilder
用来在业务代码当中收集已经编写好的规则，然后对这些规则文件进行编译，最终产生一批编译好的规则包（KnowledgePackage）给其它的应用程序使用。

##### KnowledgeBase
收集应用当中 knowledge 定义的知识库对象，包含普通规则，规则流，函数定义，对象模型等。

##### KnowledgeBaseConfiguration
用来存放规则引擎运行的环境参数。

##### Stateful 和 Stateless
Stateful 对象会和规则引擎建立一个持续的交互通道，多次触发同一数据集，使用完成要调用 ```dispose()``` 方法释放资源。
Stateless 对象内部处理 ```dispose```， 不用用户处理。

##### Fact
一个普通的 JavaBean 插入到规则的 WorkingMemory 当中后被称为 Fact 。Fact 是 JavaBean 的引用，规则引擎可以操作 Fact ，运行数据存放在 Fact 中。Fact 是规则引擎和应用的数据载体。

##### Agenda
规则和 fact 在调用 ```fireAllRules()``` 之前都存放在 Agenda 对象中，存放在 Agenda 的对象被称为 Activation 。

