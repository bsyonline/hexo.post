---
title: Java 调用 shell 命令
date: 2017-08-09 09:26:40
tags:
 - Java
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---



Java 调用 shell 命令主要用到 Process 和 Runtime 两个类。

```java
Process process = Runtime.getRuntime().exec(cmd);
int c = process.waitFor();
if (c != 0) {
  System.out.println("执行shell异常终止");
} 
```

waitFor() 用来判断 Process 进程是否终止，0代表正常终止。

如果想读取 shell 命令的输出，可以用 Java I/O 读取。

```java
//读取标准输出流
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(process.getInputStream()));
String line;
while ((line = bufferedReader.readLine()) != null) {
  System.out.println(line);
}
```

```java
//读取标准错误流
BufferedReader brError = new BufferedReader(new InputStreamReader(process.getErrorStream()));
String errline = null;
while ((errline = brError.readLine()) != null) {
    System.out.println(errline);
}
```

基本用法很简单，但是还有一些小细节需要注意。在使用上边代码的时候，经常会出现执行一段时间之后程序就 hang 住不动，无法结束。原因是因为缓冲区被充满，造成 I/O 阻塞，导致程序无法结束，所以在使用应在调用wartFor() 之前先读取流，并且要先读取标准错误再读取标准输出。参考：[Java Runtime exec can hang](http://brian.pontarelli.com/2005/11/11/java-runtime-exec-can-hang/) 。

完整程序参考：

```java
import java.io.*;
public class Executor {
    public void execute() {
        BufferedReader bReader = null;
        InputStreamReader sReader = null;
        try {
          Process p = Runtime.getRuntime().exec(cmd);
          //为"错误输出流"单独开一个线程读取之,否则会造成标准输出流的阻塞
          Thread t = new Thread(new InputStreamRunnable(p.getErrorStream(), "ErrorStream"));
          t.start();
          //"标准输出流"就在当前方法中读取
          BufferedInputStream bis = new BufferedInputStream(p.getInputStream());
          sReader = new InputStreamReader(bis, "UTF-8");
          bReader = new BufferedReader(sReader);
          StringBuilder sb = new StringBuilder();
          String line;
          while ((line = bReader.readLine()) != null) {
            sb.append(line);
            sb.append("/n");
          }
          bReader.close();
          p.destroy();
          return sb.toString();
        } catch (Exception e) {
          e.printStackTrace();
        } 
    } 
}
```

```java
import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;

public class InputStreamRunnable implements Runnable {
    BufferedReader bReader = null;
    
    public InputStreamRunnable(InputStream is) {
        try {
            bReader = new BufferedReader(new InputStreamReader(new BufferedInputStream(is), "UTF-8"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public void run() {
        String line;
        try {
            while ((line = bReader.readLine()) != null) {
                System.err.println(line);
            }
            bReader.close();
        } catch (Exception ex) {
        }
    }
} 
```

