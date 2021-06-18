---
title: Elasticsearch 6 REST API Practice III - Cat
date: 2017-12-30 13:58:36
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

## III. Cat
#### Cat Count
```
curl -X GET "localhost:9200/_cat/count/change?v"
```
#### Cat health
```
curl -X GET "localhost:9200/_cat/health?v"
```
查看状态是 yellow 的索引
```
curl -X GET "localhost:9200/_cat/indices?v&health=yellow"
```
按 count 排序
```
curl -X GET "localhost:9200/_cat/indices?v&s=docs.count:desc"
```
#### Cat Template
```
curl -X GET "localhost:9200/_cat/templates?v&s=name"
```
#### Cat Snapshot
```
curl -X GET "localhost:9200/_cat/snapshots/repo1?v&s=id"
```
#### Cat Recovery
```
curl -X GET "localhost:9200/_cat/recovery?v"
```


