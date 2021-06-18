---
title: Hbase Shell 操作
date: 2015-11-05 15:49:44
tags:
 - Hbase
category: 
 - Big Data
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 帮助文档:
```shell
Group name: general
Commands: status, version, whoami

Group name: ddl
Commands: alter, alter_async, alter_status, create, describe, disable, disable_all, drop, drop_all, enable, enable_all, exists, is_disabled, is_enabled, list, show_filters

Group name: dml
Commands: count, delete, deleteall, get, get_counter, incr, put, scan, truncate, truncate_preserve

Group name: tools
Commands: assign, balance_switch, balancer, close_region, compact, flush, hlog_roll, major_compact, move, split, unassign, zk_dump

Group name: replication
Commands: add_peer, disable_peer, enable_peer, list_peers, list_replicated_tables, remove_peer, start_replication, stop_replication

Group name: snapshot
Commands: clone_snapshot, delete_snapshot, list_snapshots, restore_snapshot, snapshot

Group name: security
Commands: grant, revoke, user_permission
```
### 示例：
#### put : 循环插入记录
```shell
for i in 'a'..'z' do for j in 'a'..'z' do put 't1', "row#{i}#{j}", "F:#{j}", "#{j}" end end
```
#### alter : 修改表结构
```shell
disable 't1'
alter 't1',{NAME => 'F'}
enable 't1'
```
#### scan: 扫描前10条记录
```shell
scan 't1', {LIMIT => 10}
```
#### 扫描F:a-F:c的记录
```shell
scan 't1', {COLUMNS => ['F:a','F:c']}
```
#### 扫描行健范围
```shell
scan 't1',{STARTROW => 'row-aa', STOPROW => 'row-az'}
```
#### 删除某列
```shell
deleteall 't1','row1','f:A'
```
