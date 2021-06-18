---
title: 每天一个 Linux 命令（28）： sed
date: 2017-03-07 10:10:45
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

sed 命令的全名是 stream editor ，用来把文档或字符串里面的文字经过一系列编辑命令转换为另一种格式输出，是简化版的 awk 。

sed 命令格式:

```shell
Usage: sed [OPTION]... {script-only-if-no-other-script} [input-file]...

  -n, --quiet, --silent
                 suppress automatic printing of pattern space
  -e script, --expression=script
                 add the script to the commands to be executed
  -f script-file, --file=script-file
                 将替换的内容放到文件
  --follow-symlinks
                 follow symlinks when processing in place
  -i[SUFFIX], --in-place[=SUFFIX]
                 替换结果应用于源文件
  -b, --binary
                 open files in binary mode (CR+LFs are not processed specially)
  -l N, --line-length=N
                 specify the desired line-wrap length for the 'l' command
  --posix
                 disable all GNU extensions.
  -r, --regexp-extended
                 use extended regular expressions in the script.
  -s, --separate
                 consider files as separate rather than as a single continuous
                 long stream.
  -u, --unbuffered
                 load minimal amounts of data from the input files and flush
                 the output buffers more often
  -z, --null-data
                 separate lines by NUL characters
      --help     display this help and exit
      --version  output version information and exit
```

可接受的命令：
```shell
       i	  之前插入
       a	  追加
       c \

       text   Replace the selected lines with text, which has each embedded newline preceded by a backslash.

       d      删除.

       D      If pattern space contains no newline, start a normal new cycle as if the d command was issued.  Otherwise, delete text in the pattern space up to the first newline, and restart cycle with the resultant  pattern  space,
              without reading a new line of input.

       h H    Copy/append pattern space to hold space.

       g G    Copy/append hold space to pattern space.

       l      List out the current line in a ``visually unambiguous'' form.

       l width
              List out the current line in a ``visually unambiguous'' form, breaking it at width characters.  This is a GNU extension.

       n N    Read/append the next line of input into the pattern space.

       p      Print the current pattern space.

       P      Print up to the first embedded newline of the current pattern space.

       s/regexp/replacement/
              替换.

       t label
              If a s/// has done a successful substitution since the last input line was read and since the last t or T command, then branch to label; if label is omitted, branch to end of script.

       T label
              If no s/// has done a successful substitution since the last input line was read and since the last t or T command, then branch to label; if label is omitted, branch to end of script.  This is a GNU extension.

       w filename
              Write the current pattern space to filename.

       W filename
              Write the first line of the current pattern space to filename.  This is a GNU extension.

       x      Exchange the contents of the hold and pattern spaces.

       y/source/dest/
              转换.
```



举例：

将文件中的 java 替换成 scala：

```shell
sed 's/java/scala/' a.txt
```
> 默认情况下 sed 不会修改源文件，所有的操作都是在缓冲区的副本中进行的。每次读入一行到缓冲区，处理完成后将结果发送到标准输出，然后删除缓冲区中的行，再读入下一行。

将替换结果应用于源文件：

```shell
sed -i 's/java/scala/' a.txt
```

移除空白行：

```shell
sed '/^$/d' a.txt 
```

> ^$行尾标记紧临表示空白行。

&代表已经匹配的字符串：

```shell
echo this is a test | sed 's/\w\+/[&]/g'
--
[this] [is] [a] [test]
```

> \w\+ 正则表达式表示匹配每一个单词，&对应于之前所匹配到的单词

多个表达式：

```shell
cat a.txt | sed 's/linux/LINUX/' | sed 's/sed/SED/'
or
cat test.txt | sed 's/linux/LINUX/;s/sed/SED/'
```

文本中包含分界符需要转义：

```shell
echo "te/st1 te/st2 te/st3 te/st4 " | sed "s/te\/st/TEST/g"
```

在包含 java 的行前插入：

```shell
sed -i '/java/i\learning ' a.txt
--
learning
java
```

在包含 java 的行后追加：

```shell
sed -i '/java/a\scala' a.txt
--
java
scala
```

删除包含 java 的行：

```shell
sed -i '/java/d' a.txt
```

