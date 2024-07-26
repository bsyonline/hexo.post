---
title: one question one answer for jvm
tags:
  - JVM
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-11-04 22:52:39
thumbnail:
---



#### Q: 写一段将目录中指定的.class文件加载到JVM的程序，并通过Class对象获取到完整类名等信息。

A: 通常会继承 URLClassLoader ，重写 findClass 方法，通过 ByteArrayOutputStream 读取文件，通过反射执行。也可以重新loadClass 方法，但是会破坏双亲委托机制。

#### Q: 一段展示代码，里面包含一个全局静态整型变量，问如果用两个ClassLoader加载此对象，执行这个整型变量++操作后结果会是怎么样的？

A:  

#### Q: 自己写一个 String 类能否替换掉 jdk 的 String 类？

A: String 类在 rt.jar 中，是由 BootstrapClassLoader 来加载的，所以即使我们自己写自定义加载器来加载 String 类，由于双亲委托机制，最后都会先交给 BootstrapClassLoader 来加载，所以自己写的 String 是无法加载的。（除非修改 BootstrapClassLoader 的源码，猜的。）

#### Q: A a=new A(); a.execute(); 和 IA a=new A(); a.execute(); 执行有什么不同?

A: 多态属性运行时绑定，会调用父类的方法。

#### Q: 反射的性能低的原因是？

A: 

#### Q: 编写一段程序，动态的创建一个接口的实现，并加载到JVM中执行；（可以允许用BCEL等工具）

A:

#### Q: 经典的String比较程序题：

   String a="a";
   String b="b";
   String ab="ab";
   (a+b)==ab;  ??  (引深题，如何才能让(a+b)==ab）
   ("a"+"b")==ab; ?? 

A:

#### Q: 写一段程序，让其OutOfMemory，或频繁执行Minor GC，但又不触发Full GC，又或频繁执行Full GC，但不执行minor GC，而且不OutOfMemory，甚至可以是控制几次Minor GC后发生一次Full GC；

A: 

#### Q: 详细讲解GC的实现，例如minor GC的时候导致是怎么回收对象内存的，Full GC的时候是怎么回收对象内存的。

A: 

#### Q: i++的执行过程

A: 

#### Q: 一个线程需要等待另外一个线程将某变量置为true才继续执行，如何编写这段程序，或者如何控制多个线程共同启动等；

A: 

#### Q: 控制线程状态的转换的方法，或者给几个thread dump，分析下哪个线程有问题，问题出在哪。

A: 

#### Q: static属性加锁、全局变量属性加锁、方法加锁的不同点？

A: 