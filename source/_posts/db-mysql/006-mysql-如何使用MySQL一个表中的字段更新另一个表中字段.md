---
title: 006-mysql-如何使用MySQL一个表中的字段更新另一个表中字段
categories:
  - db-mysql
abbrlink: f669f815
date: 2021-01-01 14:08:12
---
摘要：006-mysql-如何使用MySQL一个表中的字段更新另一个表中字段
<!-- more -->

# 查看MYSQL数据库中所有用户

## 方式一、join
```sql
update personal a inner join user_info b  on a.user_id = b.id 
set a.create_time = b.createtime;
```

## 方式二、where条件
```sql
update personal a , user_info b
set a.create_time = b.createtime
where a.user_id = b.id;
```

## 方式三、子查询
```sql
update personal a 
set a.create_time = (select name from user_info b where b.id = a.user_id);
```
