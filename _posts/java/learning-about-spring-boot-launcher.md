---
title: Learning About Spring Boot Launcher
tags:
  - Interview
category:
  - Spring Boot
author: bsyonline
lede: 没有摘要
date: 2020-02-25 09:25:52
thumbnail:
---

Spring Boot 是如何通过 java -jar 方式运行的？为了搞清楚其中缘由，我们需要先构建一个 Spring Boot 工程。


通过 mvn package 将 Spring Boot 项目打包，然后解压缩，我们可以看到 jar 中的结构。这是一个 fat-jar ，其中主要有 3 部分：
1. BOOT-INF，用来存放应用的 class 文件和 Spring BOOT 依赖的 jar 。
2. META-INF，用来存放 MANIFEST.MF 文件。
3. spring-boot-loader 的 class 文件。

打开 MANIFEST.MF 文件我们会发现有两个信息：
```
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.rolex.springbootlearning.SpringBootLearningApplication
```
我们知道，如果我们需要打一个可执行的 jar ，那么 Main-Class 就是这个 jar 的入口。但是我们工程实际的程序入口是 Start-Class 对应的类，所以我们可以大胆猜测 Spring Boot 是通过 JarLauncher 来引导执行应用程序的入口类的。
接下来我们就要 debug 一下 JarLauncher 的执行过程。正常构建 Spring Boot 工程不需要加 spring-boot-loader 依赖，Spring Boot 会通过插件自动加入。由于我们需要通过 JarLauncher 启动 debug ，所以加入 spring-boot-loader 依赖。
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-loader</artifactId>
	<scope>provided</scope>
</dependency>
```		
然后配置运行配置。
<img src="https://s2.ax1x.com/2020/02/25/3JcSvn.png" alt="3JcSvn.png" border="0" />
最后在 JarLauncher 中打上断点就可以启动调试了。
1. 首先获取文件路径，加载 fat-jar 中所有的 jar 的 URL path 。
<img src="https://s2.ax1x.com/2020/02/25/3JR4dP.png" alt="20200225103627" border="0">
2. 然后加载 Manifest 信息，读取 Start-Class 。
<img src="https://s2.ax1x.com/2020/02/25/3JR5If.png" alt="20200225102947" border="0">
3. 最后通过反射 invoke 应用程序的 main 方法。
<img src="https://s2.ax1x.com/2020/02/25/3JRhZt.png" alt="20200225103200" border="0">

通过以上 3 步就实现了通过 java -jar 方式启动 Spring Boot 应用程序。