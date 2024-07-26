---
title: BlackBerry 开发：XML 解析
date: 2012-05-14 15:53:17
tags:
 - BlackBerry
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


```Java
public class XmlParser {
    /**
     * 递归方法，用于将每一个节点元素遍历出来放入到hashtable（注：bb的api基于JDK1.3，不支持hashmap，泛型等）中
     *
     * @param node
     *            document中的节点
     * @param depth
     *            节点的深度，用于做递归
     */
    public static void fetchNode(Node node, int depth, Hashtable ht) {
        /*
         * 判断是否为一个node，如过是一个节点，找到所有的子节点和子节点的数量。
         * 如果一个节点是一个文本节点，则将其放入到hashtable中，如果是一个元素节点， 则将该节点放入hashtable中，
         * 并对其子元素进行递归，直到遍历到最后一个文本 元素，递归结束。
         */
        if (node.getNodeType() == Node.ELEMENT_NODE) {
            // 如果是一个节点，找到所有的子节点
            NodeList childNodes = node.getChildNodes();
            int numChildren = childNodes.getLength();

            Node firstChild = childNodes.item(0);
            if (numChildren == 1 && firstChild.getNodeType() == Node.TEXT_NODE) {
                  ht.put(node.getNodeName(),
                              firstChild.getNodeValue() == null ? "" : firstChild
                                          .getNodeValue());
            } else {
                  ht.put(node.getNodeName(),
                              firstChild.getNodeValue() == null ? "" : firstChild
                                          .getNodeValue());
                  for (int i = 0; i < numChildren; ++i) {
                        fetchNode(childNodes.item(i), depth + 1, ht);
                  }
            }
        } else {
            // 如果不是node，那就是一个value
            String nodeValue = node.getNodeValue();
            // 如果不value不是空，则放入hashtable中
            if (nodeValue.trim().length() != 0) {
                  ht.put(node.getNodeName(), node.getNodeValue() == null ? ""
                              : node.getNodeValue());
            }
        }
    }
}
```
