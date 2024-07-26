---
title: 阶乘
toc: false
date: 2015-04-08 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

求1+2!+3!+...+20!的和
```java
public class Prog0408 {

    public static void main(String[] args) {
          Prog0408 prog = new Prog0408();
          System.out.println(prog.print(20));
    }

     public int print(int n){
          int r=0;
          for(int i=1;i<=n;i++){
               r += factorial(i);
          }
          return r;
     }

     public int factorial(int n){
          int r = 1;
          for(int i=1;i<=n;i++){
               r = r * i;
          }
          return r;
     }


}
```
