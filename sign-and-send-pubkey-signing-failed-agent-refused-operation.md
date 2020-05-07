---
title: 'sign_and_send_pubkey: signing failed: agent refused operation 错误的解决办法'
tags:
  - Git
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2018-01-04 17:04:47
thumbnail:
---

Git 安装配置公钥后，报错：
```
sign_and_send_pubkey: signing failed: agent refused operation
```
执行以下代码即可：
```
eval "$(ssh-agent -s)"
ssh-add
```