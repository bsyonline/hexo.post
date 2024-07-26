---
title: Ubuntu 16.04 软件安装及配置
date: 2016-07-15 22:25:09
tags:
 - Ubuntu
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有留下前言"
---


### 0. 更新源
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list_bak
```

```
deb http://mirrors.sohu.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.sohu.com/ubuntu/ xenial-updates main restricted
deb http://mirrors.sohu.com/ubuntu/ xenial universe
deb-src http://mirrors.sohu.com/ubuntu/ xenial universe
deb http://mirrors.sohu.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.sohu.com/ubuntu/ xenial-updates universe
deb http://mirrors.sohu.com/ubuntu/ xenial multiverse
deb-src http://mirrors.sohu.com/ubuntu/ xenial multiverse
deb http://mirrors.sohu.com/ubuntu/ xenial-updates multiverse
deb-src http://mirrors.sohu.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.sohu.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.sohu.com/ubuntu/ xenial-backports main restricted universe multiverse

deb http://mirror.bjtu.edu.cn/ubuntu/ xenial main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ xenial-backports main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ xenial-proposed main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ xenial-security main multiverse restricted universe
deb http://mirror.bjtu.edu.cn/ubuntu/ xenial-updates main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ xenial main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ xenial-backports main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ xenial-proposed main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ xenial-security main multiverse restricted universe
deb-src http://mirror.bjtu.edu.cn/ubuntu/ xenial-updates main multiverse restricted universe
```
```
sudo apt-get update
```
###1. 安装搜狗输入法
>sudo apt-get install fcitx libssh2-1

####1.1 下载deb包
####1.2 安装
>sudo dpkg -i sogoupinyin_2.0.0.0072_amd64.deb
####1.3 设置
打开语言支持，键盘输入法系统选择fcitx。
    打开fcitx配置，添加搜狗输入法，注销后生效。

###2. 安装Unity Tweak Tool
启动器图标单击收起。minimize single window applications on click

###3. 删除libreoffice
>sudo apt-get remove libreoffice-common
###4. 删除Amazon的链接
>sudo apt-get remove unity-webapps-common
###5. 深度清理用不上的应用程序
>sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku  landscape-client-ui-install

>sudo apt-get remove onboard deja-dup

###6. 安装Terminator

###7. 安装WPS Office
```
sudo dpkg -i wps-office_10.1.0.5444~a20_amd64.deb
```
wps不能输入中文
```
sudo gedit /usr/bin/wps

#!/bin/bash
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"
gOpt=
#gOptExt=-multiply
gTemplateExt=("wpt" "dot" "dotx")
```
###8. 安装git
>sudo apt-get install git
###9. 安装Java
###10. 安装字体
####10.1 将windows字体拷贝到/usr/share/fonts/windows/
####10.2 修改权限
```
sudo chmod 664 /usr/share/fonts/windows/*
```
####10.3 安装
```
cd /usr/share/fonts/windows/
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```
###11. 安装haroopad
编辑区主题 paraiso-darkdark
###12. 安装unrar
>sudo apt-get install unrar
###13. 安装idea
###14. 安装mysql客户端navicat
###15. 安装maven
###16. 安装vmware
###17. 安装docker 未完成
###18. 安装node.js
###19. 安装gulp
###20. linux版百度网盘 bcloud
###21. thunderbird
####21.1 添加插件
Thunderbird Conversations  
Manually Sort Folders  
Color Folders
Extra Folder Columns
LookOut  
ViewAbout  
Theme Font & Size Changer  

####21.2 去除签名档之前的分割符
mail.identity.default.suppress_signature_separator=true

###22. 关闭系统错误报告
>sudo rm /var/crash/*
    sudo gedit /etc/default/apport

修改enabled=1为enabled=0

###23. dash不搜索第三方应用
>wget -q -O - https://fixUbuntu.com/fixubuntu.sh | bash

###24. chrome
```
sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
sudo apt-get update
sudo apt-get install google-chrome-stable

```
### 25. plank
安装
```
sudo add-apt-repository ppa:docky-core/stable
sudo apt-get update
sudo apt-get install plank
```
设置开机启动
dash中搜索启动应用程序，添加plank路径/usr/bin/plank

### 26. 关闭utc
/etc/default/rcS
```
utc=no
```
### 27. uGet
### 28. fileZilla
### 29. MySQL Workbench
