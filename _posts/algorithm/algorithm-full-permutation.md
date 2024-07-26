---
title: 全排列
date: 2014-10-16 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

```
public void perm(int[] arr,int start,int end){
     if(start==end){
          for(int i=0;i<arr.length;i++){
               System.out.print(arr[i]);
          }
          System.out.println();
     }else{
          for(int i=start;i<=end;i++){
               swap(arr,start,i);
               perm(arr,start+1;end);
               swap(arr,start,i);
          }
     }
}
```
