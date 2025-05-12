---
title: transformer
tags:
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2018-05-28 15:32:21
thumbnail:
---


seq2seq model 是一个功能强大的模型，基本的结构包含 2 部分，encoder 和 decoder 。由 encoder 处理 input sequence ，再将 encoder 处理的结果交给 decoder ，最后由 decoder 决定输出的 output sequence 。transformer 就是一种 seq2seq 的模型。

![[transformer.png]]

Encoder
Encoder 的作用就是输入一组向量然后输出一组向量，RNN、CNN、self-attention 都可以做，transformer 的 encoder 用的就是 self-attention 。输入的向量 $a$ 通过 self-attention 输出向量 $b$ ，将 $a + b$ 作为输入进行一次 Layer Norm 得到结果作为 Feed Forward 的输入。将 $a + b$ 作为后续输入的架构叫做 Residual 。经过 Feed Forward 得到的结果再进行一次 Residual + Layer Norm 得到最终的输出 $h$ 。

![[images/transformer-encoder.png]]

这是一个 Layer 的处理过程，整个 encoder 会包含很多个 Layer 。当然我们要知道，这种 encoder 的设计结构是最初论文的设计，可能并不是最好的设计，它还有很多变形就不在这里展开。

Decoder




