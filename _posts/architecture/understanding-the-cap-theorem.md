---
title: Understanding the CAP Theorem
tags:
  - Interview
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-05-28 17:54:57
thumbnail:
---

CAP 定理是分布式计算中的一个概念，由 Eric Brewer 提出。 CAP 是 Consistency、Availability、Partition tolerance 的缩写。最初 CAP 是一个假设，即在分布式系统中，CAP 3 项不可能同时满足，只能同时满足其中两项。后来由 Seth Gilbert 和 Nancy Lynch 对该假设进行了证明。
那么 CAP 定理如何理解呢？我们先重申一下 CAP 的释义。

* Consistency 
	一致性，即所有节点副本内容都是相同的。要强调一下，这里所说的一致性，是线性一致性或者叫顺序一致性。
* Availability
	可用性，即每一个请求都会收到一个响应，但是结果可能不是最新的。
* Partition tolerance
	分区容错，即系统产生分区后依然能够运行。分区简单讲就是节点之间不能通信，包括网络故障，宕机等。

你一定和我一样有这样的疑惑，为什么 CAP 不能同时满足呢？ 
前边提到， Seth Gilbert 和 Nancy Lynch 对该假设进行了证明，证明过程参考[https://www.glassbeam.com/sites/all/themes/glassbeam/images/blog/10.1.1.67.6951.pdf](https://www.glassbeam.com/sites/all/themes/glassbeam/images/blog/10.1.1.67.6951.pdf) 。
这里有一片文章对论文加上了一些图片，看起来更直观 [https://mwhittaker.github.io/blog/an_illustrated_proof_of_the_cap_theorem/](https://mwhittaker.github.io/blog/an_illustrated_proof_of_the_cap_theorem/) 。
基于 CAP 我们可以“简单”地将系统分为三类：
* CP
* AP
* CA

但是，大多数人认为在现实中分区是不可避免的，原因也显而易见，保证一个节点永远可用几乎是不可能，所以只能在 CP 和 AP 中选择。如果按照这样的思路，既然故障不可避免，那么当故障发生时，我们该如何选择？
* 如果系统选择在存在分区时“一致性”的权重更高，则通过拒绝响应某些请求，以保证其原子读取和写入。比如完全关闭或拒绝写入又或者只响应对“主”节点位于分区内的数据片段进行读写。只要满足业务的需要，这些都是合理的，而且这也会大大降低系统的复杂度。
* 如果系统选择在存在分区时“可用性”的权重更高，则将响应所有请求，可能会返回过时的读取并接受冲突的写入。这些不一致通过其他技术手段来解决解决，这也是合理的。

这让我们又一次陷入疑惑，到底是 CP 还是 AP ？ 先别急，我们在做出选择之前，先想一想，是不是所有的分布式系统都可以严格的分为 CP 和 AP 呢？如果答案是肯定的，那么我们可以从系统分类中找到每种分类的系统共性，从而找到规律来给下一个系统贴上 CP 或 AP 的标签。 如果答案是否定的，那么就意味着强行将系统分类成 AP 或 CP 是不合适的。 
我们先找几个系统来试着进行分类。
* master/slave 系统 
	主从结构的是很多分布式系统采用的方式。通过 master 将其他节点联系起来，如果客户端与 master 分区，则无法写入。即便可以从 slave 节点读取，但是不能写入决定了单一 master 的系统不是 CP 的。
* 关系数据库
	数据库为了提供高并发，就会放弃线性化，如果不能保证线性化，那么系统自然就不是 CP 的。
* NoSQL数据库
	大多数 NoSQL 数据的卖点都是高可用，但是如果数据的副本少于约定数量，在恢复副本到正常数量之前，是不满足 AP 的。
* 共识系统
	共识系统的主要特点就是保证一致性，如 zookeeper 。这样看 zookeeper 是一个 CP 的系统，但是 zookeeper 默认是不提供线性化读的，即在读取 zookeeper 的一个节点时，即使其他节点的数据更新，但是依旧只能读取本地节点的旧数据。为了提供更好的性能，zookeeper 并不是任何时候都执行同步的，所以在同步之前，它不是 CP 的。

好了，上面我们列举了一些系统，他们既不符合 ```CAP-一致```也不符合 ```CAP-可用```，它们只是满足 P 。再回头来看刚才提出的问题，好像到底是 CP 还是 AP ，并没有一个明确的标准来严格区分。如果这样，我们无法将系统明确的区分为 CP 或 AP ，那我们研究 CAP 的意义是什么呢？ CAP 为我们提供了一个实际问题的一个简化模型，通过这个模型，让我们对实际问题进行思考和讨论，了解我们在分区时将面对什么。 CAP 的启发意义远大于其定理本身。
