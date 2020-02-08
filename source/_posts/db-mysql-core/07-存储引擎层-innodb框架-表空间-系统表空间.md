---
title: 07-存储引擎层-innodb框架-表空间-系统表空间
categories:
  - db-mysql-core
abbrlink: da9f4def
date: 2020-02-05 10:13:21
tags:
---


摘要：系统表空间包含内容有：数据字典，双写缓冲，修改缓冲，undo日志，以及在系统表空间创建的表的数据和索引。

<!--more-->

# 表空间物理结构-页、区、段

可以看到，除了分配未使用的页外， UNDO_LOG，SYS, INDEX 页占据了不少的空间。

UNDO_LOG 页存储的是Undo log，SYS 页存储的是数据字典、回滚段、修改缓存等信息，INDEX 是索引页，TRX_SYS 页用于InnoDB的事务系统。

数据字典就是数据表的元信息，修改缓冲前面提到是为了提高IO性能也不再赘述，这里主要分析下 Undo 日志和双写缓冲。

```bash
innodb_space -s /Users/lihongxu6/docker/mymysql56/data/ibdata1 space-page-type-summary
type                count       percent     description         
ALLOCATED           4392        90.30       Freshly allocated   
UNDO_LOG            210         4.32        Undo log            
SYS                 141         2.90        System internal     
INDEX               110         2.26        B+Tree index        
INODE               7           0.14        File segment inode  
FSP_HDR             2           0.04        File space header   
TRX_SYS             1           0.02        Transaction system header
IBUF_BITMAP         1           0.02        Insert buffer bitmap
(base)
```

## Undo 日志

MySQL的MVCC(多版本并发控制)依赖Undo Log实现

MySQL的表空间文件 *.ibd 存储的是记录最新值，每个记录都有一个回滚指针(见前面图中的Roll Ptr)，指向该记录的最近一条Undo记录，

而每条Undo记录都会指向它的前一条Undo记录，如下图所示。默认情况下 undo log存储在系统表空间 ibdata1 中。

示例
这是最初的 插入后的数据
```bash
innodb_space -s /Users/lihongxu6/docker/mymysql56/data/ibdata1 -T test_innodb/table5hang -p 3 -R 127 record-history
# Transaction   Type                Undo record
# (n/a)         insert              (id=1) → ()
# (base)
```
执行一个更新
```sql
update table5hang set age=12 where id = 1;
update table5hang set age=13 where id = 1;
```
再次查看
```bash
innodb_space -s /Users/lihongxu6/docker/mymysql56/data/ibdata1 -T test_innodb/table5hang -p 3 -R 127 record-history
# Transaction   Type                Undo record
# 29910         update_existing     (id=1) → (age=12)
# 29904         update_existing     (id=1) → (age=1)
# (n/a)         insert              (id=1) → ()
# (base) 
```
需要注意的是，Undo Log 在事务执行过程中就会产生，事务提交后才会持久化，如果事务回滚了则Undo Log也会删除。

另外，删除记录并不会立即在表空间中删除该记录，而只是做个标记(delete-mark)，真正的删除则是等由后台运行的 purge 进程处理。

除了每条记录有Undo Log的列表外，整个数据库也会有一个历史列表，purge 进程会根据该历史列表真正删除已经没有再被其他事务使用的 delete-mark 的记录。

purge 进程会删除该记录以及该记录的 Undo Log。

## 双写缓冲
InnoDB的记录更新流程：先在Buffer Pool中更新，并将更新记录到 Redo Log 文件中，Buffer Pool中的记录会标记为脏数据并定期刷到磁盘。

由于InnoDB默认Page大小是16KB，而磁盘通常以扇区为单位写入，每次默认只能写入512个字节，无法保证16K数据可以原子的写入。

如果写入过程发生故障(比如机器掉电或者操作系统崩溃)，会出现页的部分写入(partial page writes)，导致难以恢复。

因为 MySQL 的重做日志采用的是物理逻辑日志，即页间是物理信息，而页内是逻辑信息，在发生页部分写入时，无法确认数据页的具体修改而导致难以恢复。


MySQL 的数据页在真正写入到表空间文件前，会先写到系统表空间文件的一段连续区域双写缓冲(Double-Write Buffer，默认大小为 2MB，128个页)并 fsync 落盘，

等双写缓冲写入成功后才会将数据页写到实际表空间的位置。

因为双写缓冲和数据页的写入时机不一致，如果在写入双写缓冲出错，可以直接丢弃该缓冲页，而如果是写入数据页时出错，则可以根据双写缓冲区数据恢复表空间文件。




