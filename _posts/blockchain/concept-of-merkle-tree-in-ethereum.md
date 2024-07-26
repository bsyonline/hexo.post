---
title: concept of merkle tree in ethereum
tags:
  - Blockchain
  - Ethereum
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-16 10:15:05
thumbnail:
---


Merkle Tree 是一种在密码学和计算机科学中常见的数据结构，它是一种 ```Hash Tree``` ，即所有的节点都是 Hash 值，叶子节点是数据块的 hash 值，非叶子节点是其子节点的 hash 值。

#### **Hash Tree 和 Hash List 的区别**
```Hash Tree``` 可以高效地对大型的数据结构的内容进行安全验证。```Hash List``` 可以看做是一种特殊的 ```Hash Tree``` ，高度为 2 的多叉 ```Hash Tree``` 。```Hash List``` 的结构简单，但是在校验某节点是否是 list 中的一部分时需要的计算所有节点的 Hash 值。而 ```Hash Tree``` 在校验某一节点是否是二进制哈希树的一部分时所需要的节点数量是 ```log(所有节点树)``` 。

#### **Merkle Tree 的应用**
下面我们用一个简化的图来说明 Merkle Tree 的结构。下图中 ```T``` 代表数据，如交易数据，```H``` 代表哈希。
![https://i.investopedia.com/content/term/merkle_tree/merkle_tree_aa.jpg](https://i.investopedia.com/content/term/merkle_tree/merkle_tree_aa.jpg)
通过两个场景来看如和应用。
1. 如果我们想验证交易 ```TD``` 是否是树的节点。
* 首先，我们可以用 ```HC``` 和 ```HD``` 计算出 ```HCD``` 。
* 第二步，用 ```HAB``` 和 ```HCD``` 计算出 ```HABCD``` 。
* 第三步，用 ```HABCD``` 和 ```HEFGH``` 计算出 ```HABCDEFGH``` ，如果相同，则是正确的数据。
2. 如果我们获取了整个树，但是发现有数据损坏或可能被篡改，那么我们如何找到异常的数据节点呢。
* 首先，比对 root 的值 ```HABCDEFGH```，如果不一样则比较子节点。
* 第二步，比较 ```HABCDEFGH``` 的子节点 ```HABCD``` 和 ```HEFGH``` 。
* 第三步，发现 ```HABCD``` 的值不同，则比较 ```HABCD``` 的子节点 ```HAB``` 和 ```HCD``` 。
* 第四步，发现 ```HCD``` 不同，则比较 ```HCD``` 的子节点 ```HC``` 和 ```HD``` ，最终定位到 ```HD``` 。

![https://i.investopedia.com/content/term/merkle_tree/merkle_tree_ab.jpg](https://i.investopedia.com/content/term/merkle_tree/merkle_tree_ab.jpg)

#### **Merkle Tree 在区块链中的应用**

 