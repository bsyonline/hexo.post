---
title: how to build riskbell on blockchain
tags:
  - Blockchain
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-11 13:09:44
thumbnail:
---


风铃目前面临的问题：
* 数据处理资源不足
* 存储中心化，要保证服务高可用

用 riskbell 基于 blockchain 需要具有哪些功能
* 分布式数据存储
* 数据就近查询
* 保证数据隐私（公司数据和客户数据）
* 交易记录可查

需要搞清楚如何实现的点：
* 如何将企业数据放到 blockchain 中
    STATUS into blockchain and data into IPFS
* 用户如何查询到企业数据（不能随意查，即要有相应的控制措施）
* 如何订正数据