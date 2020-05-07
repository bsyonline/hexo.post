---
title: 每天一个 Linux 命令（19）： mv
date: 2017-02-26 10:10:28
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

mv 的作用是移动文件和目录，也可以用来重命名文件。

<!-- more -->

mv 命令格式：

```
Usage: mv [OPTION]... [-T] SOURCE DEST

Options:
-t, --target-directory=DIRECTORY  移动多个文件到 DIRECTORY
--backup[=CONTROL]                如果文件存在，则备份。CONTROL 是备份策略，共有四种：
                                    1.CONTROL=none或off : 不备份。
                                    2.CONTROL=numbered或t：数字编号的备份
                                    3.CONTROL=existing或nil：如果存在以数字编号的备份，则继续编号备份m+1…n。
                                      执行mv操作前已存在以数字编号的文件log2.txt.~1~，那么再次执行将产生log2.txt~2~，以次类推。
                                      如果之前没有以数字编号的文件，则使用下面讲到的简单备份。
                                    4.CONTROL=simple或never：使用简单备份：在被覆盖前进行了简单备份，简单备份只能有一份，再次被覆盖时，简单备份也会被覆盖。
-b                                like --backup 但是不能指定策略，而是使用环境变量 VERSION_CONTROL 来作为备份策略。
-f                                不备份，强制覆盖
```

移动当前目录的所有文件到上一级目录

```
mv * ..
```

移动多个文件到指定目录

```
mv -t /tmp/test a.txt b.txt c.txt
```
