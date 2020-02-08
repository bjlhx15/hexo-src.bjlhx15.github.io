---
title: 03-存储引擎层、InnoDB 架构
categories:
  - db-mysql-core
abbrlink: 4ced7854
date: 2020-02-01 19:19:50
tags:
---

摘要：物理存储文件结构

<!--more-->

# 物理存储文件结构

![](/images/post/db-mysql/005/jiegou.png)

my.cnf，配置文件。

slow.log，记录慢查询日志，当语句执行时间超过参数long_query_time的值时，会被记录到该log，需要开启配置后才有。

error.log，记录错误和警告信息。

general.log，记录所有在数据库上执行的语句，可以用来追踪问题，文件增长很快，也很大，一般不会打开，需要要调试时可以打开。

系统默认库有4个，存储系统信息的，表名、列名、列属性等等。
``` text
information_scherma
mysql
performance_scherma
sys
```

# InnoDB 架构

![](/images/post/db-mysql/innodb-struct.webp)

从上面第二张图可以看到，InnoDB 主要分为两大块：

- InnoDB In-Memory Structures【内存中的结构】
- InnoDB On-Disk Structures【磁盘上的结构】

InnoDB 使用日志先行策略，将数据修改先在内存中完成，并且将事务记录成重做日志(Redo Log)，转换为顺序IO高效的提交事务。

这里日志先行，说的是日志记录到数据库以后，对应的事务就可以返回给用户，表示事务完成。

但是实际上，这个数据可能还只在内存中修改完，并没有刷到磁盘上去。内存是易失的，如果在数据落地前，机器挂了，那么这部分数据就丢失了。

InnoDB 通过 redo 日志来保证数据的一致性。如果保存所有的重做日志，显然可以在系统崩溃时根据日志重建数据。

当然记录所有的重做日志不太现实，所以 InnoDB 引入了检查点机制。即定期检查，保证检查点之前的日志都已经写到磁盘，则下次恢复只需要从检查点开始。

