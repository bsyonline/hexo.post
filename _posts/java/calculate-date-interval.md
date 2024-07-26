---
title: 计算时间间隔
date: 2016-07-29 11:58:13
tags:
 - Java
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

在计算日期或是计算程序执行时间时，经常会需要计算两个时间或日期的间隔，通常可以通过一下几种方法实现。

### 1. 使用 Java 日期 api

先计算出相差的毫秒值，再转化成天，小时，分，秒。
按时间进制自己算，代码略。

### 2. 使用 Joda 库

```java
DateTime dt1 = new DateTime(d1);
DateTime dt2 = new DateTime(d2);

int days = Days.daysBetween(dt1, dt2).getDays();
int hours = Hours.hoursBetween(dt1, dt2).getHours() % 24;
int minutes = Minutes.minutesBetween(dt1, dt2).getMinutes() % 60;
int seconds = Seconds.secondsBetween(dt1, dt2).getSeconds() % 60;
```
或
计算出两个时间的毫秒数

```java
Interval interval = new Interval(start, end);
Period p = interval.toPeriod();
int days = p.getDays();
int hours = p.getHours();
int minutes = p.getMinutes();
int seconds = p.getSeconds();
```

### 3. 使用 TimeUnit
计算出两个时间的毫秒数，使用 java.util.concurrent 包下的 TimeUint 。
```java
long diff = t2 - t1;
long days = TimeUnit.MILLISECONDS.toDays(diff);
long hours = TimeUnit.MILLISECONDS.toHours(diff) % 24;
long minutes = TimeUnit.MILLISECONDS.toMinutes(diff) % 60;
long seconds = TimeUnit.MILLISECONDS.toSeconds(diff) % 60;
```
