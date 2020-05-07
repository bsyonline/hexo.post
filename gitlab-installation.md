---
title: Gitlab 安装
date: 2016-05-16 15:45:21
tags:
 - Git
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 1. 安装依赖

```
sudo yum install curl openssh-server openssh-clients postfix cronie
sudo service postfix start  
sudo chkconfig postfix on  
sudo lokkit -s http -s ssh
```

### 2. 添加GitLab package server

```
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash  
sudo yum install gitlab-ce-8.5.0-ce.1.el6.x86_64
```

### 3. 安装

```
sudo rpm -i gitlab-ce-8.5.0-ce.1.el6.x86_64.rpm
```


### 4. 配置

```
sudo gitlab-ctl reconfigure
```

### 5. 配置邮件服务

```
sudo yum install postfix
yum install cyrus*
```

