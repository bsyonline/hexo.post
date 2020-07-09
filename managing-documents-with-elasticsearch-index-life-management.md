---
title: Managing Documents with Elasticsearch Index Life Management
date: 2020-07-09 16:58:57
tags:
---

上一篇讲了使用 ELK Stack 来搭建日志收集系统，这一篇来讲讲使用 Elasticsearch 来管理日志。日志存储在 Elasticsearch 是随着时间逐步增加的，有些日志会长期保存，而有些日志只需要保存一段时间后就删除，Elasticsearch 为 index 定义了生命周期，包括 4 个阶段：hot 、warm、cold 和 delete 。针对不同的阶段也定义了不同的策略，下面我们就来看一下如何使用策略来管理日志。

假设我们的日志规则是每 10000 条日志一个索引，日志最多保存 3 个月，那么我们可以创建一个策略。

```
curl -XPUT "http://localhost:9200/_ilm/policy/my_policy" -H 'Content-Type: application/json' -d'
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_docs": 10000
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}'
```

我们定义了一个滚动策略 rollover ，每 10000 条就会生成一个新的索引，并且 90 天后进入 delete 阶段，即被删除。

接下来需要将日志索引加上这个策略。日志我们都是通过 template 生成的，所以在 template 上加上策略。

```
curl -XPUT "http://localhost:9200/_template/my_template" -H 'Content-Type: application/json' -d'
{
  "index_patterns": ["log-*"], 
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "index.lifecycle.name": "my_policy", 
    "index.lifecycle.rollover_alias": "log-alias"
  }
}'
```

最后在初始化的时候创建第一个索引，并将索引设为可写。

```
curl -XPUT "http://localhost:9200/log-000001" -H 'Content-Type: application/json' -d'
{
  "aliases": {
	"log-alias": {
 	  "is_write_index": true
	}
  }
}'
```

以后往 log-alias 中写入新的日志是，就会使用我们设置的策略了。

> 详细的操作设置见 [https://www.elastic.co/guide/en/elasticsearch/reference/6.8/index-lifecycle-management.html](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/index-lifecycle-management.html) 。
>
> 这里附一些常用的操作：
>
> 查看策略
>
> ```
> curl -XGET "http://localhost:9200/_ilm/policy?pretty"
> ```
>
> 查看索引的信息
>
> ```
> curl -XGET "http://localhost:9200/test-*/_ilm/explain?pretty"
> ```
>
> 查看状态
>
> ```
> curl -XGET "http://localhost:9200/_ilm/status?pretty"
> ```
>
> 修改 ES 的 策略生效时间
>
> ```
> curl -XPUT "http://localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
> {
>   "transient": {
>     "indices.lifecycle.poll_interval": "3s" 
>   }
> }'
> ```

