---
title: SBT 打包
date: 2017-01-17 15:48:32
tags:
 - SBT
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

紧接上一篇（[Spark 入门](../../../../2017/01/10/Spark-入门/)），本节讨论一下 SBT 打包。

<!-- more -->

上节中，sbt 环境安装好后，执行
```
sbt package
```
即可完成打包，并且上传到 spark 集群中也能够正常运行。但是，仔细观察可以发现，上传的 jar 包只有 135 KB，显然打包只包含了编译后的文件，依赖 jar 包并没有打包，而依赖的 spark jar 在集群中都有，所以也可以正常执行。
如果程序中引用了一些其他的 jar 这样就需要指定 --jars 参数，不过集群环境中需要在每个 worker 节点都安装相同的 jar 依赖，操作还是比较繁琐，比较方便的方式是将所有的依赖 jar 都打包成一个 **fat jar** 。

### sbt-assembly 使用

SBT 的 sbt-assembly 插件可以帮助我们打包。
插件文档参考：[https://github.com/sbt/sbt-assembly](https://github.com/sbt/sbt-assembly)

按照文档，简单三步可完成打包。
1. 修改 project/plugins.sbt ，加入
```
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.3")
```
2. 在根目录下创建 assembly.sbt

  ```
  import AssemblyKeys._

  assemblySettings

  name := "spark-data-engine-batch"
  version := "1.0"
  ```

3. 执行 sbt assembly

### 解决冲突
多数情况下，打包过程会出现异常，比如文件重复。
```
[error] (*:assembly) deduplicate: different file contents found in the following:
[error] /home/rolex/.ivy2/cache/commons-collections/commons-collections/jars/commons-collections-3.2.1.jar:org/apache/commons/collections/ArrayStack.class
[error] /home/rolex/.ivy2/cache/commons-beanutils/commons-beanutils/jars/commons-beanutils-1.7.0.jar:org/apache/commons/collections/ArrayStack.class
[error] /home/rolex/.ivy2/cache/commons-beanutils/commons-beanutils-core/jars/commons-beanutils-core-1.8.0.jar:org/apache/commons/collections/ArrayStack.class
```
处理这类问题通常有两种方式：
1. 将其中一个 jar 在 sbt 中的 scope 指定为 Provided
2. 使用 sbt-assembly 的 mergeStrategy

第一种方式操作最简单，但是有时无法避免同一个包被引用多次的情况，那就只能使用第二种方式来合并冲突。
根据报错信息，参考 sbt-assembly 文档的示例进行修改。
```
mergeStrategy in assembly := {
  case PathList("javax", "servlet", xs@_*) => MergeStrategy.first
  case PathList("javax", "el", xs@_*) => MergeStrategy.first
  case PathList("org", "apache", xs@_*) => MergeStrategy.first
  case PathList("org", "objectweb", "asm", xs@_*) => MergeStrategy.first
  case PathList("org", "slf4j", xs@_*) => MergeStrategy.first
  case PathList(ps@_*) if ps.last endsWith ".properties" => MergeStrategy.first
  case x =>
    val oldStrategy = (mergeStrategy in assembly).value
    oldStrategy(x)
}
```
至此，使用 sbt assembly 就可以正确生成 fat jar 了。
