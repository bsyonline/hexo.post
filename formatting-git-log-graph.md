---
title: Formatting Git Log Graph
tags:
  - Git
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2020-02-02 22:23:06
thumbnail:
---

git log 命令可以查看 git 的提交历史，--graph 参数是将 git log 信息图形化显示。可以同过定制 format 格式来优化显示。
在 git 的安装目录下找到 etc\profile.d\aliases.sh ，加入 alias 配置。
```
alias glog="git log --graph --pretty=format:'%C(red)%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
重启终端，使用 glog 命令就可以查看定制格式的 git log 信息了。