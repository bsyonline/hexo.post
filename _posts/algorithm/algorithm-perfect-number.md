---
title: 完数
date: 2015-03-28 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

完数，所有的真因子（即除了自身以外的约数）的和（即因子函数），恰好等于它本身

```
import java.util.ArrayList;
import java.util.List;

public class Algo0328 {

    public static void main(String[] args) {
        Algo0328 p = new Algo0328();
        System.out.println(p.perfectNumber(1000));
    }

    public List<Integer> perfectNumber(int n){
        List<Integer> list = new ArrayList<Integer>();
        for(int i=1;i<n;i++){
            if(i == sum(i)){
                list.add(i);
            }
        }
        return list;
    }

    public int sum(int n){
        int sum = 0;
        for(int i=1;i<n;i++){
            if(n % i == 0){
                sum += i;
            }
        }
        return sum;
    }
}
```
