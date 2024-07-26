---
title: 每天一个 Linux 命令（18）： cp
date: 2017-02-25 10:10:13
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

cp 是 linux 中复制文件和目录的常用命令。

<!-- more -->

cp 命令格式：

```
cp [OPTION]... [-T] SOURCE DEST

Options:
  -a, --archive
                same as -dR --preserve=all
  -R, -r, --recursive
                递归拷贝目录
  -p     same as --preserve=mode,ownership,timestamps
  --preserve[=ATTR_LIST]
         保持文件指定的属性 (default: mode,ownership,timestamps), if possible additional attributes: context, links, xattr, all
```

复制一个文件

```
cp -a a.txt b.txt
```
使用选项 -a 两个文件的创建时间是一致的。

复制文件到目录

```
cp a.txt books/
```

复制目录

```
cp -r books book
```
