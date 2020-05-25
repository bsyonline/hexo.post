---
title: how to develop a web code sdudio
tags:
  - java
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2020-05-18 23:39:45
thumbnail:
---

接到这个需求，首先了解一下什么是 web ide 。

目前市面上都有哪些类似的产品，都具有什么功能。

基于 git 和 maven 构建，能够在浏览器中编辑和运行代码，并且支持远程调试。

大概的需求清楚了，接下来就考虑如何实现。

最初首先想到的是，既然是基于浏览器的，那么肯定是 B/S 架构没跑了。那么可以分成两块，前端和后端。前端渲染页面，由于前端太菜，所以一笔带过，主要说说后端。本地的 ide 我们都熟悉，我们假设把本地 ide 的界面想象成浏览器，那么我们在界面上做的操作，都可以对应到一个向 server 的请求，server 接到请求，进行处理，然后将结果返回给前端进行渲染，这样一来一回，好像整个流程就串起来了。于是就有了第一版设计。
前端有一个 text 用来写代码，然后将整个代码作为一个字符串提交给 server ，接下来就需要 server 对前端传过来的程序字符串进行处理了。以 Java 程序为例，首先需要编译，然后才能执行。那如何编译呢？学过 Java 的都知道有个命令叫 javac 。那问题来了，javac 是在命令行执行，如果我们要用 javac 来编译，就需要让 server 能够执行 shell 。所以问题又变成了 java 如何执行 shell 。于是请出了今天的第一位选手 Runtime 。

#### Runtime

每个 Java 都会对应一个 Runtime 类的实例，用来使 Java 和环境能够进行交互。比如使用 Runtime 获取 CPU 核数和内存大小：
```
int availableProcessors = Runtime.getRuntime().availableProcessors();
long freeMemory = Runtime.getRuntime().freeMemory();
System.out.println(availableProcessors);
System.out.println(freeMemory / 1024 / 1024);
```
Runtime 还可以用来在独立的进程中执行命令，比如这样：
```
Process process = Runtime.getRuntime().exec(new String[]{"powershell", "java -version"});
InputStream inputStream = process.getErrorStream();
BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
String line = null;
while((line = br.readLine()) !=null){
	System.out.println(line);
}
br.close();
```
既然是执行命令，那么就会有正常输出和错误输出，Runtime 需要我们自己处理来处理 process.getErrorStream() 和 process.getInputStream() 。这里需要注意，因为 Runtime 是 fork 子进程来执行，流是父子进程公用的，所以很容易造成阻塞。正确的做法就是创建单独的线程来处理流。
```
Process proc = Runtime.getRuntime().exec(cmd);
// any error message
StreamBeat errorGobbler = new StreamBeat(ctx, proc.getErrorStream(), "ERROR");
// any output
StreamBeat outputGobbler = new StreamBeat(ctx, proc.getInputStream(), "OUTPUT");
// kick them off
new Thread(errorGobbler).start();
new Thread(outputGobbler).start();
// wait for finished
int exitVal = proc.waitFor();
```
从 JDK1.5 以后，创建进程有一种更好的方式 ProcessBuilder ，它还提供了重定向错误输出，这样我们就不用创建线程来处理流了。
```
ProcessBuilder processBuilder = new ProcessBuilder();
processBuilder.redirectErrorStream(true);
processBuilder.command("powershell", cmd);
Process process = processBuilder.start();
InputStream inputStream = process.getInputStream();
BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
String line = null;
while ((line = br.readLine()) != null) {
	System.out.println(line);
}
br.close();
```
不管是 Runtime 还是 ProcessBuilder ，都可以解决我们执行命令的问题。 于是我们顺着这个思路就想既然有 javac 、java -jar 这样的命令，那调试应该也有命令吧，于是我们惊喜的发现在角落里有一个不起眼的家伙 jdb 。

#### JDB


命令有了，无非就再麻烦一下 Runtime 老兄嘛。燃鹅，在查看了 jdb 的用法之后，三下五除二请出了 Runtime 。纳尼？竟然卡住了。jdb 是一套交互命令， 使用 Runtime 执行之后进入到交互模式，Process 拿不到 stream ，程序卡入住，卡住了自然就没有响应给客户端，也就没办法再进行后续的操作了。在经历一阵研究之后，该方案宣告 gg 。