---
title: 06_Context Parallel
tags:
  - llm
category: 
author: bsyonline
lede: 没有摘要
date: 2019-05-25 12:01:20
thumbnail:
---

Context Parallel 是对所有的 input 输入和所有的输出 activation 在 sequence 维度上进行切分，可以看成是增强版的 Sequence Parallel 。通过 Context Parallel 可以更好解决 OOM 的问题，每个 GPU 只用处理一部分的 sequence , 同时减少 CP 倍的通信和计算，但保持 TP 不变，同时 activation 也会减少 CP 倍。

