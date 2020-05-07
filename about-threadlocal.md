---
title: About Threadlocal
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2019-11-25 08:58:49
thumbnail:
---

ThreadLocal 是一个特殊的数据存储类，它为每个线程都提供了一份变量的副本，每个线程只能够获取到自己对应的存储数据，从而实现变量被多个线程共享，数据又彼此隔离。

#### ThreadLocal 的使用场景

ThreadLocal 在什么场景下使用呢？

1. **多个线程之间作用域相同并且需要不同的数据副本。**比如我们在使用 redis 实现分布式锁时，一个线程获取锁，处理完成后要释放锁，并且要保证自己的锁只能由自己来释放，我们需要给每一把锁生成一个唯一 ID ，这个 ID 由线程自己持有，在释放锁的时候通过 ID 来进行操作，此时可以使用 ThreadLocal 来保存 ID 。

2. **对象传递。**通常我们在方法调用的时候，如果逻辑复杂，可能入参会有多个或者无法获取到某些参数的情况，这时候就可以使用 ThreadLocal 来进行对象传递。典型的如 Spring AOP ，由于业务代码是定义好的，业务代码入参中并不强制包含切面处理的对象，所以 spring 将这些切面对象放在 ThreadLocal 中，这样就可以在线程内部获取到切面所需要的参数信息了。

#### ThreadLocal 的实现原理

通过查看 ThreadLocal 的源代码，我们可以看到在 ThreadLocal 内部有一个 ThreadLocalMap ，这个 ThreadLocalMap 就是 Thread 类中用来存放线程自己的数据的。我们主要关注 ThreadLocal 的这几个方法：

1. set 

   ```
   public void set(T value) {
   	Thread t = Thread.currentThread();
   	ThreadLocalMap map = getMap(t);
   	if (map != null)
   		map.set(this, value);
   	else
   		createMap(t, value);
   }
   ```

   获取到当前线程的 ThreadLocalMap ，如果 map 不为空，就把对象 set 进去，如果 map 为空，就创建一个新的 ThreadLocalMap ，再把对象 set 进去。

2. get

   ```
   public T get() {
   	Thread t = Thread.currentThread();
   	ThreadLocalMap map = getMap(t);
   	if (map != null) {
   		ThreadLocalMap.Entry e = map.getEntry(this);
   		if (e != null) {
   			@SuppressWarnings("unchecked")
   			T result = (T)e.value;
   			return result;
   		}
   	}
   	return setInitialValue();
   }
   ```

   get 操作也是一样，先获取当前线程的 ThreadLocalMap ，如果 map 不为空，则从 Map.Entry 中获取对象。如果 map 为空，则返回初始值 null ，并把 null 值 set 到当前线程的 ThreadLocalMap 中。

3. remove

   ```
   public void remove() {
       ThreadLocalMap m = getMap(Thread.currentThread());
       if (m != null)
           m.remove(this);
   }
   ```

   remove 就更简单了，获取当前线程的 ThreadLocalMap ，然后从 Map 中删除线程的数据。

了解 ThreadLocal 的基本操作之后，我们还要再来看看 ThreadLocalMap 。ThreadLocal 提供的功能，本质上是由 ThreadLocalMap 实现的。刚才源码我们也看到了，在往 ThreadLocal 中 set 对象的时候，实际调用的是 ThreadLocalMap 的 set 方法。

```
// 遍历map
for (Entry e = tab[i];
	e != null;
	e = tab[i = nextIndex(i, len)]) {
	ThreadLocal<?> k = e.get();
    // 如果Entry的key相同，则替换
	if (k == key) {
		e.value = value;
		return;
	}
	// 如果Entry的key是null，则set新值
	if (k == null) {
		replaceStaleEntry(key, value, i);
		return;
	}
}
// 如果没有相同的对象，则生成新的Entry添加到map中
tab[i] = new Entry(key, value);
// map的大小加1
int sz = ++size;
if (!cleanSomeSlots(i, sz) && sz >= threshold)
	// 如果清除map中Entry的key为null的Entry之后，map的size
	// 还大于阈值，则进行rehash
	rehash();
```

```
private void rehash() {   
	// 清除过时的元素(Entry的key为空)
	expungeStaleEntries();
	// 如果清除之后，size 还是超过阈值，则扩容为原来的 2 倍
    if (size >= threshold - threshold / 4)
    	resize();
}
```