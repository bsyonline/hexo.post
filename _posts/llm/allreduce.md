---
title: Build Ci/CD
tags:
  - llm
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


### 什么是 All-Reduce

All-Reduce算法是分布式训练中常用的一种通信算法，用于在节点间同步参数。比如在大模型训练中节点之间同步梯度。All-Reduce算法的基本思想是先将所有节点的参数相加，然后再将结果广播到所有节点。过程如下：

1. 每个节点将本地的参数值相加，得到一个本地的总和；
2. 所有节点将总和发送给根节点；
3. 根节点将所有总和再相加，得到全局的参数总和；
4. 根节点将全局总和广播给所有节点；
5. 每个节点将本地的参数值与全局总和相减，得到新的参数值。

通过以上步骤，每个节点都得到了更新后的参数值，实现了参数同步。相比于其他通信算法，All-Reduce算法能够更好地平衡节点间的通信开销，提高训练效率。

### All-Reduce 有哪些种类

#### Reduce+Broadcast

![[images/reduce and broadcast.png]]
这种就是最典型的 PS 架构，这种算法的缺点是 PS 的带宽是最大的瓶颈。

#### Recursive Halving and Doubling（Tree）

![[images/Recursive Halving and Doubling.png]]

通信步数 2\*log2N。
优点：通信步数少；通信延迟低；扩展性好。
缺点：halving阶段有一半的节点带宽处于闲置；每步通信节点都会变。依赖网络拓扑；如果参数量过大，带宽会有压力。

#### Butterfly


通信步数 log2N。
优点：带宽利用率高；通信步数少；扩展性好。
缺点：复杂度高；依赖网络拓扑；如果参数量过大，带宽会有压力。



#### Ring

`Ring-Allreduce` 工作过程被分成两个阶段，即`Reduce-Scatter`和 `Allgather`。
优点：避免了中心节点的问题，通信的数据量只和参数总量有关，与机器数量无关。
缺点：通信步数多，2*(N-1)。

#### 2D-Torus

[https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9211480](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9211480)

#### 2D-Mesh

[https://www.researchgate.net/publication/345654115_Highly_Available_Data_Parallel_ML_training_on_Mesh_Networks](https://www.researchgate.net/publication/345654115_Highly_Available_Data_Parallel_ML_training_on_Mesh_Networks)

#### 3D-Torus

[https://dominoweb.draco.res.ibm.com/reports/rc25088.pdf](https://dominoweb.draco.res.ibm.com/reports/rc25088.pdf)

#### Double binary trees

[https://developer.nvidia.com/blog/massively-scale-deep-learning-training-nccl-2-4/](https://developer.nvidia.com/blog/massively-scale-deep-learning-training-nccl-2-4/)


All-Reduce 是并行计算和分布式计算中常用的通信操作，用于多个节点之间汇总数据，比如在模型训练中汇总各节点上的梯度信息。All-Reduce 工作过程被分成两个阶段，即 Reduce-Scatter 和  All-Gather 。

| 命令                  | 算法             | 说明                                             |
| ------------------- | -------------- | ---------------------------------------------- |
| all_gather_perf     | All-Gather     | 每个进程发送一个消息给所有其他进程，最终每个进程都会收到所有进程的消息。           |
| all_reduce_perf     | All-Reduce     | 每个进程发送一个消息给所有其他进程，所有消息在一个进程中进行聚合，然后将结果广播回所有进程。 |
| alltoall_perf       | All-to-All     | 每个进程发送一个消息给所有其他进程，每个进程也会从所有其他进程接收一个消息。         |
| broadcast_perf      | Broadcast      | 一个进程将消息发送给所有其他进程。                              |
| gather_perf         | Gather         | 所有进程将消息发送给一个指定的根进程，根进程收集所有消息。                  |
| hypercube_perf      | Hypercube      | 一种特殊的 All-Reduce 算法。                           |
| reduce_perf         | Reduce         | 所有进程将消息发送给一个指定的根进程，根进程对所有消息进行聚合，并将结果保存。        |
| reduce_scatter_perf | Reduce-Scatter | 所有进程将消息发送给所有其他进程，每个进程对部分消息进行聚合，并将结果保存。         |
| scatter_perf        | Scatter        | 一个指定的根进程将消息分发给所有其他进程，每个进程接收一部分消息。              |
| sendrecv_perf       | Send-Recv      | 每个进程发送一个消息给另一个指定的进程，并从另一个指定的进程接收一个消息。          |

All-Gather 和 Broadcast 的区别

```
All-Gather                                   Broadcast

P0: [D0] P0: [D0, D1, D2, D3]                P0: [D0] P0: [D0]

P1: [D1] P1: [D0, D1, D2, D3]                P1: [D1] P1: [D0, D1]

P2: [D2] P2: [D0, D1, D2, D3]                P2: [D2] P2: [D0, D2]

P3: [D3] P3: [D0, D1, D2, D3]                P3: [D3] P3: [D0, D3]
```


### All-Reduce 带宽计算

### 什么是带宽？algbw 是什么？busbw 又是什么？

网络或数字信号中的带宽是指在时间t内从一端流到另一端的信息量，即数据传输率。
如果我们有一个数据data 大小s=10G，从P1拷贝data到P2耗时t=5秒，那么我们可以计算P1P2之间的带宽为

$$
\displaylines{
algbw = S/t\\
algbw:alg\ bandwidth\\
S: size\\
t: time
}


$$

对于 P1 到 P2 这样的点对点通信，算法带宽是有意义的，但是对于集合操作是不准确的。因为算法带宽的峰值并不等于硬件的带宽峰值，它依赖于Rank的数量。通常随着 Rank 的增加，带宽会降低。为了体现硬件有没有被更好的使用，Nvidia 引入了总线带宽 busbw 。**总线带宽是算法带宽通过一个公式（不同的集合操作，公式也不一样）计算得到的，用来反映 GPU 之间的通信速率。换句话说，总线带宽能够反映硬件的速度瓶颈，包括 NVLink, PCI, QPI, network。**

NCCL 在 2.4 版本之前默认使用的算法是 Ring-Allreduce 。我们知道一次完整的Ring-Allreduce 做了 N-1 次 Reduce-Scatter ，N-1 次 All-Gather ，每次发送 1/N 份数据。故 Reduce-Scatter 的计算公式为：

$$
\displaylines{
busbw = (n-1) * (1/n * S) / t \\
= (n-1) * 1/n * S/t\\
=algbw * (n-1)/n
}
$$
All-Gather的计算公式为：

$$
\displaylines{
busbw = (n-1) * S/n/t \\
= (n-1) * (1/n * S) / t \\
= algbw * (n-1)/n
}
$$

最终All-Reduce的公式为：

$$
busbw = algbw * (2 * (n-1)/n)
$$




```
#                            out-of-place                           in-place

# size     count      type  redop root  time   algbw  busbw  #wrong time   algbw busbw  #wrong

# (B)      (elements)                   (us)   (GB/s) (GB/s)        (us)   (GB/s) (GB/s)

33554432   8388608    float  sum   -1   1332.0 25.19  47.23  0      1304.2 25.73 48.24   0

67108864   16777216   float  sum   -1   2615.3 25.66  48.11  0      2616.4 25.65 48.09   0

134217728  33554432   float  sum   -1   5220.7 25.71  48.20  0      5219.6 25.71 48.21   0

# Out of bounds values : 0 OK

# Avg bus bandwidth : 48.0162
```

一次完整的 All-Reduce 做了 N-1 次 Reduce-Scatter ，N-1 次 All-Gather ，每次发送 1/N 份数据。

基本的带宽计算：
$$\displaylines{
algbw=\frac{size(S)}{time(t)}
}
$$
ReduceScatter 的带宽为

AllGather 的带宽为
$$
busbw = algbw * (n-1)/n
$$
All Reduce 的带宽为
$$
busbw = algbw * (2*(n-1)/n)
$$

通过 nccl-test 结果验证：

```
# out-of-place in-place

# size count type redop root time algbw busbw #wrong time algbw busbw #wrong

# (B) (elements) (us) (GB/s) (GB/s) (us) (GB/s) (GB/s)

33554432 8388608 float sum -1 1332.0 25.19 47.23 0 1304.2 25.73 48.24 0

67108864 16777216 float sum -1 2615.3 25.66 48.11 0 2616.4 25.65 48.09 0

134217728 33554432 float sum -1 5220.7 25.71 48.20 0 5219.6 25.71 48.21 0

# Out of bounds values : 0 OK

# Avg bus bandwidth : 48.0162
```

```
algbw = （33554432/1000/1000/1000）/ （1332/1000/1000）= 25.19（GB/s）

N=8（GPU）

busbw = 25.19 * 2 *（16-1）/ 16 = 47.23（GB/s）
```


通信原语

![[通信原语.png]]