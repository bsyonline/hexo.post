---
title: JAXB 操作 xml
date: 2016-02-20 15:49:59
tags:
 - JAXB
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 使用 @XmlRootElement 和 @XmlElement 标示根节点对象和属性节点对象
```java
@XmlRootElement(name = "DATA")
public class Data {

    List<DataItem> dataItems;

    public Data() {
    }

    @XmlElement(name = "ITEM")
    public List<DataItem> getDataItems() {
        return dataItems;
    }

    public void setDataItems(List<DataItem> dataItems) {
        this.dataItems = dataItems;
    }
}
```
### xml 转对象
```java
public static Data fromXml(String xml){
    Data data = null;
    try {
        JAXBContext context = JAXBContext.newInstance(Data.class);
        Unmarshaller unmarshaller = context.createUnmarshaller();
        data = (Data) unmarshaller.unmarshal(new StringReader(xml));
    } catch (JAXBException e) {
        e.printStackTrace();
    }
    return data;
}
```
### 对象转 xml
```java
	public static void toXml(String path) {
	    try {
	        JAXBContext context = JAXBContext.newInstance(Data.class);
	        Marshaller marshaller = context.createMarshaller();
	        marshaller.marshal(fromXml(xml), new File(path));
	    } catch (JAXBException e) {
	        e.printStackTrace();
	    }
	}
```
### 设置 xml 节点的顺序
```java
@XmlType(propOrder = {"orderList", "basic", "shareholder", "person"})
public class DataItem {

    OrderList orderList;
    Basic basic;
    Shareholder shareholder;
    Person person;

     //getter and setter
}
```
