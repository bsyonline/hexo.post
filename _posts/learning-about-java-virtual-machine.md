---
title: Learning about Java Virtual Machine (JVM)
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2099-08-13 19:07:16
thumbnail:
---



### 1. JVM 架构

Java Virtual Mechine （JVM）包含三部分，分别是：类加载器子系统、运行时数据区和执行引擎。

<img src="https://s2.ax1x.com/2020/03/11/8A2ZHH.png" alt="8A2ZHH.png" border="0" style="width:500px">

#### 1.1 类加载子系统

类加载是将 ```.class``` 文件加载到虚拟机内存并创建对象的整个过程。类加载子系统提供 Java 的类动态加载功能，它有三个主要阶段：加载、链接和初始化。  

##### 加载

加载是 JVM 通过一个类的完全限定名来获取类的二进制字节流，将字节流代表的静态存储结构转化为方法区中的运行时数据结构，然后在内存中生成一个代表这个类的 Class 对象作为访问入口的过程。
在 Java 中有多种途径可以获取类的二进制字节流：

1. 从 zip 中获取，jar ，ear ，war 都属于这类。
2. 从网络中获取，比如 applet 。
3. 通过动态代理生成。
4. 通过文件生成，比如 jsp 被编译成 class 。
5. 等等。

加载既可以通过系统提供的类加载器加载，也可以通过自定义的类加载器加载。系统提供的类加载器有 3 个：Bootstrap ClassLoader 、Extention ClassLoader 和 Application ClassLoader 。

1. Bootstrap ClassLoader 负责加载 \lib 下的类。
2. Extention ClassLoader 负责加载 \lib\ext 下的类。
3. Application ClassLoader 负责加载应用程序级别的类，即系统类路径 ```classpath``` 下的类。

类加载器在加载类时使用委托算法。委托算法的原理是：如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载。

>除了顶层的类加载器，每个类加载器都要有父类加载器，这些加载器之间并不是继承关系，而是组合关系。

使用这种委托算法的优点有二：

1. 防止重复加载，父加载器加载过的类子类加载器就不用再加载一次
2. 安全，Java API 定义的类不会被篡改，因为父类已经加载了 Java 核心类，如果我们要加载自己修改的核心类，父加载器将不会加载。

##### 链接

链接包含三个步骤：验证、准备和解析。

1. 验证
   验证的目的在于确保 Class 文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。虽然一些非法的操作（比如将一个类型转成一个不存在的类型，或者将代码跳转到不存在行）在编译期间编译器拒绝编译，但是 JVM 并没有限制字节码一定要由 java 编译器编译生成，所以理论上甚至可以通过直接编写生成 class 文件，所以验证是非常重要的。
   验证包括四种：
   &nbsp;&nbsp;&nbsp;&nbsp;1) 文件格式验证，验证是否符合 class 文件格式规范。
   &nbsp;&nbsp;&nbsp;&nbsp;2) 元数据验证，对元数据信息进行语义分析，比如是否有父类，是否可以继承等。
   &nbsp;&nbsp;&nbsp;&nbsp;3) 字节码验证，通过数据流和控制流分析程序的合法性，比如在栈上放一个 int 类型的数据，使用时按 long 类型加载。
   &nbsp;&nbsp;&nbsp;&nbsp;4) 符号引用验证，对类自身信息以外的信息进行校验，比如常量池。
2. 准备
   准备是为类变量（即 ```static``` 修饰的字段变量）分配内存并且设置类变量的初始值的阶段。例如 ```static int i=5;``` 这里只将 i 初始化为 0 ，至于 5 的值将在初始化时赋值。这里不包含用 ```final``` 修饰的 ```static```，因为 ```final``` 在编译的时候就会分配了，注意这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到 Java 堆中。
3. 解析
   解析是将常量池中的符号引用替换为直接引用的过程。**符号引用**就是一组符号来描述目标，可以是任何字面量，而**直接引用**就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

>符号引用转化成直接引用分为两种：在解析阶段转化叫静态解析，运行期转化叫动态连接。

##### 初始化

初始化是类加载的最后阶段，将为所有静态变量分配原始值，并执行静态块。
在 JVM 规范中规定，有且只有 5 种情况必须进行初始化：

1. 遇到 new ，getstatic ，putstatic 和 invokestatic 4 个字节码指令时如果没有初始化，则必须进行初始化。也就是在使用 new 实例化对象时，读写类的静态变量时，以及调用静态方法时。
2. 使用反射调用时。
3. 在初始化一个类且它的父类还没有被初始化时。
4. 虚拟机启动执行主类时。
5. 使用 MethodHandle 解析 REF_getStatic ，REF_putStatic 和 REF_invokeStatic 的方法句柄时。

#### 1.2 运行时数据区

##### 1.2.1 程序寄存器

程序寄存器（Program Counter）线程私有。

存储当前线程执行程序的字节码指令的行号。

JVM 未规定异常。

##### 1.2.2 虚拟机栈

虚拟机栈（Java Vitrual Machine Stacks）线程私有。
存储使用栈帧来保存局部变量表、操作数栈、动态链接、方法出口等。

JVM 规定了两种异常：StackOutOfMemoryError 和 OutOfMemoryError 。

##### 1.2.3 本地方法栈

本地方法栈（Native Method Stack）本地方法栈和虚拟机栈类似，区别是本地方法栈用于执行本地方法。

##### 1.2.4 堆

堆（Java Heap）是共享的，是 Java 内存最大的区域。Java 内存回收主要发生在堆，按照分代收集算法进行内存回收。堆内存可以细分为新生代和老年代。新生代又分为 Eden 、From Survivor 和 To Survivor 。堆是逻辑连续的，物理可以不连续。
JVM 规定的异常为 OutOfMemoryError 。

###### JAVA 堆的结构

[https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)

堆被分成若干部分：YoungGen ，OldGen ，parmGen 。
<img src="https://s2.ax1x.com/2020/02/23/3l3SPA.png" alt="20200223093804" border="0"  style="width:400px">
上图是 hotspot 的示意图，oracle 为了整合多种 jvm 产品，在 java8 中使用 metaspace 替代了 PerGen 。

##### 1.2.5 方法区

方法区（Method Area）线程共享的。
存储已被 JVM 加载的类信息、常量、静态变量、及时编译器编译后的代码。
JVM 规定的异常为 OutOfMemoryError 。


#### 1.3 执行引擎

执行引擎用来执行分配给运行时数据区的字节码。
执行引擎分为：解释器、JIT 编译器和垃圾回收器。

##### 1.3.1 解释器

解释会读取字节码，解释并逐行执行。解释器解释的速度很快但是执行速度很慢，而且一个方法被调用多次，解释过程也会进行多次。

##### 1.3.2 JIT 编译器

JIT 编译器使用解释器进行字节码转换，遇到重复的代码，JIT 编译器会将整个字节码编译成机器码。机器码将用于重复方法的调用，从而提高性能。

##### 1.3.3 垃圾回收器

垃圾回收器用于收集 ```new``` 产生的对象。垃圾回收可以通过 ```System.gc()``` 来触发，但是不一定会立刻执行。不是用 ```new``` 生成的对象可以使用 ```finalize()``` 进行回收。



### 2. JVM 工具

#### 2.1 jps

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

#### 2.2 jstat

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





#### 2.3 jstack

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




#### 2.4 jinfo

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


#### 2.5 jmap

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

#### 2.6 jhat

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

#### 2.7 HSDIS


#### 2.8 JConsole

JConsole (Java Monitoring and Management Console) 是一种基于 JMX 的可视化工具，主要针对 JMX MBean 进行管理。使用 jconsole 可以启动。

#### 2.8 VirtualVM

### 3. JVM 选项

JVM 的选项有 3 种：

1. 基本选项，如 -version。
2. 非基本选项，即 -X 选项，如 -Xmx，-Xms，-Xmn 等。
3. 高级选项，即 -XX 选项。高级选项有两种类型，一种是 boolean 类型的，用 -XX:+ 或 -XX:- 表示，如 -XX:+PrintGCDetails ，另一种是值类型，如 -XX:ParallelGCThreads=2 。

#### 3.1 常用的高级选项

比较常用重要的有以下几个：

1. -XX:InitialHeapSize，初始堆内存大小，等价于 -Xms 。
2. -XX:MaxHeapSize，最大堆内存大小，等价于 -Xmx 。
3. -XX:NewSize，新生代初始内存，等价于 -Xmn 。
4. -XX:MaxNewSize，新生代最大内存。
5. -XX:ThreadStackSize，线程栈的内存大小，等价于 -Xss 。
6. -XX:SurvivorRatio=ratio，eden 和 survivor 的比例，默认为 8 ，即 eden:s0:s1 = 8:1:1 。
7. -XX:NewRatio=ratio，young 和 old 的比例，默认为 2 ，即 young 占 1/3 。
8. -XX:MaxTenuringThreshold=threshold，经过多少次 young GC 才能到 ParOldGen ，默认是 15 ，可以设置范围 0-15 。
9. -XX:+UseParallelGC，使用 ParallelGC 垃圾收集器。
10. -XX:+PrintCommandLineFlags，输出 JVM 选项。

在进行 JVM 调优之前首先需要确定当前环境 JVM 各项参数的值是什么。我们可以用以下几种方式查看 JVM 的参数配置。

#### 3.2 查看默认 JVM 参数的方法

1. 使用 jinfo 

查看某个具体的参数

```
D:\Dev\IdeaProjects\microlabs\effictive-java>jinfo -flag MaxHeapSize 19280
-XX:MaxHeapSize=4267704320
```

查看所有的参数

```
# jinfo -flags 19280
Attaching to process ID 19280, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.221-b11
Non-default VM flags: -XX:CICompilerCount=4 -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4267704320 -XX:MaxNewSize=1422393344 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=89128960 -XX:OldSize=179306496 -
XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
Command line:  -XX:+PrintCommandLineFlags -javaagent:D:\Program Files\JetBrains\IntelliJ IDEA 2019.2\lib\idea_rt.jar=5195:D:\Program Files\JetBrains\IntelliJ IDEA 2019.2\bin -Dfile.encoding=UTF-8
```

2. 使用 -XX:+PrintCommandLineFlags

```
# java -XX:+PrintCommandLineFlags -version
-XX:G1ConcRefinementThreads=8 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=266639552 -XX:MaxHeapSize=4266232832 -XX:+PrintCommandLineFlags -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCa
che -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation
openjdk version "11.0.3" 2019-04-16
OpenJDK Runtime Environment (build 11.0.3+12-b304.10)
OpenJDK 64-Bit Server VM (build 11.0.3+12-b304.10, mixed mode, sharing)
```

3. 使用 -XX:+PrintFlagsInitial 或 -XX:+PrintFlagsFinal

```
# java -XX:+PrintFlagsInitial -version
[Global flags]
ccstrlist AOTLibrary                               =                                           {product} {default}
      int ActiveProcessorCount                     = -1                                        {product} {default}
    uintx AdaptiveSizeDecrementScaleFactor         = 4                                         {product} {default}
    uintx AdaptiveSizeMajorGCDecayTimeScale        = 10                                        {product} {default}
    uintx AdaptiveSizePolicyCollectionCostMargin   = 50                                        {product} {default}
	…
```

-XX:+PrintFlagsInitial 显示的是初始值，-XX:+PrintFlagsFinal 显示的是最终的值，:= 表示修改过。

```
$ java -XX:+PrintFlagsInitial -version | grep UseParallelGC
     bool UseParallelGC                             = false                               {product}
$ java -XX:+PrintFlagsFinal -version | grep UseParallelGC
     bool UseParallelGC                            := true                                {product}
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

JVM 的参数配置不是固定的，默认值也会随机器，操作系统不同而不同，所以还是通过以上几种方式来查看比较好。

### 4. 垃圾回收

#### 4.1 GC的过程

1. 最初 eden 、form 、to 都是空的，新创建的对象首先放到 eden 。
   <img src="https://s2.ax1x.com/2020/02/23/3l1vUH.png" alt="20200223093404" border="0" style="width:400px">
2. 对象不断放入 eden ，当 eden 满时触发 minor GC 。
   <img src="https://s2.ax1x.com/2020/02/23/3l1jVe.png" alt="20200223093458" border="0" style="width:400px">
   将 eden 中的对象移动到 s0 ，然后清空 eden 。
   <img src="https://s2.ax1x.com/2020/02/23/3l1ObD.png" alt="20200223093528" border="0" style="width:400px">
3. 新创建的对象继续放到 eden ，当 eden 满时触发第二次 minor GC 。将 eden 中的对象移动到 s1 ，同时将 s0 中的对象也移动到 s1 ，同时对象年龄 +1 ，最后将 eden 和 s0 清空。
   <img src="https://s2.ax1x.com/2020/02/23/3l1LDO.png" alt="20200223093551" border="0" style="width:400px">
4. 当第三次 minor GC 时，eden 和 s1 中的对象会被移动到 s0 并且对象年龄 +1 ，同时将清空 eden 和 s1 ，如此往复。
   <img src="https://s2.ax1x.com/2020/02/23/3l1qKK.png" alt="20200223093611" border="0" style="width:400px">
5. 当若干次 minor GC 之后，当对象的年龄到达阈值时，会移动到 OldGen 。
   <img src="https://s2.ax1x.com/2020/02/23/3l1x5d.png" alt="20200223093639" border="0" style="width:400px">
6. 对象不断晋升到 OldGen ，当 OldGen 满时会触发 major GC 来清理 OldGen。



#### 4.2 GC 信息怎么看

规律就是： GC前内存大小->GC后内存大小(总内存大小)


<img src="https://s2.ax1x.com/2020/02/21/3uzCuQ.png" alt="20200221210744" border="0">

<img src="https://s2.ax1x.com/2020/02/21/3uzPBj.png" alt="20200221210808" border="0">

### 

#### 4.3 垃圾对象判断

垃圾回收第一步就是确定哪些对象需要被回收，方法有 2 种：

##### 4.3.1 引用计数

引用计数就是在全局维护一个 Map 保存对象的引用，当引用一个对象的时候，就给对应的 value 加 1 ，当这个引用失效的时候，就将对应的 value 减 1 。当 value 的值为 0 的时候，就说明此刻该对象是垃圾对象，可以被回收掉。引用计数的缺点：
1） 无法解决循环引用的问题。
2） 引用计数需要实时更新，开销大。

##### 4.3.2 可达性分析

可达性分析就是从 GC root 开始遍历，能遍历到的对象就不会被回收。hotspot 的定义的 GC roots 有：
1） 虚拟机栈中引用的对象（栈帧中局部变量表中的对象）。
2） 方法区中静态属性和常量引用的对象。
3） 本地方法栈中的对象。

#### 4.4 垃圾收集算法

##### 4.4.1 标记-清除算法

标记清除算法是最基本的垃圾收集算法。算法分为标记和清除两个阶段。首先，标记出所有需要回收的对象，在标记完后统一回收所有被标记的对象。标记清楚算法缺点：
1）效率不高；
2）清除后会产生大量不连续的内存碎片，碎片太多，会导致在创建大对象时没有足够的连续内存，会再次触发垃圾回收。

##### 4.4.2 复制算法

复制算法是为了解决标记清除算法的效率问题提出的。复制算法将内存分为大小相等的两份，每次使用其中一份。当其中一块用完了，就将对象复制到另一块上，然后再把原来的内存一次清除。这样每次都对一整块内存进行操作，不会出现内存碎片的问题，简单高效，但是缺点也很明显，就是每次只有一半的内存可以使用。复制算法在新生代的垃圾回收中被大量采用，新生代被分为 Eden 和 Survivor 。当进行垃圾回收时， Eden 和其中一个 Survivor 的对象会被复制到另一个 Survivor 中。如果 Survivor 空间不够，对象会进入 Old 区，这被称为分配担保（Handle Promotion）。

##### 4.4.3 标记-整理算法

由于复制算法的特点，一般不会在 Old 区使用。根据 Old 区的特点，提出了另一种标记-整理算法。首先对对象进行标记，然后将对象向一端移动，然后清理掉边界以外的内存空间。

##### 4.4.4 分代收集算法

分代收技算法是根据对象的存活时间，将内存分为几块，根据不同区域的特点选择不同的垃圾回收算法。如新生代中对象创建和死亡的频率很高，所以大多采用复制算法。老年代没有额外的内存进行分配担保，所以大多使用标记-删除和标记-整理算法。

#### 4.5 垃圾收集器

垃圾收集器是对垃圾收集算法的实现，垃圾收集器可以搭配使用。

##### 4.5.1 年轻代垃圾收集器

###### Serial 收集器

Serial 收集器是单线程的，它会在进行垃圾回收时，会暂停其他所有线程（stop-the-world）。Serial 收集器简单高效，虽然会造成停顿，但是在 Client 模式下，Serial 收集器是一个不错的选择。

###### ParNew 收集器

ParNew 是 Serial 的多线程版本，由于 CMS 只能搭配 ParNew 使用，所以 ParNew 是 Server 模式下的新生代收集器首选。

###### Parallel Scavenge 收集器

Parallel Scavenge 是新生代收集器，使用复制算法，并行的多线程收集器。Parallel Scavenge 的衡量标准是吞吐量。吞吐量是 CPU 用于运行用户代码与 CPU 总消耗时间的比值。

##### 4.5.2 老年代垃圾收集器

###### Serial Old 收集器

Serial Old 是 Serial 的老年代版本。会暂停所有线程，使用标记整理算法。

###### CMS 收集器

Concurrent Mark Sweep 目标是减少停顿。使用标记清除算法。

CMS 收集器执行过程分为 5 个阶段：

1. Initial Mark(Stop the World Event)
   暂停较短时间，标记可达对象。
2. Concurrent Marking
   应用不暂停，从 GC root 开始标记可达对象。
3. Remark(Stop the World Event)
   查找在并发标记过程中遗漏的对象。
4. Concurrent Sweep
   应用不暂停，清除不可达对象。
5. Resetting
   为下一次收集做准备。

最初堆中 eden，survivor，old 都是空的。
<img src="https://s2.ax1x.com/2020/02/23/3l4vPe.png" alt="20200223122552" border="0" style="width:300px">
当 young GC 触发，存活的对象将从 eden 和 survivor 拷贝到另一个 survivor ，到达年龄阈值的对象晋升到 old 。

<img src="https://s2.ax1x.com/2020/02/23/3l4x8H.png" alt="20200223122351" border="0" style="width:500px">

young GC 完成后，eden 和一个 survivor 被清空。

<img src="https://s2.ax1x.com/2020/02/23/3l4z2d.png" alt="20200223125905" border="0" style="width:500px">
当 old 到达一定容量触发 CMS 。
1） 首先进行 **Initial Mark** 阶段，标记出可达对象。
2） 短暂暂停之后，进入 **Concurrent Marking** 阶段，应用程序继续执行，CMS 同时找到可达的对象。
3） 在 **Remark** 阶段，找到并发标记期间丢失的对象。
4） 进入 **Concurrent Sweep** 阶段，释放掉未被标记的对象。
5） 释放之后，进入 **Resetting** 阶段，一次 CMS GC 完成。

<img src="https://s2.ax1x.com/2020/02/23/3l4X5D.png" alt="20200223125952" border="0" style="width:500px">

3. Parallel Old 收集器

Parallel Old 是 Parallel Scavenge 的老年代版本，搭配 Parallel Scanvenge 使用。

##### 4.5.3 G1 收集器

###### 什么是 G1 ?

[https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html)

Garbage-First (G1) 垃圾收集器是服务器端的垃圾收集器，适用于大内存多处理器的服务器。G1 的目标是替代 CMS ，和 CMS 相比，G1 具有 2 个优势：
1） G1 是压缩的垃圾收集器，可以避免内存碎片。
2） G1 支持垃圾收集暂停时间预测，时间可以由用户设置。

###### 内存结构

和之前的垃圾收集器架构不同，G1 使用了另一种架构。

<img src="https://s2.ax1x.com/2020/02/23/3l6Bad.png" alt="3l6Bad.png" border="0" style="width:500px"/>

堆内存被分成若干大小相等的 region ，大小 1-32Mb 不等，数量大约 2000 个，每个 region 中都有 eden，survivor ，old ，humogous 和 unused 。G1 的内存分配是一种逻辑划分，不一定是连续的。

###### GC 过程

1. Initial Mark(Stop the World Event)
   应用暂停，标记 survivor region （root region），这些 region 可能含有 old gen 的引用。
2. Root Region Scanning
   应用不暂停，扫描 root region 获取 old 的引用。
3. Concurrent Marking
   查找存活的对象。
4. Remark(Stop the World Event)
   完成存活对象的标记，使用的是 snapshot-at-the-beginning(SATB) 算法。
5. Cleanup(Stop the World Event and Concurrent)
   首先，统计存活对象和完全空闲的 region (Stop the World Event) 。第二步，清除记录的区域 (Stop the World Event) 。最后，将空白的 region 重置，然后加入到空闲列表 (Concurrent) 。
6. Copying(Stop the World Event)
   将存活的对象拷贝到 unused region 。


### 5. 串行，并行和并发

串行：应用和 GC 是顺序执行的，在 GC 的时候应用需要停止，只有一个线程在 GC 。
并行：应用和 GC 是顺序执行的，在 GC 的时候应用需要停止，有多个线程在 GC 。
并发：应用和 GC 是同时执行的，可能同时执行，也可能交替执行。





### 6. JIT

#### 6.1 JIT 是什么？ 

JIT(Just-In-Time Compilation) 即时编译，是一段代码在第一次将要被执行之前，进行的编译。属于动态编译（运行时编译）的范畴，与之相对的是 AOT(ahead-of-time Compilation)  也叫静态编译。

java 文件经过 javac 编译成字节码文件，再由 JVM 的解释器 interpreter 解释执行。当 JVM 发现某段代码执行地非常频繁，就会认为这段代码是热点代码 (hot spot code) 。JVM 就会将这些代码编译成机器指令，这样就可以直接调用机器指令执行，而不用再进行解释执行。这个将热点代码编译成机器指令的过程就是 JIT 编译，是通过 JIT 编译器来完成的。

**什么样的代码算热点代码?**

1. 被多次调用的代码，如果调用次数到达阈值，就会被加入到编译队列中等待编译，这种编译叫标准编译。
2. 循环体，如果方法中有很长的循环，当计数器到达阈值，整个方法也会被编译，这种编译叫栈上替换 (On Stack Replacement, OSR) 。

**为什么 JIT 不是针对所有代码，而只是热点代码？**

因为 JIT 也是有成本的。

1. 首先是时间成本，JIT 编译可能非常耗时，并且不是 100% 能够获取运行速度提升，即便获得运行速度提升，也不一定能抵消 JIT 编译的时间消耗，而且有些代码可能就只执行一次，所以相对来说还是解释执行的成本更低。
2. 其次是空间成本，将字节码编译成机器码会占用更多的空间。

所以只有热点代码才值得进行 JIT 编译。

#### 6.2 JIT 编译器

HotSpotVM 中，解释器和 JIT 编译器都属于执行引擎。JIT 编译器有两种，一个是 Client Compiler ，简称 C1 ，另一个是 Server Compiler ，简称 C2 。JVM 会根据运行模式选择 C1 或 C2 搭配解释器一起使用。默认使用的是 Server Compiler ，通过 ```-client``` 或 ```-server``` 选项可以指定使用哪种编译器。C1 编译器主要针对客户端程序设计，优化程度小，启动速度快。C2 编译器主要针对长时间执行的程序，更加关注代码的执行时间，所以会对代码进行更大程度的优化，因此启动时间比 C1 长。为了利用各自的优势，达到启动时间和执行效率之间的平衡，从 jdk1.6 开始引入了分层编译 (tiered compilation) ，并在 jdk1.7 作为默认策略。

**分层编译是如何分层的？**

1. 第 0 层，解释器解释执行。可以触发第 1 层编译。
2. 第 1 层，C1 编译，进行简单的优化，将字节码编译成机器码。可以触发第 2 层编译。
3. 第 2 层及以上，C2 编译，进行耗时较长的优化，可能会进行激进的优化。

**解释器和 JIT 是如何配合工作的？**

HotSpotVM 为每个方法准备了两个计数器：Invocation Counter 和 Back Edge Counter 。解释器首先解释执行字节码，每执行一次某个方法，对应的计数器加 1 ，当达到阈值，触发 C1 编译，C1 将字节码编译成机器指令，只进行简单的优化。随着执行次数增多，优化会升级，触发 C2 编译，在 C2 编译过程中 JVM 可能会进行一些激进优化（大多数情况下能够提升运行速度的优化），但不一定成功。当出现罕见陷阱 (Uncommon Trap) 即优化失败时，通过逆优化 (deopimization) 进行回退，继续由解释器解释执行。编译过程需要花费时间，程序在运行期间不会等待编译完成，编译是异步进行的，编译完成后会替换代码，再下一次执行时使用。

**Java 为什么要有 JIT ，为什么不在 javac 阶段就进行优化?**

这是由于 javac 本身就是这么设计的，至于为什么这么设计，一是因为 Java 有一些动态特性导致在 javac  阶段进行优化难度很大，所以干脆不进行优化。举个例子：
有一个类 Foo 有两个方法 m1 和 m2  。

```
class Foo {
    public String m1() {
	    return m2();
	}
	public String m2() {
		return null;
	}
}

Foo foo = new Foo();
foo.m1();
```

对于这样的类，我们能不能在 javac 编译时将 m1 优化成 return null 呢？不能。
因为我们无法确定会不会在其他时刻会出现一个 Bar 类。

```
class Bar extends Foo {
	public String m2(){
	    return "bar"
	}
}

Foo foo = new Bar();
foo.m1();
```

此时 ```m1()``` 就不应该为 null 了。显然在 javac 编译时想要确定这个问题是十分困难的。二是将优化放在 JIT ，可以让其他运行在 JVM 上的其他语言也可以享受到优化的收益。

**有 JIT 比没有 JIT 要快么?**

其实不一定，还要具体问题具体分析。如果没有 JIT ，那么执行过程是 ```字节码->解释器解释执行->结果```，有 JIT 的过程是 ```字节码->JIT 编译->执行编译后的代码->结果```。经过 JIT 编译之后的代码执行要比解释器解释执行要快，但是 JIT 编译也是需要耗时的，并不一定 JIT 编译的时间要肯定小于解释执行的时间。

#### 6.3 JIT 进行了哪些优化？

##### 6.3.1 公共子表达式消除

如果一个表达式 E 已经计算得到结果 R ，并且再后续再次出现 E 时它的参数没有发生变化，则可以直接使用 R 代替 E 。

##### 6.3.2 方法内联

方法内联是在编译过程中遇到方法调用时，将目标方法的方法体纳入编译范围之中，并取代原方法调用的优化手段。比如 getter/setter 方法，如果没有内联，在调用 getter/setter 方法时，程序需要保存当前方法的执行位置，并创建 getter/setter 的栈帧入栈，执行完调用后再出栈，恢复到当前方法继续执行。如果有内联，会将方法调用优化成字段访问，从而提高程序运行速度。内联发生在 C2 阶段，由 JIT compiler 解析字节码生成 IR 图，并在 IR 图上进行优化，如果进行方法内联，就将 IR 图中的对应节点替换成目标方法的 IR 图，然后对新的 IR 图进行进一步优化。 

##### 6.3.3 逃逸分析

###### 6.3.3.1 什么是逃逸分析

逃逸分析是 JVM 中使用的一种优化技术。逃逸分析算法是利用连通图来构建对象和引用对象之间的可达性，以此进行数据流分析的方法，在论文 Escape Analysis for Java 中有介绍。

###### 6.3.3.2 逃逸的种类

逃逸分析针对的对象的分析，对象的逃逸状态有 3 种：

1. GlobalEscape ，全局逃逸
   当对象符合以下条件，认为是全局逃逸：
   1）对象作为方法的返回结果。
   2）存储在全局变量或静态变量中的对象。
   3）重写了 finalize() 方法的对象。
2. ArgEscape ，参数逃逸
   对象作为参数传递或被参数引用，但是没有发生全局逃逸。
3. NoEscape 没有逃逸

###### 6.3.3.3 优化方式

利用逃逸分析，JVM 可以对代码进行优化：

1. **栈上分配**
   如果对象没有逃逸，可以将对象分配在方法的栈上而不用在堆中创建对象，这样当方法执行完，栈帧弹出，对象就自动被回收，可以加快内存回收，减少 GC 。
   
   ```java
   public class EscapeAnalysisExample1 {
       public static void main(String[] args) {
           for (int i = 0; i < 5_000_000; i++) {
               createObject();
           }
       }
   
       public static void createObject() {
           new Object();
       }
   }
   ```
   
   上面的示例代码使用 jvm 参数 -XX:+PrintGC -Xms5M -Xmn5M -XX:-DoEscapeAnalysis 运行，会打印 GC 日志。
   
   ```
   [GC (Allocation Failure)  4096K->709K(5632K), 0.0007185 secs]
   [GC (Allocation Failure)  4805K->749K(5632K), 0.0006822 secs]
   [GC (Allocation Failure)  4845K->797K(5632K), 0.0004460 secs]
   [GC (Allocation Failure)  4893K->797K(5632K), 0.0004623 secs]
   [GC (Allocation Failure)  4893K->805K(5632K), 0.0003993 secs]
   [GC (Allocation Failure)  4901K->837K(4608K), 0.0003775 secs]
   [GC (Allocation Failure)  3909K->829K(5120K), 0.0003963 secs]
   [Full GC (Ergonomics)  829K->658K(7168K), 0.0054946 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002696 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002715 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002137 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002416 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002771 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002459 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002483 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0003253 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0003556 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0005012 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0003222 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002627 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0003585 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0003137 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0004831 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0004348 secs]
   [GC (Allocation Failure)  3730K->658K(7168K), 0.0002738 secs]
   
   Process finished with exit code 0
   ```
   
   如果使用 -XX:+PrintGC -Xms5M -Xmn5M -XX:+DoEscapeAnalysis，这时对象都是分配在栈上，不会输出 GC 日志。
   
   
   
2. **锁消除**
   当 JVM 认为一个对象只在当前线程内部使用，那么就会优化掉对象的同步锁。

   ```java
   //优化前
   public static void m1() {
   	synchronized (new Object()) {
   		System.out.println("m1");
   	}
   }
   //优化后
   public static void optimizedM1() {
   	System.out.println("m1");
   }
   ```

   上边的例子只是为了说明，因为实际中我们应该不会对 new Object() 进行加锁。实际中最常见的还有在一个方法内部使用 StringBuffer.append() ，StringBuffer 的方法是 synchronized ，如果对象没有逃逸，JVM 会帮我们优化掉 synchronized 。



3. **标量替换**
   标量不能再拆分，比如基本类型，聚合量可以被拆分，比如对象。将对象分解成标量，使用标量代替对象就叫做标量替换。如果对象没有逃逸，就可以用标量代替，变量只存储在栈上，就不用在堆中创建对象，既减少了内存占用又提高了运行速度。
   
   ```java
   class Point {
   	int x;
   	int y;
   }
   //优化前
   public void show() {
   	Point point = new Point();
   	point.x = 1;
   	point.y = 2;
   	System.out.println("point=(" + point.x + ", " + point.y + ")");
   }
   
   //优化后
   public void optimizedShow() {
   	int x = 1;
   	int y = 2;
   	System.out.println("point=(" + x + ", " + y + ")");
   }
   ```
   
   

4. **锁粗化**
   加锁操作是很消耗资源的，如果在循环内部进行多次，会将锁的范围扩大到循环外。
   
   ```java
   //优化前
   public static void m1() {
   	for (int i = 0; i < 5_000_000; i++) {
   		synchronized (new Object()) {
   
   		}
   	}
   }
   //优化后
   public static void optimizedM1() {
   	synchronized (new Object()) {
   		for (int i = 0; i < 5_000_000; i++) {
   
   		}
   	}
   }
   ```
   
   ```java
   public class EscapeAnalysisExample4 {
       public static void main(String[] args) {
           long start = System.currentTimeMillis();
           foo();
           System.out.println("foo cost = " + (System.currentTimeMillis() - start) + "ms");
           start = System.currentTimeMillis();
       }
   
       public static void foo() {
           for (int i = 0; i < 5_000_000; i++) {
               synchronized (new Object()) {
   
               }
           }
       }
   }
   ```
   
   使用 jvm 参数 -XX:-DoEscapeAnalysis 关闭逃逸分析优化
   
   ```
   foo cost = 118ms
   ```
   
   使用 jvm 参数 -XX:+DoEscapeAnalysis 打开逃逸分析优化
   
   ```
   foo cost = 6ms
   ```
   
   

### 7. 让 YGC 和 FGC 交替出现

-Xms41m 
-Xmx41m 
-Xmn10m 
-XX:+UseParallelGC 
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps

```
public class GCTest {
    public static void main(String[] args) throws Exception {
        List caches = new ArrayList();
        for (int i = 0; i < 7; i++) {
            System.out.println("次数：" + (i + 1));
            caches.add(new byte[1024 * 1024 * 3]);
        }
        caches.clear();
        for (int i = 0; i < 2; i++) {
            System.out.println("clear后次数：" + (i + 1));
            caches.add(new byte[1024 * 1024 * 3]);
        }
    }
}
```

```
java -jar GCTest -Xms30m -Xmx30m -Xmn10m -XX:+UseParallelGC -XX:+PrintGCDetails
```

每次放3M
1、在YGC执行前，min(目前 eden 已使用的大小,之前平均晋升到old的大小中的较小值) > old剩余空间大小 ? 不执行YGC，直接执行Full GC : 执行YGC；
2、在YGC执行后，平均晋升到old的大小 > old剩余空间大小 ? 触发Full GC ： 什么都不做。

<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;width: 50px;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">eden</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">old</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">YGC</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">FGC</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">说明</th></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第1次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第2次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第3次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是0）=0，小于old剩余的空间20，所以触发YGC，eden的对象进入old，old变成6，eden变成3。平均晋升到old的大小是6，小于old剩余的空间14，不触发FGC。</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第4次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第5次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">12</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是6）=6，小于old剩余的空间12，所以触发YGC，eden的对象进入old，old变成12，eden变成3。平均晋升到old的大小是6，小于old剩余的空间8，不触发FGC。</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第6次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">12</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第7次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">18</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是6）=6，小于old剩余的空间8，所以触发YGC，eden的对象进入old，old变成18，eden变成3。平均晋升到old的大小是6，大于old剩余的空间2，触发FGC。</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第8次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">18</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第9次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">18</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是6）=6，大于old剩余的空间2，直接触发FGC。</td></tr></table>    