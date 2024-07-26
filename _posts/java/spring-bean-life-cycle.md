---
title: Spring Bean Life Cycle
tags:
  - Interview
category:
  - Spring
author: bsyonline
lede: 没有摘要
date: 2019-11-07 18:39:29
thumbnail:
---

<img src="https://s1.ax1x.com/2020/04/04/GwMsOK.png" alt="GwMsOK.png" border="0" style="width:500px;"/>

Spring bean 的生命周期有 5 个阶段：
1. Prepare 阶段
2. Instantation 阶段
3. PopulateBean 阶段
4. Initailization 阶段
5. Destory 阶段

Instantation 之前会执行 postProcessBeforeInstantiation 。
Instantation 之后会执行 postProcessAfterInstantiation 和 postProcessProperties 。
Initailization 之前执行 postProcessBeforeInitialization 。
Initailization 之后执行 postProcessAfterInitialization 。