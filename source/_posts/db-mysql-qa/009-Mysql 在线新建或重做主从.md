---
title: 009-Mysql 在线新建或重做主从
categories:
  - db-mysql-qa
abbrlink: 4f016687
date: 2020-03-19 22:16:27
---

摘要：009-Mysql 在线新建或重做主从
以前停服做主从的主要目的是想锁表，是想找到 master_log_file 和 master_log_pos 两个参数。如果有方法在不停服的情况下，能确定这两个参数，那么在线建立主从架构的功能，就可以实现了。
<!-- more -->
注意：主端不停服的前提是，它已经开启了bin-log 日志！！

# 主从新建过程
如果之前在主库没有开启 bin-log 日志，那就没有办法，因为配置 bin-log 日志之后，主库一定要重启才能生效。不过，如果现在的情况是重做主库，那就证明之前是做过主从的，只是可能主从失效了需要重做。这种情况，主库也不需要重启，只要重新备份一下数据库，就可以重建从库了。

## 主从新建过程-主库操作

### 主库配置
在主库修改配置文件 my.cnf ，添加开启 bin-log 日志，格式用 row

server-id=1
log-bin=mysql-bin
binlog-format=ROW

如果之前已经开启了bin-log 功能，就不用修改了。
### 主库创建用户以及授权
接着在主库上进行备份用户的授权操作：
```sql
grant replication slave on *.* to 'repel'@'172.10.0.1' identified by 'password';
```
授权给从库的 ip 地址，备份的用户名是 repel，建议不要使用 'repel'@'%' 这种方式进行授权操作，主要是为了安全问题，限制授权的 ip 白名单。

### 主库将需要备份的数据库导出来：
用 mysqldump 的方式，以下是导出整个数据库：
```sql
mysqldump -uroot -p  --single-transaction --no-autocommit --master-data=2 -A >test2.sql
```
用 mysqldump 的方式，如果主端 Mysql 上有不止一个项目的业务库，但是只想导出其中一个业务库，假设叫做 hr_mysql 数据库：
```sql
mysqldump -uroot -p  --single-transaction --no-autocommit --master-data=2  hr_mysql > hr_mysql_dump.sql
```
关键的参数 "--master-data=2"，能帮助实现在线重建主从数据库。
将hr_mysql_dump.sql传输给从库。

## 主从新建过程-从库操作

### 从库配置
修改从库 my.cnf ，只备份 hr_mysql 业务库，所以只增加了：
```
server-id=2
replicate_wild_do_table=hr_mysql.%
```
其实如果进行的是全库备份，那么只要配置 server-id 和主库不一样就可以了，其他的不需要增加。
但是，如果只想备份某些特定的业务库，就需要使用 replicate_wild_do_table 这个参数了。它的作用是告诉从库只需要备份特定的库，如果有多个库，就继续用逗号隔开添加即可。
还有一个功能相反的参数 replicate_wild_ignore_table ,是配置忽略的数据库，即不备份的数据库，这个参数使用起来比较多限制，不熟悉最好不使用。

### 导入主库数据
接下来是导入数据库，在导入数据库前，需要重置一下从库binlog：
```sql
mysql -uroot -p   -e 'reset master'
mysql -uroot -p   hr_mysql < hr_mysql_dump.sql
```

### 主从配置
在备份数据库的时候，使用了一个关键的参数 --master-data=2。在备份文件中，可以看到需要的 master_log_file 和 master_log_pos 两个参数。有了这两个参数，我们就不需要像以往那样停服锁表，来查看了。

具体怎么查找这两个参数？它们大概在dump文件的前 30 行以内，所以可以用 head 命令找到：
```sql
head -n 30 feitian_dump.sql
```
在上面的输出，可以看到 binlog 是 mysql-bin.000063，position 是 144333309.有了这两个参数，就可以配置从库同步了。

一般在从库 change master to 就可以了。

所以，在从库做以下配置：