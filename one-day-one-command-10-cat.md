---
title: 每天一个 Linux 命令（10）： cat
date: 2017-02-17 10:08:08
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

当我们想查看一个文件中的内容的时候， cat 是一种方法。cat 命令的作用就是连接文件到标准输出。

<!-- more -->

cat 命令格式：

```
Usage: cat [OPTION]... [FILE]...
```

cat 最基本的用法就是讲文件的全部内容输出到标准输出。

```
cat /etc/hosts
```

cat 还可以创建一个新文件。

```
cat > hello.txt
```

还有一种比较常用的用法，将多个文件合并成一个文件。

```
cat file1 file2 > file3
```
