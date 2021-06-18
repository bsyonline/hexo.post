---
title: A Guide to MongoDB API and Java Driver
tags:
  - MongoDB
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-05-25 12:01:20
thumbnail:
---





查询所有

```
db.getCollection('order').find()
```

查询 customerId=1 的记录

```
db.getCollection('order').find({"customerId":1})
```

查询 customerId=1 的记录，按 userId 降序排序

```
db.getCollection('order').find({"customerId":1}).sort({"userId":-1})
```



按 userId 分组

```
db.getCollection('order').aggregate([
  {$group:{_id:"$userId",count:{"$sum":1}}}
])
```

```
/* 1 */
{
    "_id" : 6,
    "count" : 2.0
}
/* 2 */
{
    "_id" : 5,
    "count" : 2.0
}
/* 3 */
{
    "_id" : 2,
    "count" : 4.0
}
/* 4 */
{
    "_id" : 1,
    "count" : 3.0
}
/* 5 */
{
    "_id" : 3,
    "count" : 2.0
}
/* 6 */
{
    "_id" : 4,
    "count" : 2.0
}
```

使用 java driver

```
public void groupByUserId() {
    MongoCollection<Document> col = mongoClient.getDatabase(database).getCollection(collection);
    MongoCursor<Document> cursor = col.aggregate(Lists.newArrayList(
            group("$userId", sum("count", new BsonInt32(1)))
    )).iterator();
    while (cursor.hasNext()) {
        Document document = cursor.next();
        System.out.println(new Gson().toJson(document));
    }
}
```

```
{"_id":6,"count":2}
{"_id":5,"count":2}
{"_id":2,"count":4}
{"_id":1,"count":3}
{"_id":3,"count":2}
{"_id":4,"count":2}
```



按多个字段分组

```
db.getCollection('order').aggregate([
  {$group:{_id:{customerId:"$customerId",userId:"$userId"},count:{"$sum":1}}}
])
```

```
/* 1 */
{
    "_id" : {
        "customerId" : 4,
        "userId" : 4
    },
    "count" : 1.0
}
/* 2 */
{
    "_id" : {
        "customerId" : 3,
        "userId" : 2
    },
    "count" : 1.0
}
/* 3 */
{
    "_id" : {
        "customerId" : 2,
        "userId" : 1
    },
    "count" : 1.0
}
/* 4 */
{
    "_id" : {
        "customerId" : 1,
        "userId" : 6
    },
    "count" : 2.0
}
/* 5 */
{
    "_id" : {
        "customerId" : 1,
        "userId" : 2
    },
    "count" : 3.0
}
/* 6 */
{
    "_id" : {
        "customerId" : 1,
        "userId" : 5
    },
    "count" : 2.0
}
/* 7 */
{
    "_id" : {
        "customerId" : 1,
        "userId" : 1
    },
    "count" : 2.0
}
/* 8 */
{
    "_id" : {
        "customerId" : 1,
        "userId" : 3
    },
    "count" : 2.0
}
/* 9 */
{
    "_id" : {
        "customerId" : 1,
        "userId" : 4
    },
    "count" : 1.0
}
```

使用 java driver

```
public void groupByCustomerAndUser() {
    MongoCollection<Document> col = mongoClient.getDatabase(database).getCollection(collection);
    MongoCursor<Document> cursor = col.aggregate(Lists.newArrayList(
            group(new BasicDBObject("customerId", "$customerId").append("userId", "$userId"), sum("count", new BsonInt32(1)))
    )).iterator();
    while (cursor.hasNext()) {
        Document document = cursor.next();
        System.out.println(new Gson().toJson(document));
    }
}
```

```
{"_id":{"customerId":4,"userId":4},"count":1}
{"_id":{"customerId":3,"userId":2},"count":1}
{"_id":{"customerId":2,"userId":1},"count":1}
{"_id":{"customerId":1,"userId":6},"count":2}
{"_id":{"customerId":1,"userId":2},"count":3}
{"_id":{"customerId":1,"userId":5},"count":2}
{"_id":{"customerId":1,"userId":1},"count":2}
{"_id":{"customerId":1,"userId":3},"count":2}
{"_id":{"customerId":1,"userId":4},"count":1}
```



使用 pipeline ，组合多个操作，按用户去重，查询客户id为1的订单数量

```
db.getCollection('order').aggregate([
  {$match:{"customerId": 1}},
  {$group:{_id:{customerId:"$customerId",userId:"$userId"}}},
  {$group:{_id:"$_id.customerId",count:{"$sum":1}}}
])
```

```
/* 1 */
{
    "_id" : 1,
    "count" : 6.0
}
```

使用 java driver

```
public void findByCustomerDistinctByUser(Integer customerId) {
    MongoCollection<Document> col = mongoClient.getDatabase(database).getCollection(collection);
    MongoCursor<Document> cursor = col.aggregate(Lists.newArrayList(
            match(eq("customerId", customerId)),
            group(new BasicDBObject("customerId", "$customerId").append("userId", "$userId"), sum("count", new BsonInt32(1))),
            group("$_id.customerId", sum("count", new BsonInt32(1)))
    )).iterator();
    while (cursor.hasNext()) {
        Document document = cursor.next();
        System.out.println(new Gson().toJson(document));
    }
}
```

```
{"_id":1,"count":6}
```

也可以将查询语句转成 bson 对象，这样写

```
public void findByCustomerDistinctByUser1() {
    MongoCollection<Document> col = mongoClient.getDatabase(database).getCollection(collection);
    String query1 = "{$match:{\"customerId\": 1}}";
    String query2 = "{$group:{_id:{customerId:\"$customerId\",userId:\"$userId\"}}}";
    String query3 = "{$group:{_id:\"$_id.customerId\",count:{\"$sum\":1}}}";
    Bson bson1 = (Bson) JSON.parse(query1);
    Bson bson2 = (Bson) JSON.parse(query2);
    Bson bson3 = (Bson) JSON.parse(query3);
    MongoCursor<Document> cursor = col.aggregate(Lists.newArrayList(bson1, bson2, bson3)).iterator();
    while (cursor.hasNext()) {
        Document document = cursor.next();
        System.out.println(new Gson().toJson(document));
    }
}
```



小数据量分页使用 skip() 和 limit()

```
MongoCursor<Document> cursor = col.find()
                .skip(page * pageSize)
                .limit(pageSize)
                .iterator();
```

简单，但是数据量大时有效率问题。因为会把所有记录查出来，从头变例，并且无法使用索引。

大数据量

```
MongoCursor<Document> cursor = col.find(gt("_id", new ObjectId(head)))
                .batchSize(batchSize).limit(limit).iterator();
```

效率高，但是必须回传当前页的最后一条记录的 id，同理上一页，需要回传当前页的第一条记录id，find 条件需要改成 lt() 。还有一个缺点是不能跳页。