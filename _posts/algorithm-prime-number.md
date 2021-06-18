---
title: 分解质因数
date: 2015-03-26 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

每个合数都可以写成几个质数相乘的形式。其中每个质数都是这个合数的因数，叫做这个合数的分解质因数。如 90 = 2 * 3 * 5

1.给点数n，求1到n的所有素数。  
2.如果n不是素数，用第一个质数2开始除，能整除则商继续用从2开始除。  
3.如果不能整除，换下一个质数，直到商等于质数。

```
import java.util.ArrayList;
import java.util.List;

public class Algo0326 {

    private boolean flag = false;

    public static void main(String[] args) {
        Algo0326 qf = new Algo0326();
        qf.print(5);

    }

    private void print(int n) {
        List list = prime(n);
        if (list != null && list.size() > 0) {
            System.out.print(n + " = ");
            decom(n, list);
        }
    }

    private void decom(int n, List list) {
        if (!list.contains(n)) {
            for (int i = 0; i < list.size(); i++) {
                if (!flag) {
                    int p = (Integer) list.get(i);
                    if (n % p == 0) {
                        System.out.print(p + " * ");
                        decom(n / p, list);
                    }
                } else {
                    break;
                }
            }
        } else {
            System.out.print(n);
            flag = true;
        }
    }

    public List prime(int n) {
        List list = new ArrayList();
        for (int i = 2; i <= n; i++) {
            int j;
            for (j = 2; j <= Math.sqrt(i); j++) {
                if (i % j == 0) {
                    break;
                }
            }
            if (j > Math.sqrt(i)) {
                list.add(i);
            }
        }
        return list;
    }

}
```
