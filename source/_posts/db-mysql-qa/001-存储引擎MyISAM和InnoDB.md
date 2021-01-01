---
title: 001-存储引擎MyISAM和InnoDB
categories:
  - db-mysql-qa
abbrlink: b39e95d3
date: 2020-02-19 08:49:48
---

摘要：存储引擎是数据库管理系统用来从数据库创建、读取和更新数据的软件模块。MySQL中有两种类型的存储引擎：事务性和非事务性。
对于MySQL 5.5及更高版本，默认的存储引擎是InnoDB。在5.5版本之前，MySQL的默认存储引擎是MyISAM。
<!-- more -->

# 存储引擎

## 查看安装mysql版本的支持的
``` sql
show engines;
-- Enginne      Supports     Trabsactions    XA  Savepoints  Comment
-- FEDERATED	    NO				                                Federated MySQL storage engine
-- MRG_MYISAM	    YES	        NO	        NO	   NO	Collection of identical MyISAM tables
-- MyISAM	        YES	        NO	        NO	   NO	MyISAM storage engine
-- BLACKHOLE	    YES	        NO	        NO	   NO	/dev/null storage engine (anything you write to it disappears)
-- CSV	            YES	        NO	        NO	   NO	CSV storage engine
-- MEMORY	        YES	        NO	        NO	   NO	Hash based, stored in memory, useful for temporary tables
-- ARCHIVE	        YES	        NO	        NO	   NO	Archive storage engine
-- InnoDB	        DEFAULT	    YES	        YES	   YES	Supports transactions, row-level locking, and foreign keys
-- PERFORMANCE_SCHEMA	YES	    NO	        NO	   NO	Performance Schema
```
说明
　　engine：引擎名称。
　　suppot：是否支持。
　　comment：说明。
　　transactions：是够支持事务。
　　xa：是否支持XA事务。
　　savepoints：是否支持保存savepoints之间的内容。

# 常用引擎（常用的MyISAM和InnoDB）

## MyISAM
mysql5.5之前默认的存储引擎，由MYD和MYI组成。
查看数据库的data目录/数据库名称/，在查找相对应的表名。frm,myd,myi这三个结尾的文件。
　　.myd　　//数据库文件
　　.myi　　//索引文件  又叫非聚集索引
 
### 特性：
- 并发性与锁级别-表解锁
- 支持全文索引
- 支持数据压缩   命令：进入到mysql的bin文件夹， .\myisampack.exe -b -f "需要压缩的test.MYI地址" （此命令实在Windows运行）。
- 运行完以后，会出现一个以OLD结尾的文件，删除OLD可能会出现问题。需要恢复 CHECK table 表名，REPAIR table 表名      

### 适用场景：
- 非事务的类型
- 只读类应用，读取数据的速度快
- 空间类型（坐标，空间函数）

## innodb
　　mysql5.5以后默认的存储引擎，innodb_file_per_table  on：表示独立表空间，OFF：表示系统表空间。5.6之前是系统表空间，之后为独立表空间。
　　独立表空间：.frm .ibd  存储数据+索引。 可以通过 optimize table 表名 .ibd收缩数据文件，同时可以向多个文件刷新数据。
　　系统表空间：.frm是放在数据库的文件下的。 .ibdata1是放在data文件夹下的，表公用的，会产生IO的瓶颈。 系统表空间无法简单的收缩文件大小
　　建议使用独立表空间。

### 特性：
是一种事务性存储引擎。完全支持事务的ACID特性。执行行级锁，并发程度高。Redo Log和Undo Log。
### 适用场景：
大多数的OLTP应用。

```
对比项  MyISAM  InnoDB
外键    不支持      支持
事务    不支持      支持
锁      表锁       行锁
关注点  性能        事务
表空间  小          大
缓存    缓存索引    缓存索引和数据
场景    不合适高并发 适合高并发
```

## CSV
　　数据以文本方式存储，表的字段不能为空，不能有主键。
　　.frm , .csv数据的内容， .csm存储表的元数据 。
　　使用文本编辑器可以直接编辑.csv数据，然后保存，在数据库里面执行flush  tables;
　　注：要在最后一行数据回车一下，要不然最后一条数据不展示。
　　在excel里面操作提示兼容性问题，无法操作成功，编辑完以后修复一下，可能是excel版本的问题吧。
### 特点：
以CSV格式进行数据存储，所有列的字段都不能为null，不支持索引，可以对数据文件在线编辑。

## Archive
　　以zlib对表数据进行压缩，磁盘I/O更少，数据存储在.ARZ。
　　.frm , .ARZ数据的内容。
### 特点：
　　　　只支持insert和select操作，只允许在自增ID列上加索引。
### 使用场景：
　　　　日志和数据采集应用

## Memory
　　在data文件夹里面只有一个frm。
　　数据保存在内存中，支持hash索引和BTree索引，所有字段都是固定的长度varchar(10)=char(10),不支持Blog和Text等字段
　　使用表级索，最大有max_heap_table_size 决定。 重启会丢失数据。
　　在系统使用临时表的时候，超过限制会使用MyISAM，未超过的时候使用Memory
　　临时表：在同一个session（会话）里面，才能使用。重启服务会丢失数据。
　　应用场景：mysql后台服务使用Memory

## Federated
　　访问远程的数据库表，本地只保存数据库结构和连接信息，数据保存在远程的服务器中。在本地只保存.frm
　　默认不是开启的引擎，在my.ini  增加 federated=1，重启。
　　只能用命令创建。create table ‘aaa’(里面的字段，要和连接的服务器一样) engine=federated connection='mysql://用户名:密码@地址:IP/数据库名/表名'