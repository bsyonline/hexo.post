---
title: 每天一个 Linux 命令（24）： top
date: 2017-03-03 10:10:45
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

top 命令是 Linux 下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于 Windows 的任务管理器。

<!-- more -->

top 命令格式：

```shell
Usage:  top -hv | -abcHimMsS -d delay -n iterations [-u user | -U user] -p pid [,pid ...]
Options:
  -b					批处理
  -S					积累模式
  -i <时间>			   设置间隔时间
  -u <用户名> 			  指定用户名
  -p <进程号> 			  指定进程
  -n <次数>  			   循环显示的次数
```

举例：

比较两个文件：

```shell
top
```
> PID — 进程id
>
> USER — 进程所有者
>
> PR — 进程优先级
>
> NI — nice值。负值表示高优先级，正值表示低优先级
>
> VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES 
>
> RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
>
> SHR — 共享内存大小，单位kb
>
> S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
>
> %CPU — 上次更新到现在的CPU时间占用百分比
>
> %MEM — 进程使用的物理内存百分比
>
> TIME+ — 进程使用的CPU时间总计，单位1/100秒
>
> COMMAND — 进程名称（命令名/命令行）

显示完整命令：

```shell
top -c
```
以批处理模式显示程序信息：

```shell
top -b
```

以累积模式显示程序信息：

```
top -S
```

设置信息更新次数：

```
top -n 2
```

设置信息更新时间：

```
top -d 3
```

