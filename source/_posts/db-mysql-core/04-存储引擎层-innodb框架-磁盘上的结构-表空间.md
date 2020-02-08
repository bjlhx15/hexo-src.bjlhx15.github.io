---
title: 04-存储引擎层-innodb框架-磁盘上的结构-表空间
categories:
  - db-mysql-core
abbrlink: e9a15921
date: 2020-02-02 07:30:44
tags:
---

摘要：
物理存储， InnoDB 里的所有表数据，都以索引（聚簇索引+二级索引）的形式存储起来，索引包含了表数据。
<!--more-->

# 磁盘上的结构

磁盘中的结构分为两大类：表空间和重做日志。

- 表空间：分为系统表空间(MySQL 目录的 ibdata1 文件)，临时表空间，常规表空间，Undo 表空间以及 file-per-table 表空间(MySQL5.7默认打开file_per_table 配置）。
系统表空间又包括了InnoDB数据字典，双写缓冲区(Doublewrite Buffer)，修改缓存(Change Buffer），Undo日志等。
- Redo日志：存储的就是 Log Buffer 刷到磁盘的数据。

## innodb整体文件结构

物理存储文件结构中黑框里的就是innodb。

``` text
ibdata1
ib_logfile0
ib_logfile1
```
有两个默认的日志文件，logfile0和logfile1，大小可以手工设定。

InnoDB 中用于存储数据的文件总共有两个部分，一是系统表空间文件，包括 ibdata1、 ibdata2 等文件，其中存储了 InnoDB 系统信息和用户数据库表数据和索引，是所有表公用的。另一部分是idb。

### InnoDB 表结构

InnoDB 与 MyISAM 不同，它在系统表空间存储数据字典信息，因此它的表不能像 MyISAM 那样直接拷贝数据表文件移动。

MySQL5.7 采用的文件格式是 Barracuda，它支持 COMPACT 和 DYNAMIC 这两种新的行记录格式。

创建表时可以通过 ROW_FORMAT 指定行记录格式，默认是 DYNAMIC。查看表信息，
``` sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_TABLES WHERE NAME='test_innodb/table15w'
或
SHOW TABLE STATUS from test_innodb like 'table15w'
```
```
*************************** 1. row ***************************
           Name: table15w
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 149641
 Avg_row_length: 52
    Data_length: 7880704
Max_data_length: 0
   Index_length: 0
      Data_free: 4194304
 Auto_increment: 150001
    Create_time: 2020-01-31 03:40:25
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 
```
InnoDB表使用上有一些限制，如一个表最多只能有64个辅助索引，一行大小不能超过65535等，组合索引不能超过16个字段等，一般应该不会突破限制，详细见 [innodb-restrictions](https://dev.mysql.com/doc/refman/5.7/en/innodb-restrictions.html)。

### InnoDB 表空间

表空间根据类型可以分为：系统表空间，File-Per-Table 表空间{即ibd}，常规表空间，Undo表空间，临时表空间等。

- 系统表空间：包含内容有数据字典，双写缓冲，修改缓冲以及undo日志，以及在系统表空间创建的表的数据和索引。

- 常规表空间：类似系统表空间，也是一种共享的表空间，可以通过 CREATE TABLESPACE 创建常规表空间，多个表可共享一个常规表空间，也可以修改表的表空间。
注意：必须删除常规表空间中的表后才能删除常规表空间。
``` sql
CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;
CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1;
CREATE TABLE t2 (c2 INT PRIMARY KEY) TABLESPACE ts1;
ALTER TABLE t2 TABLESPACE=innodb_file_per_table;

DROP TABLE t1;
DROP TABLESPACE ts1;
```
- File-Per-Table表空间：MySQL InnoDB新版本提供了 innodb_file_per_table 选项，每个表可以有单独的表空间数据文件(.ibd)，而不是全部放到系统表空间数据文件 ibdata1 中。在 MySQL5.7 中该选项默认开启。

- 其他表空间：其他表空间中Undo表空间存储的是Undo日志。除了存储在系统表空间外，Undo日志也可以存储在单独的Undo表空间中。
临时表空间则是非压缩的临时表的存储空间，默认是数据目录的 ibtmp1 文件，所有临时表共享，压缩的临时表用的是 File-Per-Table 表空间。


# innodb承上启下的结构

![](/images/post/db-mysql/005/innodb-jiegou.png)

Handler API是供mysql server层调用的，server层定义了一些接口，譬如insert、delete，具体怎么实现，是由每个存储引擎自己实现的。

以上图中间虚线为分界，上面的是逻辑层，每个访问都会产生事务，事务处理会产生锁（表锁、行锁），操作对象是表、索引、b+ tree。对数据页面的访问需要物理事务，为了读写一致性，需要读写锁（物理锁）。为了高效定位和管理“页”，需要用到文件管理系统。

这些都是基于逻辑的处理，再往下就是物理层。

在逻辑处理和磁盘文件之间，都是有一层缓存的，这里主要是日志缓冲区和innodb_buffer_pool。和其他常用的kafka、elasticsearch、rocksDB等等一样，要保持性能，必然都遵循相同或类似的规则，那就是写pageCache、顺序写磁盘，这是决定任何一个带存储功能的性能的关键点。

innodb_buffer_pool，未来能对性能起决定性作用的一个重要因子。决定读写速度的都是内存，只要要读的数据在内存里，它就比在磁盘上快。redis就是靠内存，mysql的数据缓存，就取决于innodb_buffer_pool。

缓冲层提供了高效的读写性能，再下面就是物理文件层了，是落到磁盘上的。

磁盘上重要的地方有REDO日志，和表数据（页）




