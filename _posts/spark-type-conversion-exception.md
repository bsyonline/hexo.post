---
title: Spark 类型转换异常的处理
date: 2017-02-03 10:16:52
tags:
 - Spark
category: 
 - Spark
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

在调试 Spark 程序的时候报了一个类型转换异常，信息如下：
```
java.lang.ClassCastException: scala.Tuple2 cannot be cast to java.lang.Comparable
```

<!-- more -->
查看代码，发现是在调用 top() 方法时出现的，如果是调用 collect() 方法不会出现异常。


分析原因为在 top 时要对 RDD 的内容进行排序，但是 RDD 中存放的是 tuple ，并没有实现 Comparator 接口。
所以可以自己实现一个 tuple 的比较器。
```
public class Tuple2Comparator implements Serializable, Comparator {

    @Override
    public int compare(Object o1, Object o2) {
        Tuple2 o11 = (Tuple2) o1;
        Tuple2 o12 = (Tuple2) o2;
        if (o11._1.hashCode() > o12._1.hashCode()) {
            return 1;
        } else {
            return -1;
        }
    }
}
```

在使用 top 的时候，指定自定义的比较器。
```
rdd.top(5, new Tuple2Comparator());
```

因为我只用到了 Tuple2 ，所以只定义了 Tuple2 的比较器，等对 scala 熟悉一些，再改进成通用所有 tuple 的。
