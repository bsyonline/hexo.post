---
title: Elasticsearch 6 REST API Practice IV - Search
date: 2017-12-31 13:58:36
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

## IV. Search
#### Routing
在执行搜索时，查询会被广播到所有的索引/索引碎片。由于数据是哈希存储的，所以一般也不存在性能问题。如果我们已经知道数据存储的位置，那么使用 routing 直接告诉系统去指定的位置查询数据，无疑会减少查询时的开销。要按数据的存储位置查询，自然是要在存储的时候就指定好数据的位置。指定位置可能会出现数据热点，所以需要根据业务需要来确定是否需要 routing 。就算没有指定 routing 也是生效的，只不过是按照 document 的 id 去做 routing 。
```
curl -X POST "localhost:9200/change/change?routing=560E8C402E5914FAE0531ECDA8C0CF0D" -H 'Content-Type: appli cation/json' -d' 
{     
	"pripid" : "560E8C402E5914FAE0531ECDA8C0CF0D" 
}'
```
将 pripid 为 ```560E8C402E5914FAE0531ECDA8C0CF0D``` 的文档指定 routing ，才能用 routing 查询。
```
curl -X GET "localhost:9200/change/_search?routing=560E8C402E5914FAE0531ECDA8C0CF0D&q=pripid:560E8C402E5914FAE0531ECDA8C0CF0D"
```
上边使用的是 uri 的查询方式，“q” 是一个参数，还有其他参数可以查询 [https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-uri-request.html](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search-uri-request.html) 。

#### Query Body Search
除了 uri 的查询方式，通常更常用的 Query Body 的方式。
```
curl -X GET "localhost:9200/_search" -H 'Content-Type: application/json' -d'
{
    "from" : 0, "size" : 10,
    "sort" : [
        { "date" : {"order" : "asc"}},
        "_score"
    ],
    "_source": ["date", "*_value"],
    "query" : {
        "term" : { "column" : "E_ENT_BASEINFO.NAME" }
    }
}
'
```
#### Scroll
通常 query 会返回少量的结果，如果要返回大量数据就需要用到 scroll 。
```
curl -X POST "localhost:9200/change/_search?scroll=1m" -H 'Content-Type: application/json' -d'
{
    "size": 1,
    "query": {
        "match" : {
            "column" : "E_ENT_BASEINFO.NAME"
        }
    }
}
'
```
scroll 会返回一个 scroll_id ，以供获取后续数据时作为输入使用。
```
curl -X POST "localhost:9200/_search/scroll" -H 'Content-Type: application/json' -d'
{
    "scroll" : "1m", 
    "scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAHdFk5ZR0hvdFRsU1dhZkx5alE5UkhWWUEAAAAAAAAB3hZOWUdIb3RUbFNXYWZMeWpROVJIVllBAAAAAAAAAd8WTllHSG90VGxTV2FmTHlqUTlSSFZZQQAAAAAAAAHgFk5ZR0hvdFRsU1dhZkx5alE5UkhWWUEAAAAAAAAB4RZOWUdIb3RUbFNXYWZMeWpROVJIVllB" 
}
'
```
使用 scroll_id 来获取下一批数据，直到 scroll_id 为空，表示数据全部获取完成。
scroll 方式获取数据对 \_doc 升序优化，如果对结果没有顺序要求，使用 scroll 可以获得很好的性能。 scroll_id 在到达失效时间后会被自动删除，但是如果确定 scroll_id 已被使用，并且以后不再使用，可以主动删除，已节省资源。

