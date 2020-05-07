---
title: 每天一个 Linux 命令（22）： du
date: 2017-03-01 10:10:45
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


Linux du 命令也是查看使用空间的，但是与 df 命令不同的是 du 命令是对文件和目录磁盘使用的空间的查看。

<!-- more -->

du 命令格式：

```shell
Usage: du [OPTION]... [FILE]...
   or: du [OPTION]... --files0-from=FUsage: df [OPTION]... [FILE]...
显示每个文件和目录的磁盘使用空间。
Options:
  -a, --all             write counts for all files, not just directories
  -b, --bytes           等价 '--apparent-size --block-size=1'
  -h, --human-readable  以 K，M，G 为单位显示 (e.g., 1K 234M 2G)
  -m                    等价 --block-size=1M
  -c, --total           显示总计
```

举例：

显示目录或者文件所占空间：

```shell
du
```
显示指定文件所占空间：

```shell
du 1.txt
```
查看指定目录的所占空间：

```shell
du /etc
```

显示多个文件所占空间：

```
du 1.txt 2.txt
```

显示多个文件总计：

```
du -c 1.txt 2.txt
```

