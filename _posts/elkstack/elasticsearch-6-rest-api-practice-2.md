---
title: Elasticsearch 6 REST API Practice II - Document
date: 2017-12-29 13:58:36
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

## II. Document
#### Write
```
curl -X PUT -H 'Content-Type: application/json' 'http://localhost:9200/change/change/c6673e3c3c29af5c' -d '
{
    "id": "c6673e3c3c29af5c",
    "node": 140000,
    "pripid": "560E8C402E5914FAE0531ECDA8C0CF0D",
    "date": 20180418,
    "column": "E_ENT_BASEINFO.OPTO",
    "new_value": "2027-08-01",
    "old_value": "",
    "type": "UPDATE"
}'
```
#### Get
```
curl -X GET 'http://localhost:9200/change/change/c6673e3c3c29af5c'
```
不显示索引内容
```
curl -X GET 'http://localhost:9200/change/change/c6673e3c3c29af5c?_source=false'
```
只显示索引内容
```
curl -X GET 'http://localhost:9200/change/change/c6673e3c3c29af5c/_source'
```
#### Multi Get
```
curl -X GET "localhost:9200/_mget" -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_index" : "change",
            "_type" : "change",
            "_id" : "c6673e3c3c29af5c"
        },
        {
            "_index" : "change",
            "_type" : "change",
            "_id" : "3750ab2377453d13"
        }
    ]
}
'
```

```
curl -X GET "localhost:9200/change/_mget" -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_type" : "change",
            "_id" : "c6673e3c3c29af5c"
        },
        {
            "_type" : "change",
            "_id" : "3750ab2377453d13"
        }
    ]
}
'
```

```
curl -X GET "localhost:9200/change/change/_mget" -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_id" : "c6673e3c3c29af5c"
        },
        {
            "_id" : "3750ab2377453d13"
        }
    ]
}
'
```
```
curl -X GET "localhost:9200/change/change/_mget" -H 'Content-Type: application/json' -d'
{
  "ids": ["c6673e3c3c29af5c","3750ab2377453d13"]
}'
```
```
curl -X GET "localhost:9200/change/change/_mget" -H 'Content-Type: application/json' -d'
{
  "docs":[
      {
        "_id":"c6673e3c3c29af5c",
        "_source":["old_value","date"]
      },
      {
        "_id":"3750ab2377453d13",
        "_source":["new_value"]
      }
  ]
}'
```
#### 删除索引
```
curl -X DELETE 'http://localhost:9200/change/change/5c0d81efc98b8954'
```
#### 按条件删除

```
curl -X POST 'http://localhost:9200/change/change/_delete_by_query' -H 'Content-Type: application/json' -d'
{
  "query": { 
    "match": {
      "pripid": "560E8C402E5914FAE0531ECDA8C0CF0D"
    }
  }
}'
```

#### 更新索引

```
curl -X POST 'http://localhost:9200/change/change/c6673e3c3c29af5c/_update' -H 'Content-Type: application/json' -d'
{
  "doc": {
      "node": 110000
  }
}'
```


#### 批量操作

```
curl -X POST 'http://localhost:9200/_bulk' -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "change", "_type" : "change", "_id" : "c6673e3c3c29af5c" } }
{"id":"c6673e3c3c29af5c","node":140000,"pripid":"560E8C402E5914FAE0531ECDA8C0CF0D","date":20180418,"column":"E_ENT_BASEINFO.OPTO","new_value":"2027-08-01","old_value":"","type":"UPDATE"}
{ "index" : { "_index" : "change", "_type" : "change", "_id" : "5c0d81efc98b8954" } }
{"id":"5c0d81efc98b8954","node":500000,"pripid":"500107010100010395","date":20180418,"column":"E_ENT_BASEINFO.OPTO","new_value":"2099-12-31","old_value":"","type":"UPDATE"}
{ "index" : { "_index" : "change", "_type" : "change", "_id" : "3750ab2377453d13" } }
{"id":"3750ab2377453d13","node":500000,"pripid":"5001071201403190469324","date":20180418,"column":"E_ENT_BASEINFO.OPTO","new_value":"2099-12-31","old_value":"","type":"UPDATE"}
'
```
每行数据都要指定 index， type 和 id 。

#### 重建索引
```
curl -X POST 'http://localhost:9200/_reindex' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "change"
  },
  "dest": {
    "index": "new_change"
  }
}'
```
对部分索引重建
```
curl 'http://localhost:9200/_reindex' -H 'Content-Type: application/json'  -d '
{
  "source":{
    "index":"change",
    "type": "change",
    "query":{
      "term":{
        "node": 650000
      }
    }
  },
  "dest":{
    "index":"change_650000"
  }
}'
```
合并为新索引
```
curl 'http://localhost:9200/_reindex' -H 'Content-Type: application/json' -d '
{
  "source":{
    "index":["change_650000", "change_410000"],
    "type": ["change", "change"]
  },
  "dest":{
    "index":"change_410000_650000"
  }
}'
```
index 可以使用通配符。

#### 远程重建索引

```
curl -X POST 'http://localhost:9200/_reindex' -d '
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200",
      "username": "user",
      "password": "pass"
    },
    "index": "source",
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}'
```



## III. 检索

**初始化数据**
```
curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '
{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}'

curl -XPUT 'http://localhost:9200/twitter/tweet/2' -d '
{
    "user" : "allen",
    "postDate" : "2010-01-25T11:02:10",
    "message" : "using Elasticsearch"
}'
```
####  分页
分页使用的参数 from 和 size ，from 表示从第一个文档开始，默认从 0 开始，size 表示一页几条记录，默认是 10 条。
```
curl '192.168.207.14:29200/change/_search' -d '
{
    "from":0,
    "size":10,
    "query":{
        "term" : {"date" : "20170518"}
    }
}'
```
>和数据库分页不同，elasticsearch 的分页数量 from + size 不能超出 index.max_result_window 参数的限制（默认为 10000）。该值可在 elasticsearch 的配置文件中修改。

#### 查询得分
每个查询到的文档都有一个得分，查询结果可以通过得分来过滤，不过这个不常用，因为无法知道每个文档在每次查询中的具体得分。
```
curl 'localhost:9200/twitter/_search' -d '
{
    "min_score":0.5,
    "query":{
        "term" : {"user" : "kimchy"}
    }
}'
```

#### 指定返回的字段
```
curl 'localhost:9200/twitter/_search' -d '
{
    "fields":["user","message"],
    "query":{
        "term" : {"user" : "kimchy"}
    }
}'

curl 'localhost:9200/twitter/_search' -d '
{
    "query":{
        "term" : {"user" : "kimchy"}
    }
}'
```
>如果不指定 fields ，elasticsearch 默认返回 \_source 字段。

#### 指定查询的类型
elasticsearch 提供了一下几种查询类型：
* query_and_fetch
  1.3 以后移除
* query_then_fetch
  先获文档排序信息，在获取相关分片，返回结果最大为 size 取值。
* dfs_query_and_fetch
  1.3 以后移除
* dfs_query_then_fetch
  比 query_then_fetch 多了一步计算分布式词频。
* count
  2.0 废弃
* scan
  2.0 废弃
  可在查询是指定类型
```
curl 'localhost:9200/twitter/_search?pretty=true&search_type=query_then_fetch' -d '{
    "query":{
        "term" : {"user" : "kimchy"}
    }
}'
```
>默认是 query_then_fetch

#### 指定检索执行的位置（preference）
elasticsearch 默认在分片之间随机执行，可以修改为一下方式：
* \_primary
  在主分片上执行。
* \_primary_first
  优先在主分片上执行，如果主分片不可用，则在其他分片上执行。
* \_local
  在请求发送到的节点上的分片上执行。
* \_replica
  在副本上执行。
* \_replica_first
  优先在副本上执行。
* \_only_node:xyz
  在 node_id 为 xyz 的节点上执行。
* \_prefer_node:xyz
  优先在 node_id 为 xyz 的节点上执行。
* \_shards:2,3
  在 2 和 3 分片上执行。
* \_only_nodes
  在子节点上执行。
* custome
  一个字符串值，带有相同的值的请求会在相同的节点上执行。

```
curl 'localhost:9200/twitter/_search?preference=_local' -d '{
    "query":{
        "term" : {"user" : "kimchy"}
    }
}'
```
#### DSL 查询
DSL （Domain Specific Language）包含两种类型的子句：
* Leaf query clauses
  用特定的值查找特定的字段
* 复合子句
  Leaf query clauses 和 复合子句的包装
##### 全文检索
###### match_all
```
curl 'localhost:9200/twitter/_search' -d '{
    "query":{
        "match_all": { "boost" : 1.2 }
    }
}'
```
###### match
```
curl 'localhost:9200/twitter/_search' -d '{
    "query":{
        "match" : {
            "message" : "Elasticsearch"
        }
    }
}'
```
有三种类型：
* **boolean match**
  默认的 match 查询类型。
```
curl 'localhost:9200/twitter/_search' -d '{
    "query":{
        "match" : {
            "message" : {
                "query" : "trying Elasticsearch",
                "operator" : "and"
            }
        }
    }
}'
```
operator 的取值为 or 或 and ，默认为 or 。
* **match_phrase**
  使用 slop 来定义查询的词之间间隔多少个词算查询匹配。
```
curl 'localhost:9200/twitter/_search' -d '{
    "query":{
        "match_phrase" : {
            "message" : {
                "query" : "trying Elasticsearch",
                "slop" : 2
            }
        }
    }
}'
```
* **match_phrase_prefix**
  可以将最后一个词作为前缀匹配。比 match_phrase 多了一个 max_expansions 参数，表名最后一个词可以扩展多少个前缀。
```
curl 'localhost:9200/twitter/_search' -d '{
    "query":{
        "match_phrase_prefix" : {
            "message" : {
                "query" : "using E",
                "max_expansions": 20
            }
        }
    }
}'
```




##### term 级别查询
###### term 查询
精确匹配给定的值。
```
curl 'localhost:9200/twitter/_search' -d '{
    "query":{
        "term" : {"user" : "kim"}
    }
}'
```
>term 不会解析查询的字段，即不会对字段分词

###### terms 查询
```
curl 'localhost:9200/twitter/_search' -d '{
    "query":{
        "terms" : {
            "user" : ["kimchy", "allen"]
        }
    }
}'
```


#### 排序
elasticsearch 使用的默认排序为得分逆序排序。
```
"sort":[
    {"_score": "desc"}
]
```
我们可对一个或多个字段指定排序规则。
```

```
