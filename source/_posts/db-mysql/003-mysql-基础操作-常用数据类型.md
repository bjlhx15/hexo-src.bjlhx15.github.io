---
title: 003-mysql-基础操作-常用数据类型
categories:
  - db-mysql
abbrlink: 93944b35
date: 2020-02-05 21:19:44
tags:
---

摘要：mysql 常用数据类型:数值类型、字符串类型、日期类型

<!--more-->

更多：[https://www.cnblogs.com/bjlhx/category/998292.html](https://www.cnblogs.com/bjlhx/category/998292.html)

# 数值类型
MySQL中支持5种整数类型，其实很大程度上相同的，只是存储值的大小范围不同而已。其次是浮点类型float和double类型
```
tinyint：占用1个字节，相对于java中的byte
smallint：占用2个字节，相对于java中的short
int：占用4个字节，相对于java中的int【推荐直接使用这个，已于扩展，降低转换】
bigint：占用8个字节，相对于java中的long【自增主键推荐使用】
float：4字节单精度浮点类型，相对于java中的float
double：8字节双精度浮点类型，相对于java中的double
```
# 字符串类型
```
char()------定长字符串，最长255个字符。定长会浪费空间
varchar()----变长(不定长)字符串，最长不超过 65535个字节,一般超过255个字节，会使用text类型. 不定长节省空间,剩余空间会留给别的数据使用
text--------长文本类型,最长65535个字节
```

## varchar
- VARCHAR用于存储可变长字符串，它比定长类型更节省空间。
- VARCHAR使用额外1或2个字节存储字符串长度。列长度小于255字节时，使用1字节表示，否则使用2字节表示。
- VARCHAR存储的内容超出设置的长度时，内容会被截断。

## char
- CHAR是定长的，根据定义的字符串长度分配足够的空间。
- CHAR会根据需要使用空格进行填充方便比较。
- CHAR适合存储很短的字符串，或者所有值都接近同一个长度。
- CHAR存储的内容超出设置的长度时，内容同样会被截断。

## 使用策略：
- 对于经常变更的数据来说，CHAR比VARCHAR更好，因为CHAR不容易产生碎片。
- 对于非常短的列，CHAR比VARCHAR在存储空间上更有效率。
- 使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存。
- 尽量避免使用TEXT/BLOB类型，查询时会使用临时表，导致严重的性能开销。

## 总结： char、varchar、text都可以表示字符串类型，其区别在于：
1. char在保存数据时, 如果存入的字符串长度小于指定的长度n,后面会用空格补全。
2. varchar和text保存数据时, 按数据的真实长度存储, 剩余的空间可以留给别的数据用.
3. char会造成空间浪费(不足指定长度的会用空格补全), 但是由于不需要计算数据的长度, 因此速度更快。（即浪费空间、节约时间）
4. varchar和text但是节省了空间, 但是存储的速度不如char快(因为要计算数据的实际长度)


# 日期类型
## 定义
```
date：年月日
time：时分秒
datetime：年月日 时分秒
	5.6后：使用：DEFAULT NOW()、DEFAULT CURRENT_TIMESTAMP设置
timestamp：时间戳，与datetime存储相同的数据。
	1、插入记录时，时间戳字段包含DEFAULT CURRENT_TIMESTAMP，如插入记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
  	2、更新记录时，时间戳字段包含ON UPDATE CURRENT_TIMESTAMP，如更新记录时未指定具体时间数据则将该时间戳字段值设置为当前时间
	CURRENT_TIMESTAMP表示使用CURRENT_TIMESTAMP()函数来获取当前时间，类似于NOW()函数
```
## 表达方式

```
日期类型         存储空间       日期格式                                      日期范围
datetime		8 bytes   YYYY-MM-DD HH:MM:SS   1000-01-01 00:00:00 ~ 9999-12-31 23:59:59
timestamp		4 bytes   YYYY-MM-DD HH:MM:SS   1970-01-01 00:00:01 ~ 2037-12-31 23:59:59
date			3 bytes   YYYY-MM-DD            1000-01-01 ~ 9999-12-31
```

### datetime和timestamp的区别用法：
1.datetime 的日期范围比较大；如果有1970年以前的数据还是要用datetime.但是timestamp 所占存储空间比较小。(节省空间)
2.timestamp 类型的列还有个特性：默认情况下，在 insert, update 数据时，timestamp 列会自动以当前时间（CURRENT_TIMESTAMP）填充/更新。(可设置)
3.timestamp比较受时区timezone的影响以及MYSQL版本和服务器的SQL MODE的影响.

使用一个常用的格式集的任何一个，你可以指定DATETIME、DATE和TIMESTAMP值：
'YYYY-MM-DD HH:MM:SS'或'YY-MM-DD HH:MM:SS'格式的一个字符串,允许一种"宽松"的语法:任何标点可用作在日期部分和时间部分之间的分隔符。例如，'98-12-31 11:30:45'、'98.12.31 11+30+45'、'98/12/31 11*30*45'和'98@12@31 11^30^45'是等价的。

- timestamp最大表示2038年，而datetime范围是1000~9999
- timestamp在插入数、修改数据时，可以自动更新成系统当前时间

```
TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  在创建新记录和修改现有记录的时候都对这个数据列刷新。
TIMESTAMP DEFAULT CURRENT_TIMESTAMP  在创建新记录的时候把这个字段设置为当前时间，但以后修改时，不再刷新它。
TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  在创建新记录的时候把这个字段设置为0，以后修改时刷新它。
TIMESTAMP DEFAULT ‘yyyy-mm-dd hh:mm:ss’ ON UPDATE CURRENT_TIMESTAMP  在创建新记录的时候把这个字段设置为给定值，以后修改时刷新它
```
```sql
create table tb_date(
	id int(20) AUTO_INCREMENT not null COMMENT '主键',
	name varchar(20) DEFAULT null COMMENT '名称',
	f_date date default null COMMENT '日期测试',
	f_datetime datetime default null COMMENT '日期时间测试',
	f_createtime TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '时间戳',
	f_updatetime TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '数据库级别更新时间戳',
	PRIMARY key (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8 DEFAULT COLLATE=utf8_unicode_ci  COMMENT '日期测试表';
```