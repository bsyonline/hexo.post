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





#### Recursive Halving and Doubling（Tree）





#### Butterfly





#### Ring


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
```
algbw = size (S) / time (t)
```

```
All Reduce：busbw = algbw * (2*(n-1)/n)

ReduceScatter：busbw = algbw * (n-1)/n

AllGather：busbw = algbw * (n-1)/n
```

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


