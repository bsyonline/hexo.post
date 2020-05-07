---
title: 素数
date: 2015-03-24 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

素数：除了1和它本身以外不再有其他的因数。

```
public class Algo0324 {

    public static void main(String[] args) {
        Algo0324 p = new Algo0324();
        long l1 = System.currentTimeMillis();
        p.print(10, 200);
        long l2 = System.currentTimeMillis();

        System.out.println("");
        long l3 = System.currentTimeMillis();
        p.print1(10, 200);
        long l4 = System.currentTimeMillis();
        System.out.println("");
        System.out.println("print用时" + (l2 - l1) + "s");
        System.out.println("print1用时" + (l4 - l3) + "s");
    }

    public void print(int start, int end) {
        int[] arr = new int[end - start + 1];
        int count = 0;
        for (int i = start; i <= end; i++) {
            arr[count] = i;
            count++;
        }
        for (int i = arr[0]; i <= arr[arr.length - 1]; i++) {
            int j;
            for (j = 2; j <= Math.sqrt(i); j++) {
                if (i % j == 0) {
                    break;
                }
            }
            if (j > Math.sqrt(i)) {
                System.out.print(i + ",");
            }
        }
    }

    public void print1(int start, int end) {
        int[] arr = new int[(end - start + 1) / 2];
        int begin = 0;
        if (start % 2 == 0) {
            begin = start + 1;
        } else {
            begin = start;
        }
        for (int i = 0; i < arr.length; i++) {
            arr[i] = begin + i * 2;
        }


        for (int i = 0; i <= arr.length - 1; i++) {
            int j;
            if (arr[i] > 0) {
                for (j = 2; j <= Math.sqrt(arr[i]); j++) {
                    if (arr[i] % j == 0) {
                        setValue(arr, i);
                        break;
                    }
                }
                if (j > Math.sqrt(arr[i])) {
                    if (arr[i] > 0) {
                        System.out.print(arr[i] + ",");
                    }
                }
            }
        }
    }

    private void setValue(int[] arr, int currentIndex) {
        int currentValue = arr[currentIndex];
        int count = 2;
        while ((currentIndex + count * currentValue) < arr.length) {
            arr[currentIndex + count * currentValue] = 0;
            count = count + 2;
        }
        arr[currentIndex] = 0;
    }

}
```
