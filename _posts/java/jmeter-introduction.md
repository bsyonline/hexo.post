---
title: Jmeter 使用入门
date: 2016-09-07 10:54:11
tags:
 - Jmeter
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

Jmeter 是一款开源的压力测试工具，虽功能不及 Loadrunner 强大，但是对于一些简单的如接口api的测试完全够用。

### 基本使用方法
通常使用 Jmeter 通常需要以下几步。以 http api 接口测试为例：

1. 添加线程组
   ![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/%E7%BA%BF%E7%A8%8B%E7%BB%84.png)
2. 添加 Sampler，这里选择 Http请求
   ![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/HTTP%E8%AF%B7%E6%B1%82.png)
3. 添加 listener
   Jmeter 提供了不同的 Listener 可以按需求添加。

点击启动，将按照计划开始执行测试，完成后可看到报告。

### 如何使用变量
如果一个测试计划内容很多，有一些参数如 IP 等会出现多次，我们可以将这样的信息设置成用户定义变量。
![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/%E7%94%A8%E6%88%B7%E5%AE%9A%E4%B9%89%E7%9A%84%E5%8F%98%E9%87%8F.png)
定义之后，可以在其他地方通过 ${} 进行引用。

### 如何使用函数
Jmeter 提供了一些内置函数，点击 函数助手对话框 可查看内置函数的说明。
举个例子，如果我们在发送 http 请求时想使用更多的测试数据，而不是在参数栏将一组测试数据写死，可以将多组测试数据放在 csv 文件中，然后通过 \__CSVRead() 函数来调用。
1. 在 HTTP请求中加入配置元件 CSV Data Set Config ，并在 fileName 中指定文件的全路径。
   ![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/%E4%BC%81%E4%B8%9A%E5%90%8D%E5%8D%95.png)
2. 在 Http 参数列表中的 value 位置使用 ${\__CSVRead(${entNameFile},0)} 。第一个参数是需要读取的 CSV 文件，第二个参数是每组数据的位置，CSV 文件中的测试数据默认使用 ‘,’ 分割，0 表示第一列数据。

### OutOfMemoryError 
在执行测试过程中，执行一段时间后出现 Java.lang.OutOfMemoryError: Java heap space 。解决方法参考如下：
1. 修改启动参数。
   在 bin/jmeter 最后加入启动参数。
```
java $ARGS $JVM_ARGS -Xms1G -Xmx5G -XX:MaxPermSize=512m -Dapple.laf.useScreenMenuBar=true $JMETER_OPTS -jar "$PRGDIR/ApacheJMeter.jar" "$@"
```
2. 如果在计划中加入了察看结果树的 Listener ，将会有大量数据写入。通常在计划调试阶段使用，正式测试时应去掉。
