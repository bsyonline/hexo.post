---
title: Elasticsearch 6 REST API Practice I - Index
date: 2017-12-28 13:58:36
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

[TOC]

学习 Elasticsearch 6.2 REST API 时，使用过的实验例子。

## I. Index
#### Create Index
```
curl -X PUT -H 'Content-Type: application/json' 'http://localhost:9200/change' -d '
{
  "mappings": {
    "change": {
      "properties": {
        "date": {
          "format": "yyyyMMdd",
          "type": "date"
        },
        "new_value": {
          "type": "keyword"
        },
        "node": {
          "type": "integer"
        },
        "pripid": {
          "type": "keyword"
        },
        "column": {
          "type": "keyword"
        },
        "id": {
          "type": "keyword"
        },
        "old_value": {
          "type": "keyword"
        },
        "type": {
          "type": "keyword"
        },
        "apprDate": {
          "format": "yyyy-MM-dd",
          "type": "date"
        }
      }
    }
  }
}'
```

从 6.0 开始 REST API 需要指定 header 信息。

#### Get Index
```
curl -X GET 'http://localhost:9200/change'
```

#### Delete Index
```
curl -X DELETE 'http://localhost:9200/change'
```

#### Indices Exists
```
curl -I 'http://localhost:9200/change'
```

存在返回 200 ，不存在返回 404 。
#### Types Exists
```
curl -I 'http://localhost:9200/change/_mapping/change'
```
存在返回 200 ，不存在返回 404 。
#### Get Mapping
```
curl -X GET 'http://localhost:9200/change/_mapping/_doc' 
```
#### Get Field Mapping
```
curl -X GET 'http://localhost:9200/change/_mapping/field/node'
```
#### Delete Mapping
不提供删除 Mapping 的操作，可以删除 Index ，再重建 Mapping 。

#### Update Mapping
我们发现有一个字段 apprDate 写错了，需要新加一个 appr_date 字段。
```
curl -XPUT -H 'Content-Type: application/json' 'http://localhost:9200/change/_mapping/change' -d '
{
  "properties": {
    "appr_date": {
      "format": "yyyy-MM-dd",
      "type": "date"
    }
  }
}'
```
#### Add Index Alias
```
curl -X POST "http://localhost:9200/_aliases" -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "add" : { "index" : "change", "alias" : "CHANGE" } }
    ]
}'
```
#### Remove Index Alias
```
curl -X POST 'http://localhost:9200/_aliases' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "remove" : { "index" : "change", "alias" : "CHANGE" } }
    ]
}'
```
#### Rename Index Alias
```
curl -X POST 'http://localhost:9200/_aliases' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "remove" : { "index" : "change", "alias" : "CHANGE" } },
        { "add" : { "index" : "change", "alias" : "CHANGE1" } }
    ]
}'
```
