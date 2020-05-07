---
title: 每天一个 Linux 命令（3）： scp
date: 2017-02-11 10:06:17
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

scp 是 secure copy 的简写，用于在Linux下进行远程拷贝文件的命令，而且 scp 传输是加密的。

<!-- more -->

scp 命令格式：
```shell
Usage: scp [-12346BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file]
           [-l limit] [-o ssh_option] [-P port] [-S program]
           [[user@]host1:]file1 ... [[user@]host2:]file2

Options:
  -1  强制scp命令使用协议ssh1  
  -2  强制scp命令使用协议ssh2  
  -4  强制scp命令只使用IPv4寻址  
  -6  强制scp命令只使用IPv6寻址  
  -B  使用批处理模式（传输过程中不询问传输口令或短语）  
  -C  允许压缩。（将-C标志传递给ssh，从而打开压缩功能）  
  -p  保留原文件的修改时间，访问时间和访问权限。  
  -q  不显示传输进度条。  
  -r  递归复制整个目录。  
  -v  详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。   
  -c  cipher  以cipher将数据传输进行加密，这个选项将直接传递给ssh。   
  -F  ssh_config  指定一个替代的ssh配置文件，此参数直接传递给ssh。  
  -i  identity_file  从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。    
  -l  limit  限定用户所能使用的带宽，以Kbit/s为单位。     
  -o  ssh_option  如果习惯于使用ssh_config(5)中的参数传递方式，   
  -P  port  注意是大写的P, port是指定数据传输用到的端口号   
  -S  program  指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。
```

实际使用很简单，只用过选项 r :
拷贝目录到远程

```
scp -r /etc ops@hadoop1:/tmp/etc/
```

拷贝文件到远程

```
scp /etc/hosts ops@hadoop1:/tmp/
```

拷贝文件到本地

```
scp ops@hadoop1:/etc/hosts /tmp/
```
