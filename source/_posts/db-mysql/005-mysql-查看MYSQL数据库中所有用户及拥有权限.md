---
title: 005-mysql-查看MYSQL数据库中所有用户及拥有权限
categories:
  - db-mysql
abbrlink: b9b0f2a9
date: 2020-03-19 21:43:47
---
摘要：005-mysql-查看MYSQL数据库中所有用户及拥有权限
<!-- more -->

# 查看MYSQL数据库中所有用户
```sql
SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;
-- +------------------------------------+
-- | query                              |
-- +------------------------------------+
-- | User: 'root'@'%';                  |
-- | User: 'mysql.session'@'localhost'; |
-- | User: 'mysql.sys'@'localhost';     |
-- | User: 'root'@'localhost';          |
-- +------------------------------------+
```
查看数据库中具体某个用户的权限
```sql
show grants for 'root'@'%'; 
-- +-------------------------------------------------------------+
-- | Grants for root@%                                           |
-- +-------------------------------------------------------------+
-- | GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION |
-- +-------------------------------------------------------------+
-- 1 row in set (0.00 sec)
```

```sql
select * from mysql.user where user='root' \G   
-- *************************** 1. row ***************************
--                   Host: localhost
--                   User: root
--            Select_priv: Y
--            Insert_priv: Y
--            Update_pr
--              ……
```

查看user表结构　需要具体的项可结合表结构来查询
```sql
desc mysql.user;
```