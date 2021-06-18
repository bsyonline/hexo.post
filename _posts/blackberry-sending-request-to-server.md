---
title: BlackBerry 开发：向服务器发请求
date: 2012-04-22 15:53:17
tags:
 - BlackBerry
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
<!--more-->
```JAVA
public String httpFetch(String xml,String url) {
    String content = "";
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    HttpConnection hconn = null;
    OutputStream out = null;




    String bodystr = xml.toString();
    try {
        hconn = (HttpConnection) Connector.open(url);
        //UrlRequestedEvent
        hconn.setRequestMethod(HttpConnection.POST);
        hconn.setRequestProperty(HttpProtocolConstants.HEADER_USER_AGENT, "BlackBerry/5.0.0");
        hconn.setRequestProperty(HttpProtocolConstants.HEADER_CONTENT_LENGTH, String.valueOf(bodystr.getBytes().length));
        hconn.setRequestProperty(HttpProtocolConstants.HEADER_CONTENT_TYPE,"application/x-www-form-urlencoded");
        out = hconn.openOutputStream();
        out.write(bodystr.getBytes());
        out.flush();

          int status = hconn.getResponseCode();

          if (status == HttpConnection.HTTP_OK) {
                // Is this html?
                String contentType = hconn.getHeaderField(HEADER_CONTENTTYPE);
                boolean htmlContent = (contentType != null && contentType.startsWith(CONTENTTYPE_TEXTHTML));

                InputStream input = hconn.openInputStream();

                int len = 0;
                int size = 0;
                byte[] data = new byte[256];
                StringBuffer raw = new StringBuffer();

                while (-1 != (len = input.read(data))) {
                      raw.append(new String(data, 0, len));
                      size += len;
                }

                raw.insert(0, "bytes received]\n");
                raw.insert(0, size);
                raw.insert(0, '[');
                content = raw.toString();

                if (htmlContent) {
                      content = prepareData(raw.toString());
                }
                input.close();
          } else {
                content = "response code = " + status;
          }
          hconn.close();
    } catch (IOException e) {
          content = e.toString();
    }
    _fetchStarted = false;

    return content;
}
```
