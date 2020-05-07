---
title: Future
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-09 18:24:10
thumbnail:
---

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
这实际是利用了 [happens-before](../../../../2020/03/10/happens-before/) 规则。通过刚才的分析我们可以知道:
1. state 是 volatile 的。
2. ```outcome = v``` happens-before ```state == NORMAL``` 。
3. ```state == NORMAL``` happens-before ```get outcome``` 。

>所以利用 happens-before 的传递性可以保证 ```outcome = v``` happens-before ```get outcome``` 。