---
title: 每天一个 Linux 命令（14）： less
date: 2017-02-21 10:07:49
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

less 的作用和 more 相同，选项更多，比 more 更复杂功能更强。

<!-- more -->

less 命令格式：

```
Usage：less [-[+]aABcCdeEfFgGiIJKLmMnNqQrRsSuUVwWX~]
            [-b space] [-h lines] [-j line] [-k keyfile]
            [-{oO} logfile] [-p pattern] [-P prompt] [-t tag]
            [-T tagsfile] [-x tab,...] [-y lines] [-[z] lines]
            [-# shift] [+[+]cmd] [--] [filename]...

```

和 more 一样，进入 less 后也有一大堆命令可用，不过 less 是基于 more 和 vi 的，所以你懂得。


常用操作：

```
b         向后翻一页
d         向后翻半页
u         向前滚动半页
Ctrl+B    返回上一屏
Ctrl+F    向后翻一屏
回车       向下一行
y         向前一行
空格       向下一屏
q         退出 less
```
