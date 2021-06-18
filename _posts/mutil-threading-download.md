---
title: 多线程下载
date: 2015-12-18 15:49:57
tags:
 - Concurrent
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

<!--more-->

```java
import java.io.InputStream;
import java.io.RandomAccessFile;
import java.net.URL;
import java.net.URLConnection;

public class MutiThreadDownloadTest {

     /**
     * @param args
     */
     public static void main(String[] args) {
          String urlString = "http://www.7-zip.org/a/7z920-x64.msi";
          String path = "D:\\7z.msi";
          try {
               new MutiThreadDownloadTest().new MutiThreadDownload(urlString, path, 3).download();
          } catch (Exception e) {
               e.printStackTrace();
          }
     }

     class MutiThreadDownload {
          private String urlString;// 下载路径
          private String path;// 文件存储路径
          private int threadNum;// 线程数量
          private int fileSize;// 文件大小

          public MutiThreadDownload(String urlString, String path, int threadNum) {
               super();
               this.urlString = urlString;
               this.path = path;
               this.threadNum = threadNum;
          }

          public void download() throws Exception {
               URL url = new URL(urlString);
               URLConnection conn = url.openConnection();
               fileSize = conn.getContentLength();// 获取文件大小
               RandomAccessFile destFile = new RandomAccessFile(path, "rw");// 创建空文件
               // 设置下载文件相同大小
               destFile.setLength(fileSize);
               destFile.close();
               // 计算每个线程的下载大小
               int sizePerThread = fileSize / threadNum;
               // 开启线程
               for (int i = 0; i < threadNum; i++) {
                    // 计算每条线程的下载的开始位置
                    int startPos = i * sizePerThread;// 开始位置
                    int endPos = sizePerThread * (i + 1) - 1;// 结束位置

                    new DownloadThread(startPos,endPos,urlString,path).start();
               }
               destFile.close();
               conn = null;
          }

     }

     class DownloadThread extends Thread {
          // 定义下载的起始点
          private long startPos;
          // 定义下载的结束点
          private long endPos;
          private String urlString;
          private String path;

          // 构造器，传入输入流，输出流和下载起始点、结束点
          public DownloadThread(long startPos, long endPos, String urlString, String path) {
               // 输出该线程负责下载的字节位置
               System.out.println("startPos="+startPos + " - endPos:" + endPos);
               this.startPos = startPos;
               this.endPos = endPos;
               this.urlString = urlString;
               this.path = path;
          }

          public void run() {
               URL url = null;
               URLConnection conn = null;
               InputStream is = null;
               RandomAccessFile out = null;
               try {
                    url = new URL(urlString);
                    conn = url.openConnection();
                    is = conn.getInputStream();
                    byte[] buffer = new byte[1024];
                    int hasRead = 0;
                    // 读取网络数据，并写入本地文件
                    int length = 0;
                    out = new RandomAccessFile(path, "rw");
                    is.skip(this.startPos);//流从什么位置读
                    out.seek(startPos);//文件从什么位置写

                    // 读取网络数据，并写入本地文件
                    while (length < endPos - startPos
                              && (hasRead = is.read(buffer)) > 0) {
                         out.write(buffer, 0, hasRead);
                         // 累计该线程下载的总大小
                         length += hasRead;
                    }
                    System.out.println("线程:" + Thread.currentThread().getId() + "下载完成!");
               } catch (Exception ex) {
                    ex.printStackTrace();
               }
               // 使用finally块来关闭当前线程的输入流、输出流
               finally {
                    try {
                         if (out != null) {
                              out.close();
                         }
                         if (is != null) {
                              is.close();
                         }
                         if (conn != null){
                              conn = null;
                         }
                    } catch (Exception ex) {
                         ex.printStackTrace();
                    }
               }
          }
     }
}
```
