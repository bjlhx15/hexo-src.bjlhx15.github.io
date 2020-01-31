---
title: 001-mysql-物理存储、InnoDB数据B+树存储
categories:
  - db-mysql
abbrlink: 83b4374e
date: 2020-01-31 09:12:52
tags:
---

摘要：MySQL InnoDB数据存储是以B+树索引方式存储
存储单元、mysql InnoDB-B+树组织数据、查询数据、mysql InnoDB-B+树存储数据量、实际操作查看

<!--more-->

# 概述

## 存储单元

磁盘扇区、文件系统、InnoDB 存储引擎都有各自的最小存储单元。存储数据的最小单位，类比放菜的盘子。

![](/images/post/db-mysql/001/mysql-disk.jpg)

### 磁盘扇区存储单元-扇区-512字节
在计算机中磁盘存储数据最小单元是扇区，一个扇区的大小是 512 字节

### 文件系统存储单元-块-4k
文件系统(例如 XFS/EXT4)他的最小单元是块，一个块的大小是 4K。

文件系统中一个文件大小只有 1 个字节，但不得不占磁盘上 4KB 的空间。\[0k不占空间]

![](/images/post/db-mysql/001/file-0k.jpg)  ![](/images/post/db-mysql/001/file-1k.jpg)

### InnoDB存储单元-页-16k
InnoDB 存储引擎也有自己的最小储存单元——页(Page)，一个页的大小是 16K。

InnoDB 的所有数据文件(后缀为 ibd 的文件)，他的大小始终都是 16384(16K)的整数倍。

``` text
$ ll |grep ibd 
-rw…………    96K 10 31 11:22 innodb_index_stats.ibd
-rw…………    96K 10 31 11:22 innodb_table_stats.ibd
-rw…………    96K 10  2  2018 slave_master_info.ibd
-rw…………    96K 10  2  2018 slave_relay_log_info.ibd
-rw…………    96K 10  2  2018 slave_worker_info.ibd
```

## mysql InnoDB-B+树组织数据、查询数据

在MySQL中，InnoDB页的大小默认是16k，当然也可以通过参数设置：
``` mysql
show variables like 'innodb_page_size';
-- innodb_page_size	16384
```
### 方式一、直接按页存储【假想】

数据表中的数据都是存储在页中的，所以一个页中能存储多少行数据呢？假设一行数据的大小是 1K，那么一个页可以存放 16 行这样的数据。

如果数据库按这样的方式存储，那么查找数据就成为一个问题。不知道要查找的数据存在哪个页中，每次查询都需要把所有的页遍历一遍，时间复杂度为 n。

### 方式二、用 B+ 树的方式组织数据【实际】

示例解说：
  
1. B+树-组织数据
![](/images/post/db-mysql/001/index-01.jpg)

先将数据记录按主键进行排序，分别存放在不同的页中（为了便于理解这里一个页中只存放 3 条记录，实际情况可以存放很多）。

除了存放数据的页以外，还有存放键值+指针的页，如图中 page number=3 的页，该页存放键值和指向数据页的指针，这样的页由 N 个键值+指针组成。

当然它也是排好序的。这样的数据组织形式，我们称为索引组织表。  

2. B+树-查询数据
```sql
select * from user where id=5;
```
id 是主键，我们通过这棵 B+ 树来查找，首先找到根页，怎么知道 user 表的根页在哪呢？

其实每张表的根页位置在表空间文件中是固定的，即 page number=3 的页（后续说明）。

找到根页后通过二分查找法，定位到 id=5 的数据应该在指针 P5 指向的页中，那么进一步去 page number=5 的页中查找，同样通过二分查询法即可找到 id=5 的记录：
```text
 5    zhao2   27
```
小结：  InnoDB 中主键索引 B+ 树是如何组织数据、查询数据
 
InnoDB 存储引擎的最小存储单元是页，页可以用于存放数据也可以用于存放键值+指针，在 B+ 树中叶子节点存放数据，非叶子节点存放键值+指针。

索引组织表通过非叶子节点的二分查找法以及指针确定数据在哪个页中，进而在去数据页中查找到需要的数据。

## mysql InnoDB-B+树存储数据量

### 数据量估算

**以B+ 树高为 2为例计算**

假设 B+ 树高为 2，即存在一个根节点和若干个叶子节点，那么这棵 B+ 树的存放总记录数为：根节点指针数*单个叶子节点记录行数。

上文已经说明单个叶子节点（页）中的记录数=16K/1K=16。（这里假设一行记录的数据大小为 1K，实际上现在很多互联网业务数据记录大小通常就是 1K 左右）。

那么现在需要计算出非叶子节点能存放多少指针？假设主键 ID 为 bigint 类型，长度为 8 字节，而指针大小在 InnoDB 源码中设置为 6 字节，这样一共 14 字节。

一个页中能存放多少这样的单元，其实就代表有多少指针，即 16384/14=1170。

可以算出一棵高度为 2 的 B+ 树，能存放 1170*16=18720 条这样的数据记录。

根据同样的原理可以算出一个高度为 3 的 B+ 树可以存放：1170*1170*16=21902400 条这样的记录。

所以在 InnoDB 中 B+ 树高度一般为 1-3 层，它就能满足千万级的数据存储。

在查找数据时一次页的查找代表一次 IO，所以通过主键索引查询通常只需要 1-3 次 IO 操作即可查找到数据。

###  InnoDB 主键索引 B+ 树的高度

上面通过推断得出 B+ 树的高度通常是 1-3，下面我们从另外一个侧面证明这个结论。

在 InnoDB 的表空间文件中，约定 page number 为 3 的代表主键索引的根页，而在根页偏移量为 64 的地方存放了该 B+ 树的 page level。

如果 page level 为 1，树高为 2，page level 为 2，则树高为 3。即 B+ 树的高度=page level+1；

查询系统表：page level。

在实际操作之前，可以通过 InnoDB 元数据表确认主键索引根页的 page number 为 3，也可以从《InnoDB 存储引擎》这本书中得到确认：
``` mysql
SELECT b.name, a.name, index_id, type, a.space, a.PAGE_NO
FROM
    information_schema.INNODB_SYS_INDEXES a,
    information_schema.INNODB_SYS_TABLES b
WHERE a.table_id = b.table_id AND a.space <> 0;
```
输出
``` text
blockchain_manager/member	    PRIMARY	    22	3	6	3
blockchain_manager/member_group	PRIMARY	    23	3	7	3
blockchain_manager/permission	PRIMARY	    24	3	8	3
blockchain_manager/permission	idx_group	56	0	8	4
```
可以看出数据库 blockchain_manager 下的 member 表、member 表、permission表主键索引根页的 page number 均为 3，而其他的二级索引 page number 为 4。

关于二级索引与主键索引的区别请参考 MySQL 相关书籍，

## 实际操作查看

### 创建测试数据

#### 测试表
``` mysql
CREATE TABLE `table5hang` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
修改表名，依次创建 table15w、table500w、table1000w

#### 测试数据 添加
```  sql
delimiter //  #定义标识符为双斜杠
DROP PROCEDURE IF EXISTS my_procedure ; #如果存在 my_procedure 存储过程则删除
CREATE PROCEDURE my_procedure () #创建无参存储过程
BEGIN
    DECLARE n INT DEFAULT 1 ; # 申明变量
    set @execSql='insert into table5hang(username,age)  values ';
    set @execdata = '';

    WHILE n < 6 DO
        set @execdata=concat(@execdata,"(","'name-",n,"',",n%100,")");

        if n%5=0
        then
            set @execSql = concat(@execSql,@execdata,";");
            #select @execSql;
            prepare stmt from @execSql;
            execute stmt;
            DEALLOCATE prepare stmt;
            commit;  

            set @execSql='insert into table5hang (username,age)  values ';
            set @execdata = '';
        ELSE
            set @execdata = concat(@execdata,',');
        end if;

        SET n = n + 1 ; #循环一次,i加一
    END WHILE ; #结束while循环
    #select count(*) from test_table;
END
//
delimiter ;
call my_procedure(); #调用存储过程
DROP PROCEDURE IF EXISTS my_procedure ; #如果存在 my_procedure 存储过程则删除

```
插入数据，依次修改表名：table15w、table500w、table1000w 以及插入条数

根据性能 500w、1000w 数据量的需要分批插入。

查看文件结构，mysql每次建库，会在data下创建以库名为名的文件夹，内部文件是表名，如下
``` text
-rw-rw----  1    54B  1 31 11:36 db.opt
-rw-rw----  1   8.4K  1 31 11:39 table1000w.frm
-rw-rw----  1   460M  1 31 12:41 table1000w.ibd
-rw-rw----  1   8.4K  1 31 11:39 table15w.frm
-rw-rw----  1    15M  1 31 11:54 table15w.ibd
-rw-rw----  1   8.4K  1 31 11:39 table500w.frm
-rw-rw----  1   236M  1 31 12:19 table500w.ibd
-rw-rw----  1   8.4K  1 31 11:40 table5hang.frm
-rw-rw----  1    96K  1 31 11:51 table5hang.ibd
```

因为主键索引 B+ 树的根页在整个表空间文件中的第 3 个页开始，所以可以算出它在文件中的偏移量：16384*3=49152（16384 为页大小）。

另外根据《InnoDB 存储引擎》中描述在根页的 64 偏移量位置前 2 个字节，保存了 page level 的值。

因此想要的 page level 的值在整个文件中的偏移量为：16384*3+64=49152+64=49216，前 2 个字节中。

``` bash
hexdump -s 49216 -n 10 table5hang.ibd

000c040 00 00 00 00 00 00 00 00 00 3c                  
000c04a

hexdump -s 49216 -n 10 table15w.ibd

000c040 00 01 00 00 00 00 00 00 00 3b                  
000c04a

hexdump -s 49216 -n 10 table500w.ibd

000c040 00 02 00 00 00 00 00 00 00 3a                  
000c04a

hexdump -s 49216 -n 10 table1000w.ibd

000c040 00 02 00 00 00 00 00 00 00 39                  
000c04a
```
#### 小结
table5hang 表数据行数为 5 条，B+ 树高度为 1，
table15w   表数据行数为 15 万，B+ 树高度为 2，
table500w  表数据行数为 500 万，B+ 树高度为 3，
table1000w 表数据行数为 1000 万，B+ 树高度为 3。

500w、1000w 两个表树的高度都是 3。换句话说这两个表通过索引查询效率并没有太大差异，因为都只需要做 3 次 IO。
那么如果有一张表行数是一千万，那么他的 B+ 树高度依旧是 3，查询效率仍然不会相差太大。region 表只有 5 行数据，当然他的 B+ 树高度为 1。

``` sql
select * from table500w where id=4230000
select * from table1000w where id=4230000
```








