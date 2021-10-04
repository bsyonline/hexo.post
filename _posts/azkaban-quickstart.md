---
title: azkaban quickstart
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-10-04 09:18:04
thumbnail:
---





### 简介



### 编译

Azkaban 没有提供编译好的包，需要自己编译。

```shell
cd azkaban; 
./gradlew build installDist -x test
```

> 将 azkaban/build.gradle 中的 
>
> https://linkedin.bintray.com/maven 
>
> 替换成
>
> https://linkedin.jfrog.io/artifactory/open-source/

### Solo Server

solo server 是单机运行模式，编译好之后直接运行。

```shell
cd azkaban-solo-server/build/install/azkaban-solo-server 
bin/start-solo.sh
```

访问 localhost:8081，azkaban/azkaban 。