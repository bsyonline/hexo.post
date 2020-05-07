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

Java 锁的种类：
1. **公平锁**：多个线程按照申请锁的顺序获得锁，效率不高，通过 new ReentrantLock(true) 可以创建公平锁。
2. **非公平锁**：多个线程不是申请锁的顺序获得锁，后申请的有可能先获得锁。性能比公平锁高，会有优先级低的线程长时间无法获得锁的情况。synchornized 是非公平锁，ReentrantLock 默认也是非公平锁。
3. **可重入锁/递归锁**：如果一个线程获得了锁，那么该方法内部调用的其他方法可以直接获得该对象的锁，不需要重新申请锁。
4. **自旋锁**：线程获取锁失败后不会阻塞，而是通过循环再次尝试获取锁。自旋可以减少线程上下文切换，但是会消耗 CPU 资源，典型的用法如 CAS 。
5. **读锁/写锁**：读读可以共存，读写、写写互斥。
6. **共享锁**：一个锁可以被多个线程占有。
6. **独占锁**：一个锁只能被一个线程占有，synchornized 和 ReentrantLock 都是独占锁。
7. **悲观锁**：认为在一个线程操作数据的时候，其他线程也会同时进行操作，如果不加锁就一定会出问题，Java 中的锁就是悲观锁。
8. **乐观锁**：认为在一个线程操作数据的时候，其他线程不一定会同时进行操作，从而先尝试进行操作，失败则进行重试，典型的就是 CAS 。
9. **分段锁**：为提高性能，将数据分成若干部分，每部分分别加锁，典型如 concurrentHashMap 。
10. **偏向锁/轻量级锁/重量级锁**：如果一段代码被一个线程多次访问，该线程会自动获得锁，这就是偏向锁。如果又有一个线程来访问这段代码，偏向锁就升级成轻量级锁，没有获得锁得锁的线程则自旋。如果自旋次数到达阈值则阻塞，轻量级锁会升级成重量级锁。

##### **ReentrantLock**
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

##### **ReadWriteLock**
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