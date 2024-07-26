---
title: BlackBerry 开发：文件操作
date: 2012-04-12 15:53:17
tags:
 - BlackBerry
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```JAVA
/**
 * 文件操作
 *
 * @author Rolex
 * @date 2011-9-28
 */
public class FileUtil {

    /**
     *
     * 将内容保存到SD卡上的文件中，如果文件不存在，则新建
     *
     * @param content
     * @return
     */
    public static boolean writeFile(String content) {     
        boolean isWrite=false;
        try {
            DataOutputStream os = null;
            FileConnection file = null;            

            file = (FileConnection) Connector.open("file:///SDCard/config.xml" , Connector.READ_WRITE);
            if(!file.exists()){
              file.create();
            }
            os = file.openDataOutputStream();
            os.write(content.getBytes());

            os.close();
            file.close();
            isWrite = true;
        }
        catch (Exception e) {
            e.printStackTrace();
        }
        return isWrite;
    }
}
```
