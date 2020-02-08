---
title: 003-mysql-基础操作-操作库表字段索引信息
categories:
  - db-mysql
abbrlink: 6c082091
date: 2020-02-05 21:19:44
tags:
---

摘要：操作库表字段索引信息

<!--more-->

其他：[https://www.cnblogs.com/bjlhx/category/998292.html](https://www.cnblogs.com/bjlhx/category/998292.html)

数值类型
MySQL中支持5种整数类型，其实很大程度上相同的，只是存储值的大小范围不同而已。
其次是浮点类型float和double类型

tinyint：占用1个sss字节，相对于java中的byte
smallint：占用2个字节，相对于java中的short
int：占用4个字节，相对于java中的int
bigint：占用8个字节，相对于java中的long
float：4字节单精度浮点类型，相对于java中的float
double：8字节双精度浮点类型，相对于java中的double
1
2
3
4
5
6
字符串类型
char()------定长字符串，最长255个字符。定长会浪费空间
varchar()----变长(不定长)字符串，最长不超过 65535个字节,一般超过255个字节，会使用text类型. 不定长节省空间,剩余空间会留给别的数据使用

长文本类型
text--------最长65535个字节

总结： char、varchar、text都可以表示字符串类型，其区别在于：
(1)char在保存数据时, 如果存入的字符串长度小于指定的长度n,后面会用空格补全。
(2)varchar和text保存数据时, 按数据的真实长度存储, 剩余的空间可以留给别的数据用.
(3)char会造成空间浪费(不足指定长度的会用空格补全), 但是由于不需要计算数据的长度, 因此速度更快。（即浪费空间、节约时间）
(4)varchar和text但是节省了空间, 但是存储的速度不如char快(因为要计算数据的实际长度)

日期类型
1、date：年月日
2、time：时分秒
3、datetime：年月日 时分秒
4、timestamp：时间戳，与datetime存储相同的数据。
timestamp最大表示2038年，而datetime范围是1000~9999
timestamp在插入数、修改数据时，可以自动更新成系统当前时间

字段约束
创建表时, 除了要给每个列指定对应的数据类型, 有时也需要给列添加约束。常见的约束有：主键约束、唯一约束、非空约束、外键约束。

主键(primary key)
主键是数据表中，一行记录的唯一标识。比如学生的编号，人的身份证号;
当主键为数值时，为了方便维护，可以设置主键为自增（auto_increment）

唯一(unique)
保证所约束的列必须是唯一的，即不能重复出现，例如：用户注册时，保存的用户名不可以重复。

非空(not null)
保证所约束的列必须是不为空的，即在插入记录时，该列必须要赋值，例如：用户注册时，保存的密码不能为空。
创建user表, 指定密码不能为空

create table user(
	id int primary key auto_increament,
	username varchar(50) unique,
	password varchar(50) not null,
	...
);
1
2
3
4
5
6
外键
外键是用于表和表之间关系的列
————————————————
版权声明：本文为CSDN博主「奋斗中的小码农」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_44458228/article/details/87861584