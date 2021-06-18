---
title: Bubble Sort
toc: false
date: 2014-04-26 15:53:17
tags:
 - Interview
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


冒泡排序（Bubble Sort）是一种比较简单的排序。它的原理是依次比较相邻的两个元素，如果他们的顺序相反，就交换位置，依次类推，直到最后一个元素。这是一轮比较，经过一轮比较之后，最后一个元素为最大。如果有 n 个数，需要循环 n-1 轮，每次循环中只需要比较未排序部分，即 n-1-i 。

<img src="https://s2.ax1x.com/2020/02/11/1TMcpF.png" alt="1TMcpF.png" border="0" />

正常情况下，需要循环 n-1 轮，但是如果在一次循环中没有比较，那么说明已是有序，后边再继续循环也没有意思，直接退出即可。

冒泡排序的时间复杂度为 O(n^2)。

```
public class BubbleSortExample {
    public static void main(String[] args) {
        int[] arr = {6, 2, 4, 7, 1};
        bubbleSort(arr);
        System.out.println(Arrays.toString(arr));
    }

    /**
     * 1. n 个数只需要循环 n-1 次
     * 2. 每次循环中只需要比较为排序部分，即 n-1-i
     * 3. 如果在一次循环中没有比较，则已是有序
     * 4. 时间复杂度为 O(n^2)
     *
     * @param arr
     */
    public static void bubbleSort(int[] arr) {
        boolean flag = false;
        for (int i = 0; i < arr.length - 1; i++) {
            for (int j = 0; j < arr.length - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    int tmp = arr[j + 1];
                    arr[j + 1] = arr[j];
                    arr[j] = tmp;
                    flag = true;
                }
                System.out.println("第" + (i + 1) + "轮-第" + (j + 1) + "次：" + Arrays.toString(arr));
            }
            if (!flag) {
                break;
            } else {
                flag = false;
            }
        }
    }

}

```