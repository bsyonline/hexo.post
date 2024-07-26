---
title: 每天一个 Linux 命令（11）： head
date: 2017-02-18 10:07:20
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

head 和 cat 一样是链接文件和标准输出的命令，不同的是 head 只输出文件的头部（默认前 10 行）。

<!-- more -->

head 命令格式：

```
Usage: head [OPTION]... [FILE]...

Options:
  -n, --lines=[-]NUM       print the first NUM lines instead of the first 10;
                               with the leading '-', print all but the last
                               NUM lines of each file
  -c, --bytes=[-]NUM       print the first NUM bytes of each file;
                               with the leading '-', print all but the last
                               NUM bytes of each file

```

实际上用过的也只有 -n 了。
