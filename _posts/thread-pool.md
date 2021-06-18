---
title: Thread Pool
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2019-11-07 16:10:44
thumbnail:
---

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

