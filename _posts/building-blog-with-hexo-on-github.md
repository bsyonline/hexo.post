---
title: Github + Hexo 搭建 Markdown 博客系统
date: 2016-07-14 19:20:12
tags:
 - Hexo
category: 
 - Wiki
thumbnail: 
author: bsyonline
lede: "没有摘要"
---



为了更好的积累和保存学习笔记，需要满足以下要求：

1. 免费；
2. 界面漂亮
3. 能使用markdown语法书写；
4. 能够导出到evernote。

试了很多markdown的博客系统，在线编辑器，markdown软件，都无法满足，后来发现了Hexo。


Hexo是一个 markdown 博客系统，初步了解优点如下：

1. 没有数据库，直接保存 markdown 文件，所以迁移应该很方便；
2. 可以自选theme；
3. 在没有网络主机的情况下，可在本地搭建博客系统，通过github页可以实现远程备份；
4. 开源。

以上基本可以满足个人需求。

## 如何搭建本地Hexo环境？

搭建Hexo环境很简单，Hexo是基于nodejs的，所以首先需要有node环境，如何安装node可参考[源码编译安装nodejs](www.aaa.com)。

### 1. 安装Hexo
```shell
npm install hexo -g
```

### 2. 升级
```shell
npm update hexo -g
```

### 3. 初始化
```shell
mkdir hexo
cd hexo
hexo init
```

### 4. 生成
```shell
hexo generate
or
hexo g
```

### 5. 启动服务
```shell
hexo server
or
hexo s
```

启动后可在[http://localhost:4000](http://localhost:4000)访问。

## 如何更换主题

Hexo有很多主题，[https://hexo.io/themes/](https://hexo.io/themes/)网站上列举了一些主题，并带有效果展示。如果没有中意的主题，可以继续google一下。

安装主题方法也很简单，以 maupassant 为例：
### 1. 下载主题
```shell
cd hexo/themes
git clone git@github.com:tufu9441/maupassant-hexo.git maupassant
npm install hexo-renderer-jade --save
npm install hexo-renderer-sass --save
```
### 2. 更换主题
```shell
vi hexo/_config.yml

language: zh-CN
theme: maupassant
```
从新启动后可看到效果。

## 第一篇博客

现在可以开始写第一篇博客了，使用
```shell
hexo new helloworld
or
hexo n helloworld
```
就创建了一篇新博客，存放在 **hexo/source/_posts/** 下，刷新网页可看到效果。

## 部署到github
使用github的好处除了进行远程备份之外，还能直接通过github网站访问博客，非常方便。
### 1. 修改配置
```shell
vi hexo/_config.yml

url: http://bysonline.github.io
deploy:
  type: git
  repository: git@github.com:bsyonline/bsyonline.github.io.git
  branch: master
```
### 2. 部署
```shell
hexo g
hexo d
```

完成之后可以访问bsyonline.github.io看到效果。
注：github的仓库名字必须是xxx.github.io


至此，本地的hexo博客系统就搭建好了。
