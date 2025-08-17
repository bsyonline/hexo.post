---
title: verl训练PPO
tags:
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2018-05-28 15:32:21
thumbnail:
---

![[Pasted image 20250813095728.png]]


#### 生成阶段

从 dataloader 中取数据转成 DataProto 格式放到 batch 中。从 batch 中取出指定 key 的数据 gen_batch ，然后对 gen_batch 进行 repeat ，这个是 GRPO 的需要的，在 PPO 中不需要设置 rollout.n ，默认是 1 。对 gen_batch 进行 rollout 之后，再合并到 batch 。这个过程实际就是对 batch 的输入生成输出，然后将输入和输出合起来。为了减少数据传输的开销和内存占用，所以才先放到 gen_batch 。

#### 计算奖励阶段

计算奖励阶段，verl 使用多源奖励融合机制。开启了 reward_model 就使用 reward_model 计算奖励，没有开启 reward_model 就使用 reward_fn 计算奖励。reward_model 和 reward_fn 可以同时生效。

#### 优势估计阶段






