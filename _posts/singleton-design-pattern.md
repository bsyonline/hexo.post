---
title: Singleton Design Pattern
date: 2016-08-02 00:00:47
tags:
 - Interview
category: 
 - Design Pattern
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

单例模式是最简单的设计模式之一，用于保证系统中只有一个类的实例的场景。

单例模式的特点：
* 只有一个实例
* 只能自己创建自己的实例
* 能够对外提供自己创建的实例


### 1. 程序示例
#### 1.1 懒汉式

在 Gof 的设计模式中介绍单例模式使用的说明代码叫做懒汉式，java 版代码如下：
```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```
这是最简单的代码实现，但是这个代码存在问题，即在多线程的程序中，不能保证一定是单例的。原因也很简单，代码被编译成字节码被执行，有可能在刚进入 if 判断后时间分片就用完了，另一个线程也进入 if 判断，从而创建多个实例。一种简单的处理方法是将方法同步。
```java
synchronized public static Singleton getInstance() {
}
```



#### 1.2 饿汉式

另外还有一种比较常见的叫做饿汉式，这种方式更符合 java 的风格。
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

#### 1.3 静态内部类
通常情况下，直接使用恶汉式是最安逸的，况且内存越来越不是瓶颈了。但是如果还对装载类时初始化耿耿于怀，可以使用这种方式。
```java
public class Singleton {

    private Singleton() {}

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
使用静态内部类的好处是不会在装载 Singleton 的时候就实例化，而是在调用 getInstance() 方法时才创建实例。

#### 1.4 枚举
在 jdk1.5 以后加入了枚举，也可以使用枚举来实现单例。
```java
public enum Singleton {
    INSTANCE;
    public void whateverMethod(){

    }
}
```
使用 enum 方式创建单例的好处是简单，简单到让人看不懂。怎么使用可参考一个例子 [枚举单例读取配置文件](../../../../2016/08/02/枚举单例读取配置文件/) 。



#### 1.5 双重校验锁
网上很多单例的文章中都提到了双重校验锁的方式。这种方式出现的目的是对懒汉式进行优化。
```java
public class Singleton {

    private volatile static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
在《java 与模式》一书中针对这种方式也做了分析，这种方式并不能真正达到效果，所以还是安心使用饿汉吧。

2016-08-08 补充：
网上一些资料中提到，自 JDK 1.5 以后，修复了 volatile 关键字的 bug ，所以，JDK 1.5 以后双重校验锁可以正常工作了。关于 volatile 关键字更多知识点，参考 [Java 关键字 volatile](../../../../2016/08/08/Java 关键字 volatile/)

### 2. 类图
如果要画出常用设计模式的类图，单例无疑是最简单的了。

![单例模式](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/singleton.png)

