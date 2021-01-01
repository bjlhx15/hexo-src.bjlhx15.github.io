---
title: 006-count优化
categories:
  - db-mysql-qa
abbrlink: 1e6de73e
date: 2020-02-20 09:44:28
---

摘要：006-count优化
<!-- more -->

# 增加占用空间小的非聚簇索引优化count

```sql
CREATE TABLE `table500w` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5000001 DEFAULT CHARSET=utf8
```
然后，插入500w数据
参看[010-MySQL批量插入测试数据](https://www.cnblogs.com/bjlhx/p/11949479.html)
1. 查看索引
```sql
show index from table500w
-- table500w	0	PRIMARY	1	id	A	4967713				BTREE		
```
只有主键id索引
2. count查询
```sql
select count(id) from table500w; 
select count(*) from table500w;
select count(1) from table500w; 
```
尝试多次后，耗时几乎一致，本机约6s
查看explain执行计划，
```sql
explain select count(id) from table500w; 
-- Id  select_type table    type    possiable_key   key     key_len ref rows  Extra
-- 1	  SIMPLE	  table500w	index		PRIMARY	        8		                4967713	Using index
```
三个均为一致，故网上说的count(id)、count(*)、count(1)快慢说法没有什么科学依据。用啥写法都是这么慢。
3. 非聚簇索引优化
```sql
alter table table500w add index index_age(age);
show index from table500w;
-- table500w	0	PRIMARY	1	id	A	4967713				BTREE		
-- table500w	1	index_age	1	age	A	202			YES	BTREE		
```
执行查询
```sql
select count(id) from table500w; 
select count(*) from table500w;
select count(1) from table500w; 
```
发现耗时，几乎约600ms，原因是mysql引擎做的优化，会使用占用空间较少的索引作为count（*）的命中统计
重复上述，查看执行计划，三个都一样
```sql
explain select count(id) from table500w; 
-- 1	SIMPLE	table500w	index		index_age	5		4967713	Using index
```
上述可以通过force强制使用指定索引,来查看使用聚集索引以及非聚集索引 对count影响
```sql
select count(*) from table500w FORCE index(`PRIMARY`); 
select count(*) from table500w FORCE index(index_age); 
```

## 小结
非聚集索引所占空间的大小往往，远小于聚集索引或堆表所占用的空间大小；
同样的，表中占用较少字节的字段的非聚集索引，对于速度的提升效果，也要远大于，占用较多字节的字段的非聚集索引，因为占用字节少，那么索引占用的空间也少，同样是扫描，只需要更少的时间，对硬盘的访问次数也更少，那么速度就会更快了。

- 情况一、只有主键索引，
1. 数据和主键索引存储在一个ibd中，总大小224mb，所加载文件也为全部


- 情况二、找一个短小的列age，为它建立辅助索引。
仅加载索引页统计，索引页为90mb，
1. 二级索引的存储空间仅包含length字段值(4) 、数据主键(8)，假设二级索引辅助结构不占用空间（仅计算数据占用空间）
2. 在默认情况下，MySQL的一个数据页大小为16K，一个页可存储的数据条数为 16*1024/(4+8) =1365 
3. 按照单页存储空间占用为50%（页分裂现象导致页不满）计算，500万条数据的统计需要读取约:500 0000/(1365*0.5)=7331 个物理页
4. 而页在连续的情况下，数据库一次可读取多个连续的页，数据读取总量为 16k*7331 约 114MB，使用附注方法查看约 90mb，以
5. 因mysql空间分配为按区分配，每个区1M，一次分配1-5个连续区，当数据量较小，一次仅分配一个区，112M数据会分配在114个区中，
6. 固态硬盘读取均速约 679m/s ，整个过程：io寻址时间(0ms)+读取时间（114m/679m=167ms）= 167 ms，
7. 而数据解析统计约为 30-100ms，故总耗时会在300ms加。

综上所述，纠结 count(id)、count(*) 、count(1)写法上性能没有任何意义，通过执行几化发现没有任何效率差异。关注语句真正命中的索引意义重大。

# 附注
## mac固态硬盘测试
``` bash
sudo time dd if=/dev/zero bs=1024k of=tstfile count=1024
# 1024+0 records in
# 1024+0 records out
# 1073741824 bytes transferred in 1.506577 secs (712702911 bytes/sec)
#         1.51 real         0.00 user         0.49 sys
```

速度：712702911/1024/1024=679m/s

## 索引占用空间
如上述，先删除age索引，则只剩下主键索引，主键索引和数据在一起，不被计算
```sql
-- 删除索引
alter table table500w drop index index_age;
-- 查看表中存在的索引
show index from table500w;
-- table500w	0	PRIMARY	1	id	A	4988148				BTREE		
-- 优化 清理空间
OPTIMIZE table table500w;
-- 查看 索引存储空间
select 
concat(round(SUM(DATA_LENGTH / 1024 / 1024), 2), 'MB') AS DATA_SIZE, 
concat(round(SUM(INDEX_LENGTH / 1024 / 1024), 2), 'MB') AS INDEX_SIZE, 
concat(round(SUM(DATA_FREE / 1024 / 1024), 2), 'MB') AS DATA_FREE
from information_schema.tables where table_schema='test_innodb' and table_name='table500w';
-- DATA_SIZE  INDEX_SIZE  DATA_FREE
--  223.78MB	0.00MB	0.00MB
-- 创建索引
alter table table500w add index index_age(age);
-- 查看表中存在的索引
show index from table500w;
-- table500w	0	PRIMARY	1	id	A	4988148				BTREE		
-- table500w	1	index_age	1	age	A	202			YES	BTREE		
-- 查看 索引存储空间
select 
concat(round(SUM(DATA_LENGTH / 1024 / 1024), 2), 'MB') AS DATA_SIZE, 
concat(round(SUM(INDEX_LENGTH / 1024 / 1024), 2), 'MB') AS INDEX_SIZE, 
concat(round(SUM(DATA_FREE / 1024 / 1024), 2), 'MB') AS DATA_FREE
from information_schema.tables where table_schema='test_innodb' and table_name='table500w';
-- DATA_SIZE  INDEX_SIZE  DATA_FREE
--  223.78MB	89.66MB	0.00MB
```