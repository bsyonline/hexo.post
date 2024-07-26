---
title: Bloom Filter
toc: false
date: 2015-11-24 15:45:57
tags:
 - Data Structures
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

bloom filter是一种节省空间的数据结构。

内在表现为m个比特位的数组。

优势在于大小固定且在初始化时被设置。

一个典型的bloom filter包含add和contrans操作，和set类似。

	class BloomFilter<E> {
	     public BloomFilter(int m, int k) { ... }
	     public void add(E obj) { ... }
	     public boolean contains(E obj) { ... }
	}

初始状态时，Bloom Filter是一个包含m位的位数组，每一位都置为0。

![1](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/2.png)

对于任意元素x，bloom filter将其使用hash函数计算k个索引，存放在数组中，每个位置的值为1。

![2](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/3.png)

判断一个元素y是否属于集合时，对y进行k次hash函数计算，如果所有索引位置在数组中的值都为1，则y属于这个集合。如果有任意个索引位置的值为0，则y不属于这个集合。

![1](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/4.png)

增加元素到bloom filter中不会改变其大小，只会改变误报率。

误报率的近似值为0.7*(m/n)
