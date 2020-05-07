---
title: 死锁
date: 2016-09-14 17:37:05
tags:
 - Interview
category: 
 - Concurrent
thumbnail: 
author: bsyonline
lede: "死锁是2个以上的线程由于竞争同一资源形成阻塞。"
---

死锁是2个以上的线程由于竞争同一资源形成阻塞。

### 如何写一个死锁
最后一个未阻塞的线程在调用 Condition.signal() / Condition.signalAll() / Object.notify() / notifyAll() 之前调用了 Condition.await() 或 Object.wait() 或调用了 Condition.signal() / Object.notify()，被通知的线程没有达到执行条件，又阻塞了。
```java
class DeadLock implements Runnable {
    ReentrantLock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    @Override
    public void run(){
        try{
            lock.lock();
            condition.await();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args){
        DeadLock deadlock = new DeadLock();
        Thread t1 = new Thread(deadlock);
        Thread t2 = new Thread(deadlock);
        t1.start();
        t2.start();
    }
}
```

使用 jstack 工具查看：
```
"Thread-1" #13 prio=5 os_prio=0 tid=0x00007f5b4815d000 nid=0x39c5 waiting on condition [0x00007f5b2621b000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000000d8093868> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
	at com.chinadaas.riskbell.tools.DeadLockTest.run(DeadLockTest.java:20)
	at java.lang.Thread.run(Thread.java:745)

"Thread-0" #12 prio=5 os_prio=0 tid=0x00007f5b4815b000 nid=0x39c4 waiting on condition [0x00007f5b2631c000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000000d8093868> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
	at com.chinadaas.riskbell.tools.DeadLockTest.run(DeadLockTest.java:20)
	at java.lang.Thread.run(Thread.java:745)

```

如果线程是可以被打断的，可以使用程序打断死锁。(程序作为示例，执行时未达到效果。)
```java
public void DeadlockChecker(){
    new Thread(new Runnable() {
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        @Override
        public void run() {
            while (true) {
                long[] deadlockThreadIds = threadMXBean.findDeadlockedThreads();
                if (deadlockThreadIds != null) {
                    ThreadInfo[] threadInfos = threadMXBean.getThreadInfo(deadlockThreadIds);
                    for (Thread t : Thread.getAllStackTraces().keySet()) {
                        for (int i = 0; i < threadInfos.length; i++) {
                            System.out.println(threadInfos[i].getThreadId());
                            if (t.getId() == threadInfos[i].getThreadId()) {
                                t.interrupt();
                            }
                        }
                    }
                }
            }
        }
    }).start();
}
```
