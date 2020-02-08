---
title: 11-InnoDB-事务隔离级别、ACID模型
categories:
  - db-mysql-core
abbrlink: 3890ccea
date: 2020-02-05 14:40:35
tags:
---

摘要：mysql InnoDB 使用 B+树组织数据、查询数据、mysql InnoDB-B+树存储数据量、实际操作查看

<!--more-->
# MVCC

MVCC (MultiVersion Concurrency Control) 叫做多版本并发控制。
理解为：事务对数据库的任何修改的提交都不会直接覆盖之前的数据，而是产生一个新的版本与老版本共存，使得读取时可以完全不加锁。

同一行数据会有多个版本，某事务对该数据的修改并不会直接覆盖老版本，而是产生一个新版本和老版共存。
然后在该行追加两个虚拟的列，列就是进行数据操作的事务的ID（created_by_txn_id），是一个单调递增的ID；还有一个deleted_by_txn_id，将来用来做删除的。

那么在另一个事务在读取该行数据时，由具体的隔离级别来控制到底读取该行的哪个版本。同时，在读取过程中完全不加锁，除非用select * xxx for update强行加锁。

譬如read committed级别，每次读取，总是取事务ID最大的那个就好了。

对于Repeatable read，每次读取时，总是取事务ID小于等于当前事务的ID的那些数据记录。在这个范围内，如果某一数据有多个版本，则取最新的。

MVCC在mysql中的实现依赖的是undo log与read view

undo log记录某行数据的多个版本的数据；read view用来判断当前版本数据的可见性。

mysql就是用MVCC来实现读写分离不加锁的。

那么MVCC里多出来的那些版本的数据最终是要删除的，支持MVCC的数据库套路一般差不多，都会有一个后台线程来定时清理那些肯定没用的数据。只要一个数据的deleted_by_txn_id不为空，并且比当前还没结束的事务ID最小的一个还小，该数据就可以被清理掉了。在PostgreSQL中，该清理任务叫“vacuum”，在Innodb中，叫做“purge”。

# InnoDB 事务隔离级别
InnoDB的多版本并发控制是基于事务隔离级别实现的，而事务隔离级别则是依托前面提到的 Undo Log 实现的。
当读取一个数据记录时，每个事务会使用一个读视图(Read View)，读视图用于控制事务能读取到的记录的版本。

InnoDB的事务隔离级别分为：Read UnCommitted，Read Committed，Repeatable Read以及Serializable。
其中Serializable是基于锁实现的串行化方式，严格来说不是事务可见性范畴。

- Read Uncommitted：未提交读也称为脏读，它读取的是当前最新修改的记录，即便这个修改最后并未生效。
- Read Committed：提交读。它基于的是当前事务内的语句开始执行时的最大的事务ID。如果其他事务修改同一个记录，在没有提交前，则该语句读取的记录还是不会变。
但是这种情况会产生不可重复读，即一个事务内多次读取同一条记录可能得到不同的结果(该记录被其他事务修改并提交了)。
- Repeatable Read：可重复读。它基于的是事务开始时的读视图，直到事务结束。不读取其他新的事务对该记录的修改，保证同一个事务内的可重复读取。
InnoDB提供了 next-key lock来解决幻读问题，不过在一些特殊场景下，可重复读还是可能出现幻读的情况。在实际开发中影响不大。

# ACID 模型

事务有 ACID 四个属性， InnoDB 是支持事务的，它实现 ACID 的机制如下：

## Atomicity
innodb的原子性主要是通过提供的事务机制实现，与原子性相关的特性有：

- Autocommit 设置。
- COMMIT 和 ROLLBACK 语句(通过 Undo Log实现)。

## Consistency
innodb的一致性主要是指保护数据不受系统崩溃影响，相关特性包括：

- InnoDB 的双写缓冲区(doublewrite buffer)。
- InnoDB 的故障恢复机制(crash recovery)。

## Isolation
innodb的隔离性也是主要通过事务机制实现，特别是为事务提供的多种隔离级别，相关特性包括：

- Autocommit设置。
- SET ISOLATION LEVEL 语句。
- InnoDB 锁机制。

## Durability
innodb的持久性相关特性：

- Redo log。
- 双写缓冲功能。可以通过配置项 innodb_doublewrite 开启或者关闭。
- 配置 innodb_flush_log_at_trx_commit。用于配置innodb如何写入和刷新 redo 日志缓存到磁盘。默认为1，表示每次事务提交都会将日志缓存写入并刷到磁盘。innodb_flush_log_at_timeout 可以配置刷新日志缓存到磁盘的频率，默认是1秒。
- 配置 sync_binlog。用于设置同步 binlog 到磁盘的频率，为0表示禁止MySQL同步binlog到磁盘，binlog刷到磁盘的频率由操作系统决定，性能最好但是最不安全。为1表示每次事务提交前同步到磁盘，性能最差但是最安全。MySQL文档推荐是 sync_binlog 和 innodb_flush_log_at_trx_commit 都设置为 1。
- 操作系统的 fsync 系统调用。
- UPS设备和备份策略等。

参考资料
https://dev.mysql.com/doc/refman/5.7/en/innodb-storage-engine.html
http://ourmysql.com/archives/1228
https://www.jianshu.com/p/d4cc0ea9d097
https://zhuanlan.zhihu.com/p/103487968?utm_source=wechat_session




