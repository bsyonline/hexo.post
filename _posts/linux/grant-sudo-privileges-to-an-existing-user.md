---
title: 添加 sudo 权限
date: 2015-08-09 15:53:17
tags:
 - Linux
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有留下前言"
---

错误消息：xxx is not in the sudoers file
```
su root
visudo
```
root ALL=(ALL) ALL下添加一行
```
xxx  ALL=(ALL) ALL
```
