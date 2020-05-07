---
title: Java 关键字 volatile 详解
date: 2016-08-08 11:39:01
tags:
 - Interview
category: 
 - Concurrent
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

volatile 是 Java 语言的关键字，常常和 synchronized 进行比较。

**volatile 和 synchronized 简单比较**

<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;width:100%"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">volatile</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">synchronized</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">作用位置</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">变量</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">方法，代码块</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">同步对象</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">主内存和线程内存之间某个变量的值</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">主内存和线程内存之间所有变量的值</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">消耗资源</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">少</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">多</td></tr></table>


volatile 不能像 synchronized 一样广泛的用于线程安全，因为 volatile 不能保证原子性，所以 volatile 不能保证线程安全。但是在某些特殊场景下使用 volatile 要比 synchronized 和锁简单和高效，还能使程序更加简单。
##### **内存可见性**
Java 的每个线程都拥有自己的内存，在某个时间点，多个线程中间的同一个变量的值可能是不同的，volatile 的作用就是让变量对所有线程都是一致的，每次获得的都是该变量的最新值，即可见性。比如程序需要有一个标识来指示一个一次性操作，像资源初始化，那么使用 volatile 是非常方便的。
```java
public class VolatileExample {
    volatile boolean init = false;

    public boolean isInit() {
        return init;
    }

    public void setInit(boolean init) {
        this.init = init;
    }

    public static void main(String[] args) throws InterruptedException {
        VolatileExample volatileExample = new VolatileExample();
        new Thread(() -> {
            while (!volatileExample.isInit()) {
            }
            System.out.println("t2 completed");
        }, "t2").start();

        Thread.sleep(5000);
        volatileExample.setInit(true);
        System.out.println("init = true");
    }
}
```
volatile 变量比使用 synchronized 代码要简单一些，在这种只有一种状态转换的情况使用 volatile 是合适的。
##### **禁止指令重排**

volatile 的另一个作用是禁止指令重排。JVM 为了提高程序的执行效率，可能会对没有前后依赖关系的程序指令进行优化，从而改变执行的顺序。比如双重校验单例的代码：

```
public class Singleton {
	private static volatile Singleton instance = null;
	
	public static Singleton getInstance() {
		if (instance == null) {
			sychornized(Singleton.class) {
				if (instance == null) {
					instance = new Singleton(); 
				}
			}
		}
		return instance;
	}
}
```
instance = new Singleton() 并不是原子操作，它对应了 3 条 JVM 指令：
```
memory = allocate();   //1：分配对象的内存空间 
ctorInstance(memory);  //2：初始化对象 
instance = memory;     //3：设置instance指向刚分配的内存地址
```
由于 2 和 3 并没有依赖顺序，所以如果 JVM 对其进行指令重排，顺序可能会变成 1 3 2 。如果在多线程的场景中，就可能出现 instance 不为空，但是还没有初始化完成的情况，所以为了避免出现此种问题，可以使用 volatile 来禁止指令重排。
在使用 volatile 关键字修饰变量后，JVM 会通过内存屏障来保证指令的顺序。
1. 在每个 volatile 写操作前插入 StoreStore 屏障，在写操作后插入 StoreLoad 屏障。
2. 在每个 volatile 读操作前插入 LoadLoad 屏障，在读操作后插入 LoadStore 屏障。
