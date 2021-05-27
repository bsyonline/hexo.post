---
title: Escape Analysis
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2020-03-27 09:39:40
thumbnail:
---

### 什么是逃逸分析

逃逸分析是 JVM 中使用的一种优化技术。逃逸分析算法是利用连通图来构建对象和引用对象之间的可达性，以此进行数据流分析的方法，在论文 Escape Analysis for Java 中有介绍。

### 逃逸的种类

逃逸分析针对的对象的分析，对象的逃逸状态有 3 种：

1. GlobalEscape ，全局逃逸
当对象符合以下条件，认为是全局逃逸：
	1）对象作为方法的返回结果。
	2）存储在全局变量或静态变量中的对象。
	3）重写了 finalize() 方法的对象。
2. ArgEscape ，参数逃逸
对象作为参数传递或被参数引用，但是没有发生全局逃逸。
3. NoEscape 没有逃逸

### 优化方式

利用逃逸分析，JVM 可以对代码进行优化：

1. 栈上分配
如果对象没有逃逸，可以将对象分配在方法的栈上而不用在堆中创建对象，这样当方法执行完，栈帧弹出，对象就自动被回收，可以加快内存回收，减少 GC 。
2. 锁消除
当 JVM 认为一个对象只在当前线程内部使用，那么就会优化掉对象的同步锁。
```
public static void m1() {
	synchronized (new Object()) {
		System.out.println("m1");
	}
}
public static void optimizedM1() {
	System.out.println("m1");
}
```
上边的例子只是为了说明，因为实际中我们应该不会对 new Object() 进行加锁。实际中最常见的还有在一个方法内部使用 StringBuffer.append() ，StringBuffer 的方法是 synchronized ，如果对象没有逃逸，JVM 会帮我们优化掉 synchronized 。
3. 标量替换
标量不能再拆分，比如基本类型，聚合量可以被拆分，比如对象。将对象分解成标量，使用标量代替对象就叫做标量替换。如果对象没有逃逸，就可以用标量代替，变量只存储在栈上，就不用在堆中创建对象，既减少了内存占用又提高了运行速度。
```
class Point {
	int x;
	int y;
}
public void optimizedShow() {
	int x = 1;
	int y = 2;
	System.out.println("point=(" + x + ", " + y + ")");
}
public void show() {
	Point point = new Point();
	point.x = 1;
	point.y = 2;
	System.out.println("point=(" + point.x + ", " + point.y + ")");
}
```
4. 锁粗化
加锁操作是很消耗资源的，如果在循环内部进行多次，会将锁的范围扩大到循环外。
```
public static void m1() {
	for (int i = 0; i < 5_000_000; i++) {
		synchronized (new Object()) {

		}
	}
}
public static void optimizedM1() {
	synchronized (new Object()) {
		for (int i = 0; i < 5_000_000; i++) {

		}
	}
}
```