---
title: Default JVM Options
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2020-02-21 11:59:09
thumbnail:
---

在进行 JVM 调优之前首先需要确定当前环境 JVM 各项参数的值是什么。我们可以用以下几种方式查看 JVM 的参数配置。
1. **使用 jinfo** 
查看某个具体的参数
```
D:\Dev\IdeaProjects\microlabs\effictive-java>jinfo -flag MaxHeapSize 19280
-XX:MaxHeapSize=4267704320
```
查看所有的参数
```
>jinfo -flags 19280
Attaching to process ID 19280, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.221-b11
Non-default VM flags: -XX:CICompilerCount=4 -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4267704320 -XX:MaxNewSize=1422393344 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=89128960 -XX:OldSize=179306496 -
XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
Command line:  -XX:+PrintCommandLineFlags -javaagent:D:\Program Files\JetBrains\IntelliJ IDEA 2019.2\lib\idea_rt.jar=5195:D:\Program Files\JetBrains\IntelliJ IDEA 2019.2\bin -Dfile.encoding=UTF-8
```
2. **使用 -XX:+PrintCommandLineFlags**
```
>java -XX:+PrintCommandLineFlags -version
-XX:G1ConcRefinementThreads=8 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=266639552 -XX:MaxHeapSize=4266232832 -XX:+PrintCommandLineFlags -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCa
che -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation
openjdk version "11.0.3" 2019-04-16
OpenJDK Runtime Environment (build 11.0.3+12-b304.10)
OpenJDK 64-Bit Server VM (build 11.0.3+12-b304.10, mixed mode, sharing)
```
3. **使用 -XX:+PrintFlagsInitial 或 -XX:+PrintFlagsFinal**
```
>java -XX:+PrintFlagsInitial -version
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