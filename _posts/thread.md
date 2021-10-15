---
title: Thread
date: 2016-07-23 21:50:03
tags:
  - Interview
category:
  - Concurrent
thumbnail: 
author: bsyonline
lede: 
---



### 线程的状态

参考 jdk 源码 java.lant.Thread.State 。Java 的线程有以下几种状态：

* NEW
  线程未启动。
* RUNNABLE
  线程在 JVM 中运行，也可能在等待其他资源。
* BLOCKED
  线程阻塞并等待锁，在调用 wait() 方法后进入或重新进入同步方法。
* WAITTING
  线程在执行特定的方法如： Object.wait(), Thread.join(), LockSupport.park() 后等待另一个线程执行特定的方法，如：Object.notify()/Object.notifyAll() 。
* TIMED_WAITTING
  线程在执行特定方法后进入执行时间的等待状态，如： Thread.sleep(), Object.wait(), Thread.join(), LockSupport.parkNanos, LockSupport.parkUntil
* TERMINATED
  线程执行完成。



### synchronized 

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



### volatile 

volatile 是 Java 语言的关键字，常常和 synchronized 进行比较。

#### volatile 和 synchronized 比较

<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;width:100%"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">volatile</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">synchronized</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">作用位置</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">变量</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">方法，代码块</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">同步对象</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">主内存和线程内存之间某个变量的值</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">主内存和线程内存之间所有变量的值</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">消耗资源</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">少</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">多</td></tr></table>

volatile 不能像 synchronized 一样广泛的用于线程安全，因为 volatile 不能保证原子性，所以 volatile 不能保证线程安全。但是在某些特殊场景下使用 volatile 要比 synchronized 和锁简单和高效，还能使程序更加简单。

#### 内存可见性

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

#### 禁止指令重排

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



### CAS

在 Java 并发编程中，为了保证并发安全，一种方式是使用锁，另一种方式是使用 CAS（compare and swap） 。CAS 有 3 个参数，内存地址 V ，旧的预期值 A ，要修改的新值 B ，更新一个变量的时候，只有当变量的预期值 A 和内存地址 V 当中的实际值相同时，才会将内存地址 V 对应的值修改为 B ，如果和预期不相等则更新失败。

#### CAS 是如何实现的？

Unsafe 类中有 3 个 cas 方法：

```
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

它们都是本地方法，在 x86 架构的系统中，是 Unsafe 类通过来调用 **cmpxchg** 指令来完成 CAS 操作的。

>compareAndSwapInt() 方法 4 个参数含义：
>Object var1 需要操作的对象。
>long var2	需要操作的属性的内存偏移量。
>int var4	期望值。
>int var5	更新值。

#### 如何使用 Unsafe

Unsafe 是 sun.misc 包下的类，如果我们使用 Unsafe.getUnsafe() 来创建实例，则会报 java.lang.SecurityException: Unsafe 。原因是因为 Unsafe 类是 Java 的底层类，必须由 bootstrap classloader 加载，我们的应用是通过 application classloader 加载的，所以会报异常。

```
@CallerSensitive
public static Unsafe getUnsafe() {
	Class var0 = Reflection.getCallerClass();
	if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
		throw new SecurityException("Unsafe");
	} else {
		return theUnsafe;
	}
}
```

既然不能直接使用，那就只能换一种方式，借鉴 java.util.concurrent.atomic.AtomicInteger ，利用反射来获取 Unsafe 。

```
public class UnsafeExample {

    private static Unsafe unsafe;
    private static long valueOffset;
    private volatile int value;

    static {
        try {
            Field theUnsafeField = Unsafe.class.getDeclaredField("theUnsafe");
            theUnsafeField.setAccessible(true);
            unsafe = (Unsafe) theUnsafeField.get(null);
            valueOffset = unsafe.objectFieldOffset(UnsafeExample.class.getDeclaredField("value"));
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        //Unsafe unsafe = Unsafe.getUnsafe();
        System.out.println(unsafe);
        System.out.println(valueOffset);
    }
}
```

#### CAS 的优点与问题

和 Java 的锁相比，CAS 不会阻塞线程，也不会有线程间通信带来的线程切换和唤醒的消耗，在性能上会好于 Java 的锁。但是 CAS 也有一些问题：

1. 线程自旋造成的 CPU 占用高。
2. ABA 问题。
3. 只能保证单个属性的线程安全，多个属性的线程安全无法使用 CAS 。

#### ABA 问题

CAS 没有加锁，而是通过比较内存地址的值是否和预期相等来判断是否能更新成功。举个例子，比如一个栈中有 ABC 三个元素，栈顶元素是 C ，如果进行 2 次出栈操作，在将 C 入栈，此时栈顶元素还是 C ，但是栈中只有 A 和 C 了。为了避免这种情况，可以使用 ```AtomicStampedReference<V>``` 或 ```AtomicMarkableReference<V>``` 通过每次操作都添加一个标签来，虽然值相同但是不同的操作标签已经改变，从而避免 ABA 问题。

>```AtomicStampedReference<V>``` 和 ```AtomicMarkableReference<V>``` 内部都维护了一个 pair ，AtomicStampedReference 中的 pair 是 (reference, int) ，AtomicMarkableReference 中的 pair 是 (reference, boolean) 。AtomicMarkableReference 只是标识是否有修改。



### LockSupport 

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



### AQS

AQS(AbstractQueuedSynchronizer) 是 java.util.concurrent.locks 包中提供的同步框架。AQS 是一个抽象类，java.util.concurrent 中很多并发工具类都是基于 AQS 实现的。AQS 比较复杂，但是我们可以先比较概括的来描述一下 AQS 。简单来说，AQS 维护了一个 state 属性和一个由双向链表构成的同步队列。线程通过 CAS 操作 state 获取锁，获取锁失败的线程被构造成 Node 放到同步队列中阻塞。当锁被释放，再将队列中的线程唤醒来重新获取锁。AQS 中提供了很多获取和释放资源的操作，子类继承和扩展这些方法来完成具体的功能，AQS 比较抽象和难以理解，我们可以通过学习具体场景的类来学习 AQS 。

### 锁

#### 锁的种类

##### 1. 公平锁

多个线程按照申请锁的顺序获得锁，效率不高，通过 new ReentrantLock(true) 可以创建公平锁。

##### 2. 非公平锁

多个线程不是申请锁的顺序获得锁，后申请的有可能先获得锁。性能比公平锁高，会有优先级低的线程长时间无法获得锁的情况。synchornized 是非公平锁，ReentrantLock 默认也是非公平锁。

##### 3. 可重入锁/递归锁

如果一个线程获得了锁，那么该方法内部调用的其他方法可以直接获得该对象的锁，不需要重新申请锁。

##### 4. 自旋锁

线程获取锁失败后不会阻塞，而是通过循环再次尝试获取锁。自旋可以减少线程上下文切换，但是会消耗 CPU 资源，典型的用法如 CAS 。

##### 5. 读锁/写锁

读读可以共存，读写、写写互斥。

##### 6. 共享锁

一个锁可以被多个线程占有。

##### 7. 独占锁

一个锁只能被一个线程占有，synchornized 和 ReentrantLock 都是独占锁。

##### 8. 悲观锁

认为在一个线程操作数据的时候，其他线程也会同时进行操作，如果不加锁就一定会出问题，Java 中的锁就是悲观锁。

##### 9. 乐观锁

认为在一个线程操作数据的时候，其他线程不一定会同时进行操作，从而先尝试进行操作，失败则进行重试，典型的就是 CAS 。

##### 10. 分段锁

为提高性能，将数据分成若干部分，每部分分别加锁，典型如 concurrentHashMap 。

##### 11. 偏向锁/轻量级锁/重量级锁

如果一段代码被一个线程多次访问，该线程会自动获得锁，这就是偏向锁。如果又有一个线程来访问这段代码，偏向锁就升级成轻量级锁，没有获得锁得锁的线程则自旋。如果自旋次数到达阈值则阻塞，轻量级锁会升级成重量级锁。

#### ReentrantLock

**基本用法**

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

**源码分析**

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

**加锁**

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

**入队**

入队操作包含两步：

1. addWaiter()
   首先将当前线程封装成 Node 节点 n ，如果队列没有初始化，则通过 enq() 方法来初始化队列，会进行两次循环，第一次循环初始化一个空节点 t ，将 head 和 tail 都指向该 t ，第二次循环将 tail 指向 n ，将 t.next 指向 n ，这样就将线程加入到 AQS 队列中。
   <img src="https://s2.ax1x.com/2020/03/05/3HjMWQ.png" alt="3HjMWQ.png" border="0" style="width:600px"/>
2. acquireQueued()
   这个方法会判断线程是否会阻塞。判断逻辑放在 acquireQueued() 方法的死循环中。当线程加入到 AQS 队列中后，并不会马上阻塞，会在第一次循环中 再次尝试获取锁，如果获取锁失败，就将线程的 waitStatus 条件标记为需要 signal 来唤醒。第二次循环，还是获取锁失败之后，才真正 park 该线程，然后跳出循环。
   <img src="https://s2.ax1x.com/2020/03/05/3HX0qP.png" alt="3HX0qP.png" border="0" />

**解锁**

解锁就很简单了，从 AQS 对列中获取 head.next 然后 unpark 。

#### ReadWriteLock

**基本用法**

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





### Condition

Condition 是 java.util.concurrent 包中提供的接口，Condition 可以用来阻塞和唤醒线程，如果有多个线程，还可以指定唤醒某个线程。具体用法我们可以看一个例子：

```
public class ConditionTest {

    public static void main(String[] args) throws InterruptedException {
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();
        Thread t1 = new Thread(() -> {
            reentrantLock.lock();
            try {
                condition.await();
                System.out.println(Thread.currentThread().getName() + " wake up");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }, "t1");
        t1.start();
        Thread.sleep(1000);
        Thread t2 = new Thread(() -> {
            reentrantLock.lock();
            try {
                condition.await();
                System.out.println(Thread.currentThread().getName() + " wake up");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                reentrantLock.unlock();
            }
        }, "t2");
        t2.start();
        Thread.sleep(1000);
        new Thread(() -> {
            reentrantLock.lock();
            condition.signalAll();
            reentrantLock.unlock();
        }, "t5").start();
    }
}
```

这个例子展示了 Condition 的用法，我们来看一下每一步都做了什么。

1. 首先 lock.newCondition() 返回的是 sync.newCondition() sync 继承自 AQS ，最终创建了一个 AQS 的 ConditionObject() 对象。ConditionObject 中包含了两个引用 firstWaiter 和 lastWaiter ，其类型是 AbstractQueuedSynchronizer.Node ，Node 中又包含了一个指针 nextWaiter ，这样就可以来构建一个链表。
2. 当调用 await() 方法时，先创建一个包含当前线程的 node 节点 n1 ，然后如果 condition queue 的 lastWaiter 节点 t 为空，说明condition queue 中没有节点，就将 firstWaiter 节点指向 n1 ，否则就将 lastWaiter.nextWaiter 指向 n1 。
   <img src="https://s2.ax1x.com/2020/03/06/3bd2sf.png" alt="20200306102035" border="0" style="width:500px">
3. 当调用 signal() 方法时，从 condition queue 中取出 firstWaiter 节点 n1 执行 doSignal() 。在 doSignal() 中，先将 firstWaiter 指向 n1.next 节点，如果 n1.next 为空，就将 lastWaiter 置空。然后将 condition queue 的节点 n1 取出来，加入到 AQS 队列中。
   <img src="https://s2.ax1x.com/2020/03/06/3bdRL8.png" alt="20200306101634" border="0" style="width:500px">
4. 当执行 unlock() 方法，就可以从 AQS 队列中执行出队操作，从而 unpark 线程。

>signal() 和 signalAll() 的区别：
>signal() 方法是从 condition queue 中取出 firstWaiter 节点执行 transferForSignal() 操作。signalAll() 方法是遍历 condition queue 执行 transferForSignal() 直到 firstWaiter 为空。



### CountDownLatch

CountDownLatch 是 java.util.concurrent 包中提供的一个倒计时器工具类。基本用法如下：

```
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(3);
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " task done");
                latch.countDown();
            }, "t" + i).start();
        }
        latch.await();
        System.out.println("all thread wake up");
    }
}
```

接下来我们来看看每一步都做了什么。

1. 初始化 CountDownLatch 。

```
public CountDownLatch(int count) {
	if (count < 0) throw new IllegalArgumentException("count < 0");
	this.sync = new Sync(count);
}
//
Sync(int count) {
	setState(count);
}
```

可以看到我们创建的计数 count 通过 setState 保存到 AQS 的 state 中。

2. await() 
   首先判断 state ，因为是倒计数器，所以是判断 state 是不是等于 0 。如果不为 0 ，将包含主线程信息的 node 节点加入到 AQS 队列，然后再进行循环。第一次循环会再判断一下计数器是否到 0 了，如果还没有到 0 ，说明任务还没有就绪，那就将 node 的 waitStatus 标记为需要 signal 来唤醒。然后进行第二次循环，如果还没有到 0 ，则 park 主线程。
3. countDown()
   每执行一次 countDown() 方法，会通过 CAS 将 state-1 ，直到 state=0 才执行 doReleaseShared() 。当最后一次 countDown() 执行，AQS 队列中有两个节点，一个是虚拟的 node 节点 head ，另一个是包含主线程信息的 node 节点。取出 head 节点，并且由于 waitStatus 是 signal ，会执行 unparkSuccessor() 。在 unparkSuccessor 中会 unpark head.next 即包含主线程信息的节点。



### BlockingQueue 

BlockingQueue 是 java.util.concurrent 包中的接口，实现了 Collection 和 Queue 接口，提供了 3 类操作。
BlockingQueue 操作

<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Throws exception</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Special value</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Blocks</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Times out</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">Insert</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">add(e)</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">offer(e)</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">put(e)</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">offer(e, time, unit)</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">Remove</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">remove()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">poll()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">take()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">poll(time, unit)</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">Examine</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">element()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">peek()</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">not applicable</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;">not applicable</td></tr></table>
和普通的 queue 相比，BlockingQueue 满时 put 元素和 BlockingQueue 为空时 take 元素都会阻塞。下面我们就以 LinkedBlockingQueue 为例来看看具体实现。
首先在 LinkedBlockingQueue 中定义了一些属性。
<img src="https://s2.ax1x.com/2020/03/07/3jTjvd.png" alt="3jTjvd.png" border="0" style="width:400px"/>
通过这些属性定义，我们大概可以猜到元素在是以 Node 的形式保存在队列中，并且入队和出队操作都会用锁，元素个数也是通过 AtomicInteger 来保存，所以 LinkedBlockingQueue 是一个线程安全的队列。
接下来我们主要看看两个 block 操作：

1. put(e)
   <img src="https://s2.ax1x.com/2020/03/07/3jqyM4.png" alt="3jqyM4.png" border="0" style="width:500px" />
   put() 中有 3 处判断:
   1) 当元素个数小于 capacity 则继续执行，如果等于 capacity 则执行阻塞 notFull 线程。
   2) 添加完元素之后，如果元素个数小于 capacity 则唤醒 notFull 线程。
   3) 释放锁之前元素个数为 c+1 ，如果 c==0 那么说明队列中有一个元素，所以唤醒 notEmpty 线程。
2. take()
   <img src="https://s2.ax1x.com/2020/03/07/3jjU6U.png" alt="3jjU6U.png" border="0" style="width:350px" />
   和 put(e) 类似，take() 方法也是有 2 处判断：
   1) 当元素个数等于 0 ，则阻塞 notEmpty 线程。
   2) 当元素个数大于 0 ，则唤醒 notEmpty 线程。
   3) take 元素个数会减少，在释放锁之前做了 take 操作，所以释放锁之后队列实际是小于 capacity 的，所以唤醒 notFull 线程。

总结一下：

1. **LinkedBlockingQueue 有两把锁 putLock 和 takeLock ，有两个 condition 条件 notFull 和 notEmpty 。**
2. **元素添加或删除操作先获取锁，再操作链表。**
3. **如果队列满时阻塞 notFull 线程，反之则唤醒 notFull 线程。**
4. **如果队列中没有元素时，阻塞 notEmpty 线程，反之则唤醒 notEmpty 线程。**



### Future

Future 是一个可终止的异步计算接口，它提供了一种异步计算方式。Future 代表了异步计算的结果，当计算没有完成，获取结果会被阻塞，直到计算完成。FutureTask 是 Future 接口的实现，可以接收 Runnable 和 Callable 对象，也可以使用 Executor.submit() 执行。

```
public class FutureExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        FutureTask<Integer> f1 = new FutureTask((Callable<Integer>) () -> {
            Thread.sleep(2000);
            return 10;
        });
        FutureTask<Integer> f2 = new FutureTask((Callable<Integer>) () -> {
            Thread.sleep(3000);
            return 20;
        });
        FutureTask<Integer> f3 = new FutureTask((Callable<Integer>) () -> {
            Thread.sleep(5000);
            return 20;
        });
        new Thread(f1).start();
        new Thread(f2).start();        
		new Thread(f3).start();
        foo();
        f3.cancel(true);
        System.out.println("--");
        while (!f1.isDone() || !f2.isDone()) {
            System.out.println("f1" + (f1.isDone() ? "完成，" : "未完成，") + "f2" + (f2.isDone() ? "完成" : "未完成"));
            Thread.sleep(300);
        }
        System.out.println("f1" + (f1.isDone() ? "完成，" : "未完成，") + "f2" + (f2.isDone() ? "完成" : "未完成"));
        int result = f1.get() + f2.get();
        System.out.println("f3 cancelled state: "+f3.isCancelled());
        System.out.println("result=" + result + ", costs " + (System.currentTimeMillis() - start) + "ms");
    }

    static void foo() throws InterruptedException {
        System.out.println("同步执行foo");
        Thread.sleep(1000);
        System.out.println("foo done");
    }
}
```

执行结果

```
同步执行foo
foo done
--
f1未完成，f2未完成
f1未完成，f2未完成
f1未完成，f2未完成
f1未完成，f2未完成
f1完成，f2未完成
f1完成，f2未完成
f1完成，f2未完成
f1完成，f2完成
f3 cancelled state: true
result=30, costs 3146ms
```

在上面的例子就是 FutureTask 的用法。主线程执行 foo() ，f1 ，f2 f3 是三个异步任务，执行完成后通过 future.get() 获取直接结果，如果 f1 或 f2 没有执行完，future.get() 会阻塞，直到任务完成。通过 future.isDone() 可以查看任务是否完成。在 f3 任务之前完成之前可以通过 future.cancel() 终止任务，任务完成后则无法终止。
接下来我们来分析一下 Future 是如何工作的。
首先看看 FutureTask 中定义的属性和构造器。
<img src="https://s2.ax1x.com/2020/03/09/8Cif4x.png" alt="8Cif4x.png" border="0" style="width:300px"/>
FutureTask 有两个构造器，一个接收 Callable 一个接收 Runnable 。
<img src="https://s2.ax1x.com/2020/03/09/8Ci58K.png" alt="8Ci58K.png" border="0" style="width:450px"/>
Runnable 最终通过 RunnableAdapter 构造成为 Callable 。
<img src="https://s2.ax1x.com/2020/03/09/8Ck9W6.png" alt="8Ck9W6.png" border="0" style="width:600px"/>
由于 FutureTask 实现了 Runnable 接口，所以任务启动后首先会执行 run() 方法。在 run() 方法中，任务的初始状态为 NEW ，通过 CAS 将当前线程赋值为 runner 。然后在 run() 方法中执行 callable.call() 。
<img src="https://s2.ax1x.com/2020/03/09/8CEKsS.png" alt="8CEKsS.png" border="0" style="width:600px"/>
不管是成功还是异常，都会通过 CAS 更新状态，将结果或异常交给 outcome 。
<img src="https://s2.ax1x.com/2020/03/09/8CE2QK.png" alt="8CE2QK.png" border="0" style="width:600px"/>
再来看如何获取结果。根据状态，如果完成就直接获取结果，如果没有完成，则执行 awaitDone() 。
<img src="https://s2.ax1x.com/2020/03/10/8CmgtH.png" alt="8CmgtH.png" border="0" style="width:600px"/>
如果一个任务没有执行完，在第一次循环中将构造一个包含当前线程的 WaitNode ，第二次循环将 WaitNode 加入到队列，第三次循环才 park 线程。当任务执行完成后，调用 finishCompletion() 唤醒线程，再重新获取结果。
cancel() 操作就比较简单了，更新状态，唤醒线程而已。
<img src="https://s2.ax1x.com/2020/03/10/8CncrV.png" alt="8CncrV.png" border="0" style="width:600px"/>

总结一下：FutureTask 实现了 Runnable 接口，在 run() 方法中调用 callable.call() ，将结果保存到 outcome 。通过 get() 获取结果，如果任务没有执行完，则通过自旋将当前线程加入到 waiter 队列的头部，然后 park 线程。等到任务执行完成，通过 finishCompletion() 唤醒 waiter 队列中的线程，再次获取结果。

>我们注意到，在 FutureTask 中声明的 outcome 并没有用 volatile 修饰，那是如何保证在多线程环境下修改了 outcome 立刻对其他线程可见的呢？
>这实际是利用了 [happens-before](../../../../2020/03/10/happens-before/) 规则。通过刚才的分析我们可以知道:

1. state 是 volatile 的。
2. ```outcome = v``` happens-before ```state == NORMAL``` 。
3. ```state == NORMAL``` happens-before ```get outcome``` 。

>所以利用 happens-before 的传递性可以保证 ```outcome = v``` happens-before ```get outcome``` 。

### 线程池

#### 线程池简介

Java 多线程可以提搞程序的执行效率，单同时会带来一些问题。

1. 线程过多会带来额外开销，包括创建销毁线程等。
2. 调度线程是线程切换带来的开销。

线程池是 Pooling 的一种应用。使用线程池可以避免上述问题。

JDK 提供了 ThreadPoolExecutor 类来创建线程池。


```
public ThreadPoolExecutor(int corePoolSize,                  // 线程池初始大小
						  int maximumPoolSize,               // 线程池最大线程数
						  long keepAliveTime,                // 线程存活的时间
						  TimeUnit unit,                     // 时间单位
						  BlockingQueue<Runnable> workQueue, // 任务等待队列
						  ThreadFactory threadFactory,       // 线程工厂用来创建线程
						  RejectedExecutionHandler handler   // 线程数量超过线程池大小后的处理策略
						  ) 
```

#### ThreadPoolExecutor 的体系

ThreadPoolExecutor 继承关系如下：

<a href="https://imgtu.com/i/gyyJds"><img src="https://z3.ax1x.com/2021/05/15/gyyJds.png" alt="gyyJds.png" border="0" /></a>

#### ThreadPoolExecutor 的运行机制

ThreadPoolExecutor 的运行机制如下：

<a href="https://imgtu.com/i/gyy7TA"><img src="https://z3.ax1x.com/2021/05/15/gyy7TA.png" alt="gyy7TA.png" border="0" /></a>

线程池内部通过生产者消费者模型讲线程和任务解耦。

#### ThreadPoolExecutor 的生命周期

前面说到 ThreadPoolExecutor 的一个工作就是维护线程池的生命周期，这是在 ThreadPoolExecutor 内部维护的，整个生命周期包含 5 种状态：RUNNING、SHUTWOWN、STOP、TIDYING 和 TERMINATED 。

| 运行状态   | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| RUNNING    | 能接受新提交的任务，并能够处理阻塞队列种的任务。             |
| SHUTWOWN   | 关闭状态，不再接受新提交的任务，单可以继续处理阻塞队列中的任务。 |
| STOP       | 不能接受新任务，也不能处理队列中的任务，并会中断正在处理任务的线程。 |
| TIDYING    | 所有任务都已，workerCount有效线程数为 0 。                   |
| TERMINATED | 执行 terminated() 方法后进入该状态。                         |

<a href="https://imgtu.com/i/gy0XNV"><img src="https://z3.ax1x.com/2021/05/15/gy0XNV.png" alt="gy0XNV.png" border="0"></a>



#### JDK 提供的线程池

除了使用 new ThreadPoolExecutor ，Executors 提供了一些静态方法用来创建线程池，如：

```
Executor executor = Executors.newSingleThreadExecutor();
//new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>()));
```

```
Executor executor = Executors.newFixedThreadPool(5);
//new ThreadPoolExecutor(5, 5, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
```

```
Executor executor = Executors.newCachedThreadPool();
//new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
```

```
Executor executor = Executors.newScheduledThreadPool();
```

本质上和我们自己 new 一个线程池是一样的。

LinkedBlockingQueue 是阻塞队列，会缓存任务。SynchronousQueue 不缓存任务，会将任务直接发给消费端执行。所以如果我们创建以下线程池：

```
ThreadPoolExecutor executor = new ThreadPoolExecutor(10, 20, 20L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>(), new ThreadPoolExecutor.AbortPolicy());
```

当我们提交 50 个任务，那么只有 20 个任务能够执行，剩下 30 个线程会被丢弃。


虽然 JDK 提供了 Executors 几种创建线程的方式，但是实际使用中并不建议直接使用，因为 LinkedBlockingQueue 虽然是有界队列，但是队列大小默认是 Integer.MAX_VALUE ，如果任务很多会导致 OOM 。所以我们还是需要根据实际情况自己创建线程池。

>线程的数量需要根据实际情况设置，如果任务是 CPU 密集型，那么线程数和 CPU 核数相同，可以减少线程上下文切换，如果是 IO 密集型，线程数量可以是 CPU 核数的若干倍。这个数据只是经验值，实际多少最好需要压测。



#### 线程池参数分析

我们要创建一个核心线程数 5 ，最大线程数 20 ，队列大小是 10 的一个线程池。

```
ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 20,
            5, TimeUnit.SECONDS, new LinkedBlockingQueue<>(5), Executors.defaultThreadFactory(), defaultHandler);
```


如果我们提交 10 个任务，那么首先会将前 5 个任务交给 5 个线程执行，同时 5 个线程进入队列等待。这时队列有 5 个线程工作，5 个任务等待。

如果我们提交 15 个任务，那么首先会将前 5 个任务交给 5 个线程执行，同时 5 个线程进入队列等待，再新创建 5 个线程来执行 11-15 号任务。这时队列有 10 个线程工作，5 个任务等待。

如果我们提交 50 个任务，那么首先会将前 5 个任务交给 5 个线程执行，同时 5 个线程进入队列等待，再新创建 15 个线程来执行任务，剩下的 25 个任务会丢弃掉。这时队列有 20 个线程工作，5 个任务等待，等任务执行完，超过线程存活时间，线程会被销毁，剩下 5 个线程。



#### 动态调整线程池参数

ThreadPoolExecutor 的主要参数有 3 个，corePoolSize ，maximumPoolSize 和 blockingQueue 的 capacity 。ThreadPoolExecutor 提供了 setCorePoolSize 和 setMaximumPoolSize 方法可以修改 corePoolSize 和 maximumPoolSize 。

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 5, 30, TimeUnit.SECONDS, new LinkedBlockingDeque<>(10),
                new ThreadPoolExecutor.AbortPolicy());
// main - coreSize=2, active=5, maxSize=5, poolRate=100.00%, completed=0, queueSize=10, queued=10, remain=0, queueRate=100.00%
executor.setCorePoolSize(10);
executor.setMaximumPoolSize(10);
executor.prestartAllCoreThreads(); // 此方法是预热线程池，如果不调用此方法，活动线程数不一定是10
// main - coreSize=10, active=10, maxSize=10, poolRate=100.00%, completed=0, queueSize=10, queued=5, remain=5, queueRate=50.00%
```

但是没有提供修改 capacity 的方法，因为 LinkedBlocklingQueue 的 capacity 是 final 的，所以无法直接修改，不过可以自定义一个 BlockingQueue 增加一个修改 capacity 的方法。



### ThreadLocal 

ThreadLocal 是一个特殊的数据存储类，它为每个线程都提供了一份变量的副本，每个线程只能够获取到自己对应的存储数据，从而实现变量被多个线程共享，数据又彼此隔离。

#### ThreadLocal 的使用场景

ThreadLocal 在什么场景下使用呢？

1. **多个线程之间作用域相同并且需要不同的数据副本。**比如我们在使用 redis 实现分布式锁时，一个线程获取锁，处理完成后要释放锁，并且要保证自己的锁只能由自己来释放，我们需要给每一把锁生成一个唯一 ID ，这个 ID 由线程自己持有，在释放锁的时候通过 ID 来进行操作，此时可以使用 ThreadLocal 来保存 ID 。

2. **对象传递。**通常我们在方法调用的时候，如果逻辑复杂，可能入参会有多个或者无法获取到某些参数的情况，这时候就可以使用 ThreadLocal 来进行对象传递。典型的如 Spring AOP ，由于业务代码是定义好的，业务代码入参中并不强制包含切面处理的对象，所以 spring 将这些切面对象放在 ThreadLocal 中，这样就可以在线程内部获取到切面所需要的参数信息了。

#### ThreadLocal 的实现原理

通过查看 ThreadLocal 的源代码，我们可以看到在 ThreadLocal 内部有一个 ThreadLocalMap ，这个 ThreadLocalMap 就是 Thread 类中用来存放线程自己的数据的。我们主要关注 ThreadLocal 的这几个方法：

1. set 

   ```java
   public void set(T value) {
   	Thread t = Thread.currentThread();
   	ThreadLocalMap map = getMap(t);
   	if (map != null)
   		map.set(this, value);
   	else
   		createMap(t, value);
   }
   ```

   获取到当前线程的 ThreadLocalMap ，如果 map 不为空，就把对象 set 进去，如果 map 为空，就创建一个新的 ThreadLocalMap ，再把对象 set 进去。

2. get

   ```java
   public T get() {
   	Thread t = Thread.currentThread();
   	ThreadLocalMap map = getMap(t);
   	if (map != null) {
   		ThreadLocalMap.Entry e = map.getEntry(this);
   		if (e != null) {
   			@SuppressWarnings("unchecked")
   			T result = (T)e.value;
   			return result;
   		}
   	}
   	return setInitialValue();
   }
   ```

   get 操作也是一样，先获取当前线程的 ThreadLocalMap ，如果 map 不为空，则从 Map.Entry 中获取对象。如果 map 为空，则返回初始值 null ，并把 null 值 set 到当前线程的 ThreadLocalMap 中。

3. remove

   ```java
   public void remove() {
       ThreadLocalMap m = getMap(Thread.currentThread());
       if (m != null)
           m.remove(this);
   }
   ```

   remove 就更简单了，获取当前线程的 ThreadLocalMap ，然后从 Map 中删除线程的数据。

了解 ThreadLocal 的基本操作之后，我们还要再来看看 ThreadLocalMap 。ThreadLocal 提供的功能，本质上是由 ThreadLocalMap 实现的。刚才源码我们也看到了，在往 ThreadLocal 中 set 对象的时候，实际调用的是 ThreadLocalMap 的 set 方法。

```java
private void set(ThreadLocal<?> key, Object value) {
	Entry[] tab = table;
	int len = tab.length;
    // 根据hash找到数组中的一个位置
    int i = key.threadLocalHashCode & (len-1);
	// 遍历map，看是否有值，如果没有则不进入循环
    // 如果有值，entry的key相同就替换，key为空就直接set，否则就找数组的下一个位置
    for (Entry e = tab[i];
        e != null;
        e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        // 如果Entry的key相同，则替换
        if (k == key) {
            e.value = value;
            return;
        }
        // 如果Entry的key是null，则set新值
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // 直到找到一个空的位置，生成新的Entry添加到map中
    tab[i] = new Entry(key, value);
    // map的大小加1
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        // 如果清除map中Entry的key为null的Entry之后，map的size
        // 还大于阈值，则进行rehash
        rehash();
}
```

```java
private void rehash() {   
	// 清除过时的元素(Entry的key为空)
	expungeStaleEntries();
	// 如果清除之后，size 还是超过阈值，则扩容为原来的 2 倍
    if (size >= threshold - threshold / 4)
    	resize();
}
```

#### ThreadLocal 需要注意的问题

1. 脏数据

   由于是线程隔离的，且清理之前不会失效，如果线程复用，在新线程 set 前就去 get ，可能会得到旧线程的数据。

2. 内存泄漏

   ThreadLocalMap 中的 key 是弱引用，不存在外部引用时就可以被回收，ThreadLocalMap 中的 entry 的 key 是强引用，所以当线程退出，value 才会被回收。如果线程一直不退出，就会造成内存泄漏。

所以在使用 ThreadLocal 是应该尽可能的主动 remove 。

 3. 还有一种场景，在主线程和子线程共享数据时，直接使用 ThreadLocal 是不行的，可以使用 InheritableThreadLocal 。但是子线程需要是新创建的，线程复用是不行的。另外在从

    ```java
    public class ThreadLocalTest {
        static InheritableThreadLocal<User> threadLocal = new InheritableThreadLocal<>();
        static ExecutorService executor = Executors.newFixedThreadPool(5);
    
        public static void main(String[] args) throws InterruptedException {
            for (int i = 0; i < 10; i++) {
                threadLocal.set(new User(i, Thread.currentThread().getName()));
                new Thread(() -> {
                    System.out.println(Thread.currentThread() + " - " + threadLocal.get());
                }).start();
            }
        }
    }
    ```

    输出结果，10 个线程能拿到 10 个不同的值。

    ```
    Thread[Thread-0,5,main] - User{id=0, name='main'}
    Thread[Thread-2,5,main] - User{id=2, name='main'}
    Thread[Thread-1,5,main] - User{id=1, name='main'}
    Thread[Thread-3,5,main] - User{id=3, name='main'}
    Thread[Thread-6,5,main] - User{id=6, name='main'}
    Thread[Thread-4,5,main] - User{id=4, name='main'}
    Thread[Thread-7,5,main] - User{id=7, name='main'}
    Thread[Thread-5,5,main] - User{id=5, name='main'}
    Thread[Thread-8,5,main] - User{id=8, name='main'}
    Thread[Thread-9,5,main] - User{id=9, name='main'}
    ```

    ```java
    public class ThreadLocalTest {
        static InheritableThreadLocal<User> threadLocal = new InheritableThreadLocal<>();
    
        static ExecutorService executor = Executors.newFixedThreadPool(5);
    
        public static void main(String[] args) throws InterruptedException {
            for (int i = 0; i < 10; i++) {
                threadLocal.set(new User(i, Thread.currentThread().getName()));
                executor.execute(() -> {
                    System.out.println(Thread.currentThread() + " - " + threadLocal.get());
                });
            }
    }
    ```

    如果使用线程池，输出结果只能拿到 5 个不同的值。

    ```
    Thread[pool-1-thread-2,5,main] - User{id=1, name='main'}
    Thread[pool-1-thread-3,5,main] - User{id=2, name='main'}
    Thread[pool-1-thread-1,5,main] - User{id=0, name='main'}
    Thread[pool-1-thread-4,5,main] - User{id=3, name='main'}
    Thread[pool-1-thread-1,5,main] - User{id=0, name='main'}
    Thread[pool-1-thread-3,5,main] - User{id=2, name='main'}
    Thread[pool-1-thread-2,5,main] - User{id=1, name='main'}
    Thread[pool-1-thread-5,5,main] - User{id=4, name='main'}
    Thread[pool-1-thread-1,5,main] - User{id=0, name='main'}
    Thread[pool-1-thread-4,5,main] - User{id=3, name='main'}
    ```

    

### 死锁

死锁是2个以上的线程由于竞争同一资源形成阻塞。

**如何写一个死锁**

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

```
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



### 多线程下载

```java
import java.io.InputStream;
import java.io.RandomAccessFile;
import java.net.URL;
import java.net.URLConnection;

public class MutiThreadDownloadTest {

     /**
     * @param args
     */
     public static void main(String[] args) {
          String urlString = "http://www.7-zip.org/a/7z920-x64.msi";
          String path = "D:\\7z.msi";
          try {
               new MutiThreadDownloadTest().new MutiThreadDownload(urlString, path, 3).download();
          } catch (Exception e) {
               e.printStackTrace();
          }
     }

     class MutiThreadDownload {
          private String urlString;// 下载路径
          private String path;// 文件存储路径
          private int threadNum;// 线程数量
          private int fileSize;// 文件大小

          public MutiThreadDownload(String urlString, String path, int threadNum) {
               super();
               this.urlString = urlString;
               this.path = path;
               this.threadNum = threadNum;
          }

          public void download() throws Exception {
               URL url = new URL(urlString);
               URLConnection conn = url.openConnection();
               fileSize = conn.getContentLength();// 获取文件大小
               RandomAccessFile destFile = new RandomAccessFile(path, "rw");// 创建空文件
               // 设置下载文件相同大小
               destFile.setLength(fileSize);
               destFile.close();
               // 计算每个线程的下载大小
               int sizePerThread = fileSize / threadNum;
               // 开启线程
               for (int i = 0; i < threadNum; i++) {
                    // 计算每条线程的下载的开始位置
                    int startPos = i * sizePerThread;// 开始位置
                    int endPos = sizePerThread * (i + 1) - 1;// 结束位置

                    new DownloadThread(startPos,endPos,urlString,path).start();
               }
               destFile.close();
               conn = null;
          }

     }

     class DownloadThread extends Thread {
          // 定义下载的起始点
          private long startPos;
          // 定义下载的结束点
          private long endPos;
          private String urlString;
          private String path;

          // 构造器，传入输入流，输出流和下载起始点、结束点
          public DownloadThread(long startPos, long endPos, String urlString, String path) {
               // 输出该线程负责下载的字节位置
               System.out.println("startPos="+startPos + " - endPos:" + endPos);
               this.startPos = startPos;
               this.endPos = endPos;
               this.urlString = urlString;
               this.path = path;
          }

          public void run() {
               URL url = null;
               URLConnection conn = null;
               InputStream is = null;
               RandomAccessFile out = null;
               try {
                    url = new URL(urlString);
                    conn = url.openConnection();
                    is = conn.getInputStream();
                    byte[] buffer = new byte[1024];
                    int hasRead = 0;
                    // 读取网络数据，并写入本地文件
                    int length = 0;
                    out = new RandomAccessFile(path, "rw");
                    is.skip(this.startPos);//流从什么位置读
                    out.seek(startPos);//文件从什么位置写

                    // 读取网络数据，并写入本地文件
                    while (length < endPos - startPos
                              && (hasRead = is.read(buffer)) > 0) {
                         out.write(buffer, 0, hasRead);
                         // 累计该线程下载的总大小
                         length += hasRead;
                    }
                    System.out.println("线程:" + Thread.currentThread().getId() + "下载完成!");
               } catch (Exception ex) {
                    ex.printStackTrace();
               }
               // 使用finally块来关闭当前线程的输入流、输出流
               finally {
                    try {
                         if (out != null) {
                              out.close();
                         }
                         if (is != null) {
                              is.close();
                         }
                         if (conn != null){
                              conn = null;
                         }
                    } catch (Exception ex) {
                         ex.printStackTrace();
                    }
               }
          }
     }
}
```



### 交替打印线程

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

