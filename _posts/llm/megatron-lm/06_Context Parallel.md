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

> [!论文]
> [Reducing Activation Recomputation in Large Transformer Models](https://arxiv.org/pdf/2205.05198)




Context Parallel 是对所有的 input 输入和所有的输出 activation 在 sequence 维度上进行切分，Sequence Parallel 仅对 Dropout 和 Layernorm 进行切分，Context Parallel 除了对 Dropout 和 Layernorm 切分之外，还对 attention 部分进行切分，可以看成是增强版的 Sequence Parallel 。通过 Context Parallel ，每个 GPU 只用处理一部分的 sequence , 同时减少 CP 倍的通信和计算，但保持 TP 不变，同时 activation 也会减少 CP 倍。所以  Context Parallel 可以更大程度上降低激活的显存占用，更好解决 OOM 的问题。

SP 的侧重点是减少序列长度的限制，CP 的侧重点是降低激活显存占用。在 megatron 中 CP 基于 ring-attention 实现。

ring attention
计算和通信可以overlapped
GPU只需要向固定的GPU发数据
通信量小

ulysses
受限于head的数量，因为tp也需要用head
all2all对网络拓扑要求高
大多数情况通信量高于ring attention