---
title: Selection Sort
tags:
  - Interview
category:
  - Data Structure and Algorithm
author: bsyonline
lede: 没有摘要
date: 2020-02-08 11:39:46
thumbnail:
---

n 个数需要循环 n-1 次
默认选择第一个数，假定其最小，从下一个数开始依次向后比较，如果有更小的数，记录下值和索引
如果第一个数和选定的数不是同一个数，将第一个位置的数与选定的数交换
时间复杂度为 O(n^2)