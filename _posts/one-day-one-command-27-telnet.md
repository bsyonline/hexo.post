---
title: 每天一个 Linux 命令（27）： telnet
date: 2017-03-06 10:10:45
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

free 命令可以显示 Linux 系统中空闲的、已用的物理内存及 swap 内存,及被内核使用的 buffer，是最经常使用的系统监控工具之一。

<!-- more -->

free 命令格式：

```shell
Usage:
 free [options]
Options:
 -b, --bytes         以 bytes 为单位显示
 -k, --kilo          以 kilobytes 为单位显示
 -m, --mega          以 megabytes 为单位显示
 -g, --giga          以 gigabytes 为单位显示
     --tera          以 terabytes 为单位显示
 -h, --human         show human-readable output
     --si            use powers of 1000 not 1024
 -l, --lohi          show detailed low and high memory statistics
 -t, --total         显示 RAM + swap
 -s N, --seconds N   每 N 秒刷新显示
 -c N, --count N     刷新 N 次后退出
 -w, --wide          wide output
```

举例：

比较两个文件：

```shell
free -hw
              total        used        free      shared     buffers       cache   available
Mem:           7.5G        5.6G        327M        199M        385M        1.2G        1.3G
Swap:          7.6G        3.2G        4.5G

--
total:总计物理内存的大小。
used:已使用多大。
free:可用有多少。
Shared:多个进程共享的内存总额。
Buffers/cached:磁盘缓存的大小。
```
> buffers 和 cached 的区别
>
> 为了提高磁盘存取效率, Linux做了一些精心的设计, 除了对dentry进行缓存(用于VFS,加速文件路径名到inode的转换), 还采取了两种主要Cache方式：Buffer Cache和Page Cache。前者针对磁盘块的读写，后者针对文件inode的读写。这些Cache有效缩短了 I/O系统调用(比如read,write,getdents)的时间。
>
> 磁盘的操作有逻辑级（文件系统）和物理级（磁盘块），这两种Cache就是分别缓存逻辑和物理级数据的。
>
> Page cache实际上是针对文件系统的，是文件的缓存，在文件层面上的数据会缓存到page cache。文件的逻辑层需要映射到实际的物理磁盘，这种映射关系由文件系统来完成。当page cache的数据需要刷新时，page cache中的数据交给buffer cache，因为Buffer Cache就是缓存磁盘块的。但是这种处理在2.6版本的内核之后就变的很简单了，没有真正意义上的cache操作。
>
> Buffer cache是针对磁盘块的缓存，也就是在没有文件系统的情况下，直接对磁盘进行操作的数据会缓存到buffer cache中，例如，文件系统的元数据都会缓存到buffer cache中。
>
> 简单说来，page cache用来缓存文件数据，buffer cache用来缓存磁盘数据。在有文件系统的情况下，对文件操作，那么数据会缓存到page cache，如果直接采用dd等工具对磁盘进行读写，那么数据会缓存到buffer cache。
>
> 所以我们看linux,只要不用swap的交换空间,就不用担心自己的内存太少.如果常常swap用很多,可能你就要考虑加物理内存了.这也是linux看内存是否够用的标准.
>
> 如果是应用服务器的话，一般只看第二行，+buffers/cache,即对应用程序来说free的内存太少了，也是该考虑优化程序或加内存了。

