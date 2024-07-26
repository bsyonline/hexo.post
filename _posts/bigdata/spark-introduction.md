---
title: Spark 入门
date: 2016-05-05 09:37:43
tags:
 - Spark
category: 
 - Spark
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


spark 环境搭建完成后，就可以搭建 spark 的开发环境进一步学习 spark 了。

### 几种构建方式差异
搭建 spark 的开发环境有多种方式，各有利弊。
* 使用 maven 构建
* 创建 scala 工程
* 使用 sbt 构建

**使用 maven 构建**

使用 maven 方式构建项目，在 Java 开发中十分常见，所以可以平滑过度，只需要添加 scala-tools 和 scala-library 的依赖，其他和 Java 无异。
但是使用缺点就是各个环境的 jar 的版本不好控制，一不小心就会报各种各样的找不到包的错误，因为 spark 和 scala 不同版本编译包是不兼容的。

**创建 scala 工程**
这种方式使用起来比较繁琐，因为各种环境，路径都需要自己配置，但优点是只要包导入没问题，基本不会出现版本问题。

**使用 sbt 构建**
sbt 是 scala 官方推出的构建工具，和 maven 使用方式类似，除了第一次下载时间超超超级长之外，其他都比较方便，各种资料介绍 sbt 的也最多，所以推荐使用。

综合各种，这里还是使用第三种方式。

### 构建 IDEA + SBT 环境
#### 1. 安装 sbt

下载 sbt 包并解压，配置环境变量

```
SBT_HOME=/u01/sbt
PAHT=$SBT_HOME/bin:$PATH
export SBT_HOME PATH
```

其他参考 [http://www.scala-sbt.org/download.html](http://www.scala-sbt.org/download.html) 。

#### 2. 修改国内源

sbt 需要先下载依赖的 jar 包，但是下载速度太慢，可以更换成国内的源提高速度。
在 ~/.sbt 下创建 repositories 文件
```
[repositories]
#local
public: http://maven.aliyun.com/nexus/content/groups/public/
typesafe:http://dl.bintray.com/typesafe/ivy-releases/ , [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
ivy-sbt-plugin:http://dl.bintray.com/sbt/sbt-plugin-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
sonatype-oss-releases
maven-central
sonatype-oss-snapshots
```
#### 3. 创建工程

先使用 IDEA 选择 SBT 方式创建工程，然后在终端切换到项目目录运行 sbt
```
sbt
> compile
```
依赖下载完成，开发环境就搭建好了。

> 使用 IDEA 为什么还要使用命令行？

>IDEA 中 sbt 默认使用国外的源，所以下载速度很慢，命令行方式由于修改了国内的源，所以下载速度快很多。而且先通过 IDEA 创建工程，也不用自己再去创建 build.sbt 等文件。


### 测试
拷贝 $SPARK_HOME/examples/src/main/java/org/apache/spark/examples/JavaWordCount.java 到工程中。

#### 1. 使用 sbt 编译打包
```
sbt compile package
```

生成的 jar 包默认路径为 target/scala-2.10 下。

#### 2. 添加部署脚本
```shell
#!/usr/bin/env bash

SCRIPT_PATH=`readlink -f "$0"`;
cd `dirname ${SCRIPT_PATH}`

/u01/spark/bin/spark-submit --name JavaWordCount  \
--class com.sparkone.test.JavaWordCount \
--master spark://localhost:7077 \
--executor-memory 512M \
../target/scala-2.10/sparkone_2.10-1.0.jar hdfs://localhost:9000/user/rolex/input/NOTICE.txt
```

执行即可在单机环境执行测试。
