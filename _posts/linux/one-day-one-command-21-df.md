---
title: 每天一个 Linux 命令（21）： df
date: 2017-02-28 10:10:45
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


linux 中 df 命令的功能是用来检查 linux 服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

<!-- more -->

df 命令格式：

```shell
Usage: df [OPTION]... [FILE]...
显示指定文件或所有文件的占用磁盘信息.
Options:
 -a, --all 全部文件系统列表
 -h, --human-readable  				以 1K=1024B 显示 (e.g., 1023M)
 -H, --si              				以 1K=1000B 显示 (e.g., 1.1G)
 -T, --print-type      				显示文件系统类型
 -t <TYPE>, --type=TYPE				显示指定类型的文件系统
 -x <TYPE>, --exclude-type=TYPE		不显示指定类型的文件系统
```

举例：

显示磁盘使用情况：

```shell
df
```
显示指定类型磁盘：

```shell
df -t ext4
```
以更易读的方式显示目前磁盘空间和使用情况：

```shell
df -h
```
