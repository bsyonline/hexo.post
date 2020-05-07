---
title: 斐波那契数列
toc: false
date: 2015-03-23 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
有一对兔子，从出生后第3个月起每个月都生一对兔子，小兔子长到第三个月后每个月又生一对兔子，假如兔子都不死，问每个月的兔子总数为多少？

|  月份   |  1   |  2   |  3   |  4   |  5   |  6   |  7   |  8   |  9   |  10  |  11  |  12  |
| :---: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 小兔子对数 |  1   |  0   |  1   |  1   |  2   |  3   |  5   |  8   |  13  |  21  |  34  |  55  |
| 中兔子对数 |  0   |  1   |  0   |  1   |  1   |  2   |  3   |  5   |  8   |  13  |  21  |  34  |
| 大兔子对数 |  0   |  0   |  1   |  1   |  2   |  3   |  5   |  8   |  13  |  21  |  34  |  55  |
|  总对数  |  1   |  1   |  2   |  3   |  5   |  8   |  13  |  21  |  34  |  55  |  89  | 144  |

```
public class Algo0323 {

    public static void main(String[] args) {
        Algo0323 f = new Algo0323();
        for (int i = 0; i < 20; i++) {
            System.out.print(f.fibonacci(i) + ",");
        }
    }

    public int fibonacci(int month) {
        if (month == 0) {
            return 0;
        } else if (month == 1 || month == 2) {
            return 1;
        } else {
            return fibonacci(month - 1) + fibonacci(month - 2);
        }
    }
}
```
