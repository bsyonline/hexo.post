---
title: synchronized
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-02-20 09:36:42
thumbnail:
---


synchronized 是 java 关键字，是最常见多线程同步方式，可以作用于方法和代码块。
```
// 同步方法
synchronized void method(){

}

// 同步代码块
synchronized (this) {

}
```
不论是同步方法还是同步代码块，在 JVM 中都是通过管程 (Monitor) 来支持的。同步方法是隐式的，通过 ACC_SYNCHRONIZED 访问标识来声明。同步代码则是在 javac 编译器在字节码中添加 monitorenter 和 monitorexit 指令来实现的。无论正常退出还是异常结束， monitorenter 都必须有 monitorexit 与之对应。
synchronized 具有以下特点：
1. **非公平锁**
2. **可重入**：是指在如果一个线程获得了锁进入了 synchronized 方法，那么该方法内部调用的其他方法可以直接获得该对象的锁。
3. **不可中断**：是指一旦一个线程获得了锁，只有执行完成或抛出异常才能释放，执行过程中其他线程只能等待，直到获得锁的线程释放锁之后其他线程才能获取锁。

synchronized 使用使用时需要注意以下几点：

1. synchronized 代码块的作用的对象不能为空。因为 synchronized 的信息是保存在对象的头信息中，所以作用的对象必须进行实例化。
2. synchronized 虽然简单，但是效率不高，所以在使用是尽可能保证 synchronized 的作用域不能过大。
3. 要注意避免死锁