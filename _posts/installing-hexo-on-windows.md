---
title: Windows 安装 hexo 小计
date: 2017-03-24 18:22:49
tags:
 - Hexo
category: 
 - Wiki
thumbnail: 
author: bsyonline
lede: "没有摘要"
---



一直在 linux 上用 hexo ，自己还有一个 windows 的机器，也想利用起来。对于我来说，需求非常简单，windows 作为 linux 的一个补充，能在本地运行，能通过 git 同步数据即可。

<!-- more -->

### 1. 安装

首先需要按装 node.js 。和 linux 一样，下载 node.js windows 的安装包安装 node.js 环境。

然后安装 hexo 。

```
npm config set registry "https://registry.npm.taobao.org"
npm install -g hexo-cli
```

等待几分钟即可完成安装。

### 2. 配置

linux 上已经安装了 hexo ，相关的配置已经在 github 上，所以，不用重新配置，初始化完成后，同步git 即可。

```
hexo init hexo
cd hexo
git init
git remote add <url>
git pull origin master
hexo clean
hexo g
hexo s
```

以上。

