---
title: self-attention
tags:
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2018-05-28 15:32:21
thumbnail:
---

假设有输入向量 $a$ ，通过 $fn$ 得到输出 $b$ 。如果 $a$ 是一组向量 $a_1$，$a_2$，$a_3$，$a_4$，向量之间没有关系，那么很容易根据 $a_1$ 就能得到 $b_1$ ，根据 $a_2$ 就能得到 $b_2$，根据 $a_3$ 就能得到 $b_3$ ，根据 $a_4$ 就能得到 $b_4$ 。

如果相邻输入向量之间有关系，那么我们可以根据 $a_1$，$a_2$ 得到 $b_1$，根据 $a_1$，$a_2$，$a_3$ 得到 $b_2$ ，根据 $a_2$，$a_3$，$a_4$ 得到 $b_3$ ，根据 $a_3$，$a_4$ 得到 $b_4$ 。同理如果 $a_1$ 需要考虑整个序列的关系才能 $b_1$，也是类似的，即根据 $a_1$，$a_2$，$a_3$，$a_4$ 得到 $b_1$，根据 $a_1$，$a_2$，$a_3$，$a_4$ 得到 $b_2$ ，根据 $a_1$，$a_2$，$a_3$，$a_4$ 得到 $b_3$ ，根据 $a_1$，$a_2$，$a_3$，$a_4$ 得到 $b_4$ 。那么如何根据 $a_1$，$a_2$，$a_3$，$a_4$ 计算出 $b_1$ 呢？

第一步：
根据 $a_1$ 找出 sequence 中跟 $a_1$ 相关的项量，也就是需要计算出 $a_1$ 和其他向量的相关性，常用的做法是 $a_1$ 通过和其他向量做点乘得到。

$$
\displaylines{
Q_i=a_i \cdot W_q \tag{1} 
}
$$
$$
\displaylines{
K_i=a_i \cdot W_k \tag{2}
}
$$

比如要计算 $a_1$ 和 $a_2$ 的相关性，那么就将 $a_1$ 和 $a_2$ 分别乘上 2 个向量 $W_q$ 和 $W_k$ 得到 $Q_1$ 和 $K_2$ 。

第二步：
再将 $Q_1$ 和 $K_2$ 做点乘得到 $a_1$ 和 $a_2$ 的 attention score $R_{12}$ 。

$$
R_{12}=Q_1 \cdot K_2 \tag3
$$

同理可以得到 $R_{11}$，$R_{13}$，$R_{14}$ 。

第三步：
得到 4 个  attention score 做一次归一化之后得到 ${R_{11}}^\prime$，${R_{12}}^\prime$，${R_{13}}^\prime$，${R_{14}}^\prime$ 。

$$
{R_{11}}^\prime=Softmax(R_{11}) \tag4
$$

第四步：
将 $a_1$ 点乘 $W_v$ 得到 $V_1$ 。

$$
V_i=a_i \cdot W_v \tag 5
$$

同理得到 $V_2$，$V_3$， $V_4$ 。

第五步：
将 ${R_{11}}^\prime V_1$，${R_{12}}^\prime V_2$，${R_{13}}^\prime V_3$，${R_{14}}^\prime V_4$ 求和得到 $b_1$ 。

$$
b_1=\sum{{R_{1i}}^\prime V_i} \tag6
$$

同理可以计算出 $b_2$，$b_3$，$b_4$ 。

由（1）（2）（5）可以得到

$$
\displaylines{
Q=[a_1,a_2,a_3,a_4]\cdot W_q \tag 7
}
$$
$$
\displaylines{
K=[a_1,a_2,a_3,a_4]\cdot W_k \tag8
}
$$
$$
\displaylines{
V=[a_1,a_2,a_3,a_4]\cdot W_v \tag9
}
$$

由（3）可得

$$
\displaylines{
\begin{bmatrix}
R_{11}\\R_{12}\\R_{13}\\R_{14}
\end{bmatrix}=Q_1
\begin{bmatrix}
K_1\\K_2\\K_3\\K_4
\end{bmatrix} \tag{10} }
$$

由（10）可得

$$
\displaylines{
R=\begin{bmatrix}
{R_{11}\ \\R_{21}\ \\R_{31}\ \\R_{41}}\\
{R_{12}\ \\R_{22}\ \\R_{32}\ \\R_{42}}\\
{R_{13}\ \\R_{23}\ \\R_{33}\ \\R_{43}}\\
{R_{14}\ \\R_{24}\ \\R_{34}\ \\R_{44}}
\end{bmatrix}=\begin{bmatrix}Q_1\ Q_2\ Q_3\ Q_4\end{bmatrix}
\begin{bmatrix}
K_1\\K_2\\K_3\\K_4
\end{bmatrix}=QK^T \tag{11}
}
$$

$R$ 这个矩阵中存储的就是 attention score 。对 $R$ 做 Soft-max 得到 $R^\prime$ 

$$
\displaylines{
R^\prime=Softmax(R)=\begin{bmatrix}
{R_{11}^\prime\ \\R_{21}^\prime\ \\R_{31}^\prime\ \\R_{41}^\prime}\\
{R_{12}^\prime\ \\R_{22}^\prime\ \\R_{32}^\prime\ \\R_{42}^\prime}\\
{R_{13}^\prime\ \\R_{23}^\prime\ \\R_{33}^\prime\ \\R_{43}^\prime}\\
{R_{14}^\prime\ \\R_{24}^\prime\ \\R_{34}^\prime\ \\R_{44}^\prime}
\end{bmatrix} \tag{12}
}
$$

由（6）可得

$$
\displaylines{
[b_1\ b_2\ b_3\ b_4]=[V_1\ V_2\ V_3\ V_4]\begin{bmatrix}
{R_{11}^\prime\ \\R_{21}^\prime\ \\R_{31}^\prime\ \\R_{41}^\prime}\\
{R_{12}^\prime\ \\R_{22}^\prime\ \\R_{32}^\prime\ \\R_{42}^\prime}\\
{R_{13}^\prime\ \\R_{23}^\prime\ \\R_{33}^\prime\ \\R_{43}^\prime}\\
{R_{14}^\prime\ \\R_{24}^\prime\ \\R_{34}^\prime\ \\R_{44}^\prime}
\end{bmatrix}
}
$$

这样我们就从输入 $[a_1\ a_2\ a_3\ a_4]$ 得到了输出 $[b_1\ b_2\ b_3\ b_4]$ 。以上的过程就是 self-attention 。

self-attention 的进阶版本叫做 mutil-head self-attention 。self-attention 只生成了一组 QKV ，mutil-head self-attention 是在 QKV 的基础上再乘上一个矩阵生成出新的一组 QKV ，需要几个 head 就生成几组 QKV 。然后和 self-attention 一样对应位置 QKV 计算，算出每个位置的 $b_{ij}$ 再把他们拼起来再乘上一个矩阵得到 $b_i$ 。

self-attention v.s. CNN
CNN 是 self-attention 的特例。数据量小的时候 CNN 的效果优于 self-attention 。在数据量大的时候 self-attention 由于 CNN 。

self-attention v.s. RNN
1. 单项的 RNN 只能考虑已输入的 input ，self-attention 会考虑全部的 input 。RNN 如果需要使用输入，必须要将 input 存到内存中一层一层传递。self-attention 只需要用 QK 就可以。
2. RNN 是串行的，self-attention 是并行的。

$Q_1=a_1 \cdot W_q$

$K_1=a_1 \cdot W_k$

$K_2=a_2 \cdot W_k$

$\mathcal{K_3=a_3 \cdot W_k}$
$K_4=a_4 \cdot W_k$

$V_1=a_1 \cdot W_v$
$V_2=a_2 \cdot W_v$
$V_3=a_3 \cdot W_v$
$V_4=a_4 \cdot W_v$



