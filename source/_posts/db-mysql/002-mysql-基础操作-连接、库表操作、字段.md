---
title: 002-mysql-基础操作-连接、库表操作、字段
categories:
  - db-mysql
abbrlink: 8d83348a
date: 2020-02-05 20:33:40
tags:
---

摘要：mysql 库表操作

<!--more-->

其他：[https://www.cnblogs.com/bjlhx/category/998292.html](https://www.cnblogs.com/bjlhx/category/998292.html)

# 连接登录
```bash
# 帮助
mysql -?
# 常用参数
# -h 表示服务器名字。localhost表示本地 可以省略
# -P 端口
# -u 表示用户名
# -p 表示密码。直接在-p后面输入密码即可，中间不能有空格。 新版本不能指定，需要手工输入
# -D 指定数据库，权限不够时。
```
连接 
``` bash
mysql -hlocalhost -uroot -p123456
```
进入后，语句以 ; 结尾

# 常用命令

## 库
- 查看所有数据库: `show databases;`
- 进入数据库: `use 库名;`
- 查看数据库使用端口:`show variables like 'port';`
- 数据库编码:`show variables like 'character%';`
```
character_set_client      为客户端编码方式；
character_set_connection  为建立连接使用的编码；
character_set_database    为数据库的编码；
character_set_results     为结果集的编码；
character_set_server      为数据库服务器的编码；
```
- 查看数据库最大连接数:`show variables like '%max_connections%';`
- 查看数据库当前连接数，并发数:`show status like 'Threads%';`
```
Threads_cached : 代表当前此时此刻线程缓存中有多少空闲线程。
Threads_connected :代表当前已建立连接的数量，因为一个连接就需要一个线程，所以也可以看成当前被使用的线程数。
Threads_created :代表从最近一次服务启动，已创建线程的数量。
Threads_running :代表当前激活的（非睡眠状态）线程数。并不是代表正在使用的线程数，有时候连接已建立，但是连接处于sleep状态，这里相对应的线程也是sleep状态。
```


## 表
- 查看正在使用的数据库：`select database();`
- 查看库中所有的表: `show tables;`
- 查看表结构: `desc/describe 表名`;或：`show columns from table_name [from database_name];`
- 查看表-列结构: `desc/describe 表名 列名;`
- 查看表生成的DDL sql语句:`show create table tname;`
- 查看库表信息：`SHOW TABLE STATUS [{FROM | IN} db_name] [LIKE 'pattern' | WHERE expr]`
``` text 字段含义
返回列	说明
Name	表名称
Engine	表的存储引擎
Version	版本
Row_format	行格式
Rows	表中的行数。对于非事务性表，这个值是精确的，对于事务性引擎，这个值通常是估算的。
Avg_row_length	平均每行的大下（字节）
Data_length	表的数据量(单位：字节)
Max_data_length	表可以容纳的最大数据量
Index_length	索引占用磁盘的空间大小
Data_free	标识已分配，但现在未使用的空间，并且包含了已被删除行的空间。
Auto_increment	下一个Auto_increment的值
Create_time	表的创建时间
Update_time	表的最近更新时间
Check_time	最近一次使用 check table 或myisamchk工具检查表的时间
Collation	表的字符集和字符排序规则
Checksum	如果启用，则对整个表的内容计算时的校验和
Create_options	表创建时的其它
Comment	表在创建是添加的注释说明
```
或者 `select * from information_schema.tables where TABLE_SCHEMA='库名'`

## 库、表
- 显示数据库状态：`status;`
- 退出/断开连接:`exit;或quit;或 \q;或ctrl+c;`

# 数据库操作

## 库操作

### 建库
字符集： 1.若没有显式设置，则自动使用服务器级的配置 ； 2.显式设置：在创建库时指定 ，如下
```sql
create database if not  exists  库名 default charset utf8 collate utf8_general_ci;
```
查看 库信息
```sql
select * from information_schema.schemata where schema_name = 'test_sql'; 
-- def	test_sql	utf8	utf8_general_ci	
```
### 字段约束
创建表时, 除了要给每个列指定对应的数据类型, 有时也需要给列添加约束。常见的约束有：主键约束、唯一约束、非空约束、外键约束。

#### 主键(primary key)【主键索引，聚集索引】
主键是数据表中，一行记录的唯一标识。比如学生的编号，人的身份证号;
当主键为数值时，为了方便维护，可以设置主键为自增（auto_increment）

#### 唯一(unique)
保证所约束的列必须是唯一的，即不能重复出现，例如：用户注册时，保存的用户名不可以重复。

> - 约束 全称完整性约束，它是关系数据库中的对象，用来存放插入到一个表中一列数据的规则，用来确保数据的准确性和一致性。
> - 索引 数据库中用的最频繁的操作是数据查询，索引就是为了加速表中数据行的检索而创建的一种分散的数据结构。可以把索引类比成书的目录，有目录的肯定比没有目录的书，更方便查找。
> - 唯一约束 保证在一个字段或者一组字段里的数据都与表中其它行的对应数据不同。和主键约束不同，唯一约束允许为 NULL，只是只能有一行。
> - 唯一索引 不允许具有索引值相同的行，从而禁止重复的索引或键值。

在mysql 中唯一约束 与 唯一索引 一样。

```sql
-- 唯一约束
create table t1(
	id int primary key AUTO_INCREMENT  COMMENT '主键约束',
	username varchar(50) UNIQUE comment '唯一约束',
	password varchar(50) not null comment '非空约束',
	address varchar(50) default null comment '默认为空'
);
```
```sql
-- 建立唯一索引
create table t2(
	id int primary key AUTO_INCREMENT  COMMENT '主键约束',
	username varchar(50) default null comment '约束',
	password varchar(50) not null comment '非空约束',
	address varchar(50) default null comment '默认为空'
);
ALTER  TABLE  `t2`  ADD  UNIQUE (`username` ) ;
```
查看 ddl t1,t2均为
```sql
CREATE TABLE `t1` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键约束',
  `username` varchar(50) COLLATE utf8_bin DEFAULT NULL COMMENT '唯一约束',
  `password` varchar(50) COLLATE utf8_bin NOT NULL COMMENT '非空约束',
  `address` varchar(50) COLLATE utf8_bin DEFAULT NULL COMMENT '默认为空',
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
```

#### 非空(not null)
保证所约束的列必须是不为空的，即在插入记录时，该列必须要赋值，例如：用户注册时，保存的密码不能为空。
创建user表, 指定密码不能为空

#### 外键
外键是用于表和表之间关系的列

#### 示例
```sql
create table user(
	id int primary key auto_increament  COMMENT '主键约束',
	username varchar(50) unique comment '唯一约束',
	password varchar(50) not null comment '非空约束',
	address varchar(50) default null comment '默认为空',
	...
);
```

### 删库
```sql
drop database 库名;
```

## 表、字段操作

### 建表
1. 是否需要删除
```sql
drop table if exists 表名;
```
2. 创建【推荐】
create table 表名 (字段设定列表) default charset=utf8 default collate=utf8_bin; 
  - 字符集：1.若没有显式设置，则自动使用数据库级的配置 ；2.显式设置：在创建表时指定 
  - 字段列表类型：查看下文
  - 成功后，可以使用`desc 表名`或`show create table 表名`查看
  
```sql
CREATE TABLE IF NOT EXISTS `user`(
   `user_id` INT(20) AUTO_INCREMENT,
   `user_name` VARCHAR(100) NOT NULL,
   `user_age` int NOT NULL,
   `regist_date` DATE,
   PRIMARY KEY ( `user_id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8 DEFAULT COLLATE=utf8_unicode_ci  COMMENT '用户信息表';
```


### 删表
```sql
drop table if exists 表名;
```

### 修改表名

```sql
alter table 表名 rename [TO|AS] 新表名;
-- 或
rename table 表名 to 新表名;
```

### 表字段操作

#### 改表-加字段
```sql
alter table table_name add COLUMN new_name VARCHAR(20) DEFAULT NULL;
```

#### 改表-删段名
```sql
alter table user drop column old_name;
```

#### 改表-改字段名
```sql
alter table table_name change column old_name new_name varchar(255) default NULL comment '注释';
```
#### 改表-改字段类型或大小
```sql
alter table table_name modify column column1  decimal(10,1) DEFAULT NULL COMMENT '注释';
```

# 索引

参看：[012-MySQL 索引添加以及优化说明](https://www.cnblogs.com/bjlhx/p/11953939.html)


