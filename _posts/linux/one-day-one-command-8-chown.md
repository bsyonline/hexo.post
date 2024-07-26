---
title: 每天一个 Linux 命令（8）： chown
date: 2017-02-15 10:10:26
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

chown 用于修改文件的所有者和组。

<!-- more -->

命令格式：
```shell
Usage: chown [OPTION]... [OWNER][:[GROUP]] FILE...
  or:  chown [OPTION]... --reference=RFILE FILE...

Options:
  -R, --recursive        operate on files and directories recursively
```

修改文件的所有者和组
```
chown ops:ops text.txt
```
修改目录下所有文件的所有者和组

```
chown -R ops:ops /tmp/logs/
```
