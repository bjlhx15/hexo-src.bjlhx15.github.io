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

总结： char、varchar、text都可以表示字符串类型，其区别在于：
1. char在保存数据时, 如果存入的字符串长度小于指定的长度n,后面会用空格补全。
2. varchar和text保存数据时, 按数据的真实长度存储, 剩余的空间可以留给别的数据用.
3. char会造成空间浪费(不足指定长度的会用空格补全), 但是由于不需要计算数据的长度, 因此速度更快。（即浪费空间、节约时间）
4. varchar和text但是节省了空间, 但是存储的速度不如char快(因为要计算数据的实际长度)

# 日期类型
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
- timestamp最大表示2038年，而datetime范围是1000~9999
- timestamp在插入数、修改数据时，可以自动更新成系统当前时间
