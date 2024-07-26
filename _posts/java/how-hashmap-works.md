---
title: HashMap 的工作原理
date: 2016-08-09 11:37:57
tags:
 - Interview
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

HashMap 是最常用的集合类之一，在面试中也出镜率颇高。

#### HashMap 和 Hashtable
经常会问 **HashMap 的特点** 及 **HashMap 和 Hashtable 的区别** 等等，那么就先做一下简单总结。

<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">HashMap</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">Hashtable</th></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">所在位置</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">实现 Map 接口，JDK 1.2 加入到 Java Collections Framework</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">Hashtable 集成自 Dictionary ，JDK 1.2 实现了 Map 接口</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">是否支持 null key 或 null value</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">是</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">否</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">线程安全</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">不安全</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">安全</td></tr></table>

以上是 3 点是我们耳熟能详的二者之间的区别，Hashtable 不是我们此篇的重点，暂且放在一旁。
#### HashMap 的数据结构
上边这个面试题是通常只是开胃菜，每个熟悉 Java 的人都能知晓。上题热身之后，经常会有这样的问题提出： **HashMap 是如何存储数据的?** Java8 中是通过数组链表和红黑树来实现的。
<img src="https://s2.ax1x.com/2020/02/29/3sqfXT.png" alt="3sqfXT.png" border="0" style="width:600px"/>
Java8 中，当我们向 HashMap 中 put 元素时，会进行一下几步操作：
1. 首先判断数组是否初始化，如果没有初始化，会首先调用 resize() 对 Node<K,V>[] 数组进行初始化。
2. 然后通过 hash(key) 定位到数组元素 Node<K,V>[i] ，如果 Node<K,V>[i] 为空，就创建一个新的 Node<K,V> 放到 Node<K,V>[i] 。
3. 如果 Node<K,V>[i] 不为空，则判断 Node<K,V>[i] 的类型是否是 HashMap.TreeNode ，如果是就按照树的方式 put 新值。
4. 如果 Node<K,V>[i] 的类型不是 HashMap.TreeNode ，就按照链表方式进行遍历，将新值加到链表末尾。
5. 添加完成，再判断链表中元素的数量是否超过 treeify 的阈值，如果超过则进行 treeifyBin 操作。
6. 最后再判断一下 HashMap 中元素的数量是否超过负载值，如果超过则再进行一次 resize() 。

<img src="https://s2.ax1x.com/2020/02/29/3sxupd.png" alt="3sxupd.png" border="0" />

和之前的版本不同，为了避免单个 Node<K,V> 上的节点过多，导致遍历时间复杂度高，Java8 中对链表的数量超过阈值（默认时 8 ）时，会将链表转成红黑树，这个过程叫做 treeify() ，其流程如下。

<img src="https://s2.ax1x.com/2020/02/29/3sza5D.png" alt="3sza5D.png" border="0" style="width: 400px"/>

>treeify 操作并不一定会将链表转成树，会根据 HashMap 中元素的数量是否超过 treeify 的最小阈值来决定。



#### resizing
什么是 resizing ？ 我们知道 HashMap 中数组的元素在 Node<K,V>[] 中的位置是通过 hash(key) 来确定的，如果多个元素的 hash(key) 相同，它们以链表或树的形式的会被存储在相同的位置，这个过程叫做**哈希碰撞**。如果 HashMap 中元素分布足够离散，那么不会出现哈希碰撞，时间复杂度为 O(1) 。如果元素不是离散分布，那么会频繁出现碰撞，数据在链表中存储，时间复杂度为 O(n) 。随着元素增多，哈希碰撞的几率会增加，为了减少碰撞的几率，当达到一定的阈值，HashMap 对 Node<K,V>[] 进行扩容，这个过程叫做 resizing 。Java8 中的 resizing 过程如下。

<img src="https://s2.ax1x.com/2020/02/29/3yNs4P.png" alt="3yNs4P.png" border="0" />

将旧的数组扩容成新的数组，旧数组中的元素需要重新放置到新数组中，这个过程不会再重新计算 hash(key) ，而是先遍历链表算出 hiTail 和 loTail ，然后根据 hiTail 和 loTail 得到元素在新数组中的位置。说简单点，由于扩容是向左移一位，那么不用每次都重新计算 hash(key) ，只要看左边新扩展的一位是 0 还是 1 , 0 就留在原位置，1 就将原位置索引加上原容量得到新位置的索引。计算结果参考下图：
<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20181014/hashmap_04.png" style="width: 850px">


>Java8 和之前的版本在实现上有很大的不同。Java8 之前，后加入的元素在链表的头部。Java8 是最先加入的元素在链表头部。

**为什么使用 String 作为 key 是一个不错选择？**
其实，答案可以从上边获得。因为，String 是 final 的，并且有固定的 hashCode() 和 equals() 方法，所以能有效减少碰撞的发生，同时由于不可变，可以缓存 key 的 hashcode ，提高 get 对象的速度。同理，如果自定义对象作为 key ，应保证对象是不可变的，即保证 equals() 和 hashCode() 方法正确重写。

**多线程环境下 resize 导致 CPU 100%**
这个是 Java8 之前多线程环境下 HashMap 会出现一个问题，导致问题原因是在链表中后加入的元素会在链表的头部，在扩容通过遍历将旧数组中的元素移动到新数组中，在多线程中有可能出现环形链表，get 的时候导致死循环，最终导致 CPU 100% 。Java8 的 resize 不再使用这种方式，也就不会出现这个问题了。

**初始化容量的设计**
HashMap 的初始容量为 16 ，容量超过 75% 就会 resizing 。虽然 HashMap 的 resizing 性能在不断提升，但是如果能预估 HashMap 的大小，就能够避免不必要的 resizing 。比如，有 1000 个元素， 下面的写法必定会触发 resizing 。
```java
Map map = new HashMap();
```
或
```java
Map map = new HashMap(1000);
```
不考虑空间因素，2 倍 size 是最简单的方法。
```java
Map map = new HashMap(1000 * 2);
```
但这样并不是太好，因为 HashMap 本身就不省空间。所以靠谱的做法还是自己算一个 init size 。
```java
float size = 1000 / 0.75f;
Map map = new HashMap(size);
```
或
```java
Map map = Maps.newHashMapWithExpectedSize(1000);
```

**Key 的设计**
我们知道 Map 获取元素是通过 Key 来比较的，Integer/String 这些常见的类型都可以作为 key 。这些类都有一个共同的特点，即重写了 hashCode() 和 equals() 方法，保证了 key 的离散，并且这些类是 final 的，保证不会有子类重写这些方法。如果需要使用自定义对象类型的 key ，最好还是要重写 hashcode 和 equals 方法来保证元素分布是离散的。

**几种同步的 HashMap 的区别**
Hashtable 对方法加 synchronized 。
Collections.synchronizedMap(new HashMap<>()); 使用 synchronized 代码块。
ConcurrentHashMap 利用分段锁，java8 之前是先根据 hash 定位到 segment 然后在 segment 内部使用 lock 加锁。Java8 是对 Node<K,V>[] 的元素（链表的第一个元素）用 CAS 加锁，在链表内部使用 synchronized 进行同步。