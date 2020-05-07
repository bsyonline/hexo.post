---
title: Heapsort
tags:
  - Interview
category:
  - Data Structure and Algorithm
author: bsyonline
lede: 没有摘要
date: 2020-02-08 14:10:42
thumbnail:
---

堆排序（heapsort） 用到了一种数据结构叫做**堆**。堆由一颗**完全二叉树**构成，且完全二叉树的每一个父节点都大于或小于叶子节点。如果父节点大于叶子节点，就叫做最大堆，反之就叫做最小堆。排序按照升序或降序需要对应用到最大堆或最小堆。
>完全二叉树具有一个特点，就是树的生成顺序是自上至下，从左到右。

由于完全二叉树的特点，元素是连续，所以完全二叉树可以用数组来表示。

<img src="https://s2.ax1x.com/2020/02/10/152KRe.png" alt="152KRe.png" border="0" style="width:600px" />

由上图可以看到，完全二叉树种的任意一个节点都可以通过计算得到：如果 i 是索引位置，那么 parent = (i - 1) / 2 , c1 = i * 2 + 1， c2 = i * 2 + 2 。

如果我们需要利用堆来对数组进行排序，那么就需要用数组构造出一个最大堆或最小堆。而我们当前的数组并不是一个最大堆，所以首先需要将完全二叉树调整成堆，这个步骤叫做 heapify ，其过程如下：
从 root 节点开始，找到 parent ，c1，c2 三个元素，parent 和 c1 比较，如果 parent 小于 c1 ，则将 parent 和 c1 互换。将所有的子树都按照这个过程再做一次，直到满足堆的特性。
有了这个基础，我们就可以利用堆来进行数组排序了。步骤为：
1. 从树的 h-1 层的最左节点开始做 heapify 直到构造一个堆。
2. 然后将 root 和最后一个节点，也就是数组的第一个元素和最后一个元素交换，然后将堆的最后一个元素删除。
3. 由于交换破坏了堆，所以再重复1，2步，依次类推，最终就得到了一个有序的数组。


<img src="https://s2.ax1x.com/2020/02/10/152MxH.png" alt="152MxH.png" border="0" />
<img src="https://s2.ax1x.com/2020/02/10/152GZt.png" alt="152GZt.png" border="0" />
<img src="https://s2.ax1x.com/2020/02/10/152YIf.png" alt="152YIf.png" border="0" />
<img src="https://s2.ax1x.com/2020/02/10/15W3UP.png" alt="20200210180057" border="0">
<img src="https://s2.ax1x.com/2020/02/10/152aRg.png" alt="152aRg.png" border="0" />