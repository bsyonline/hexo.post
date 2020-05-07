---
title: 每天一个 Linux 命令（2）： grep
date: 2017-02-09 16:51:00
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

grep 是比较常用的一个命令，用来搜索文件或标准输出中的匹配模式的内容，默认打印出匹配的行。
<!-- more -->

grep 命令格式：
```shell
Usage: grep [OPTION]... PATTERN [FILE]...

Options:
  --help                    Print a usage message briefly summarizing these command-line options and the bug-reporting address, then exit.
  -i, --ignore-case         Ignore  case  distinctions  in  both  the PATTERN and the input files.  (-i is specified by POSIX.)
  -c, --count               Suppress normal output; instead print a count of matching lines for each input file.
                            With the -v,  --invert-match option (see below), count non-matching lines.  (-c is specified by POSIX.)
  -l, --files-with-matches  Suppress normal output; instead print the name of each input file from which output would normally have
                            been printed.  The scanning will stop on the first match.  (-l  is  specified by POSIX.)
  -n, --line-number         print line number with output lines
      --line-buffered       flush output on every line
```

用过的选项也就上面几个。
举个例子吧，查找 test.txt 文件中 linux 出现的次数，忽略大小写。

```shell
grep -ic linux test.txt
```

grep 也支持正则表达式。
```
^         #锚定行的开始 如：'^grep'匹配所有以grep开头的行。

$         #锚定行的结束 如：'grep$'匹配所有以grep结尾的行。

.         #匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。

*         #匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。

.*        #一起用代表任意字符。

[]        #匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。

[^]       #匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。

\(..\)    #标记匹配字符，如'\(love\)'，love被标记为1。

\<        #锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。

\>        #锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。

x\{m\}    #重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。

x\{m,\}   #重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。

x\{m,n\}  #重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。

\w        #匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。

\W        #\w的反置形式，匹配一个或多个非单词字符，如点号句号等。

\b        #单词锁定符，如: '\bgrep\b'只匹配grep。
```

```shell
grep -ic lin.x test.txt
```
效果和第一个一样。
