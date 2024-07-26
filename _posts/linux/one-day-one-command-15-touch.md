---
title: 每天一个 Linux 命令（15）： touch
date: 2017-02-22 10:08:29
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

touch 可以刷新文件的访问和修改时间，但是在印象中用途就是创建一个空文件。

<!-- more -->

touch 命令格式：

```
Usage: touch [OPTION]... FILE...

Options:
  -d, --date=STRING      使用指定日期代替创建日期
  -t STAMP               使用 [[CC]YY]MMDDhhmm[.ss] 代替当前时间

```

创建一个空文件

```
touch a.txt
```

修改文件创建日期

```
touch -d 20170222 a.txt
```

修改时间

```
touch -t 1702220930 a.txt
```
