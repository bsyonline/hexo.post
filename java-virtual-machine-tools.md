---
title: Java Virtual Machine Tools
tags:
  - JVM
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2018-09-12 10:14:48
thumbnail:
---



### jps

jps (JVM Process Status Tool) 列出正在运行的 JVM 的进程，以及执行主类的名称和进程的本地虚拟机 id 。jps 通常是查看 jvm 信息的第一步。

```
$ jps -help
Usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
    
Options:
       jps 命令支持下列选项，使用这些选项会改变 jsp 的输出内容。
       -q  显示 JVM ID
       -m  显示 main 方法的参数
       -l  显示 main 方法的包名或 jar 文件路径
       -v  显示 JVM 参数
       -V  等同与 jps
```

jps 的使用示例：

```
$ jps
4611 SubApplication
25782 Main
7415 Jps
4504 RemoteMavenServer
4072 Main
24617 EurekaServerApplication
26297 ZuulApplication
25835 Main
```

### jstat

jstat (JVM Statistics Monitor Tool) 是用于监视 JVM 的运行状态信息的命令行工具。

```
$ jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
		

Options:
   General Options
       	General Options 只有两个选项
       	-help 帮助
       	-options	显示 statis 选项
   Output Options
       	输入选项决定 stat 命令的输出内容和格式。stat 命令的输出是类似表格，列之间用空格分隔。选项可以组合使用。
		class: 类加载器行为监视
		compiler: Java HotSpot VM Just-in-Time 编译器行为监视
		gc: GC 行为监视
		gccapacity: GC 容量和占用空间监视
		gccause: 当前或上一次 GC 监视及原因
		gcnew: 新生代行为监视
		gcnewcapacity: 新生代容量和占用空间监视
		gcold: 老年代行为监视
        gcoldcapacity: 老年代容量和占用空间监视
		gcmetacapacity: metaspace 大小监视，metaspace 就是原来的持久代，从 jdk8 开始使用
		gcutil: 垃圾回收监视汇总
		printcompilation: 显示被 JIT 编译的方法
```

示例：每 250 毫秒查询一次 GC 的情况一共查看 20 次
```
$ jstat -gc 28511 250 20
```

Stat Options 说明


*  class 选项
   * Loaded: 加载的类的数量
   * Bytes: 加载的类的大小（kBs）
   * Unloaded: 未加载的类的数量
   * Bytes: 未加载的类的大小 （Kbytes）
   * Time: 加载或卸载所消耗的时间

```
$ jstat -class 28511
Loaded  Bytes  Unloaded  Bytes     Time   
 14547 26373.1        0     0.0      17.13
```

* compiler 选项
   * Compiled: 执行 JIT 编译任务的数量
   * Failed: JIT 编译任务失败的数量
   * Invalid: JIT 编译任务不合法的数量
   * Time: 执行 JIT 编译任务的时间
   * FailedType: 上一个编译失败的类型
   * FailedMethod: 上一次编译失败的类和方法名字

```
$ jstat -compiler 28511
Compiled Failed Invalid   Time   FailedType FailedMethod
    7910      2       0    60.44          1 java/net/URLClassLoader$1 run
```

* gc 选项
   * S0C: Survivor 0 容量 (kB)
   * S1C: Survivor 1 容量 (kB)
   * S0U: Survivor 0 使用量 (kB)
   * S1U: Survivor 1 使用量 (kB)
   * EC: Eden 区容量 (kB)
   * EU: Eden 区使用量 (kB)
   * OC: 老年代容量 (kB)
   * OU: 老年代使用量 (kB)
   * MC: Metaspace 容量 (kB)
   * MU: Metacspace 使用量 (kB)
   * CCSC: 类压缩空间容量 (kB) 也是 Java8 新出的概念，配合 matespace 使用
   * CCSU: 类压缩空间使用量 (kB)
   * YGC: Young GC 次数
   * YGCT: Young GC 消耗时间
   * FGC: Full GC 次数
   * FGCT: Full GC 消耗时间
   * GCT: GC 总耗时

```
$ jstat -gc 28511 
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
6144.0 17408.0  0.0   17393.6 997888.0 919208.4  226304.0   43056.4   75136.0 73724.1 9856.0 9576.9     21    0.279   3      0.451    0.731
```

* gccapacity 选项
   * NGCMN: 新生代最小容量 (kB)
   * NGCMX: 新生代最大容量 (kB)
   * NGC: 新生代容量 (kB)
   * S0C: Survivor 0 容量 (kB)
   * S1C: Survivor 1 容量 (kB)
   * EC: Eden 区容量 (kB)
   * OGCMN: 老年代最小容量 (kB)
   * OGCMX: 老年代最大容量 (kB)
   * OGC: 老年代容量 (kB)
   * OC: 老年代容量 (kB)
   * MCMN: 元空间最小容量 (kB)
   * MCMX: 元空间最大容量 (kB)
   * MC: 元空间容量 (kB)
   * CCSMN: 类压缩空间最小容量 (kB)
   * CCSMX: 类压缩空间最大容量 (kB)
   * CCSC: 类压缩空间容量 (kB)
   * YGC: Young GC 次数
   * FGC: Full GC 次数

```
$ jstat -gccapacity 28511 
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC 
 84480.0 1344000.0 1076736.0 6144.0 17408.0 997888.0   169472.0  2688512.0   226304.0   226304.0      0.0 1114112.0  75136.0      0.0 1048576.0   9856.0     21     3
```

* gccause 选项
和 gcutil 选项相同，但是多了下面两个选项
  * LGCC: 上一次 GC 原因
  * GCC: 当前 GC 原因

```
$ jstat -gccause 28511 
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
  0.00  99.92  92.12  19.03  98.12  97.17     21    0.279     3    0.451    0.731 Allocation Failure   No GC 
```

* gcnew 选项
   * S0C: Survivor 0 容量 (kB)
   * S1C: Survivor 1 容量 (kB)
   * S0U: Survivor 0 使用量 (kB)
   * S1U: Survivor 1 使用量 (kB)
   * TT: 新生代需要经历多少次 GC 晋升到老年代
   * MTT: 新生代需要经历多少次 GC 晋升到老年代中的最大阈值
   * DSS: 期望的 Survivor 大小 (kB)
   * EC: Eden 区容量 (kB)
   * EU: Eden 区使用量 (kB)
   * YGC: Young GC 次数
   * YGCT: Young GC 耗时

```
$ jstat -gcnew 28511 
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT  
6144.0 17408.0    0.0 17393.6  4  15 20992.0 997888.0 919208.4     21    0.279
```

* gcnewcapacity 选项
   * NGCMN: 新生代最小容量 (kB)
   * NGCMX: 新生代最大容量 (kB)
   * NGC: 新生代容量 (kB)
   * S0CMX: Survivor 0 最大容量 (kB)
   * S0C: Survivor 0 容量 (kB)
   * S1CMX: Survivor 1 最大容量 (kB)
   * S1C: Survivor 1 容量 (kB)
   * ECMX: Eden 区最大容量 (kB)
   * EC: Eden 区容量 (kB)
   * YGC: Young GC 次数
   * FGC: Full GC 次数

```
$ jstat -gcnewcapacity 28511 
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC 
   84480.0  1344000.0  1076736.0 448000.0   6144.0 448000.0  17408.0  1342976.0   997888.0    21     3
```

* gcold 选项
   * MC: 元空间容量 (kB)
   * MU: 元空间使用量 (kB)
   * CCSC: 类压缩空间容量 (kB)
   * CCSU: 类压缩空间使用量 (kB)
   * OC: 老年代容量 (kB)
   * OU: Old space 使用量 (kB)
   * YGC: Young GC 次数
   * FGC: Full GC 次数
   * FGCT: Full GC 耗时
   * GCT: GC 总耗时

```
$ jstat -gcold 28511 
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT   
 75136.0  73724.1   9856.0   9576.9    226304.0     43056.4     21     3    0.451    0.731
```

* gcoldcapacity 选项
   * OGCMN: 老年代最小容量 (kB)
   * OGCMX: 老年代最大容量 (kB)
   * OGC: 老年代容量 (kB)
   * OC: Old space 容量 (kB)
   * YGC: Young GC 次数
   * FGC: Full GC 次数
   * FGCT: Full GC 耗时
   * GCT: GC 总耗时

```
$ jstat -gcoldcapacity 28511 
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT   
   169472.0   2688512.0    226304.0    226304.0    21     3    0.451    0.731
```

* gcmetacapacity 选项
    * MCMN: 元空间最小容量 (kB)
    * MCMX: 元空间最大容量 (kB)
    * MC: 元空间容量 (kB)
    * CCSMN: 类压缩空间最小容量 (kB)
    * CCSMX: 类压缩空间最大容量 (kB)
    * YGC: Young GC 次数
    * FGC: Full GC 次数
    * FGCT: Full GC 耗时
    * GCT: GC 总耗时

```
$ jstat -gcmetacapacity 28511 
   MCMN       MCMX        MC       CCSMN      CCSMX       CCSC     YGC   FGC    FGCT     GCT   
       0.0  1114112.0    75136.0        0.0  1048576.0     9856.0    21     3    0.451    0.731
```

* gcutil 选项
    * S0: Survivor 0 使用量对空间容量占比
    * S1: Survivor 1 使用量对空间容量占比
    * E: Eden 区使用量对空间容量占比
    * O: 老年代使用量对空间容量占比
    * M: 元空间使用量对空间容量占比
    * CCS: 类压缩空间使用量对空间容量占比
    * YGC: Young GC 次数.
    * YGCT: Young generation garbage collection time.
    * FGC: Full GC 次数.
    * FGCT: Full GC 耗时.
    * GCT: GC 总耗时.

```
$ jstat -gcutil 28511 
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
  0.00  99.92  94.69  19.03  98.12  97.17     21    0.279     3    0.451    0.731
```

* printcompilation 选项
  * Compiled: 最近编译的方法中 JIT 编译任务数量
  * Size: 最近编译方法字节码的字节数
  * Type: 最近编译的方法的编译类型
  * Method: 标识最近编译的方法的类名和方法名，类名使用 ``/`` 代替 ``.`` ，类名和方法名的格式和 HotSpot -XX:+PrintCompilation 选项一致

```
$ jstat -printcompilation 28511 
Compiled  Size  Type Method
    7913     21    1 java/net/InetAddress$InetAddressHolder init
```





### jstack
jstack (Stack Trace for Java) 用于生成 JVM 当前时刻的线程快照（threaddump 或 javacore）。线程快照是当前 JVM 每个线程正在执行的方法的堆栈集合，生成线程快照的目的是定位线程出现长时间停顿的原因，比如死锁。用法如下：
```
$ jstack -help
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  强制，线程无响应时使用
    -m  输出 java 和 native 堆栈信息
    -l  显示锁信息
    -h or -help to print this help message
```




### jinfo
jinfo (Configuration Info for Java) 的作用是查看调整 JVM 参数。用法如下：
```
$ jinfo -help
Usage:
    jinfo [option] <pid>

where <option> is one of:
    -flag <name>         打印指定名称的参数
    -flag [+|-]<name>    打开或关闭参数
    -flag <name>=<value> 设置参数
    -flags               打印所有参数
    -sysprops            打印 Java system properties
    <no option>          打印 flags 和 sysprops
```
如果我们想查看某进程的 GC 日志信息，可以使用命令
```
$ jinfo -flag GCLogFileSize 14801
-XX:GCLogFileSize=8192
```


### jmap

jmap (Memory Map for Java) 用于生成堆的存储快照（dump 文件），查询 finalize 执行队列， Java 堆和永久代信息。用法如下：
```
$ jmap -help
Usage:
    jmap [option] <pid>

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                显示堆的详细信息
    -histo[:live]        显示堆的统计信息
    -clstats             信息 classloader 的统计信息
    -finalizerinfo       显示等待执行 finalize 方法的对象
    -dump:<dump-options> 生成堆的存储快照
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   强制生成 dump
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

如果我们想生成进程的 dump 文件，可以使用
```
$ jmap -dump:file=14801.dump 14801 
Dumping heap to /home/root/14801.dump ...
Heap dump file created
```

### jhat
jhat （JVM Heap Analysis Tool） 是用来分析 dump 文件的，通常和 jmap 搭配使用。
```
$ jhat -help
Usage:  jhat [-stack <bool>] [-refs <bool>] [-port <port>] [-baseline <file>] [-debug <int>] [-version] [-h|-help] <file>

  -J<flag>          Pass <flag> directly to the runtime system. For
        example, -J-mx512m to use a maximum heap size of 512MB
  -stack false:     Turn off tracking object allocation call stack.
  -refs false:      Turn off tracking of references to objects
  -port <port>:     Set the port for the HTTP server.  Defaults to 7000
  -exclude <file>:  Specify a file that lists data members that should
        be excluded from the reachableFrom query.
  -baseline <file>: Specify a baseline object dump.  Objects in
        both heap dumps with the same ID and same class will
        be marked as not being "new".
  -debug <int>:     Set debug level.
          0:  No debug output
          1:  Debug hprof file parsing
          2:  Debug hprof file parsing, no server
  -version          Report version number
  -h|-help          Print this help and exit
  <file>            The file to read

For a dump file that contains multiple heap dumps,
you may specify which dump in the file
by appending "#<number>" to the file name, i.e. "foo.hprof#3".

All boolean options default to "true"
```
jmap 提供了一个 http 服务，可以通过浏览器查看 dump 文件信息
```
$ jhat 14801.dump 
Reading from 14801.dump...
Dump file created Mon Feb 18 17:10:37 CST 2019
Snapshot read, resolving...
Resolving 2366175 objects...
Chasing references, expect 473 dots.............
Eliminating duplicate references................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

访问 http://localhost:7000 可以查看 dump 信息，不过并不常用。

### HSDIS


### JConsole
JConsole (Java Monitoring and Management Console) 是一种基于 JMX 的可视化工具，主要针对 JMX MBean 进行管理。使用 jconsole 可以启动。

### VirtualVM

