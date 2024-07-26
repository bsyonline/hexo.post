---
title: Binary Search
date: 2014-04-17 15:53:17
tags:
 - Interview
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

二分查找，又叫折半查找，时间复杂度为 O(logn)

**groovy 版**
```java
def s = [1, 2, 3, 4, 5, 6, 7, 8, 9] as int[]

def binarySearch(int[] a, int b) {
    int low = 0;
    int high = a.length - 1
    while (low < high) {
        int middle = (low + high) / 2 as int
        if (b < a[middle]) {
            high = middle - 1
            continue
        }
        if (b > a[middle]) {
            low = middle + 1
            continue
        }
        if (b == a[middle]) {
            return middle
        }
    }
    return -1
}

println binarySearch(s, 80)
```
**Java 版**

迭代：
```java
public class BinarySearch {
    public int search(int[] arr, int x) {
        if (arr == null || arr.length == 0) {
            return -1;
        }
        int low = 0;
        int high = arr.length - 1;
        while (low < high) {
            int middle = (low + high) / 2;
            if (x > arr[middle]) {
                low = middle + 1;
                continue;
            }
            if (x < arr[middle]) {
                high = middle - 1;
                continue;
            }
            if (x == arr[middle]) {
                return middle;
            }
        }
        return -1;
    }
}
```

递归：

```java
public class BinarySearch {
    public int search(int[] arr, int start, int end, int x) {
        if (arr == null || arr.length == 0) {
            return -1;
        }
        int low = start;
        int high = end;
        int middle = (low + high) / 2;
        while (low <= high) {
            if (x == arr[middle]) {
                return middle;
            }
            if (x < arr[middle]) {
                return search(arr, start, middle - 1, x);
            }
            if (x > arr[middle]) {
                return search(arr, middle + 1, high, x);
            }
        }
        return -1;
    }
}
```
