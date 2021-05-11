---
title: data warehouse data analysis project
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-05-05 11:33:09
thumbnail:
---



### 需求分析



### 架构设计

<img src="D:\Dev\pic\20210425\image-20210505114144027.png" alt="image-20210505114144027" style="zoom: 50%;" />

| 技术/工具 | 版本   |
| --------- | ------ |
| Hadoop    | 2.10.1 |
| ZooKeeper | 3.7.0  |
| Flume     | 1.9.0  |
| Hive      | 2.3.8  |
| HBase     | 2.3.5  |
| Sqoop     | 1.4.7  |
| MySQL     | 5.7.24 |

### 数据仓库命名

ODS(Operation Data Store) 原始数据层，ods_ 开头。

DWD(Data Warehouse Detail) 明细数据层，dwd_ 开头。

DWB(Data Warehouse Basic) 基础数据层，dwb_ 开头。

DWS(Data Warehouse Service) 服务数据层，dws_ 开头。

ADS(Application Data Store) 数据应用层，ads_ 开头。

临时表 _tmp 结尾。

备份 _bak 结尾。

### 数据准备

#### 1. launch log

log-launch-hdfs.conf

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /home/data/log/launch
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop-cluster/mall_data/log/launch/%Y-%m-%d
a3.sinks.k3.hdfs.filePrefix = log-launch-
a3.sinks.k3.hdfs.round = true
a3.sinks.k3.hdfs.roundValue = 1
a3.sinks.k3.hdfs.roundUnit = hour
a3.sinks.k3.hdfs.useLocalTimeStamp = true
a3.sinks.k3.hdfs.batchSize = 100
a3.sinks.k3.hdfs.fileType = DataStream
a3.sinks.k3.hdfs.rollInterval = 600
a3.sinks.k3.hdfs.rollSize = 134217700
a3.sinks.k3.hdfs.rollCount = 0
a3.sinks.k3.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```

启动

```
bin/flume-ng agent --conf conf/ --name a3 --conf-file job/log-launch-hdfs.conf
```

查看 HDFS 的 /mall_data/log 目录。

#### 2. event log

log-event-hdfs.conf

```
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = spooldir
a2.sources.r2.spoolDir = /home/data/log/event
a2.sources.r2.fileSuffix = .COMPLETED
a2.sources.r2.fileHeader = true
a2.sources.r2.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop-cluster/mall_data/log/event/%Y-%m-%d
a2.sinks.k2.hdfs.filePrefix = log-event-
a2.sinks.k2.hdfs.round = true
a2.sinks.k2.hdfs.roundValue = 1
a2.sinks.k2.hdfs.roundUnit = hour
a2.sinks.k2.hdfs.useLocalTimeStamp = true
a2.sinks.k2.hdfs.batchSize = 100
a2.sinks.k2.hdfs.fileType = DataStream
a2.sinks.k2.hdfs.rollInterval = 600
a2.sinks.k2.hdfs.rollSize = 124217700
a2.sinks.k2.hdfs.rollCount = 0
a2.sinks.k2.hdfs.minBlockReplicas = 1

# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

启动

```
bin/flume-ng agent --conf conf/ --name a2 --conf-file job/log-event-hdfs.conf
```

查看 HDFS 的 /mall_data/log 目录。

### 数据仓库搭建

#### 创建 ODS 层

```
hive> create database mall;
hive> use mall;
hive> show tables;
# 创建表
hive> CREATE EXTERNAL TABLE ods_launch_log (`line` string) PARTITIONED BY (`dt` string) LOCATION '/warehouse/mall/ods/ods_launch_log';
# 加载数据
hive> load data inpath '/mall_data/log/launch/2021-05-05' into table mall.ods_launch_log partition(dt='2021-05-05');
# 创建表
hive> CREATE EXTERNAL TABLE ods_event_log (`line` string) PARTITIONED BY (`dt` string) LOCATION '/warehouse/mall/ods/ods_event_log';
# 加载数据
hive> load data inpath '/mall_data/log/event/2021-05-05' into table mall.ods_event_log partition(dt='2021-05-05');
```

#### 创建 DWD 层

对ODS层数据进行清洗（去除空值，脏数据，超过极限范围的数据，行式存储改为列存储）。

##### 1. 日志启动表

创建表

```sql
CREATE EXTERNAL TABLE dwd_launch_log (
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string,  
    `app_time` string,
    `network` string, 
    `lng` string, 
    `lat` string, 
    `entry` string, 
    `open_ad_type` string, 
    `action` string, 
    `loading_time` string, 
    `detail` string, 
    `extend1` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_launch_log/';
```

导入数据

```sql
insert overwrite table dwd_launch_log
PARTITION (dt='2021-05-05')
select 
    get_json_object(line,'$.mid') mid_id,
    get_json_object(line,'$.uid') user_id,
    get_json_object(line,'$.vc') version_code,
    get_json_object(line,'$.vn') version_name,
    get_json_object(line,'$.l') lang,
    get_json_object(line,'$.sr') source,
    get_json_object(line,'$.os') os,
    get_json_object(line,'$.ar') area,
    get_json_object(line,'$.md') model,
    get_json_object(line,'$.ba') brand,
    get_json_object(line,'$.sv') sdk_version,
    get_json_object(line,'$.g') gmail,
    get_json_object(line,'$.hw') height_width,
    get_json_object(line,'$.t') app_time,
    get_json_object(line,'$.nw') network,
    get_json_object(line,'$.ln') lng,
    get_json_object(line,'$.la') lat,
    get_json_object(line,'$.entry') entry,
    get_json_object(line,'$.open_ad_type') open_ad_type,
    get_json_object(line,'$.action') action,
    get_json_object(line,'$.loading_time') loading_time,
    get_json_object(line,'$.detail') detail,
    get_json_object(line,'$.extend1') extend1
from ods_launch_log 
where dt='2021-05-05';
```

##### 2. 日志事件表

###### 事件日志基础明细表

创建表

```sql
CREATE EXTERNAL TABLE dwd_base_event_log(
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string, 
    `app_time` string, 
    `network` string, 
    `lng` string, 
    `lat` string, 
    `event_name` string, 
    `event_json` string, 
    `server_time` string
)
PARTITIONED BY (`dt` string)
stored as parquet
location '/warehouse/mall/dwd/dwd_base_event_log/';
```

导入数据

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_base_event_log 
PARTITION (dt='2021-05-05')
select
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    event_name,
    event_json,
    server_time
from
(
    select
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[0] as mid_id,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[1] as user_id,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[2] as version_code,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[3] as version_name,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[4] as lang,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[5] as source,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[6] as os,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[7] as area,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[8] as model,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[9] as brand,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[10] as sdk_version,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[11] as gmail,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[12] as height_width,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[13] as app_time,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[14] as network,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[15] as lng,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[16] as lat,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[17] as ops,
        split(base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la'),'\t')[18] as server_time
    from ods_event_log 
    where dt='2021-05-05' and base_analizer(line,'mid,uid,vc,vn,l,sr,os,ar,md,ba,sv,g,hw,t,nw,ln,la')<>'' 
) sdk_log lateral view flat_analizer(ops) tmp_k as event_name, event_json;
```

##### 3. 商品点击表

```sql
CREATE EXTERNAL TABLE dwd_display_log (
    `mid_id` string,
    `user_id` string,
    `version_code` string,
    `version_name` string,
    `lang` string,
    `source` string,
    `os` string,
    `area` string,
    `model` string,
    `brand` string,
    `sdk_version` string,
    `gmail` string,
    `height_width` string,
    `app_time` string,
    `network` string,
    `lng` string,
    `lat` string,
    `action` string,
    `goodsid` string,
    `place` string,
    `extend1` string,
    `category` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_display_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_display_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.action') action,
    get_json_object(event_json,'$.kv.goodsid') goodsid,
    get_json_object(event_json,'$.kv.place') place,
    get_json_object(event_json,'$.kv.extend1') extend1,
    get_json_object(event_json,'$.kv.category') category,
    server_time
from dwd_base_event_log 
where dt='2021-05-05' and event_name='display';
```



##### 4. 商品详情页表

```sql
CREATE EXTERNAL TABLE dwd_detail_log (
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string, 
    `app_time` string,  
    `network` string, 
    `lng` string, 
    `lat` string, 
    `entry` string,
    `action` string,
    `goodsid` string,
    `showtype` string,
    `news_staytime` string,
    `loading_time` string,
    `type1` string,
    `category` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_detail_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_detail_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.entry') entry,
    get_json_object(event_json,'$.kv.action') action,
    get_json_object(event_json,'$.kv.goodsid') goodsid,
    get_json_object(event_json,'$.kv.showtype') showtype,
    get_json_object(event_json,'$.kv.news_staytime') news_staytime,
    get_json_object(event_json,'$.kv.loading_time') loading_time,
    get_json_object(event_json,'$.kv.type1') type1,
    get_json_object(event_json,'$.kv.category') category,
    server_time
from dwd_base_event_log
where dt='2021-05-05' and event_name='newsdetail';
```



##### 5. 商品列表页表

```sql
CREATE EXTERNAL TABLE dwd_list_log(
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string,
    `height_width` string,  
    `app_time` string,
    `network` string, 
    `lng` string, 
    `lat` string, 
    `action` string,
    `loading_time` string,
    `loading_way` string,
    `extend1` string,
    `extend2` string,
    `type` string,
    `type1` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_list_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_list_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.action') action,
    get_json_object(event_json,'$.kv.loading_time') loading_time,
    get_json_object(event_json,'$.kv.loading_way') loading_way,
    get_json_object(event_json,'$.kv.extend1') extend1,
    get_json_object(event_json,'$.kv.extend2') extend2,
    get_json_object(event_json,'$.kv.type') type,
    get_json_object(event_json,'$.kv.type1') type1,
    server_time
from dwd_base_event_log
where dt='2021-05-05' and event_name='loading';
```



##### 6. 广告表

```sql
CREATE EXTERNAL TABLE dwd_ad_log (
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string,  
    `app_time` string,
    `network` string, 
    `lng` string, 
    `lat` string, 
    `entry` string,
    `action` string,
    `content` string,
    `detail` string,
    `ad_source` string,
    `behavior` string,
    `newstype` string,
    `show_style` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_ad_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_ad_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.entry') entry,
    get_json_object(event_json,'$.kv.action') action,
    get_json_object(event_json,'$.kv.content') content,
    get_json_object(event_json,'$.kv.detail') detail,
    get_json_object(event_json,'$.kv.source') ad_source,
    get_json_object(event_json,'$.kv.behavior') behavior,
    get_json_object(event_json,'$.kv.newstype') newstype,
    get_json_object(event_json,'$.kv.show_style') show_style,
    server_time
from dwd_base_event_log 
where dt='2021-05-05' and event_name='ad';
```



##### 7. 消息通知表

```sql
CREATE EXTERNAL TABLE dwd_notification_log (
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string,
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string,  
    `app_time` string,
    `network` string, 
    `lng` string, 
    `lat` string, 
    `action` string,
    `noti_type` string,
    `ap_time` string,
    `content` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_notification_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_notification_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.action') action,
    get_json_object(event_json,'$.kv.noti_type') noti_type,
    get_json_object(event_json,'$.kv.ap_time') ap_time,
    get_json_object(event_json,'$.kv.content') content,
    server_time
from dwd_base_event_log
where dt='2021-05-05' and event_name='notification';
```



##### 8. 用户前台活跃表

```sql
CREATE EXTERNAL TABLE dwd_active_foreground_log (
    `mid_id` string,
    `user_id` string,
    `version_code` string,
    `version_name` string,
    `lang` string,
    `source` string,
    `os` string,
    `area` string,
    `model` string,
    `brand` string,
    `sdk_version` string,
    `gmail` string,
    `height_width` string,
    `app_time` string,
    `network` string,
    `lng` string,
    `lat` string,
    `push_id` string,
    `access` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_foreground_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_active_foreground_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.push_id') push_id,
    get_json_object(event_json,'$.kv.access') access,
    server_time
from dwd_base_event_log
where dt='2021-05-05' and event_name='active_foreground';
```



##### 9. 用户后台活跃表

```sql
CREATE EXTERNAL TABLE dwd_active_background_log (
    `mid_id` string,
    `user_id` string,
    `version_code` string,
    `version_name` string,
    `lang` string,
    `source` string,
    `os` string,
    `area` string,
    `model` string,
    `brand` string,
    `sdk_version` string,
    `gmail` string,
     `height_width` string,
    `app_time` string,
    `network` string,
    `lng` string,
    `lat` string,
    `active_source` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_background_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_active_background_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.active_source') active_source,
    server_time
from dwd_base_event_log
where dt='2021-05-05' and event_name='active_background';
```



##### 10. 评论表

```sql
CREATE EXTERNAL TABLE dwd_comment_log (
    `mid_id` string,
    `user_id` string,
    `version_code` string,
    `version_name` string,
    `lang` string,
    `source` string,
    `os` string,
    `area` string,
    `model` string,
    `brand` string,
    `sdk_version` string,
    `gmail` string,
    `height_width` string,
    `app_time` string,
    `network` string,
    `lng` string,
    `lat` string,
    `comment_id` int,
    `userid` int,
    `p_comment_id` int, 
    `content` string,
    `addtime` string,
    `other_id` int,
    `praise_count` int,
    `reply_count` int,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_comment_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_comment_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.comment_id') comment_id,
    get_json_object(event_json,'$.kv.userid') userid,
    get_json_object(event_json,'$.kv.p_comment_id') p_comment_id,
    get_json_object(event_json,'$.kv.content') content,
    get_json_object(event_json,'$.kv.addtime') addtime,
    get_json_object(event_json,'$.kv.other_id') other_id,
    get_json_object(event_json,'$.kv.praise_count') praise_count,
    get_json_object(event_json,'$.kv.reply_count') reply_count,
    server_time
from dwd_base_event_log
where dt='2021-05-05' and event_name='comment';
```



##### 11. 收藏表

```sql
CREATE EXTERNAL TABLE dwd_favorites_log (
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string,  
    `app_time` string,
    `network` string, 
    `lng` string, 
    `lat` string, 
    `id` int, 
    `course_id` int, 
    `userid` int,
    `add_time` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_favorites_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_favorites_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.id') id,
    get_json_object(event_json,'$.kv.course_id') course_id,
    get_json_object(event_json,'$.kv.userid') userid,
    get_json_object(event_json,'$.kv.add_time') add_time,
    server_time
from dwd_base_event_log 
where dt='2021-05-05' and event_name='favorites';
```



##### 12. 点赞表

```sql
CREATE EXTERNAL TABLE dwd_praise_log (
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string,  
    `app_time` string,
    `network` string, 
    `lng` string, 
    `lat` string, 
    `id` string, 
    `userid` string, 
    `target_id` string,
    `type` string,
    `add_time` string,
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_praise_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_praise_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.id') id,
    get_json_object(event_json,'$.kv.userid') userid,
    get_json_object(event_json,'$.kv.target_id') target_id,
    get_json_object(event_json,'$.kv.type') type,
    get_json_object(event_json,'$.kv.add_time') add_time,
    server_time
from dwd_base_event_log
where dt='2021-05-05' and event_name='praise';
```



##### 13. 错误日志表

```sql
drop table if exists dwd_error_log;
CREATE EXTERNAL TABLE dwd_error_log (
    `mid_id` string,
    `user_id` string, 
    `version_code` string, 
    `version_name` string, 
    `lang` string, 
    `source` string, 
    `os` string, 
    `area` string, 
    `model` string,
    `brand` string, 
    `sdk_version` string, 
    `gmail` string, 
    `height_width` string,  
    `app_time` string,
    `network` string, 
    `lng` string, 
    `lat` string, 
    `errorBrief` string, 
    `errorDetail` string, 
    `server_time` string
)
PARTITIONED BY (dt string)
location '/warehouse/mall/dwd/dwd_error_log/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dwd_error_log
PARTITION (dt='2021-05-05')
select 
    mid_id,
    user_id,
    version_code,
    version_name,
    lang,
    source,
    os,
    area,
    model,
    brand,
    sdk_version,
    gmail,
    height_width,
    app_time,
    network,
    lng,
    lat,
    get_json_object(event_json,'$.kv.errorBrief') errorBrief,
    get_json_object(event_json,'$.kv.errorDetail') errorDetail,
    server_time
from dwd_base_event_log 
where dt='2021-05-05' and event_name='error';
```



#### 创建 DWS 层

以DWD层为基础，进行轻度的汇总。

##### 1. 每日活跃设备明细

```sql
create external table dws_uv_day (
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识', 
    `version_code` string COMMENT '程序版本号', 
    `version_name` string COMMENT '程序版本名', 
    `lang` string COMMENT '系统语言', 
    `source` string COMMENT '渠道号', 
    `os` string COMMENT '安卓系统版本', 
    `area` string COMMENT '区域', 
    `model` string COMMENT '手机型号', 
    `brand` string COMMENT '手机品牌', 
    `sdk_version` string COMMENT 'sdkVersion', 
    `gmail` string COMMENT 'gmail', 
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度'
)
partitioned by(dt string)
stored as parquet
location '/warehouse/mall/dws/dws_uv_day';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dws_uv_day 
partition(dt='2021-05-05')
select  
    mid_id,
    concat_ws('|', collect_set(user_id)) user_id,
    concat_ws('|', collect_set(version_code)) version_code,
    concat_ws('|', collect_set(version_name)) version_name,
    concat_ws('|', collect_set(lang))lang,
    concat_ws('|', collect_set(source)) source,
    concat_ws('|', collect_set(os)) os,
    concat_ws('|', collect_set(area)) area, 
    concat_ws('|', collect_set(model)) model,
    concat_ws('|', collect_set(brand)) brand,
    concat_ws('|', collect_set(sdk_version)) sdk_version,
    concat_ws('|', collect_set(gmail)) gmail,
    concat_ws('|', collect_set(height_width)) height_width,
    concat_ws('|', collect_set(app_time)) app_time,
    concat_ws('|', collect_set(network)) network,
    concat_ws('|', collect_set(lng)) lng,
    concat_ws('|', collect_set(lat)) lat
from dwd_launch_log
where dt='2021-05-05'
group by mid_id;
```

##### 2. 每周活跃设备明细

```sql
create external table dws_uv_week ( 
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识', 
    `version_code` string COMMENT '程序版本号', 
    `version_name` string COMMENT '程序版本名', 
    `lang` string COMMENT '系统语言', 
    `source` string COMMENT '渠道号', 
    `os` string COMMENT '安卓系统版本', 
    `area` string COMMENT '区域', 
    `model` string COMMENT '手机型号', 
    `brand` string COMMENT '手机品牌', 
    `sdk_version` string COMMENT 'sdkVersion', 
    `gmail` string COMMENT 'gmail', 
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度',
    `monday_date` string COMMENT '周一日期',
    `sunday_date` string COMMENT  '周日日期' 
) COMMENT '活跃用户按周明细'
PARTITIONED BY (`wk_dt` string)
stored as parquet
location '/warehouse/mall/dws/dws_uv_week/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dws_uv_week partition(wk_dt)
select  
    mid_id,
    concat_ws('|', collect_set(user_id)) user_id,
    concat_ws('|', collect_set(version_code)) version_code,
    concat_ws('|', collect_set(version_name)) version_name,
    concat_ws('|', collect_set(lang)) lang,
    concat_ws('|', collect_set(source)) source,
    concat_ws('|', collect_set(os)) os,
    concat_ws('|', collect_set(area)) area, 
    concat_ws('|', collect_set(model)) model,
    concat_ws('|', collect_set(brand)) brand,
    concat_ws('|', collect_set(sdk_version)) sdk_version,
    concat_ws('|', collect_set(gmail)) gmail,
    concat_ws('|', collect_set(height_width)) height_width,
    concat_ws('|', collect_set(app_time)) app_time,
    concat_ws('|', collect_set(network)) network,
    concat_ws('|', collect_set(lng)) lng,
    concat_ws('|', collect_set(lat)) lat,
    date_add(next_day('2021-05-05','MO'),-7),
    date_add(next_day('2021-05-05','MO'),-1),
    concat(date_add( next_day('2021-05-05','MO'),-7), '_' , date_add(next_day('2021-05-05','MO'),-1) 
)
from dws_uv_day 
where dt>=date_add(next_day('2021-05-05','MO'),-7) and dt<=date_add(next_day('2021-05-05','MO'),-1) 
group by mid_id;
```

##### 3. 每月活跃设备明细

```sql
create external table dws_uv_month ( 
    `mid_id` string COMMENT '设备唯一标识',
    `user_id` string COMMENT '用户标识', 
    `version_code` string COMMENT '程序版本号', 
    `version_name` string COMMENT '程序版本名', 
    `lang` string COMMENT '系统语言', 
    `source` string COMMENT '渠道号', 
    `os` string COMMENT '安卓系统版本', 
    `area` string COMMENT '区域', 
    `model` string COMMENT '手机型号', 
    `brand` string COMMENT '手机品牌', 
    `sdk_version` string COMMENT 'sdkVersion', 
    `gmail` string COMMENT 'gmail', 
    `height_width` string COMMENT '屏幕宽高',
    `app_time` string COMMENT '客户端日志产生时的时间',
    `network` string COMMENT '网络模式',
    `lng` string COMMENT '经度',
    `lat` string COMMENT '纬度'
) COMMENT '活跃用户按月明细'
PARTITIONED BY (`mn` string)
stored as parquet
location '/warehouse/mall/dws/dws_uv_month/';
```

```sql
set hive.exec.dynamic.partition.mode=nonstrict;
insert overwrite table dws_uv_month partition(mn)
select  
    mid_id,
    concat_ws('|', collect_set(user_id)) user_id,
    concat_ws('|', collect_set(version_code)) version_code,
    concat_ws('|', collect_set(version_name)) version_name,
    concat_ws('|', collect_set(lang)) lang,
    concat_ws('|', collect_set(source)) source,
    concat_ws('|', collect_set(os)) os,
    concat_ws('|', collect_set(area)) area, 
    concat_ws('|', collect_set(model)) model,
    concat_ws('|', collect_set(brand)) brand,
    concat_ws('|', collect_set(sdk_version)) sdk_version,
    concat_ws('|', collect_set(gmail)) gmail,
    concat_ws('|', collect_set(height_width)) height_width,
    concat_ws('|', collect_set(app_time)) app_time,
    concat_ws('|', collect_set(network)) network,
    concat_ws('|', collect_set(lng)) lng,
    concat_ws('|', collect_set(lat)) lat,
    date_format('2021-05-05','yyyy-MM')
from dws_uv_day
where date_format(dt,'yyyy-MM') = date_format('2021-05-05','yyyy-MM')
group by mid_id;
```



#### 创建 ADS 层

统计分析的结果，为更高级的服务提供数据。

##### 1.  统计出具体的日活、周活、月活

```sql
create external table ads_uv_count ( 
    `dt` string COMMENT '统计日期',
    `day_count` bigint COMMENT '当日用户数量',
    `wk_count`  bigint COMMENT '当周用户数量',
    `mn_count`  bigint COMMENT '当月用户数量',
    `is_weekend` string COMMENT 'Y,N是否是一周的最后一天(星期天),用于得到本周最终结果',
    `is_monthend` string COMMENT 'Y,N是否是一个月的最后一天(月末),用于得到本月最终结果' 
) COMMENT '活跃设备数'
row format delimited fields terminated by '\t'
location '/warehouse/mall/ads/ads_uv_count/';
```

```sql
insert into table ads_uv_count 
select  
  '2021-05-05' dt,
   daycount.ct,
   wkcount.ct,
   mncount.ct,
   if(date_add(next_day('2021-05-05','MO'),-1)='2021-05-05','Y','N') ,
   if(last_day('2021-05-05')='2021-05-05','Y','N') 
from 
    (
       select  
          '2021-05-05' dt,
           count(*) ct
       from dws_uv_day
       where dt='2021-05-05'  
    ) daycount join 
    ( 
       select  
         '2021-05-05' dt,
         count (*) ct
       from dws_uv_week
       where wk_dt=concat(date_add(next_day('2021-05-05','MO'),-7),'_' ,date_add(next_day('2021-05-05','MO'),-1) )
    ) wkcount on daycount.dt=wkcount.dt
    join 
    ( 
       select  
         '2021-05-05' dt,
         count (*) ct
       from dws_uv_month
       where mn=date_format('2021-05-05','yyyy-MM')  
    )mncount on daycount.dt=mncount.dt;
```

#### 导出 ADS 层数据

使用 sqoop 导出到 MySQL 。