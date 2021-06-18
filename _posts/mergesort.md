---
title: Mergesort
tags:
  - Interview
category:
  - Data Structure and Algorithm
author: bsyonline
lede: 没有摘要
date: 2020-02-08 11:43:37
thumbnail:
---


归并排序（mergesort）是采用分治法的一种排序算法。它的操作步骤分为两个阶段：拆分和归并。
拆分阶段，先将数组分成左右两段，再将左边再分成左右两段，依此执行直到左右两段只有 1 个元素（1 个元素即认为有序）。右边也执行相同操作。
归并阶段，从左边开始依次合并直到成为一个完成数组。归并具体步骤如下：
1. 假如有两个数组 a1 和 a2 ，先从 a1 和 a2 各取出第一个元素进行比较，a1 的元素较小，则放到临时数组，然后再从 a1 取出下一个元素，继续比较。
2. 如果 a2 元素全都放到临时数组，a1 还有元素，则按顺序加入到临时数组。

<img src="https://s2.ax1x.com/2020/02/11/1Tbq2j.png" alt="1Tbq2j.png" border="0" />

了解了归并的操作之后，将两个阶段合起来就可以完成排序了，n 个数的排序需要合并 n-1 次。

<img src="https://s2.ax1x.com/2020/02/11/1TRCPP.png" alt="1TRCPP.png" border="0" />

归并排序是稳定排序，它的时间复杂度为 O(n * logn)。

```
public class MergesortExample {
    public static void main(String[] args) {
        int[] arr = {6, 2, 4, 7, 1};
        mergesort(arr, 0, arr.length - 1);
        System.out.println(Arrays.toString(arr));
    }

    /**
     * 1. n 个数需要合并 n-1 次
     * 2. 时间复杂度为 O(n * logn)
     *
     * @param arr
     * @param left
     * @param right
     */
    public static void mergesort(int[] arr, int left, int right) {
        if (left == right) {
            return;
        }
        int middle = (left + right) / 2;
        mergesort(arr, left, middle);
        mergesort(arr, middle + 1, right);
        merge(arr, left, middle, right);
    }

    private static void merge(int[] arr, int left, int middle, int right) {
        System.out.printf("left=%d,right=%d\n", left, right);
        int[] tmp = new int[right - left + 1];
        int t = 0;
        int i = left;
        int j = middle + 1;
        //从左右两端开始比较，较小的数放到临时数组
        while (i <= middle && j <= right) {
            tmp[t++] = arr[i] < arr[j] ? arr[i++] : arr[j++];
        }
        //如果左边或右边还有剩余，就依次拷贝到临时数组
        while (i <= middle) {
            tmp[t++] = arr[i++];
        }
        while (j <= right) {
            tmp[t++] = arr[j++];
        }
        //将临时数组拷贝回原数组
        for (int k = 0; k < t; k++) {
            arr[left + k] = tmp[k];
        }
    }
}
```