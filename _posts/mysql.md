---
title: mysql
tags:
  - MySQL
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-08-11 23:17:11
thumbnail:
---


mysql 架构
连接器（connectors）
系统管理和控制工具（services & utilies）
连接池（connection pool）
sql接口（sql interface）
解析器（parser）
	1. 词法分析 分词，获取子句
	2. 语法分析 检查是否符合sql92语法
	形成语法树
查询优化器
	1. 索引优化
	2. 多表关联 小表驱动大表
	3. where条件，从左到右，选择过滤力度最大的条件先执行
查询缓存（cache buffer）
	执行的sql hash 作为key， 结果是value
	当数据改变，清除缓存
存储引擎
	5.6 默认 innoDB
	MyISAM 查询和插入速度快，但不支持事务，行锁，外键；
	InnoDB 支持事务，行锁，外键；

执行流程
	1. 连接管理模块
	2. 连接进程模块
	3. 用户模块
	4. 命令分发器
	5. 记录日志
	6. 命令解析器
	7. 查询优化器
	8. 访问控制 grant
	9. 表管理模块
	10. 存储引擎接口

物理架构
	日志文件和索引文件 /var/lib/mysql
	日志用顺序IO方式存储，数据使用随机IO方式存储
	顺序IO和随机IO的优劣
	
	日志分类
		错误日志
		二进制日志（bin log）
			记录数据变化所有的ddl和dml，但不包括select。作用数据备份，恢复，同步
		通用查询日志，性能不高，生产中不开启
		慢查询日志，默认关闭。
			slow_query_log,slow_query_time,
		redo log
		undo log
		中继日志 relay log
		看日志开启情况 show variables like 'log_%'
	数据文件
		看数据文件 show variables like 'datadir%'
		InnoDB
			.frm文件 表结构信息
			.ibd 独享表空间
			.ibdata 共享表空间
		MyISAM
			.frm文件 表结构信息
			.myd文件 存储表数据
			.myi文件 存储索引信息
			
索引
	优势
		查询快
		降低排序成本
	劣势
		增删改慢
		
索引使用场景
	1. 主键唯一索引
	2. 频繁查询的字段
	3. 关联查询，关联字段建索引
	4. 排序字段要建索引
	5. 索引覆盖
	6. 分组统计
不需要索引的场景
	1. 记录太少
	2. 频繁更新

组合索引
	优势： 
		省空间
		容易形成索引覆盖
	最左前缀原则：索引从左向右直到遇到范围查询，索引中断


查看执行计划 explain 
	1. id 大的先执行，相同的顺序执行
	2. select_type 能够看出sql语句使用了什么结构
		premary
		subquery
		dependent subquery
		union
		dependent union 
		union result 
		derived 
	3. type
		system  
		const 唯一索引或主键索引
		eq_ref 主键关联或唯一索引关联
		ref 非唯一索引
		range 范围索引，前缀索引
		all 没有索引
		index 索引覆盖
	4. possible_keys
		可能用到的索引
	5. key
		优化器使用的索引
	6. 执行计划估算的扫描行数
	7. extra
		null 表示索引使用的好
		using temporary 使用了临时表， distinct
		using filesort 文件排序
		using index 索引覆盖
		using where 没有使用索引，在server层过滤
		using index condition 索引下推ICP
		
索引失效	
	索引上计算
	组合索引开始索引缺失
	使用!=
	主键判空
	like `%a`
	字符串没有使用单引号
	or
	
	
锁
	悲观锁
		表锁
		元数据锁（meta data lock）
		意向锁
		行锁
			共享锁
			排他写锁
			锁定范围
				记录锁record locks 主键指定
				间隙锁 gap locks 锁定记录前 记录中 记录后的行
				next-key 记录锁+间隙锁
			功能
				共享读锁 lock in share mode
				排他写锁
					1. dml 自动加
					2. for update
			两阶段锁：加锁阶段 解锁阶段
			没索引不支持行锁
			主键锁定一行，不产生间隙锁
			主键范围锁定，产生间隙锁
			
	死锁
		互相等待对方资源

事务
	由存储引擎实现
	

innodb 架构
	buffer pool
	
	
	redo log
	
	double write
	
	check point
	
	










			
			
			
			