---
title: 每天一个 Linux 命令（3）： tar
date: 2017-02-10 10:29:14
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

今天来看看 linux 的打包命令 tar 。在 linux 中打包指将多个文件或目录合成一个文件，这个过程可以用 tar 来完成，生成的文件通常以 .tar 结尾。和打包相关还有一个压缩，linux 只能针对一个文件进行压缩，所以通常我们要将多个文件或目录打成一个包，再进行压缩。
<!-- more -->
### tar
tar 命令格式：
```shell
Usage: tar [OPTION...] [FILE]...

Options:
-z, --gzip, --gunzip, --ungzip    filter the archive through gzip
-j, --bzip2                       filter the archive through bzip2
-v, --verbose                     verbosely list files processed
    --warning=KEYWORD             warning control
-x, --extract, --get              extract files from an archive
-f, --file=ARCHIVE                use archive file or device ARCHIVE
    --force-local                 archive file is local even if it has a colon
-c, --create                      create a new archive
-t, --list                        list the contents of an archive
    --test-label                  test the archive volume label and exit

```

还是举几个例子吧。

创建 tar 包
```
tar -cf test.tar /etc/hosts
```
提取 tar 包
```
tar -xf test.tar
```
打包并压缩
```
tar -zcf test.tar.gz /etc/hosts
```

解压缩 .tar.gz 文件
```
tar -zxf kernel.tar.gz
```

>在参数 f 之后的文件档名是自己取的，我们习惯上都用 .tar 来作为辨识。 如果加 z 参数，则以 .tar.gz 或 .tgz 来代表 gzip 压缩过的 tar包； 如果加 j 参数，则以 .tar.bz2 来作为tar包名。

查看包中的内容
```
tar -tvf test.tar
```
如果是 .tar.gz 包，在加上 z 选项就可以了。

提取包中的某个文件
```
tar -zxf test.tar.gz ./books/c.txt
```
