---
title: 05_Sequence Parallel
tags:
  - llm
category: 
author: bsyonline
lede: 没有摘要
date: 2019-05-25 12:01:20
thumbnail:
---

> [!论文]
> [Sequence Parallelism: Long Sequence Training from System Perspective](https://arxiv.org/pdf/2105.13120)
> 

在 self-attention 中显存占用和序列长度是平方关系 $2bs^2a$ ，所以长序列会显著增加显存占用，所以这篇论文提出了 Sequence Parallel ，它是一种内存高效的并行性方法，可以对输入以及激活的Dropout 和 Layernorm 进行切分，从而打破输入序列长度限制，在 GPU 上有效地使用更长的序列进行训练。同时 Sequence Parallel 可以和其他并行策略（DP、TP、PP）兼容。

### Ring self-attention

现有系统要求将整个序列保存在一个 GPU 中，这限制了输入序列的长度。Sequence Parallel 首先沿序列维度将输入序列拆分为多个块，并将每个子序列块送到一个相应的 GPU。每个 GPU 只包含完整序列的一部分，即子序列。每个 GPU 将根据其本地 $Q,K$ 来计算部分注意力分数 $S=QK^T$ ，然后将 $K$ 传输到下一个 GPU 上，保证一个 $Q$ 都能和之前所有的 $K$ 计算出本地的注意力分数 $S$。经过 n-1 次传输，每个 GPU 上就有所有的 $K$ 值，就可以计算出完整的 $QK^T$ 。

![[ring-attention-k.png]]

在下一阶段，每个设备根据注意力分数 $S$ 和 $V$ 计算出输出 $O$ ，并将 $V$ 传递给下一个 GPU 来计算输出 $O$ 。同样经过 n-1 次传输，每个 GPU 上就有了所有的 $V$ 值，就可以计算出完整的 $O$ 。

![[ring-attention-v.png]]


这种方式叫做 ring self-attention 。


### 通信

![[tp-sp.png]]