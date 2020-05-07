---
title: 每天一个 Linux 命令（12）： tail
date: 2017-02-19 10:07:32
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

tail 是查看 log 的最常用的命令，没有之一。

<!-- more -->

tail 命令的格式：

```
Usage: tail [OPTION]... [FILE]...

Options:
  -f, --follow[={name|descriptor}]
                           循环输出文件增加的内容；
  -n, --lines=[+]NUM       输出最后的 NUM 行;
  -s, --sleep-interval=N   和 -f 一起使用，在每次循环间隔 N 秒。
```
