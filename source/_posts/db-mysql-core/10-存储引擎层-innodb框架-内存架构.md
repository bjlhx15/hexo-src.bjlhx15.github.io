---
title: 10-存储引擎层-innodb框架-内存架构
categories:
  - db-mysql-core
abbrlink: ddb6eda9
date: 2020-02-05 10:53:04
tags:
---

摘要：mysql InnoDB 使用 B+树组织数据、查询数据、mysql InnoDB-B+树存储数据量、实际操作查看

<!--more-->


# InnoDB 内存中的结构
参看原理图 03-存储引擎层、InnoDB 架构

![](/images/post/db-mysql/innodb-struct.webp)

内存中的结构主要包括 Buffer Pool，Change Buffer、Adaptive Hash Index以及 Log Buffer 四部分。

如果从内存上来看，Change Buffer 和 Adaptive Hash Index 占用的内存都属于 Buffer Pool，Log Buffer占用的内存与 Buffer Pool独立。

缓冲池缓存的数据包括Page Cache、Change Buffer、Data Dictionary Cache等，通常 MySQL 服务器的 80% 的物理内存会分配给 Buffer Pool。

基于效率考虑，InnoDB中数据管理的最小单位为页，默认每页大小为16KB，每页包含若干行数据。为了提高缓存管理效率，InnoDB的缓存池通过一个页链表实现，很少访问的页会通过缓存池的 LRU 算法淘汰出去。

InnoDB 的缓冲池页链表分为两部分：New sublist(默认占5/8缓存池) 和 Old sublist(默认占3/8缓存池，可以通过 innodb_old_blocks_pct修改，默认值为 37)，

其中新读取的页会加入到 Old sublist的头部，而 Old sublist中的页如果被访问，则会移到 New sublist的头部。

缓冲池的使用情况可以通过 `show engine innodb status` 命令查看。其中一些主要信息如下：

由于MySQL不同版本采用InnoDB引擎版本不同，5.6后对show engine innodb status信息进行了优化
``` text
MySQL版本	InnoDB引擎版本
5.1.x	1.0.x版本（官方称为InnoDB Plugin）
5.5.x	5.5（1.1.x版本），InnoDB被Oracle收购后
5.6.x	5.6（1.2.x版本）
5.7.x	5.7
8.0.x	8.0
```
show engine innodb status显示的不是当前状态，而是过去某个时间范围内InnoDB存储引擎的状态。

向右拉
``` text
Per second averages calculated from the last 59 seconds
```
在显示前端可看到以上信息，代表查询的信息为过去59秒内每2秒的平均值。

show engine innodb status主要包括以下几个部分：
```text
BACKGROUND THREAD	后台Master线程
SEMAPHORES	信号量信息
LATEST DETECTED DEADLOCK	最近一次死锁信息，只有产生过死锁才会有
TRANSACTIONS	事物信息
FILE I/O	IO Thread信息
INSERT BUFFER AND ADAPTIVE HASH INDEX	INSERT BUFFER和自适应HASH索引
LOG	日志
BUFFER POOL AND MEMORY	BUFFER POOL和内存
INDIVIDUAL BUFFER POOL INFO	如果设置了多个BUFFER POOL实例，这里显示每个BUFFER POOL信息。可通过innodb_buffer_pool_instances参数设置
ROW OPERATIONS	行操作统计信息
END OF INNODB MONITOR OUTPU	输出结束语
```

## Buffer pool
查看：BUFFER POOL AND MEMORY
```
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 137428992 # 分配给InnoDB缓存池的内存(字节)
Dictionary memory allocated 102398  # 分配给InnoDB数据字典的内存(字节)
Buffer pool size   8191 # 缓存池的页数目
Free buffers       7893 # 缓存池空闲链表的页数目
Database pages     298  # 缓存池LRU链表的页数目
Modified db pages  0    # 修改过的页数目
......
```

Innodb的这个缓存区就是Innodb_buffer_pool，当读取数据时，就会先从缓存中查看是否数据的页（page）存在，不存在的话才去磁盘上检索，查到后缓存到这个pool里。
同理，插入、修改、删除也是先操作缓存里数据，之后再以一定频率更新到磁盘上。控制刷盘的机制，叫做Checkpoint。

![](/images/post/db-mysql/innodb_buffer_pool.png)
 
注意，左边那两个不在Innodb_buffer_pool里，是另外一块内存。只不过大部分的内存都属于Innodb_buffer_pool的。

mysql安装后，默认pool的大小是128M，可以通过show variables like 'innodb_buffer_pool%';命令查看。
```sql
show variables like 'innodb_buffer_pool%';
# innodb_buffer_pool_dump_at_shutdown	OFF
# innodb_buffer_pool_dump_now	OFF
# innodb_buffer_pool_filename	ib_buffer_pool
# innodb_buffer_pool_instances	8
# innodb_buffer_pool_load_abort	OFF
# innodb_buffer_pool_load_at_startup	OFF
# innodb_buffer_pool_load_now	OFF
# innodb_buffer_pool_size	134217728
```
可以通过show global status like '%innodb_buffer_pool_pages%';  查看已经被占用的和空闲的。共计8000多个page。

```sql
show global status like '%innodb_buffer_pool_pages%';
# Innodb_buffer_pool_pages_data	7167
# Innodb_buffer_pool_pages_dirty	0
# Innodb_buffer_pool_pages_flushed	120861
# Innodb_buffer_pool_pages_free	1024
# Innodb_buffer_pool_pages_misc	1
# Innodb_buffer_pool_pages_total	8192
```

所以如果数据很多，而pool很小，那么性能就好不了。

理论上来说，如果你给pool的内存足够大，够装下所有数据，要访问的所有数据都在pool里，那么你的所有请求都是走内存，性能将是最好的，和redis类似。

官方建议pool的空间设置为物理内存的50%-75%。

在mysql5.7.5之后，可以在mysql不重启的情况下动态修改pool的size，如果你设置的pool的size超过了1G的话，应该再修改一下Innodb_buffer_pool_instances=N，将pool分成N个pool实例，将来操作的数据会按照page的hash来映射到不同的pool实例。

这样可以大幅优化多线程情况下，并发读取同一个pool造成的锁的竞争。

### 缓冲区LRU淘汰算法

当pool的大小不够用了，满了，就会根据LRU算法（最近最少使用）来淘汰老的页面。最频繁使用的页在LRU列表的前端，最少使用的页在LRU列表的尾端。淘汰的话，就首先释放尾端的页。

InnoDB的LRU和普通的不太一样，Innodb的加入了midpoint位置的概念。最新读取到的页，并不是直接放到LRU列表的头部的，而是放到midpoint位置。
这个位置大概是LRU列表的5/8处，该参数由innodb_old_blocks_pct控制。
```
show variables like 'innodb_old_blocks_pct%';
innodb_old_blocks_pct	37
```
如37是默认值，表示新读取的页插入到LRU尾端37%的位置。在midpoint之后的列表都是old列表，之前的是new列表，可以简单理解为new列表的页都是最活跃的数据。

为什么不直接放头部？因为某些数据扫描操作需要访问的页很多，有时候这些页仅仅在本次查询有效，以后就不怎么用了，并不算是活跃的热点数据。那么真正活跃的还是希望放到头部去，这些新的暂不确定是否真正未来要活跃。所以，这可以理解为预热。还引入了一个参数innodb_old_blocks_time用来表示页读取到mid位置后需要等待多久才会被加入到LRU列表的热端。

重要的查询命令可以看到这些信息，show engine innodb status;
Database pages表示LRU列表中页的数量，pages made young显示了LRU列表中页移动到前端的次数，Buffer pool hit rate表示缓冲池的命中率，100%表示良好，该值小于95%时，需要考虑是否因为全表扫描引起了LRU列表被污染。里面还有其他的参数，可以自行查阅一下代表什么意思。

### Pool的主要空间

其实更多的、对性能影响更大的是读缓存。毕竟多数数据库是读多写少。

读缓存主要数据是索引页和数据页，如果要读取的数据在pool里没有，那就去磁盘读，读到后的新页放到pool的3/8位置，后续根据情况再决定是否放到LRU列表的头部。

注意，最小单位是页，哪怕只读一条数据，也会加载整个页进去。如果是顺序读的话，刚好又在同一个页里，譬如读了id=1的，那么再读id=2的时，大概率直接从缓存里读。 

## BACKGROUND THREAD

InnoDB存储引擎的核心操作大部分都集中在Mater Thread后台线程中。

MySQL5.5版本之前：
``` text
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 846676 1_second, 846675 sleeps, 84665 10_second, 17 background, 17 flush
srv_master_thread log flush and writes: 854189
```
MySQL 5.6之后对Master Thread进行了优化，去除了sleeps的信息，srv_active为之前的每秒的循环，srv_idle为每10秒的的循环，srv_shutdown为停止的循环，通常为0，只在MySQL关闭时才会增加。
```text
BACKGROUND THREAD
-----------------
srv_master_thread loops: 3911776 srv_active, 0 srv_shutdown, 309625 srv_idle
srv_master_thread log flush and writes: 4221384
```
上面可以看出主循环每10秒进行了309625次，每秒进行了3911776次，每10秒的操作符合1：10。

负载低的情况下日志缓冲刷盘次数，4221384 ≈ 3911776+309625。

根据循环次数可大概判断当前数据库负载情况。如果每秒循环次数少，每10秒次数多，证明当前负载很低；如果每秒循环次数多，每10秒次数少，远大于10：1，证明当前负载很高。

## SEMAPHORES

当前等待线程的列表及事件计数器，可以评估当前负载情况。
```text
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 58961200
OS WAIT ARRAY INFO: signal count 125268732
Mutex spin waits 770371493, rounds 6482840874, OS waits 20699077
RW-shared spins 0, rounds 115276716, OS waits 14655922
RW-excl spins 0, rounds 987115172, OS waits 12384598
RW-sx spins 40484350, rounds 419545112, OS waits 4476477
Spin rounds per wait: 115276716.00 RW-shared, 987115172.00 RW-excl, 10.36 RW-sx
```
``` text
OS WAIT ARRAY INFO	reservation count：表示InnoDB产生了多少次OS WAIT；
                    signal count：表示进入OS WAIT的线程被唤醒次数
Mutex(5.7后去除)	spins：空转次数，通过innodb_sync_spin_loops控制，超过则转到OS waits；
                spin waits：spin线程无法获取锁而进入Spin wait；
                rounds：spin wait进行轮询检查mutexes的次数；
                OS waits：线程放弃spin wait进入挂起状态。
RW-shared	RW-shared 共享锁
RW-excl	RW-excl 排他锁
RW-sx	5.7后新增；RW-sx 共享排他锁
Spin rounds per wait	rounds / spins = 值
```

要明白InnoDB如何处理互斥量(Mutexes)，以及什么是两步获得锁(two-step approach)。

1. 首先进程试图获得一个锁，如果此锁被它人占用。它就会执行所谓的spin wait，即所谓循环的查询“锁被释放了吗？”。
2. 如果在循环过程中，一直未得到锁释放的信息，则其转入OS WAIT，即所谓线程进入挂起(suspended)状态。
3. 直到锁被释放后，通过信号(singal)唤醒线程。

Spin wait的消耗远小于OSwaits。Spin wait利用cpu的空闲时间，检查锁的状态，OS Wait会有所谓content switch，从CPU内核中换出当前执行线程以供其它线程使用。

所以应尽量减少OS waits，可以通过innodb_sync_spin_loops参数来平衡spin wait和os wait。Mutex信息可通过show engine innodb mutex查看。

## LATEST DETECTED DEADLOCK

记录最近一次死锁信息，只有产生过死锁才会有记录。
```text
------------------------
LATEST DETECTED DEADLOCK
------------------------
190425 18:00:13
*** (1) TRANSACTION:
TRANSACTION 231E7C5DF, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1248, 3 row lock(s)
MySQL thread id 1346996, OS thread handle 0x7fd968454700, query id 760545285 10.10.x.x app_user updating
DELETE 
    FROM db_0.table_0
    WHERE ORDER_ID IN (  456787464 , 456787465 )
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 5 page no 6064 n bits 824 index `orderId_index` of table `db_0`.`table_0` trx id 231E7C5DF lock_mode X waiting
```
```text
Record lock, heap no 180 PHYSICAL RECORD: n_fields 2; compact format; info bits 32
 0: len 8; hex 80000015eb6a1041; asc      j A;;
 1: len 8; hex 800000002018fce2; asc         ;
*** (2) TRANSACTION:
TRANSACTION 231E7C5DD, ACTIVE 0 sec starting index read, thread declared inside InnoDB 1
mysql tables in use 1, locked 1
5 lock struct(s), heap size 1248, 4 row lock(s)
MySQL thread id 1348165, OS thread handle 0x7fd96669f700, query id 760545283 10.10.x.x app_user updating
DELETE 
    FROM db_0.table_0
    WHERE ORDER_ID IN (  456787464 , 456787465 )
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 5 page no 6064 n bits 824 index `orderId_index` of table `db_0`.`table_0` trx id 231E7C5DD lock_mode X locks rec but not gap
```
```text
Record lock, heap no 180 PHYSICAL RECORD: n_fields 2; compact format; info bits 32
 0: len 8; hex 80000015eb6a1041; asc      j A;;
 1: len 8; hex 800000002018fce2; asc         ;;
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 5 page no 6064 n bits 824 index `orderId_index` of table `db_0`.`table_0` trx id 231E7C5DD lock_mode X waiting
```
```text
Record lock, heap no 180 PHYSICAL RECORD: n_fields 2; compact format; info bits 32
 0: len 8; hex 80000015eb6a1041; asc      j A;;
 1: len 8; hex 800000002018fce2; asc         ;;
*** WE ROLL BACK TRANSACTION （1）
```

死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象。正常死锁会自动释放，innodb有一个内在的死锁检测工具，

当死锁超过一定时间后，会回滚其中一个事务，innodb_lock_wait_timeout可配置死锁等待超时时间。

- 死锁在两情况下最容易产生：

- 高并发同时操作同一条数据

存在主键和辅助索引，加锁顺序相反

避免死锁方法即降低并发，操作数据时使加锁顺序相同。  

## TRANSACTIONS

包含了InnoDB事务(transaction)的统计信息。
``` text
------------
TRANSACTIONS
------------
Trx id counter 2409176
Purge done for trx's n:o < 2409171 undo n:o < 0 state: running but idle
History list length 31
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 421224214038352, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421224214044736, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421224214039264, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 2409171, ACTIVE 1549 sec fetching rows, thread declared inside InnoDB 3871
mysql tables in use 1, locked 0
0 lock struct(s), heap size 1136, 0 row lock(s)
MySQL thread id 653597, OS thread handle 140289889908480, query id 2528936 127.0.0.1 root Sending data
SELECT /*!40001 SQL_NO_CACHE */ * FROM `table`
```
``` text
Trx id counter	当前事物ID
Purge done for trx's	正在清理掉的transaction ID
History list length	记录了undo spaces内未清掉的事务个数，Purge的原则是记录没有被其它事务继续使用。
LIST OF TRANSACTIONS FOR EACH SESSION	每个session的事物状态
```
当前活跃的事物状态为ACTIVE，事物的详细信息，包括线程ID、执行时间、用户、SQL等。正在使用1个表，涉及锁的表0个。

## FILE I/O

在InnoDB中大量使用了AIO（Async IO）来处理IO 请求，IO Thread主要是负责这些IO请求的回调处理，通过调用fsync()函数协调内存与磁盘之间的数据。
```text
--------
FILE I/O
--------
I/O thread 0 state: waiting for completed aio requests (insert buffer thread)
I/O thread 1 state: waiting for completed aio requests (log thread)
I/O thread 2 state: waiting for completed aio requests (read thread)
I/O thread 3 state: waiting for completed aio requests (read thread)
I/O thread 4 state: waiting for completed aio requests (read thread)
I/O thread 5 state: waiting for completed aio requests (read thread)
I/O thread 6 state: waiting for completed aio requests (read thread)
I/O thread 7 state: waiting for completed aio requests (read thread)
I/O thread 8 state: waiting for completed aio requests (read thread)
I/O thread 9 state: waiting for completed aio requests (read thread)
I/O thread 10 state: waiting for completed aio requests (write thread)
I/O thread 11 state: waiting for completed aio requests (write thread)
I/O thread 12 state: waiting for completed aio requests (write thread)
I/O thread 13 state: waiting for completed aio requests (write thread)
I/O thread 14 state: waiting for completed aio requests (write thread)
I/O thread 15 state: waiting for completed aio requests (write thread)
I/O thread 16 state: waiting for completed aio requests (write thread)
I/O thread 17 state: waiting for completed aio requests (write thread)
Pending normal aio reads: [0, 0, 0, 0, 0, 0, 0, 0] , aio writes: [0, 0, 0, 0, 0, 0, 0, 0] ,
ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
15234061 OS file reads, 304461183 OS file writes, 73899457 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.24 writes/s, 0.17 fsyncs/s
```
InnoDB1.0版本之前有4个IO线程，1.1后做了优化，Purge Thread从Master Thread独立出来，Purge Cleaner Thread从InnoDB1.2版本引入，都是为了减轻Master Thread的工作，提高CPU利用率。
```text
insert buffer thread	合并插入缓冲，insert buffer维护非唯一辅助索引
log thread	负责异步刷新事物日志
read thread	预读，innodb_read_io_threads 默认4
write thread	刷新脏页缓冲，innodb_write_io_threads 默认4
purge thread	回收已经使用并分配的undo页，可设置多个
purge cleaner Thread	刷新脏页
```
显示各个I/O thread的pending operations,pending的log和buffer pool thread的fsync()调用；
- aio：代表的是异步IO(asynchronous I/O)；
- OS file：显示了reads writes fsync() 调用次数。

 

## (Change Buffer)INSERT BUFFER AND ADAPTIVE HASH INDEX

### 插入缓冲insert buffer
它是buffer_pool的一部分，用来做insert操作时的缓存的。

如b+tree，以及数据的存放格式，那么当新插入数据时，倘若直接就插入到b+ tree里，那么可能会比较缓慢，需要读取、找到要插入的地方，还要做树的扩容、校验、寻址、落盘等等一大堆操作。

在Innodb中，主键是行唯一标识，如果你的插入顺序是按照主键递增进行插入的，那么还好，它不需要磁盘的随机读取，找到了页，就能插，这样速度还是可以的。

然而，如果你的表上有多个别的索引（二级索引），那么当插入时，对于那个二级索引树，就不是顺序的了，它需要根据自己的索引列进行排序，这就需要随机读取了。
二级索引越多，那么插入就会越慢，因为要寻找的树更多了。还有，如果你频繁地更新同一条数据，倘若也频繁地读写磁盘，那就不合适了，最好是将多个对同一page的操作，合并起来，统一操作。

所以，Innodb设计了Insert Buffer，对于非聚簇索引的插入、更新操作，不是每次都插入到索引页中，而是先判断该二级索引页是否在缓冲池中，
若在，就直接插入，若不在，则先插入一个insert buffer里，再以一定的频率进行真正的插入到二级索引的操作，这时就可以聚合多个操作，一起去插入，就能提高性能。

然而，insert buffer需要同时满足两个条件时，才会被使用：

- 索引是二级索引
- 索引不是unique

注意，索引不能是unique，因为在插入缓冲时，数据库并不去查询索引页来判断插入的记录的唯一性，如果查找了，就又会产生随机读取。

insert buffer的问题是，在写密集的情况下，内存会占有很大，默认最大可以占用1/2的Innodb_buffer_pool的空间。
很明显，如果占用过大，就会对其他的操作有影响，譬如能缓存的查询页就变少了。可以通过IBUF_POOL_SIZE_PER_MAX_SIZE来进行控制。

### 变更缓冲change buffer

INSERT BUFFER即合并插入缓存，从innodb 1.0.x(MySQL5.5 之前)版本开始引入Change Buffer，是INSERT BUFFER升级版，即MySQL 5.1.x以上版本都支持，
不仅包括INSERT BUFFER，还包括UPDATE BUFFER、DELETE BUFFER、PURGE BUFFER。

也就是所有DML操作，都会先进缓冲区，进行逻辑操作，后面才会真正落地。

通过参数Innodb_change_buffering开始查看修改各种buffer的选项。可选值有inserts\deletes\purges\changes\all\none。
``` sql
show variables like 'Innodb_change_buffering%';
-- innodb_change_buffering	all

show variables like 'Innodb_change_buffer_max_size';
-- innodb_change_buffer_max_size	25
```
默认是所有操作都入buffer，参数是控制内存大小的，25代表最多使用1/4的缓冲池空间。


通常来说，InnoDB辅助索引不同于聚集索引的顺序插入，如果每次修改二级索引都直接写入磁盘，则会有大量频繁的随机IO。
Change buffer 的主要目的是将对 非唯一 辅助索引页的操作缓存下来，以此减少辅助索引的随机IO，并达到操作合并的效果。它会占用部分Buffer Pool 的内存空间。
在 MySQL5.5 之前 Change Buffer其实叫 Insert Buffer，最初只支持 insert 操作的缓存，随着支持操作类型的增加，改名为 Change Buffer。
如果辅助索引页已经在缓冲区了，则直接修改即可；如果不在，则先将修改保存到 Change Buffer。
Change Buffer的数据在对应辅助索引页读取到缓冲区时合并到真正的辅助索引页中。Change Buffer 内部实现也是使用的 B+ 树。

查看Change Buffer信息也可以通过 show engine innodb status 命令。更多信息见 [mysqlserverteam: the-innodb-change-buffer](https://mysqlserverteam.com/the-innodb-change-buffer/)。
```text
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1356, free list len 149402, seg size 149404, 2004231 merges
merged operations:
 insert 1373793, delete mark 316276978, delete 5341003
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 298897, node heap has 1 buffer(s)
Hash table size 298897, node heap has 1 buffer(s)
Hash table size 298897, node heap has 0 buffer(s)
Hash table size 298897, node heap has 1 buffer(s)
Hash table size 298897, node heap has 1 buffer(s)
Hash table size 298897, node heap has 1 buffer(s)
Hash table size 298897, node heap has 2 buffer(s)
Hash table size 298897, node heap has 0 buffer(s)
Hash table size 298897, node heap has 1 buffer(s)
Hash table size 298897, node heap has 0 buffer(s)
Hash table size 298897, node heap has 0 buffer(s)
Hash table size 298897, node heap has 1 buffer(s)
Hash table size 298897, node heap has 1 buffer(s)
193.03 hash searches/s, 713.40 non-hash searches/s
```
```text
Ibuf：size	已经合并页的数量
free list len	空闲列表长度
seg size	Insert Buffer大小
merges	合并次数
merged operations 	
                    Change Buffer中每个操作次数；
                    insert代表Insert Buffer;
                    delete mark代表Delete Buffer；
                    delete代表Purge Buffer;
discarded operations	Change Buffer中无需合并的次数
hash searches/s	通过hash索引查询
non-hash searches/s	不能通过hash索引查询
```

可以通过 innodb_change_buffering 配置是否缓存辅助索引页的修改，默认为 all，即缓存 insert/delete-mark/purge 操作
(注：MySQL 删除数据通常分为两步，第一步是delete-mark，即只标记，而purge才是真正的删除数据)。

![](/images/post/db-mysql/changebuffer.webp)

### ADAPTIVE HASH INDEX

自适应哈希索引(AHI)查询非常快，一般时间复杂度为 O(1)，相比 B+ 树通常要查询 3~4次，效率会有很大提升。

innodb 通过观察索引页上的查询次数，如果发现建立哈希索引可以提升查询效率，则会自动建立哈希索引，称之为自适应哈希索引，不需要人工干预，可以通过 innodb_adaptive_hash_index 开启，MySQL5.7 默认开启。

考虑到不同系统的差异，有些系统开启自适应哈希索引可能会导致性能提升不明显，而且为监控索引页查询次数增加了多余的性能损耗，
 
 MySQL5.7 更改了 AHI 实现机制，每个 AHI 都分配了专门分区，通过 innodb_adaptive_hash_index_parts配置分区数目，默认是8个，如前一节命令列出所示。

通过(Change Buffer)INSERT BUFFER 可以看到自适应哈希索引大小、使用情况、每秒使用自适应哈希索引搜索情况。
自适应HASH索引，由INNODB存储引擎控制，只适合等值查询，不适合范围查询。

## LOG

事物日志的信息。
``` text
---
LOG
---
Log sequence number 33859450169594
Log flushed up to   33859450169564
Pages flushed up to 33859450169210
Last checkpoint at  33859450169201
0 pending log flushes, 0 pending chkp writes
15044267 log i/o's done, 0.10 log i/o's/second
```
InnoDB事物采用Write-Ahead log策略，即事物在提交时，先写重做日志，在修改页。

Write-Ahead Log：如果一个页在写入磁盘时，必须先将内存中小于该页LSN的日志先写入到磁盘中。

重做日志有LSN、每个页有LSN、Checkpoint也有LSN。
```
Log sequence number	最新产生的日志序列号
Log flushed up to	已刷到磁盘的重做日志的日志号
Pages flushed up to	已刷到磁盘的页的日志号
Last checkpoint at	最后一次检查点位置，数据和日志一致的状态
pending	当前挂起的日志读写操作
```
LSN记录的是重做日志的总量，单位是字节。以下三种情况会将重做日志缓存刷到重做日志文件：

- Master Thread 每秒刷重做日志缓存到重做日志文件

- innodb_flush_log_at_trx_commit=1时，控制Log Buffer如何写入和刷到磁盘，每次事务提交刷重做日志缓存到重做日志文件

- 重做日志缓冲池剩余空间小于1/2时，刷重做日志缓存到重做日志文件

innodb_flush_log_at_trx_commit 说明：
- 默认为1，表示每次事务提交都会将 Log Buffer 写入操作系统缓存，并调用配置的 "flush" 方法将数据写到磁盘。设置为 1 因为频繁刷磁盘效率会偏低，但是安全性高，最多丢失 1个 事务数据。而设置为 0 和 2 则可能丢失 1秒以上 的事务数据。
- 为 0 则表示每秒才将 Log Buffer 写入内核缓冲区并调用 "flush" 方法将数据写到磁盘。
- 为 2 则是每次事务提交都将 Log Buffer写入内核缓冲区，但是每秒才调用 "flush" 将内核缓冲区的数据刷到磁盘。

注意，除了 MySQL 的缓冲区，操作系统本身也有内核缓冲区。

Log Buffer是 重做日志在内存中的缓冲区，大小由 innodb_log_buffer_size 定义，默认是 16M。

一个大的 Log Buffer可以让大事务在提交前不必将日志中途刷到磁盘，可以提高效率。如果你的系统有很多修改很多行记录的大事务，可以增大该值。

![](/images/post/db-mysql/innodb_flush_log_at_trx_commit.png)

innodb_flush_log_at_timeout 可以配置刷新日志缓存到磁盘的频率，默认是1秒。注意刷磁盘的频率并不保证就正好是这个时间，可能因为MySQL的一些操作导致推迟或提前。
而这个 "flush" 方法并不是C标准库的 fflush 方法(fflush是将C标准库的缓冲写到内核缓冲区，并不保证刷到磁盘)，它通过 innodb_flush_method 配置的，默认是 fsync，即日志和数据都通过 fsync 系统调用刷到磁盘。

可以发现，InnoDB 基本每秒都会将 Log buffer落盘。而InnoDB中使用的 redo log 和 undo log，它们是分开存储的。
redo log在内存中有log buffer，在磁盘对应ib_logfile文件。而undo log是记录在表空间ibd文件中的，InnoDB为undo log会生成undo页，对undo log本身的操作（比如向undo log插入一条记录），也会记录redo log，因此undo log并不需要马上落盘。
而 redo log则通常会分配一块连续的磁盘空间，然后先写到log buffer，并每秒刷一次磁盘。redo log必须在数据落盘前先落盘(Write Ahead Log)，从而保证数据持久性和一致性。而数据本身的修改可以先驻留在内存缓冲池中，再根据特定的策略定期刷到磁盘。


## BUFFER POOL AND MEMORY

innodb_buffer_pool包含数据页、索引页、undo页、insert buffer、数据字典、自适应哈希索引、锁信息等。数据库缓冲池是通过LRU列表管理的。
```
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 19789774848
Dictionary memory allocated 3944999
Buffer pool size   1179504
Free buffers       8192
Database pages     1116347
Old database pages 411925
Modified db pages  3
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 74514305, not young 649973267
0.21 youngs/s, 0.17 non-youngs/s
Pages read 15233915, created 7356668, written 264739684
0.00 reads/s, 0.00 creates/s, 0.10 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 1116347, unzip_LRU len: 0
I/O sum[48]:cur[0], unzip sum[0]:cur[0]
```
``` text
Total large memory allocated	为innodb 分配的总内存数(byte)
Dictionary memory allocated	为innodb数据字典分配的内存数(byte)
Buffer pool size	innodb_buffer_pool的页数量
Free buffers	lru列表中的空闲页数量
Database pages	lru列表中的非空闲页数量
Old database pages	old子列表的页数量
Modified db pages	脏页的数量
Pending reads	挂起读的数量
```
可以看到当前Buffer Pool Size共有1179504页，即1179504*16K。新读取到的页默认插入LRU列表的5/8的位置。

此值由innodb_old_blocks_pct控制，即前5/8称为new list，后面3/8的称为old list。

Pages made young 显示LRU列表中old list移到new list的次数，not young显示仍在old list的次数。

这两个值受innodb_old_blocks_time影响，此值为微秒。如果old list中超过30微秒不再读取，则记录not young，反之记录为Pages made young。
``` text
> show global variables like '%blocks%';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_old_blocks_pct  | 37    |
| innodb_old_blocks_time | 30    |
+------------------------+-------+
```
youngs/s,non-youngs/s，表示每秒这两类操作的次数。

Pages read,created,written，表示innodb被读取，创建，写入了多少页及每秒的次数。

Buffer pool hit rate，表示缓冲池命中率，如果低于95%需要具体排查。

Pages read ahead，表示页面预读，随机预读的每秒页数。

LRU中包含unzip_LRU，unzip_LRU是管理非16KB的压缩表。

## INDIVIDUAL BUFFER POOL INFO

可通过innodb_buffer_pool_instances 来配置多个缓冲池实例，默认为1。可减少数据库内部资源竞争，增加并发处理能力。如果分配多个缓冲池实例，每个缓冲池大小为 innodb_buffer_pool_size / innodb_buffer_pool_instances 。
``` text
----------------------
INDIVIDUAL BUFFER POOL INFO
----------------------
---BUFFER POOL 0
Buffer pool size   147438
Free buffers       1024
Database pages     139530
Old database pages 51486
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 8790743, not young 77467460
0.00 youngs/s, 0.00 non-youngs/s
Pages read 1856892, created 916430, written 30727167
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 139530, unzip_LRU len: 0
I/O sum[6]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 1
---BUFFER POOL 2
---BUFFER POOL 3
```
可以通过information_schema.INNODB_BUFFER_POOL_STATS视图查看每个buffer_pool实例的信息，MySQL默认一个page大小为16K，

可以得出POOL_SIZE * innodb_buffer_pool_instances * 16K = innodb_buffer_pool_size。
```text
(root@localhost) [information_schema] >select POOL_ID,POOL_SIZE,FREE_BUFFERS,DATABASE_PAGES,OLD_DATABASE_PAGES,MODIFIED_DATABASE_PAGES,PAGES_MADE_YOUNG,PAGES_NOT_MADE_YOUNG from information_schema.INNODB_BUFFER_POOL_STATS;
+---------+-----------+--------------+----------------+--------------------+-------------------------+------------------+----------------------+
| POOL_ID | POOL_SIZE | FREE_BUFFERS | DATABASE_PAGES | OLD_DATABASE_PAGES | MODIFIED_DATABASE_PAGES | PAGES_MADE_YOUNG | PAGES_NOT_MADE_YOUNG |
+---------+-----------+--------------+----------------+--------------------+-------------------------+------------------+----------------------+
|       0 |     90112 |            0 |          90109 |              33279 |                       0 |            18064 |            132278807 |
|       1 |     90112 |            0 |          90109 |              33282 |                       0 |            18342 |            132086061 |
|       2 |     90112 |            0 |          90110 |              33282 |                       0 |            17631 |            132149779 |
+---------+-----------+--------------+----------------+--------------------+-------------------------+------------------+----------------------+
```
详细说明同上。

## ROW OPERATIONS

显示了row 操作及其他一些统计信息。
```
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=444943, Main thread ID=139899621590784, state: sleeping
Number of rows inserted 172887566, updated 227534242, deleted 56676133, read 709667077
8.77 inserts/s, 8.04 updates/s, 0.00 deletes/s, 10.92 reads/s
```
queries，表示innodb内核中有多少个线程，队列中有多少个线程。

read views open inside InnoDB，表示有多少个read view 被打开，一个read view 包含事物开始点数据库内容的MVCC快照。

Process ID=444943，表示内核的主线程状态。

Number of rows inserted、updated、deleted、read，表示多少行被插入，更新和删除，读取及每秒信息，可用于监控。

可通过以下命令查看：
```
(root@localhost) [(none)] >show global status like 'Innodb_rows_%';
+----------------------+-----------+
| Variable_name        | Value     |
+----------------------+-----------+
| Innodb_rows_deleted  | 56676133  |
| Innodb_rows_inserted | 172887566 |
| Innodb_rows_read     | 709667077 |
| Innodb_rows_updated  | 227534242 |
+----------------------+-----------+

(root@localhost) [(none)] >show global status like 'Uptime';
+---------------+---------+
| Variable_name | Value   |
+---------------+---------+
| Uptime        | 1757270 |
+---------------+---------+
END OF INNODB MONITOR OUTPUT
```

## InnoDB信息结束语。
```
----------------------------
END OF INNODB MONITOR OUTPUT
============================
如果看不到这行输出，可能是有大量事务或者有一个大的死锁截断了输出信息。
```
