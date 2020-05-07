---
title: AspectJ 和 Spring AOP 科普
date: 2017-04-01 14:29:56
tags:
 - AOP
 - AspectJ
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "AOP （面向方面编程） 是一种编程思想，是对 OOP 的有力补充。AspectJ 是对 Java 在 AOP 编程上的扩展。Spring AOP 是 Spring 提供的 AOP 编程框架。"
---


AOP （面向方面编程） 是一种编程思想，是对 OOP 的有力补充。AspectJ 是对 Java 在 AOP 编程上的扩展。Spring AOP 是 Spring 提供的 AOP 编程框架。



在接触到 AOP 的时候用的就是 Spring AOP ， Spring 也提供了 AspectJ 的支持，可以在 Spring AOP 中使用 AspectJ 的语法，但 Spring AOP 和 AspectJ 还是有区别的。

AspectJ 功能强大到令人发指（虽然没用过），并且是在编译期就对代码进行织入，所以速度快。但是 AspectJ 需要使用特殊编译器编译，学习门槛较高。
Spring AOP 功能上有所限制（1. 相同类中的方法不能互相调用；2. 只能在 Spring 体系的类中使用；3. 粒度到方法级。），但是使用起来比 AspectJ 简单，采用动态织入，所以不需要专门的 AspectJ 编译器来编译，但速度慢于 AspectJ 。


在使用 AOP 的过程中，一般会涉及以下几个概念。

*    Aspect: a modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented using regular classes (the schema-based approach) or regular classes annotated with the @Aspect annotation (the @AspectJ style).
*    Join point: a point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.
*    Advice: action taken by an aspect at a particular join point. Different types of advice include "around," "before" and "after" advice. (Advice types are discussed below.) Many AOP frameworks, including Spring, model an advice as an interceptor, maintaining a chain of interceptors around the join point.
*    Pointcut: a predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.
*    Introduction: declaring additional methods or fields on behalf of a type. Spring AOP allows you to introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an IsModified interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)
*    Target object: object being advised by one or more aspects. Also referred to as the advised object. Since Spring AOP is implemented using runtime proxies, this object will always be a proxied object.
*    AOP proxy: an object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy will be a JDK dynamic proxy or a CGLIB proxy.
*    Weaving: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime. 



以上概念抄自 [https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html) 。

AspectJ 有专门的 Java 编译器，可在官网下载 [https://eclipse.org/aspectj/downloads.php#stable_release](https://eclipse.org/aspectj/downloads.php#stable_release) 。AspectJ 也有集成开发环境，最常见的比如 AJDT 是 eclipse 的插件。IDEA 官方没有插件，具体设置参考 [https://www.jetbrains.com/help/idea/2017.1/aspectj.html](https://www.jetbrains.com/help/idea/2017.1/aspectj.html) 


这里不涉及细节，只是科普，所以具体细节可参考：
1. [跟我学 aspectj](http://blog.csdn.net/zl3450341/article/details/7673938) 
2. [使用 IDEA 开发 AspectJ 程序](../../../../2017/04/10/使用-IDEA-开发-AspectJ-程序/)
3. [Spring AOP 开发](../../../../2017/04/10/Spring-AOP-开发/)

