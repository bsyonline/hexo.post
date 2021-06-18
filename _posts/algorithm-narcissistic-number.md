---
title: 水仙花数
date: 2015-03-25 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


水仙花数是指一个 n 位数 ( n≥3 )，它的每个位上的数字的 n 次幂之和等于它本身。（例如：1^3 + 5^3+ 3^3 = 153）
```
public class Algo0325 {

    public static void main(String[] args) {
        for (int i = 100; i < 1000; i++) {
            int h = i / 100;
            int t = (i - h * 100) / 10;
            int o = (i - h * 100 - t * 10);
            if ((Math.pow(h, 3) + Math.pow(t, 3) + Math.pow(o, 3)) == i) {
                System.out.print(i + ", ");
            }
        }
    }

}
```
