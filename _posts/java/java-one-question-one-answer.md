---
title: Java 一问一答
date: 2017-07-05 17:24:43
tags:
 - Java
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

#### Q： Spring Data 的 repository 只定义了接口，没有实现，是如何完成数据访问操作的？

A： 这是通过 Java 编译器的衍生机制实现的。衍生机制基于 Java 6 的注解处理工具 APT ，它能够以编码的方式探查具有特殊注解的代码，然后调用函数生成查询元模型类。