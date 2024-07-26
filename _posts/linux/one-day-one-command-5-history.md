---
title: 每天一个 Linux 命令（5）： history
date: 2017-02-12 10:07:55
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

history 命令能够列出在终端执行的命令的列表，并且能够快速执行某条历史命令。

history 命令格式：

```
history
```
执行后会看到类似信息：
```
1804  shutdown -h now
1805  dependencyTree
1806  sbt dependencyTree
1807  sbt dependency-tree
1808  sbt dependency-graph
```

最常用的功能是快速执行历史命令中的某行，如想执行 ‘sbt dependencyTree’ ，可以使用如下命令：
```
!1806
```

history 功能还有很多，不过都没怎么用过，基本是查看历史命令记录，不过想看命令的执行时间，有个小技巧可以用的上。
```
export HISTTIMEFORMAT='%F %T '
history
```

设置格式化后，可以看到 history 中会显示日期和时间。
