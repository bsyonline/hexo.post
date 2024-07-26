---
title: 每天一个 Linux 命令（20）： find
date: 2017-02-27 10:10:45
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


find 命令提供了强大的搜索文件的功能。

<!-- more -->

find 命令格式：

```shell
Usage: find [-H] [-L] [-P] [-Olevel] [-D help|tree|search|stat|rates|opt|exec|time] [path...] [expression]
    path        find 命令所查找的目录路径。例如用 . 来表示当前目录，用 / 来表示系统根目录。默认是当前目录。
    expression  默认是 -print ，打印到控制台
Options：
    STANDARDS CONFORMANCE
       For closest compliance to the POSIX standard, you should set the POSIXLY_CORRECT environment variable.  The following options  are  specified  in  the  POSIX
       standard (IEEE Std 1003.1, 2003 Edition):

       -H     This option is supported.

       -L     This option is supported.

       -name  按照文件名查找文件。

       -type  Supported.   POSIX specifies `b', `c', `d', `l', `p', `f' and `s'.  GNU find also supports `D', representing a Door, where the OS provides these.

       -ok    Supported.   Interpretation  of  the  response is according to the "yes" and "no" patterns selected by setting the `LC_MESSAGES' environment variable.
              When the `POSIXLY_CORRECT' environment variable is set, these patterns are taken system's definition of a positive (yes) or  negative  (no)  response.
              See  the  system's  documentation for nl_langinfo(3), in particular YESEXPR and NOEXPR.    When `POSIXLY_CORRECT' is not set, the patterns are instead
              taken from find's own message catalogue.

       -newer Supported.  If the file specified is a symbolic link, it is always dereferenced.  This is a change from previous behaviour, which  used  to  take  the
              relevant time from the symbolic link; see the HISTORY section below.

       -perm  按照文件权限来查找文件。

       Other predicates
              The predicates -atime, -ctime, -depth, -group, -links, -mtime, -nogroup, -nouser, -print, -prune, -size, -user and -xdev `-atime', `-ctime', `-depth',
              `-group', `-links', `-mtime', `-nogroup', `-nouser', `-perm', `-print', `-prune', `-size', `-user' and `-xdev', are all supported.
```

Options 太多了，就不一一列举了。

create 'rb_gs_change_v3','f'

举例：

按名字查找：
```shell
find /tmp -name core
```
按权限查找：
```shell
find . -perm -664
```
按权限查找：
```shell
find . -perm /222
```
结合 exec 使用：
```shell
find /tmp -name core -type f -print | xargs /bin/rm -f
```