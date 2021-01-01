---
title: 007-binlog
categories:
  - db-mysql-qa
abbrlink: feb3a22e
date: 2020-02-20 11:36:16
---

摘要：007-binlog
<!-- more -->

# binlog

## 日志的三种模式

系统变量binlog_format 指定二进制日志的类型。分别有STATEMENT、ROW、MIXED三种值。MySQL 5.7.6之前默认为STATEMENT模式。MySQL 5.7.7之后默认为ROW模式。这个参数主要影响主从复制。

开启和停用Binlog：log-bin=mysql-bin

查看binlog的格式：show variables like 'binlog_format'

基于SQL语句的复制（statement-based replication, SBR）
基于行的复制（row-based replication, RBR）
混合模式复制（mixed-based replication, MBR）

### statement level模式

每一条会修改数据的sql都会记录到master的bin-log中。slave在复制的时候sql进程会解析成和原来master端执行过的相同的sql来再次执行。
- 适用场景：对主从数据一致性要求不太高，并且很少用到函数、存储过程、触发器等场景
- 优点：statement level下的优点，首先就是解决了row level下的缺点，不需要记录每一行数据的变化，减少bin-log日志量，节约io，提高性能。因为他只需要记录在master上所执行的语句的细节，以及执行语句时候的上下文的信息。
- 缺点：由于它是记录的执行语句，所以为了让这些语句在slave端也能正确执行，那么他还必须记录每条语句在执行的时候的一些相关信息，也就是上下文信息，以保证所有语句在slave端被执行的时候能够得到和在master端执行时候相同的结果。另外就是,由于mysql现在发展比较快，很多的新功能加入，使mysql的复制遇到了不小的挑战,自然复制的时候涉及到越复杂的内容，bug也就越容易出现。在statement level下，目前已经发现的就有不少情况会造成mysql的复制问题，主要是修改数据的时候使用了某些特定的函数或者功能的时候会出现，比如sleep()在有些版本就不能正确复制。
部分新功能（函数、存储过程、触发器）同步会有障碍，比如now()

### rowlevel模式
5.1.5版本的MySQL才开始支持row level的复制,它不记录sql语句上下文相关信息，仅保存哪条记录被修改。

日志中会记录成每一行数据被修改的形式，然后在slave端再对相同的数据进行修改
- 适用场景：对主从数据一致性要求比较高的场景。
- 优点：bin-log中可以不记录执行的sql语句的上下文相关的信息，仅仅只需要记录那一条记录被修改了，修改成什么样了。所以row level的日志的内容会非常清楚的记录下每一行数据修改的细节。而且不会出现某些特定情况下的存储过程，或function,以及trigger的调用和触发无法被正确复制的问题。
- 缺点：row level下，所有的执行的语句当记录到日志中的时候，都将以每行记录的修改记录，这样可能会产生大量的日志内容，比如有这样一条update语句：update product set owner_member_id=‘d’ where owner_member_id=‘a’,执行之后，日志中记录的不是这条update语句所对应的事件(mysql是以事件的形式来记录bin-log日志)，而是这条语句所更新的每一条记录的变化情况，这样就记录成很多条记录被更新的很多事件。自然，bin-log日志的量会很大。

### mixed模式
从5.1.8版本开始，MySQL提供了Mixed格式，实际上就是Statement与Row的结合

实际上就是前两种模式的结合，在mixed模式下，mysql会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在statement和row之间选一种。新版本中的statement level还是和以前一样，仅仅记录执行的语句。而新版本的mysql中对row level模式被做了优化，并不是所有的修改都会以row level来记录，像遇到表结构变更的时候就会以statement模式来记录，如果sql语句确实就是update或者delete 等修改数据的语句，那么还是会记录所有行的变更。

MySQL默认采用statement格式进行二进制日志文件的记录，但是在一些情况下会使用row格式，可能的情况有：
>   1）、表的存储引擎为NDB，此时对表的DML操作都会以ROW格式记录
>   2）、使用了UUID(),USER(),CURRENT_USER(),FOUND_ROWS(),ROW_count()等不确定函数时
>   3）、使用了insert delay语句
>   4）、使用了用户定义函数（UDF）
>   5）、使用了临时表
- 适用场景：对主从数据一致性要求不太高，可能会用到函数、存储过程、触发器等场景
- 优缺点介于statement和row模式之间

# 主从复制

## MySQL主从复制的原理
（1）、主库必须开启二进制日志
（2）、当有增删改的语句时，会记录到主库的binlog中
（3）、主库通过IO线程把binlog里面的内容传给从库的relay binlog（中继日志）（这是msyql复制是异步复制的原因）
（4）、从库的sql线程负责读取它的relay log里的信息并应用到数据库中
## Seconds_Behind_Master的原理。
表示sql线程和io线程之间的时间差
具体的计算：从库服务器当前的时间戳与二进制日志中的事件的时间戳相对比得到的，所以只有在执行事件时才能报告延迟。
不足：
一些错误（例如主备的max_allowed_packet不匹配，或者网络不稳定）可能中断复制，由于主从复制是异步操作，Seconds_Behind_Master可能显示为0
## 主从延迟的主要原因有哪些？
（1）、慢SQL语句过多
（2）、从库的硬件比主库差
（3）、同一个主库下有过多的从库
（4）、网络延迟
（5）、表分区过多

# CentOs7.6基于docker搭建主从集群
## 简介
Mysql主从又叫Replication、AB复制。简单讲就是A与B两台机器做主从后，在A上写数据，另外一台B也会跟着写数据，实现数据实时同步
mysql主从是基于binlog，主上需开启binlog才能进行主从

### 主从过程大概有3个步骤
1. 主将更改操作记录到binlog里
2. 从将主的binlog事件（sql语句） 同步本机上并记录在relaylog里
3. 从根据relaylog里面的sql语句按顺序执行

### 主从作用
1. 实时灾备，用于故障切换
2. 读写分离，提供查询服务
3. 备份，避免影响业务

### 主从形式
* 一主一从
* 主主复制
* 一主多从---扩展系统读取的性能，因为读是在从库读取的
* 多主一从---5.7版本开始支持
* 联级复制

### 主从复制原理
![](/images/post/db-mysql/mysqlrepl.webp)
1. 主库将所有的写操作记录在binlog日志中，并生成log dump线程，将binlog日志传给从库的I/O线程
2. 从库生成两个线程，一个是I/O线程，另一个是SQL线程
3. I/O线程去请求主库的binlog日志，并将binlog日志中的文件写入relay log（中继日志）中
4. SQL线程会读取relay loy中的内容，并解析成具体的操作，来实现主从的操作一致，达到最终数据一致的目的

### 主从复制配置步骤：
1. 确保从数据库与主数据库里的数据一致
2. 在主数据库里创建一个同步账户授权给从数据库使用
3. 配合主数据库（修改配置文件）
4. 配置从数据库（修改配置文件）

## 准备工作
搭建两台MYSQL服务器，一台作为主服务器，一台作为从服务器，主服务器进行写操作，从服务器进行读操作

### 环境说明

```
数据库角色	IP	应用与系统	有无数据
主数据库	192.168.55.130	centos7 mysql-5.7	有
从数据库	192.168.55.129	centos7 mysql-5.7	无
```

### docker安装
``` bash
    yum install docker -y
    docker version
    systemctl start docker
    systemctl enable docker #开机启动docker
    docker version
    systemctl status docker
```

详细参看：https://github.com/bjlhx15/shell 的 cmd/centos/docker

### docker安装mysql

#### docker安装mysql以及配置文件
```bash
docker pull mysql:5.7
```
##### 查看配置文件以及共享配置【过程说明】
```bash
docker run -p 53306:3306 --name mymysql57 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
docker exec -it mymysql57 /bin/bash
cat /etc/mysql/mysql.conf.d/mysqld.cnf
```
在宿主机上创建文件
```bash
mkdir -p /export/Data/docker/mymysql57/conf
vim /export/Data/docker/mymysql57/conf/mysqld.cnf
```
输入如下：
```
[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
#log-error	= /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address	= 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
```
启动并添加目录映射
``` bash
docker run -d -p 53306:3306 --name mymysql57 \
 -v /export/Data/docker/mymysql57/conf:/etc/mysql/mysql.conf.d \
 -v /export/Data/docker/mymysql57/data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=123456  mysql:5.7
```

详细参看：https://github.com/bjlhx15/shell 的 cmd/centos/docker

#### Docker搭建主从服务器

* Master(主)：

``` bash
mkdir -p /export/Data/docker/mysql_master/conf
vim /export/Data/docker/mysql_master/conf/mysqld.cnf
```
输入如下：
```
[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
#log-error	= /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address	= 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

## 开启 主从配置
## 同一局域网内注意要唯一
server-id=100  
## 开启二进制日志功能，可以随便取（关键）
log-bin=mysql-bin
#binlog-do-db=zn                   #可以被从服务器复制的库, 二进制需要同步的数据库名
#binlog-ignore-db=mysql            #不可以被从服务器复制的库
```
```bash
docker run -d -p 53306:3306 --name mysql_master \
 -v /export/Data/docker/mysql_master/conf:/etc/mysql/mysql.conf.d \
 -v /export/Data/docker/mysql_master/data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=123456  mysql:5.7
```

* Slave(从)：
``` bash
mkdir -p /export/Data/docker/mysql_slave/conf
vim /export/Data/docker/mysql_slave/conf/mysqld.cnf
```
输入如下：
```
[client]
default-character-set = utf8
 
[mysql]
default-character-set = utf8

[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
#log-error	= /var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address	= 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character-set-server = utf8

## 设置server_id,注意要唯一
server-id=101  
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin   
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin
log_slave_updates=1
replicate-do-db=contract
sql_mode=NO_AUTO_VALUE_ON_ZERO,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE
```
```bash
docker run -d -p 53307:3306 --name mysql_slave \
 -v /export/Data/docker/mysql_slave/conf:/etc/mysql/mysql.conf.d \
 -v /export/Data/docker/mysql_slave/data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=123456  mysql:5.7
```

Master对外映射的端口是53306，Slave对外映射的端口是53307。因为docker容器是相互独立的，每个容器有其独立的ip，所以不同容器使用相同的端口并不会冲突。
```bash
docker ps -a 
```
可以使用Navicat等工具测试连接mysql

### 账户配置及说明
进入主数据库
```bash
docker exec -it mysql_master /bin/bash
```
在Master数据库创建数据同步用户，授予用户 slave的 REPLICATION SLAVE权限和REPLICATION CLIENT权限，用于在主从库之间同步数据。
```sql
mysql -uroot -p123456
CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```
这里表示创建一个slaver同步账号slave，允许访问的IP地址为%，%表示通配符

### 链接Master(主)和Slave(从)
#### 进入主数据库
```bash
docker exec -it mysql_master /bin/bash
```
```sql
mysql -uroot -p123456
# 增加读锁，放置建立期间有人写入数据，查看完毕不要关闭窗口，unlock tables; 释放锁
flush tables with read lock;
show master status;
-- +------------------+----------+--------------+------------------+-------------------+
-- | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
-- +------------------+----------+--------------+------------------+-------------------+
-- | mysql-bin.000001 |      617 |              |                  |                   |
-- +------------------+----------+--------------+------------------+-------------------+
-- 1 row in set (0.00 sec)
```
File和Position字段的值后面将会用到，在后面的操作完成之前，需要保证Master库不能做任何操作，否则将会引起状态变化，File和Position字段的值变化。
mysqlbinlog mysql-bin.000001
##### 导出数据
mysqldump -h127.0.0.1 -uroot -p123456 --default-character-set=utf8 contract>contract.sql

注意查看字符集
系统参数 
```sql
 show variables like 'char%'
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```
上述导出：一般character_set_database 相一致：--default-character-set=utf8

#### 进入从数据库
```bash
docker exec -it mysql_slave /bin/bash
```
```sql
mysql -uroot -p123456
-- 创建从库
create database contract2 default character set utf8;
-- 导入已有数据
-- 本机上传文件：scp /Users/lihongxu/work/contract.sql root@10.0.01:/export/Data/contract.sql
-- 容器拷贝：docker cp /home/trace/contract_dll_dml_0326.sql mysql_hr_prod_slave:/
-- 重置binlog,导入数据
mysql -uroot -p   -e 'reset master'
mysql -uroot -p   contract < contract_dll_dml_0326.sql
-- 从库配置链接主库
change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos= 617, master_connect_retry=30;
```
命令说明：
```
master_host ：Master的地址，指的是容器的独立ip,可以通过docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称|容器id  查询容器的ip
master_port：Master的端口号，指的是容器的端口号
master_user：用于数据同步的用户
master_password：用于同步的用户的密码
master_log_file：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
master_log_pos：从哪个 Position 开始读，即上文中提到的 Position 字段的值
master_connect_retry：如果连接失败，重试的时间间隔，单位是秒，默认是60秒
```
在Slave 中的mysql终端执行show slave status \G;用于查看主从同步状态。
```sql
start slave;    -- 开启从  stop slave 停止从
show slave status \G;
# 主要关注参数 
 #  Slave_IO_Running: No
 # Slave_SQL_Running: No
```
正常情况下，SlaveIORunning 和 SlaveSQLRunning 都是No，因为还没有开启主从复制过程。
使用start slave开启主从复制过程，然后再次查询主从同步状态show slave status \G;。
```sql
start slave
show slave status \G;
```
使用start slave开启主从复制过程后，查看状态，如果SlaveIORunning一直是Connecting，则说明主从复制一直处于连接状态，这种情况一般是下面几种原因造成的，可以根据 Last_IO_Error提示予以排除。

网络不通:检查ip,端口
密码不对:检查是否创建用于同步的用户和用户密码是否正确
pos不对:检查Master的 Position

测试主从复制
在Master创建一个数据库，然后检查Slave是否存在此数据库。
Master:
```bash
docker exec -it mysql_master /bin/bash
```
```sql
mysql -uroot -p123456
create database test;
```