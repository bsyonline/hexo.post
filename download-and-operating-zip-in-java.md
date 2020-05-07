---
title: Java URL下载 zip
date: 2015-12-16 15:52:35
tags:
 - Java
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

```java
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.FileReader;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.URL;
import java.net.URLConnection;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;

public class CilentTest {

     public static void main(String[] args) throws Exception {
          String urlString = "http://115.182.16.65:9090/ONOFFIQWS/download/1485356343/151030183903530378126.zip";
          String path = "D:\\test.zip";
          download(urlString, path);

          unzip(path);

          String savePath = path.substring(0, path.lastIndexOf(File.separator))
                    + File.separator + "temp";
          read(savePath);

          removeDir(new File(path));
          removeDir(new File(savePath));

     }

     /**
     * 下载文件
     *
     * @param urlString
     *            文件地址
     * @param path
     *            本地存放路径
     * @throws Exception
     */
     public static final void download(String urlString, String path)
               throws Exception {
          InputStream in = null;
          OutputStream out = null;
          int connectTimeout = 30 * 1000; // 连接超时:30s
          int readTimeout = 1 * 1000 * 1000; // IO超时:1min
          byte[] buffer = new byte[8 * 1024]; // IO缓冲区:8KB

          URL url = new URL(urlString);
          File file = new File(path);
          URLConnection conn = url.openConnection();

          conn.setConnectTimeout(connectTimeout);
          conn.setReadTimeout(readTimeout);
          conn.connect();

          in = conn.getInputStream();
          out = new FileOutputStream(file);

          while (true) {
               int bytes = in.read(buffer);
               if (bytes == -1) {
                    break;
               }
               out.write(buffer, 0, bytes);

          }

          in.close();
          out.close();
     }

     /**
     * 解压缩zip
     *
     * @param path
     *            文件路径
     */
     public static void unzip(String path) {
          final int buffer = 2048;
          int count = -1;
          int index = -1;

          // 解压缩路径 .\temp
          String savePath = path.substring(0, path.lastIndexOf(File.separator))
                    + File.separator + "temp" + File.separator;

          File temPath = new File(savePath);
          // 创建temp文件夹
          if (!temPath.exists()) {
               temPath.mkdirs();
          }

          try {
               BufferedOutputStream bos = null;
               ZipEntry entry = null;
               FileInputStream fis = new FileInputStream(path);
               ZipInputStream zis = new ZipInputStream(
                         new BufferedInputStream(fis));

               while ((entry = zis.getNextEntry()) != null) {

                    String tempFile = entry.getName();

                    index = tempFile.lastIndexOf("/");
                    if (index > -1) {
                         tempFile = tempFile.substring(index + 1);
                    }
                    tempFile = savePath + tempFile;
                    File f = new File(tempFile);
                    f.createNewFile();

                    FileOutputStream fos = new FileOutputStream(f);
                    bos = new BufferedOutputStream(fos, buffer);

                    byte data[] = new byte[buffer];
                    while ((count = zis.read(data, 0, buffer)) != -1) {
                         bos.write(data, 0, count);
                    }

                    bos.flush();
                    bos.close();
               }

               zis.close();

          } catch (Exception e) {
               e.printStackTrace();
          }
     }

     /**
     * 读目录下所有文件内容
     *
     * @param path
     *            文件夹
     * @throws Exception
     */
     public static final void read(String path) throws Exception {
          File dir = new File(path);
          if (!dir.exists()) {
               throw new RuntimeException("doss not exist");
          }
          if (dir.isDirectory()) {
               File[] files = dir.listFiles();
               for (File f : files) {
                    StringBuffer sb = new StringBuffer();
                    String filePath = path + File.separator + f.getName();
                    BufferedReader br = new BufferedReader(new FileReader(filePath));
                    String s = null;
                    while ((s = br.readLine()) != null) {
                         sb.append(s);
                    }
                    System.out.println(sb.toString());
                    br.close();
               }
          } else {
               throw new RuntimeException("dose not a directory");
          }
     }

     /**
     * 递归删除文件夹
     *
     * @param path
     *            文件夹
     */
     public static final void removeDir(File path) {
          if (!path.exists()) {
               throw new RuntimeException("doss not exist");
          }
          if (path.isFile()) {
               path.delete();
               return;
          }
          File[] files = path.listFiles();
          for (int i = 0; i < files.length; i++) {
               removeDir(files[i]);
          }
          path.delete();
     }

}
```
