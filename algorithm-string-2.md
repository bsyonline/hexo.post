---
title: 字符串2
date: 2015-03-30 15:53:17
tags:
 - Algorithm
category: 
 - Data Structure and Algorithm
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

输入一行字符，分别统计出其中英文字母、空格、数字和其它字符的个数
```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Prog0330 {
    public static void main(String[] args) throws IOException {
        Prog0330 t = new Prog0330();
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String s = br.readLine();
        t.wordCount(s);
    }

    public void wordCount(String s){
        int wordCount=s.length();
        int digitCount = 0;
        int letterCount = 0;
        int spaceCount = 0;
        int otherCount = 0;
        for(int i=0;i<wordCount;i++){
            char c = s.charAt(i);
            if(Character.isDigit(c)){
                digitCount += 1;
            }else if(Character.isLetter(c)){
                letterCount += 1;
            }else if(Character.isSpaceChar(c)){
                spaceCount += 1;
            }else{
                otherCount += 1;
            }
        }

        System.out.println("wordCount="+wordCount);
        System.out.println("digitCount="+digitCount);
        System.out.println("letterCount="+letterCount);
        System.out.println("spaceCount="+spaceCount);
        System.out.println("otherCount="+otherCount);
    }
}
```
