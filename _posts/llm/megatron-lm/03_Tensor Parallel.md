---
title: 03_Tensor Parallel
tags:
  - llm
category: 
author: bsyonline
lede: 没有摘要
date: 2019-05-25 12:01:20
thumbnail:
---
张量并行，是把模型进行切分放置到不同设备之上，也可以理解为**把矩阵运算**分配到不同的设备之上，比如把某个矩阵乘法切分成为多个矩阵乘法放到不同设备之上。
模型并行的问题在于：
	a）all-reduce通信单次通信数据量大，并且通信频繁。但是大模型需要分布在多个节点上，节点之间的通信要比机内慢很多。
	b）GPU更适合处理大规模的矩阵运算，但是张量并行把矩阵运算分割到各节点上，运算的规模变小了，数量变多了，故GPU利用率会降低。

### TP 的原理

![[images/模型并行.png]]
对于输入 X 和权重 W：

$$\displaylines{
X\cdot W=Y
}
$$

#### 行并行

将 W 按照行切分为 $W_1$、$W_2$，对应的输入需要切分为 $X_1$、$X_2$，即：

$$\displaylines{
[X_1,X_2]\cdot
\begin{bmatrix}
W_1\\W_2
\end{bmatrix}=X_1 \cdot W_1+X_2 \cdot W_2=Y \\


}

$$

假如有2个 GPU，我们可以将 $X_1 \cdot W_1$ 放到 GPU1 上计算得到结果 $Y_1$，将 $X_2 \cdot W_2$ 放到 GPU2 上计算得到结果 $Y_2$，最后将结果 $Y_1$，$Y_2$ 相加得到最终结果 Y。

#### 列并行

将 W 按照列切分为 $W_1$、$W_2$，输入 X 不需要切分，即

$$\displaylines{ 
X \cdot W_1=Y_1\\ 
\\
X \cdot W_2=Y_2
}$$

将 $X \cdot W_1$ 放到 GPU1 上计算得到结果 $Y_1$，将 $X \cdot W_2$ 放到 GPU2 上计算得到结果 $Y_2$，最后将 $Y_1$ 和
$Y_2$ 按列拼接得到最终结果 Y。

