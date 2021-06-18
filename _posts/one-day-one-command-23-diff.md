---
title: 每天一个 Linux 命令（23）： diff
date: 2017-03-02 10:10:45
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


diff 命令是 linux 上非常重要的工具，用于比较文件的内容，特别是比较两个版本不同的文件以找到改动的地方。



<!-- more -->

diff 命令格式：

```shell
Usage: diff [OPTION]... FILES
按行比较文件。
Options:
  -q, --brief                   只显示差异
  -s, --report-identical-files  文件相同仍然显示
  -c, -C NUM, --context[=NUM]   显示 NUM (default 3) 行 of copied context
  -u, -U NUM, --unified[=NUM]   显示 NUM (default 3) 行 of unified context
  -e, --ed                      output an ed script
  -n, --rcs                     以 RCS 格式显示差异 diff
  -r, --recursive               递归比较子目录
  -i, --ignore-case             忽略大小写
  -y, --side-by-side            并列显示
  -W, --width=NUM               指定栏数 NUM (default 130) 
```

举例：

比较两个文件：

```shell
diff 1.log 3.log
```
> a - add
>
> c - change
>
> d - delete

并排格式输出：

```shell
fiff -y -W 20 1.txt 2.txt
```
> “|”表示前后2个文件内容有不同
>
> “<”表示后面文件比前面文件少了1行内容
>
> “>”表示后面文件比前面文件多了1行内容

上下文输出格式：

```shell
diff -c 1.log 4.log
```

> “＋” 比较的文件的后者比前着多一行
>
> “－” 比较的文件的后者比前着少一行
>
> “！” 比较的文件两者有差别的行

统一格式输出：

```
diff -u 4.log 3.log
```

> “—“表示变动前的文件，”+++”表示变动后的文件
>
> 变动的位置用两个@作为起首和结束

比较文件夹不同：

```
diff  test3 test6  
```

