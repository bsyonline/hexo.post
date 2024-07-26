---
title: GC
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2019-11-04 14:30:42
thumbnail:
---


-Xms41m 
-Xmx41m 
-Xmn10m 
-XX:+UseParallelGC 
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps

```
public class GCTest {
    public static void main(String[] args) throws Exception {
        List caches = new ArrayList();
        for (int i = 0; i < 7; i++) {
            System.out.println("次数：" + (i + 1));
            caches.add(new byte[1024 * 1024 * 3]);
        }
        caches.clear();
        for (int i = 0; i < 2; i++) {
            System.out.println("clear后次数：" + (i + 1));
            caches.add(new byte[1024 * 1024 * 3]);
        }
    }
}
```

```
java -jar GCTest -Xms30m -Xmx30m -Xmn10m -XX:+UseParallelGC -XX:+PrintGCDetails
```

每次放3M
1、在YGC执行前，min(目前 eden 已使用的大小,之前平均晋升到old的大小中的较小值) > old剩余空间大小 ? 不执行YGC，直接执行Full GC : 执行YGC；
2、在YGC执行后，平均晋升到old的大小 > old剩余空间大小 ? 触发Full GC ： 什么都不做。

<table style="font-size:12px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;"><tr><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;width: 50px;"></th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">eden</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">old</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">YGC</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">FGC</th><th style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;">说明</th></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第1次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第2次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第3次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是0）=0，小于old剩余的空间20，所以触发YGC，eden的对象进入old，old变成6，eden变成3。平均晋升到old的大小是6，小于old剩余的空间14，不触发FGC。</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第4次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第5次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">12</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是6）=6，小于old剩余的空间12，所以触发YGC，eden的对象进入old，old变成12，eden变成3。平均晋升到old的大小是6，小于old剩余的空间8，不触发FGC。</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第6次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">12</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第7次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">18</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是6）=6，小于old剩余的空间8，所以触发YGC，eden的对象进入old，old变成18，eden变成3。平均晋升到old的大小是6，大于old剩余的空间2，触发FGC。</td></tr><tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第8次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">6</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">18</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;"></td></tr>
<tr><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">第9次</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">3</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">18</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">0</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">1</td><td style="border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;">eden空间剩余2，小于需要的内存，min（eden已使用大小是6，平均进入old的大小是6）=6，大于old剩余的空间2，直接触发FGC。</td></tr></table>    


