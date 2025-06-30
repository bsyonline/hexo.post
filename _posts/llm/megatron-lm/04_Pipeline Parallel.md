---
title: 04_Pipeline Parallel
tags:
  - llm
category: 
author: bsyonline
lede: 没有摘要
date: 2019-05-25 12:01:20
thumbnail:
---


流水线（Pipeline Parallel）并行是对模型的层进行切分，将不同的层放到不同的 GPU 上进行处理，这样可以降低显存占用，让 GPU 能够训练更大模型。

### 标准流水线并行

假设有 4 个 GPU ，将模型分成 4 层，每个 GPU 放 1/4 的层称为 1 个 batch ，训练的过程如下：

![[标准流水线并行.png]]

GPU0 对输入进行计算得到结果作为 GPU1 的输入传给 GPU1 ，GPU1 处理的结果作为 GPU2 的输出传给 GPU2 ，依次类推，直到 GPU3 得到完整的损失，然后从 GPU3 开始 到 GPU0 依次进行反向计算梯度，最终在 GPU0 上得到完整的梯度，更新完参数后进行下一轮迭代。

![[流水线并行.png]]

我们可以看到朴素流水线并行存虽然降低了显存，但是损失和梯度的计算要所有 GPU 都完成才能进行之后的计算，所以在 GPU0 进行前向的时候 GPU1-GPU3 都处于空闲状态，GPU 资源严重浪费。

### GPipe

为了解决 GPU 资源浪费的问题，Google 提出了一种批量拆分流水线的算法 GPipe 。GPipe 的主要思想是将每个 batch 再拆分成更小的 micro-batch 来提高 GPU 的利用率。

![[images/gpipe.png]]

从图中看到对流水线进行拆分之后，一个 GPU 在工作的时候，其他 GPU 处于闲置状态，在图上形成空白的区域，被叫做气泡。气泡区域越小，说明 GPU 的利用率越高。GPipe 就是通过对 batch 切分成更小的 micro-batch 来减少气泡时间。

假设流水线并行的数量为 K （GPU 的数量），mirco-batch 的数量为 M ，朴素流水线并行的气泡时间为 $\frac{K-1}{K}$ ，GPipe 的气泡时间为

![[images/GPipe气泡时间计算.png]]

$$
\displaylines{
bubble\ time=1-\frac{2MK}{2(M+K-1) \cdot K} \\
\\
=1-\frac{M}{(M+K-1)} \\
\\
=\frac{K-1}{(M+K-1)}
}
$$

当由于 GPU 数量 K 是固定的，当不断增加 mirco-batch 的数量 M ，气泡时间趋近于 0 。所以增加micro-batch 的数量能够有效提高 GPU 的资源利用率。

### 1F1B

标准的流水线并行和 GPipe 都是做完所有的前向再进行反向，这种方式叫做 **F-then-B** 。GPipe 将 batch 切分成 micro-batch ，虽然提高了 GPU 的利用率，但是并没有降低显存占用，甚至每个 micro-batch 还要额外占用显存来存储自己的中间激活值。为了解决显存占用问题，微软提出了一种 **1F1B（One Forward pass followed by One Backward pass）** 策略 PipeDream 。1F1B 就是在做完前向计算之后立刻进行反向计算，计算完之后将中间激活值丢掉以节省显存。

#### 非交错式 1F1B

非交错式分成 3 个阶段，第一阶段是热身阶段，处理器进行不同数量的前向计算。在第二阶段，处理器进行一次前向计算，然后是一次后向计算。第三阶段，处理器完成后向计算。
![[非交错式1F1B.png]]
非交错式气泡率和 GPipe 相同，显存占用显著降低。

#### 交错式 1F1B

要求 **microbatches 的数量是流水线阶段的整数倍**。
![[交错式1F1B.png]]

交错式 1F1B 增加了 N 个 model chuck ，与 GPipe 相比气泡率为 GPipe 的 1/N 。 