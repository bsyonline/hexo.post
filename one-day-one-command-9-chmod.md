---
title: 每天一个 Linux 命令（9）： chmod
date: 2017-02-16 10:10:38
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

chmod 命令用于改变 linux 系统文件或目录的访问权限。

<!-- more -->

Linux 文件系统的权限分为读 r ，写 w ，执行 x 3 种。Linux 有 3 类用户，所有者，同组用户和其他用户。在 linux 系统中文件和目录都具有权限，每种用户对文件或目录的权限通过 1 个文件类型 和 3 组 rwx 组成的字符串表示。
```
-rw-rw-r--  1 rolex rolex   5185 Feb 13 17:16 sbt2182110252329960095.log
```
如果想修改 -rw-rw-r-- 成其他权限，可以用 chmod 命令。
chmod 命令格式：

```shell
Usage: chmod [OPTION]... MODE[,MODE]... FILE...
  or:  chmod [OPTION]... OCTAL-MODE FILE...
  or:  chmod [OPTION]... --reference=RFILE FILE...

Options:
  -R, --recursive        change files and directories recursively
```

权限有两种设定方式：字符方式和数字方式。
1. 字符方式

字符方式是使用字母和操作符来操作权限。

u ：目录或者文件的当前的用户
g ：目录或者文件的当前的群组
o ：除了目录或者文件的当前用户或群组之外的用户或者群组
a ：所有的用户及群组
r ：读权限，用数字4表示
w ：写权限，用数字2表示
x ：执行权限，用数字1表示
\- ：删除权限，用数字0表示
s ：特殊权限

比如，为所有用户增加写权限
```shell
chmod a+w sbt2182110252329960095.log
```

2. 数字方式

数字方式是用二进制方式表示权限，0 表示无权限，1 表示有权限。如 -rwxrw-rw- 可表示成 755 ， -rw-r--r-- 可表示成 644 。

例如，为所有用户开放所有权限
```shell
chmod 777 sbt2182110252329960095.log
```
