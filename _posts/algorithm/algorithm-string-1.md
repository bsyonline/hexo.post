---
title: 字符串处理1
date: 2015-04-09 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

一个5位数，判断它是不是回文数。即12321是回文数，个位与万位相同，十位与千位相同。
```
public class Prog0409 {

    public static void main(String[] args) {
          Prog0409 prog = new Prog0409();
          System.out.println(prog.print(12331));
    }

     public boolean print(int n){
          String s = String.valueOf(n);
          if(s.length() != 5){
               return false;
          }else{
               if((s.charAt(0)==s.charAt(4)) && (s.charAt(1) == s.charAt(3))){
                    return true;
               }else{
                    return false;
               }
          }
     }

}
```
