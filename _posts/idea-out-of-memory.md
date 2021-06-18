---
title: IDEA 内存溢出解决办法
date: 2014-01-13 15:49:52
tags:
 - IDEA
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 使用 idea64.exe 启动
### 2. 修改 bin 目录下的 idea.exe.vmoptions 文件
	-Xms128m
	-Xmx1024m
### 3. 添加运行VM参数
     -server -XX:PermSize=512M -XX:MaxPermSize=1024m
