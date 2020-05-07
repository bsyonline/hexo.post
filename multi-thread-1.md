---
title: 多线程
date: 2015-03-13 15:53:17
tags:
 - Interview
category: 
 - Concurrent
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
编写一个程序，开启3个线程，这3个线程的ID分别为A、B、C，每个线程将自己的ID在屏幕上打印10遍，要求输出结果必须按ABC的顺序显示；如：ABCABC….依次递推。
```java
public class JoinTest {

    class A extends Thread {
        public void run() {
            System.out.print("A");
        }
    }

    class B extends Thread {
        public void run() {
            System.out.print("B");
        }
    }

    class C extends Thread {
        public void run() {
            System.out.print("C");
        }
    }

    public void run(Thread thread) throws InterruptedException {
        Thread t = new Thread(thread);
        t.start();
        t.join();
    }

    public Thread get(int i) {
        Thread t = null;
        switch (i % 3) {
            case 0:
                t = new A();
                break;
            case 1:
                t = new B();
                break;
            case 2:
                t = new C();
                break;
        }
        return t;

    }

    public void print() throws InterruptedException {
        for (int i = 0; i < 30; i++) {
            run(get(i));
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new JoinTest().print();
    }
}
```
