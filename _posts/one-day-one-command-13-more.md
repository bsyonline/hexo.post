---
title: 每天一个 Linux 命令（13）： more
date: 2017-02-20 10:04:46
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

more 是另一个和 cat 功能类似的命令，不同的是 more 可以分页显示文件内容。

<!-- more -->

more 命令格式：

```
Usage: more [options] <file>...

Options:
 -<number>   每次显示 number 行
 +<number>   从 number 行开始显示
```

常用操作：

```
Enter     向下n行，需要定义。默认为1行
空格键     向下滚动一屏
Ctrl+B    返回上一屏
=         输出当前行的行号
:f        输出文件名和当前行的行号
q         退出 more
```
