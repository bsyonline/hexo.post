---
title: 朴素贝叶斯算法
date: 2017-05-18 15:27:15
tags:
 - Algorithm
 - Machine Learning
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

朴素贝叶斯 (Native Bayes) 是一个简单的多类分类算法，它假定每一对特征值都是彼此独立的。朴素贝叶斯算法具有很好的性能。数据训练能计算出每一个给定标签的特征可能的条件分布并可以利用它来观察特征的分布并用于预测。<!--more-->
Spark MLlib 支持多项朴素贝叶斯（multinomial naive Bayes）和伯努利朴素贝叶斯（Bernoulli naive Bayes），他们都是典型的文本分类算法。
每条观察数据都是由特征和指标构成的文本。特征表示一项在文本中出现的频率，特征的值必须是非负。指标 0 和 1 表示一项是否在文本中能找到。

MLlib 的实现的朴素贝叶斯算法接收三个参数： LabeledPoint 的 RDD ，lambda 是 The smoothing parameter ，默认值是 1.0 ， modelType 可以指定使用 multinomial 或是 Bernoulli ，输出是一个可以用于评估和预测的 NaiveBayesModel 。

```scala
def train(input: RDD[LabeledPoint], lambda: Double, modelType: String): NaiveBayesModel
```

LabeledPoint 是一个存放特征和标签的类。

```scala
LabeledPoint(label: Double, features: Vector) 
```
