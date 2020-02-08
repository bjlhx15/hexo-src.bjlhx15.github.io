---
title: 05-存储引擎层-innodb框架-表空间-ibd
categories:
  - db-mysql-core
abbrlink: 20b74871
date: 2020-02-02 08:42:20
tags:
---

摘要：
innodb文件系统是由一些log和每个表的ibd（16K的整数倍）等文件组成的。内部结构如下。

<!--more-->

# 表空间-idb简述

ibd是存表数据的，那么在计算机里，所有的存储都是有最小存储单元的。

在磁盘上，最小的单元是扇区，一个扇区是512字节，操作系统中最小单元是块（block），最小单位是4K。而innodb也有自己的最小存储单元——页（page），一页是16K。

这意味着一个文件放到电脑上，哪怕它是空的，也要占用4K，它占用的空间永远是4K的整数倍。

可以去查看每个ibd文件的大小，它永远是16384（16k）的整数倍。

后续说明索引加载表数据的前提，mysql的数据最小是16K，也就是哪怕你只取一条，可能还不到1K，那么mysql也会取出16K的数据。

因为“页”是最小单位。“页”还决定了b+ tree在某个高度下，能存放的数据量，为什么一个表存2万数据，和存1500万数据，查询速度一样。

## 单个文件表空间-frm、ibd

如果想让两个表共用一个数据文件的话,共用一个ibdata，可以通过**innodb_file_per_table**控制。默认每个表一个。

创建了一个数据库test_innodb，并且创建了几个表，可以在数据目录看一下。
``` text
db.opt
table5hang.frm
table5hang.ibd
table15w.frm
table15w.ibd
table500w.frm
table500w.ibd
table1000w.frm
table1000w.ibd
```
当新建一个库时，首先文件系统上会多一个以库名命名的文件夹。里面有ibd、frm文件，每个表对应一个ibd文件。
- db.opt 保存了数据库test的默认字符集 utf8mb4 和校验方法utf8mb4_general_ci；
存储的是mysql的一些配置信息，如编码、排序的信息，如果在创建数据库时指定了一些非默认参数的话，也会存到该文件。

- 每个表会对应一个frm文件，一个ibd文件。

  - frm文件是表结构，无论在 MySQL 中选择了哪个存储引擎，所有的 MySQL 表都会在硬盘上创建一个 .frm 文件用来描述表的格式或者说定义； .frm 文件的格式在不同的平台上都是相同的。
    数据字典信息(InnoDB数据字典信息主要是存储在系统表空间ibdata1文件中，由于历史原因才在 t.frm 多保留了一份)
    
  - ibd是表的数据和索引，是每一个表独有的表空间，文件存储了当前表的数据和相关的索引数据。

独立的表空间文件之存储对应表的B+树数据、索引和插入缓冲等信息，其余信息还是存储在默认表空间中。

这个文件所存储的内容主要就是B+树（索引），一个表可以有多个索引，也就是在一个文件中，可以存储多个索引，而如果一个表没有索引的话，用来存储数据的被称为聚簇索引，也就是说这也是一个索引。

### 采用 File-Per-Table 的优缺点

- 优点：可以方便回收删除表所占的磁盘空间。如果使用系统表空间的话，删除表后空闲空间只能被 InnoDB 数据使用。TRUNCATE TABLE 操作会更快。
可以单独拷贝表空间数据到其他数据库(使用 transportable tablespace 特性)，可以更方便的观测每个表空间数据的大小。
- 缺点：fsync 操作需要作用的多个表空间文件，比只对系统表空间这一个文件进行fsync操作会多一些 IO 操作。此外，mysqld需要维护更多的文件描述符。

## 当新建库时，innodb内部操作

innodb存储引擎在存储设计上模仿了Oracle的存储结构，其数据是按照表空间进行管理的。

新建一个数据库时，innodb存储引擎会初始化一个名为ibdata1 的表空间文件，默认情况下，这个文件会存储所有表的数据，以及我们所熟知但看不到的系统表sys_tables、sys_columns、sys_indexes 、sys_fields等。

此外，还会存储用来保证数据完整性的回滚段数据，当然这部分数据在新版本的MySQL中，已经可以通过参数来设置回滚段的存储位置了；

注：不同的表既可以共用一个ibd文件，也可以每个表自己一个ibd文件，默认是一个表一个。

虽然是一个表一个ibd，但这个ibd里只存储了该表的B+树数据、索引、插入缓存等信息，其余的信息如列、属性等信息还是存储在默认的ibdata1里面的。

```text 官网介绍

A data file that can hold data for one or more InnoDB tables and associated indexes.
 
The system tablespace contains the InnoDB data dictionary, and prior to MySQL 5.6 holds all other InnoDB tables by default.
 
The innodb_file_per_table option, enabled by default in MySQL 5.6 and higher, allows tables to be created in their own tablespaces.
 
File-per-table tablespaces support features such as efficient storage of off-page columns, table compression, and transportable tablespaces.
See Section 14.7.4, “InnoDB File-Per-Table Tablespaces” for details.
```

## ibd存储的数据

该表的所有索引数据【聚集索引】。

这里使用了B+tree,B+ tree的叶子节点，就会存放所有的数据。整个表，其实就是一棵B+ tree，一个ibd就是1-N个b+ tree。N等于你的索引数量

当新建一个表时，会给表创建一个主键primary Key，然后这个key就带着整行数据占据着一块空间，作为B+ tree的一个叶子节点里元素。

可以理解为一个key-value键值对，key就是主键，value就是整行数据。如果你根本就没创建主键（不推荐），那innodb也会给你分配一个RowId来作为将来找它的主键，只是看不到。

这棵拥有全量数据的b+ tree，就是将来提供数据的树，一般来说，这棵树最大一般4层，3层就能存2千万数据了，4层能达到n个亿，将来通过主键查询时，通过2-4次IO就能找到数据行。这个索引树，即——聚簇索引。

# 表空间-ibd内文件结构

## 
