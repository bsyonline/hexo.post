---
title: 经纬度查省市县（逆编码）
date: 2013-07-16 15:45:13
tags:
 - Java
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


阿里云提供的接口

[http://gc.ditu.aliyun.com/regeocoding?l="+lat+","+lng+"&type=010](http://gc.ditu.aliyun.com/regeocoding?l="+lat+","+lng+"&type=010)
type=010 兴趣点（poi）100代表道路，010代表POI，001代表门址，111可以同时显示前三项


[api 文档](http://gc.ditu.aliyun.com/jsdoc/geocode_api.html) 参考。

行政区划：
[http://recode.ditu.aliyun.com/dist_query?l=39.938133,116.395739](http://recode.ditu.aliyun.com/dist_query?l=39.938133,116.395739)
l 参数定义：纬度，经度


```java
public String[] getArea(float lng, float lat){
    String[] arr = new String[3];
    String path="http://gc.ditu.aliyun.com/regeocoding?l="+lat+","+lng+"&type=010";
    URL url= null;
    try {
        url = new URL(path);
        HttpURLConnection conn=(HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");
        conn.setConnectTimeout(5000);
        String jsonString = "";
        if(conn.getResponseCode()==200){
            BufferedReader is=new BufferedReader(new InputStreamReader(conn.getInputStream(),"UTF-8"));
            String str;
            if((str=is.readLine())!=null)
            {
                jsonString += str;
            }

            is.close();

            JSONObject jsonObject = JSONObject.fromObject(jsonString);
            String addrList =  jsonObject.getString("addrList");

            JSONArray jsonarry = JSONArray.fromObject(addrList);
            String poi = null;
            for(int i = 0;i<jsonarry.size();i++){
                JSONObject jsonObject2 = jsonarry.getJSONObject(i);
                poi =  jsonObject2.getString("admName");
            }
            System.out.println("----------"+poi);

            if(!"".equals(poi)){
                String[] area = poi.split(",");
                arr[0] = area[0];
                arr[1] = area[1];

                if(area.length == 3) {
                    arr[2] = area[2];
                }

            }

        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return arr;
}
```
