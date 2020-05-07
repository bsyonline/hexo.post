---
title: LockSupport
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-06 21:35:10
thumbnail:
---

在 java.util.concurrent 中大量使用了 LockSuport 来阻塞和唤醒线程。LockSupport 的 park/unpark 都是通过 Unsafe 类来实现的。
```
// 用于 debug 
public static void park(Object blocker) {
	Thread t = Thread.currentThread();
	setBlocker(t, blocker);
	UNSAFE.park(false, 0L);
	setBlocker(t, null);
}

public static void park() {
	UNSAFE.park(false, 0L);
}

public static void unpark(Thread thread) {
	if (thread != null)
		UNSAFE.unpark(thread);
}
```
LockSupport 有一个 permit 的概念，park 时，permit 置为 0 ，unpark 或 interrupt 时，permit 置为 1 。
```
public class LockSupportExample {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            LockSupport.park();
            LockSupport.park();
            LockSupport.park();
        });
        t1.start();
        Thread.sleep(3000);
        LockSupport.unpark(t1);
        Thread.sleep(3000);
        Thread.interrupted();
    }
}
```
如果同一个线程调用多次 park() 第一次执行完线程会阻塞，第二次不会执行，直到 unpark() 或 interrupt 之后 permit 恢复，才会执行后续的 park() 。