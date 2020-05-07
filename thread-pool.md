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

线程池可以重用线程资源，减少线程创建销毁的资源耗费。JDK 提供了 ThreadPoolExecutor 来创建线程池。


```
public ThreadPoolExecutor(int corePoolSize,                  // 线程池初始大小
						  int maximumPoolSize,               // 线程池最大线程数
						  long keepAliveTime,                // 线程存活的时间
						  TimeUnit unit,                     // 时间单位
						  BlockingQueue<Runnable> workQueue, // 任务等待队列
						  ThreadFactory threadFactory,       // 线程工厂用来创建线程
						  RejectedExecutionHandler handler   // 线程数量超过线程池大小后的处理策略
						  ) {}
```
#### 线程池参数效果
我们要创建一个核心线程数 5 ，最大线程数 20 ，队列大小是 10 的一个线程池。
```
ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 20,
            5, TimeUnit.SECONDS, new LinkedBlockingQueue<>(5), Executors.defaultThreadFactory(), defaultHandler);
```


如果我们提交 10 个任务，那么首先会将前 5 个任务交给 5 个线程执行，同时 5 个线程进入队列等待。这时队列有 5 个线程工作，5 个任务等待。

如果我们提交 15 个任务，那么首先会将前 5 个任务交给 5 个线程执行，同时 5 个线程进入队列等待，再新创建 5 个线程来执行 11-15 号任务。这时队列有 10 个线程工作，5 个任务等待。

如果我们提交 50 个任务，那么首先会将前 5 个任务交给 5 个线程执行，同时 5 个线程进入队列等待，再新创建 15 个线程来执行任务，剩下的 25 个任务会丢弃掉。这时队列有 20 个线程工作，5 个任务等待，等任务执行完，超过线程存活时间，线程会被销毁，剩下 5 个线程。

#### jdk 提供的线程池
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








