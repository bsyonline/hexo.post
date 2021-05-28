---
title: Lock
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-02-20 09:55:37
thumbnail:
---



### Java 锁的种类

#### 1. 公平锁

多个线程按照申请锁的顺序获得锁，效率不高，通过 new ReentrantLock(true) 可以创建公平锁。

#### 2. 非公平锁

多个线程不是申请锁的顺序获得锁，后申请的有可能先获得锁。性能比公平锁高，会有优先级低的线程长时间无法获得锁的情况。synchornized 是非公平锁，ReentrantLock 默认也是非公平锁。

#### 3. 可重入锁/递归锁

如果一个线程获得了锁，那么该方法内部调用的其他方法可以直接获得该对象的锁，不需要重新申请锁。

#### 4. 自旋锁

线程获取锁失败后不会阻塞，而是通过循环再次尝试获取锁。自旋可以减少线程上下文切换，但是会消耗 CPU 资源，典型的用法如 CAS 。

#### 5. 读锁/写锁

读读可以共存，读写、写写互斥。

#### 6. 共享锁

一个锁可以被多个线程占有。

#### 7. 独占锁

一个锁只能被一个线程占有，synchornized 和 ReentrantLock 都是独占锁。

#### 8. 悲观锁

认为在一个线程操作数据的时候，其他线程也会同时进行操作，如果不加锁就一定会出问题，Java 中的锁就是悲观锁。

#### 9. 乐观锁

认为在一个线程操作数据的时候，其他线程不一定会同时进行操作，从而先尝试进行操作，失败则进行重试，典型的就是 CAS 。

#### 10. 分段锁

为提高性能，将数据分成若干部分，每部分分别加锁，典型如 concurrentHashMap 。

#### 11. 偏向锁/轻量级锁/重量级锁

如果一段代码被一个线程多次访问，该线程会自动获得锁，这就是偏向锁。如果又有一个线程来访问这段代码，偏向锁就升级成轻量级锁，没有获得锁得锁的线程则自旋。如果自旋次数到达阈值则阻塞，轻量级锁会升级成重量级锁。

### ReentrantLock
#### 基本用法

ReentrantLock 是 J.U.C.locks 包 Lock 接口的实现类，基本用法如下：

```
public ReentrantLock lock = new ReentrantLock();

lock.lock();
try{
    // dosomething
} catch(Exception e) {

} finally {
    lock.unlock();
}
```

ReentrantLock 具有如下特点：
1. **可重入**，和 synchornized 相同。
2. **可中断**，普通的 lock.lock() 是不能响应中断的，lock.lockInterruptibly() 能够响应中断。
3. **可限时**，lock.tryLock(5, TimeUnit.SECONDS) 超时不能获得锁，就返回 false ，不会永久等待构成死锁。
4. **支持公平锁**，默认是非公平锁，通过 new ReentrantLock(true) 可以创建公平锁。
5. **支持精确唤醒线程**，在使用 notify/notifyAll 时，只能随机唤醒线程，ReentrantLock 可以通过 Condition 精确唤醒线程。

```
Lock lock = new ReentrantLock();
Condition c1 = lock.newCondition();
Condition c2 = lock.newCondition();
Condition c3 = lock.newCondition();

public void print1() {
	lock.lock();
	try {
		while (count % 3 != 0) {
			c1.await();
		}
		System.out.println(Thread.currentThread().getName() + " print " + count);
		count++;
		c2.signal();
	} catch (InterruptedException e) {
		e.printStackTrace();
	} finally {
		lock.unlock();
	}
}

public void print2() {
	lock.lock();
	try {
		while (count % 3 != 1) {
			c2.await();
		}
		System.out.println(Thread.currentThread().getName() + " print " + count);
		count++;
		c3.signal();
	} catch (InterruptedException e) {
		e.printStackTrace();
	} finally {
		lock.unlock();
	}
}

public void print3() {
	lock.lock();
	try {
		while (count % 3 != 2) {
			c3.await();
		}
		System.out.println(Thread.currentThread().getName() + " print " + count);
		count++;
		c1.signal();
	} catch (InterruptedException e) {
		e.printStackTrace();
	} finally {
		lock.unlock();
	}
}
```

使用 ReentrantLock 需要注意：
1. 由于 Lock 需要手动 lock/unlock ，所以需要注意 lock 之后一定要 unlock 以避免死锁。
2. lock/unlock 多个嵌套是可以的。

#### 源码分析

ReentrantLock 可以创建公平锁和非公平锁，默认是非公平锁。公平锁由 FairSync 实现，非公平锁由 NonfairSync 实现，它们都是 Sync 的子类，Sync 又是 AbstractQueuedSynchronizer 的子类。ReentrantLock 的基本用法如下：

```
ReentrantLock reentrantLock = new ReentrantLock();
reentrantLock.lock();
try {
	// todo
} finally {
	reentrantLock.unlock();
}
```

下面我们就来分析一下每一步都做了什么。

##### **加锁**

<img src="https://s2.ax1x.com/2020/03/05/3Hhs4x.png" alt="3Hhs4x.png" border="0" />
从代码来看，公平锁和非公平锁的 lock 都是执行 acquire() 方法，区别在于非公平锁在 acquire() 前先进行一次 CAS 操作，成功了就不执行 acquire() ， 不成功才执行 acquire() 。这个 acquire() 是父类 AbstractQueuedSynchronizer 中的方法。
<img src="https://s2.ax1x.com/2020/03/05/3H4Uit.png" alt="3H4Uit.png" border="0" style="width:350px" />
公平锁和非公平锁的 tryAcquire() 分别有各自的实现。
<img src="https://s2.ax1x.com/2020/03/05/3HRjbR.png" alt="3HRjbR.png" border="0" />
比较两个 tryAcquire() 方法发现，公平锁多了一个 hasQueuedPredecessors() 的判断。在公平锁中，当线程执行 tryAcquire() 时，如果可以进行加锁时，先执行 !hasQueuedPredecessors() 的判断。
<img src="https://s2.ax1x.com/2020/03/05/3HW3rj.png" alt="3HW3rj.png" border="0" style="width:450px"/>
如果当前线程之前有排队的线程则返回 true ，如果队列为空或队列的 head 就是当前线程则返回 false 。这是一个取反判断，所以当队列中有排队的线程，那么就不再执行 compareAndSetState() 操作了。换句话说就是如果有线程在排队，那么就不执行加锁的操作，直接判定加锁失败。这样就回到 AbstractQueuedSynchronizer 的 acquire() 中，继续执行 acquireQueued() 操作，即加入到 AQS 队列中。
相对于公平锁来说，非公平锁没有执行 hasQueuedPredecessors() 判断，如果可以进行加锁，则直接进行加锁，加锁成功返回 true ，加锁失败返回 false ，再执行 acquireQueued() 入队操作。
总结一下，**非公平锁比公平锁多了两次获取锁的机会**。

##### **入队**

入队操作包含两步：

1. addWaiter()
   首先将当前线程封装成 Node 节点 n ，如果队列没有初始化，则通过 enq() 方法来初始化队列，会进行两次循环，第一次循环初始化一个空节点 t ，将 head 和 tail 都指向该 t ，第二次循环将 tail 指向 n ，将 t.next 指向 n ，这样就将线程加入到 AQS 队列中。
   <img src="https://s2.ax1x.com/2020/03/05/3HjMWQ.png" alt="3HjMWQ.png" border="0" style="width:600px"/>
2. acquireQueued()
   这个方法会判断线程是否会阻塞。判断逻辑放在 acquireQueued() 方法的死循环中。当线程加入到 AQS 队列中后，并不会马上阻塞，会在第一次循环中 再次尝试获取锁，如果获取锁失败，就将线程的 waitStatus 条件标记为需要 signal 来唤醒。第二次循环，还是获取锁失败之后，才真正 park 该线程，然后跳出循环。
   <img src="https://s2.ax1x.com/2020/03/05/3HX0qP.png" alt="3HX0qP.png" border="0" />

##### **解锁**

解锁就很简单了，从 AQS 对列中获取 head.next 然后 unpark 。

### ReadWriteLock

#### 基本用法

ReentrantReadWriteLock 是 ReadWriteLock 接口的实现类，基本用法如下：
```
public class ReadWriteLockExample {

    public static void main(String[] args) {
        Cache cache = new ReadWriteLockExample().new Cache();
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> {
                cache.put("key-" + finalI, finalI);
            }).start();
        }

        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> {
                System.out.println(cache.get("key-" + finalI));
            }).start();
        }
    }

    class Cache {
        volatile HashMap map = new HashMap();
        ReadWriteLock lock = new ReentrantReadWriteLock();

        public void put(String key, Object val) {
            lock.writeLock().lock();
            try {
                System.out.println(Thread.currentThread().getName() + " write begin");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                map.put(key, val);
                System.out.println(Thread.currentThread().getName() + " write finish");
            } finally {
                lock.writeLock().unlock();
            }
        }

        public Object get(String key) {
            lock.readLock().lock();
            try {
                System.out.println(Thread.currentThread().getName() + " read begin");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Object obj = map.get(key);
                System.out.println(Thread.currentThread().getName() + " read finish");
                return obj;
            } finally {
                lock.readLock().unlock();
            }
        }
    }
}
```

