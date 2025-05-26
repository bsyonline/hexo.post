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

### Encoder

Encoder 的作用就是输入一组向量然后输出一组向量，RNN、CNN、self-attention 都可以做，transformer 的 encoder 用的就是 self-attention 。输入的向量 $a$ 通过 self-attention 输出向量 $b$ ，将 $a + b$ 作为输入进行一次 Layer Norm 得到结果作为 Feed Forward 的输入。将 $a + b$ 作为后续输入的架构叫做 Residual 。经过 Feed Forward 得到的结果再进行一次 Residual + Layer Norm 得到最终的输出 $h$ 。

![[images/transformer-encoder.png]]

这是一个 Layer 的处理过程，整个 encoder 会包含很多个 Layer 。当然我们要知道，这种 encoder 的设计结构是最初论文的设计，可能并不是最好的设计，它还有很多变形就不在这里展开。

### Decoder

从 transformer 的模型图我们可以看到，Encoder 的输出会作为 Decoder 的输入。在 Decoder 的处理过程中，首先会接收到一个特殊的 token 表示开始，然后生成一个词表长度的向量，对这个向量做 softmax 后挑出打分最高的 token 作为输出，然后再将输出作为下一步的输入，依次类推，直到输出一个特殊的结束 token 。整个过程如下图所示。

![[images/transformer-decoder.png]]

#### Masked Self-Attention

再回到 transformer 的模型图，我们发现 Encoder 和 Decoder 的输入都会先经过一个 Multi-Head Attention ，区别是 Decoder 的 Multi-Head Attention 是 Masked Multi-Head Attention 。在 self-attention 中，输出向量 $b1$ 由所有的输入向量 $a1,a2,a3,a4$ 计算得到，而在 Decoder 中，输出是一个一个 token 输出的，所以向量 $b1$ 只由 $b1$ 之前的输入向量 $a1$ 计算得到，向量 $b2$ 只由 $b2$ 之前的向量 $a1,a2$ 计算得到，以此类推。 

#### Cross Attention

在 Decoder 中还有一个 Multi-Head Attention ，和 Encoder 的 Multi-Head Attention 不同，Encoder 的 Multi-Head Attention 的 $Q,K,V$ 都来自 Encoder ，而 Decoder 中的这个 Encoder 的 Multi-Head Attention 的 $Q$ 来自 Decoder ，$K,V$ 来自 Encoder 。这个过程就叫做 Cross Attention 。

![[cross attention.png]]

