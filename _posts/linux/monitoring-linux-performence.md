---
title: Monitoring Linux Performence
tags:
  - Interview
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2020-02-22 15:00:39
thumbnail:
---

程序在 Linux 服务器上运行，我们需要对程序的运行状态和系统健康状况进行监控，那么如何做呢？下面一组命令可以帮助到我们。
1. **top**，监控 CPU 。
<img src="https://s2.ax1x.com/2020/02/27/3wTtfg.png" alt="3wTtfg.png" border="0" style="width:550px" />
这里我们主要关注两个地方：
1) 是右上角的 load average ，load average 有 3 个值，分别表示系统 1m ，5m ，15m 的平均负载值，`(1m+5m+15m)/3*100%` 可以作为衡量系统负载的指标，一般不要超过 60% 。
2) 是 %CPU 。可以看出 CPU 占用情况和对应的的进程 id 。
2. **vmstat**，也可以用来监控 CPU 资源。
<img src="https://s2.ax1x.com/2020/02/27/3wqi9O.png" alt="3wqi9O.png" border="0" style="width:550px" />
也是关注两块：
1) procs，如果 r 的平均值大于系统 CPU 核数的 2 倍说明负载过高。
	r 表示进行 CPU 处理的进程数。
	b 表示进行 I/O 处理的进程数。
2) cpu，如果 us + sy 大于 80% 说明 CPU 资源不足。
	us 表示用户进程消耗 CPU 时间百分比。
	sy 表示内核进程消耗 CPU 时间百分比。
3. **pidstat**，可以监控具体进程的 CPU 资源。
<img src="https://s2.ax1x.com/2020/02/27/3wj0oR.png" alt="3wj0oR.png" border="0" style="width:500px" />
也可以监控内存资源。
<img src="https://s2.ax1x.com/2020/02/27/3wvyAs.png" alt="3wvyAs.png" border="0" style="width:500px" />
4. **free**，可以监控内存。
<img src="https://s2.ax1x.com/2020/02/27/3wxebQ.png" alt="3wxebQ.png" border="0" style="width:600px" />
一般 free 小于 20% 说明内存不足。
5. **df**，可以查看磁盘占用情况。
<img src="https://s2.ax1x.com/2020/02/27/3wxvR0.png" alt="3wxvR0.png" border="0" style="width:400px"/>
6. **iostat**，可以监控磁盘 I/O 。
<img src="https://s2.ax1x.com/2020/02/27/30Sdc6.png" alt="30Sdc6.png" border="0" />
util 表示磁盘 I/O 带宽占用百分比。
7. **ifstat**，可以查看网络 I/O 。
<img src="https://s2.ax1x.com/2020/02/27/30pGKf.png" alt="30pGKf.png" border="0" style="width:200px"/>
8. **netstat**，可以查看协议相关的信息。

了解了常用的系统监控命令，再结合 java 的命令，就可以定位程序问题了。举个例子。
```
public class CPUTest {
    public static void main(String[] args) {
        new Thread(()->{
            try {
                Thread.sleep(Integer.MAX_VALUE);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(()->{
            while (true) {
                int i = new Random().nextInt();
            }
        }).start();
        System.out.println("program started...");
    }
}
```
这个程序有 2 个线程，其中一个休眠，一个一直进行 cpu 操作。在服务器上运行，在使用系统监控命令，可以看到系统的资源情况。
<img src="https://s2.ax1x.com/2020/02/27/30F6US.png" alt="30F6US.png" border="0" style="width:700px"/>
进程 id=330 的 Java 程序 CPU 已经 100% 。
<img src="https://s2.ax1x.com/2020/02/27/30knVf.png" alt="30knVf.png" border="0" style="width:500px"/>
我们使用 ps 可以查看 CPU 占用 100% 是由 id=343 的线程造成的。接下来我们将 343 转成 16 进制。
<img src="https://s2.ax1x.com/2020/02/27/30Apyn.png" alt="30Apyn.png" border="0" style="width:400px"/>
接下来我们就可以使用 jstack 来查看堆栈信息了。
<img src="https://s2.ax1x.com/2020/02/27/30Asfg.png" alt="30Asfg.png" border="0" />
找到对应的 nid 即可找到有问题的代码了。