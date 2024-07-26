---
title: 每天一个 Linux 命令（16）： which
date: 2017-02-23 10:09:09
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

which 命令的作用是在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。换句话说，可以用 which 查看一个命令是否存在。

<!-- more -->

which 命令格式：

```
Usage: which [-a] filename
```

>which 是在 PATH 中查找，不同的环境变量，查找结果不一定相同。
>如果一个命令使用 which 查不到，那么肯定在 PATH 中没有配置。
