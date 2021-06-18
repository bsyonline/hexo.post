---
title: CAS
tags:
  - Interview
category:
  - Concurrent
author: bsyonline
lede: 没有摘要
date: 2020-03-02 08:35:53
thumbnail:
---

在 Java 并发编程中，为了保证并发安全，一种方式是使用锁，另一种方式是使用 CAS（compare and swap） 。CAS 有 3 个参数，内存地址 V ，旧的预期值 A ，要修改的新值 B ，更新一个变量的时候，只有当变量的预期值 A 和内存地址 V 当中的实际值相同时，才会将内存地址 V 对应的值修改为 B ，如果和预期不相等则更新失败。
#### CAS 是如何实现的？
Unsafe 类中有 3 个 cas 方法：
```
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```
它们都是本地方法，在 x86 架构的系统中，是 Unsafe 类通过来调用 **cmpxchg** 指令来完成 CAS 操作的。

>compareAndSwapInt() 方法 4 个参数含义：
Object var1 需要操作的对象。
long var2	需要操作的属性的内存偏移量。
int var4	期望值。
int var5	更新值。

#### 如何使用 Unsafe
Unsafe 是 sun.misc 包下的类，如果我们使用 Unsafe.getUnsafe() 来创建实例，则会报 java.lang.SecurityException: Unsafe 。原因是因为 Unsafe 类是 Java 的底层类，必须由 bootstrap classloader 加载，我们的应用是通过 application classloader 加载的，所以会报异常。
```
@CallerSensitive
public static Unsafe getUnsafe() {
	Class var0 = Reflection.getCallerClass();
	if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
		throw new SecurityException("Unsafe");
	} else {
		return theUnsafe;
	}
}
```
既然不能直接使用，那就只能换一种方式，借鉴 java.util.concurrent.atomic.AtomicInteger ，利用反射来获取 Unsafe 。
```
public class UnsafeExample {

    private static Unsafe unsafe;
    private static long valueOffset;
    private volatile int value;

    static {
        try {
            Field theUnsafeField = Unsafe.class.getDeclaredField("theUnsafe");
            theUnsafeField.setAccessible(true);
            unsafe = (Unsafe) theUnsafeField.get(null);
            valueOffset = unsafe.objectFieldOffset(UnsafeExample.class.getDeclaredField("value"));
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        //Unsafe unsafe = Unsafe.getUnsafe();
        System.out.println(unsafe);
        System.out.println(valueOffset);
    }
}
```
#### CAS 的优点与问题
和 Java 的锁相比，CAS 不会阻塞线程，也不会有线程间通信带来的线程切换和唤醒的消耗，在性能上会好于 Java 的锁。但是 CAS 也有一些问题：
1. 线程自旋造成的 CPU 占用高。
2. ABA 问题。
3. 只能保证单个属性的线程安全，多个属性的线程安全无法使用 CAS 。

#### ABA 问题
CAS 没有加锁，而是通过比较内存地址的值是否和预期相等来判断是否能更新成功。举个例子，比如一个栈中有 ABC 三个元素，栈顶元素是 C ，如果进行 2 次出栈操作，在将 C 入栈，此时栈顶元素还是 C ，但是栈中只有 A 和 C 了。为了避免这种情况，可以使用 ```AtomicStampedReference<V>``` 或 ```AtomicMarkableReference<V>``` 通过每次操作都添加一个标签来，虽然值相同但是不同的操作标签已经改变，从而避免 ABA 问题。
>```AtomicStampedReference<V>``` 和 ```AtomicMarkableReference<V>``` 内部都维护了一个 pair ，AtomicStampedReference 中的 pair 是 (reference, int) ，AtomicMarkableReference 中的 pair 是 (reference, boolean) 。AtomicMarkableReference 只是标识是否有修改。