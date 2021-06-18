---
title: 每天一个 Linux 命令（7）： ln
date: 2017-02-14 10:09:37
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

ln 命令是在 linux 系统中建立一个文件连接。

<!-- more -->

ln 命令格式：

```
Usage: ln [OPTION]... [-T] TARGET LINK_NAME

Options:
  -s, --symbolic              make symbolic links instead of hard links
```

ln 比较简单，但是还有一个基本概念要明确一下。在 linux 系统中链接分两种，软链接和硬链接。
软链接，也叫符号链接，以路径方式存在，类似 windows 系统的快捷方式。通常用软链接来创建一个目录的 link ，作用就是快捷方式。
硬链接，是文件副本，不能链接目录。
ln 默认创建的是硬链接， 创建软链接要使用选项 -s 。

为一个目录创建软链接
```
ln -s /u01/jdk1.8.0_111 java
```

删除软链接
```
rm -rf java
```
>后边不能带“/”

最后在强调一点，ln 的所有文件或目录是同步，修改任何一个位置，其他都会改变。
