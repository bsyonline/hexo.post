---
title: 完全平方数
date: 2015-04-02 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Pattern;

public class Algo0402 {

    public static void main(String[] args) {
        Algo0402 algo = new Algo0402();
        List list = algo.fullSquare(1000);
        System.out.println(list);

        System.out.println(algo.isInteger("-2"));

        for(int i=0;i<=1000;i++){
            if(list.contains(i+100)&&list.contains(i+168)){
                System.out.print(i + ",");
            }
        }
    }

    public List<Integer> fullSquare(int n){
        List<Integer> list = new ArrayList<Integer>();
        for(int i=0;i<=Math.sqrt(n);i++) {
            list.add(i*i);
        }
        return list;
    }

    public boolean isInteger(String s){
        Pattern p = Pattern.compile("^[+]?[\\d]*$");
        return p.matcher(s).matches();
    }
}
```
