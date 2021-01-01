---
title: 008-mysql5.7配置文件
categories:
  - db-mysql-qa
abbrlink: c5078da8
date: 2020-03-19 18:43:59
---

摘要：008-mysql5.7配置文件
<!-- more -->

# docker
```bash
docker run -p 53306:3306 --name mymysql57 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
docker exec -it mymysql57 /bin/bash
```
进入docker后查看
```bash
ls /etc
```
没有 my.cnf,存在mysql
```bash
ls /etc/mysql
# conf.d	my.cnf	my.cnf.fallback  mysql.cnf  mysql.conf.d
cat /etc/mysql/my.cnf
# 意思是配置文件在这两个目录下
# !includedir /etc/mysql/conf.d/
# !includedir /etc/mysql/mysql.conf.d/
cat /etc/mysql/mysql.cnf
# 意思是配置文件在这两个目录下
# !includedir /etc/mysql/conf.d/
# !includedir /etc/mysql/mysql.conf.d/

# ------------------------------------------
ls /etc/mysql/conf.d/
# docker.cnf  mysql.cnf  mysqldump.cnf
cat /etc/mysql/conf.d/docker.cnf
# [mysqld]
# skip-host-cache
# skip-name-resolve
cat /etc/mysql/conf.d/mysql.cnf
# 空
cat /etc/mysql/conf.d/mysqldump.cnf
# [mysqldump]
# quick
# quote-names
# max_allowed_packet	= 16M

# ------------------------------------------
ls /etc/mysql/mysql.conf.d/
# mysqld.cnf
cat /etc/mysql/mysql.conf.d/mysqld.cnf
# [mysqld]
# pid-file	= /var/run/mysqld/mysqld.pid
# socket		= /var/run/mysqld/mysqld.sock
# datadir		= /var/lib/mysql
# #log-error	= /var/log/mysql/error.log
# # By default we only accept connections from localhost
# #bind-address	= 127.0.0.1
# # Disabling symbolic-links is recommended to prevent assorted security risks
# symbolic-links=0
```

故核心配置在： /etc/mysql/mysql.conf.d/mysqld.cnf

