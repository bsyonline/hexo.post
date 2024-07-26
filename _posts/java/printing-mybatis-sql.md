---
title: Printing Mybatis SQL
tags:
  - MyBatis
category:
  - MyBatis
author: bsyonline
lede: 没有摘要
date: 2020-01-31 20:08:40
thumbnail:
---

打印出的 sql 可以帮助调试程序。需要在配置文件添加日志配置。
```
logging:
  level:
    com.rolex.microlabs.dao: debug # 查询接口的包名
```
打印 sql 信息如下：
```
==>  Preparing: select id, name, age, gender, skill from t_user WHERE age=? and gender=? 
==> Parameters: 20(Integer), 1(Integer)
<==      Total: 1
```