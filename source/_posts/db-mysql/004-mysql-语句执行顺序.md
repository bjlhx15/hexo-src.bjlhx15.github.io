---
title: 004-mysql-语句执行顺序
categories:
  - db-mysql
abbrlink: 55a48c08
date: 2020-02-20 08:51:33
---
摘要：004-mysql-语句执行顺序
<!-- more -->

# mysql 语句语法结构
```sql
SELECT 
DISTINCT <select_list>
FROM <left_table>
<join_type> JOIN <right_table>
ON <join_condition>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

# mysql 语句执行顺序


MySQL的语句，最先执行的总是FROM操作，最后执行的是LIMIT操作。其中每一个操作都会产生一张虚拟的表，这个虚拟的表作为一个处理的输入，只是这些虚拟的表对用户来说是透明的，但是只有最后一个虚拟的表才会被作为结果返回。如果没有在语句中指定对应的操作，那么将会跳过相应的步骤。

1. from:需要从哪个数据表检索数据，对FROM的左边的表和右边的表计算笛卡尔积。产生虚表VT1
2. on：在生成临时表时使用的条件，对虚表VT1进行ON筛选，只有那些符合<join-condition>的行才会被记录在虚表VT2中。
3. join：联合多表查询返回记录时，并生成一张临时表，如果指定了OUTER JOIN（比如left join、 right join），那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3, rug from子句中包含两个以上的表的话，那么就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1~3这三个步骤，一直到处理完所有的表为止。
4. where:过滤表中数据的条件，对虚拟表VT3进行WHERE条件过滤。只有符合<where-condition>的记录才会被插入到虚拟表VT4中。
5. group by:如何将上面过滤出的数据分组，根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5.
6. CUBE | ROLLUP: 对表VT5进行cube或者rollup操作，产生表VT6.
7. having:对上面已经分组的数据进行过滤的条件，对虚拟表VT6应用having过滤，只有符合<having-condition>的记录才会被 插入到虚拟表VT7中。
8. select:查看结果集中的哪个列，或列的计算结果，执行select操作，选择指定的列，插入到虚拟表VT8中。
9. DISTINCT: 对VT8中的记录进行去重。产生虚拟表VT9.
10. order by :按照什么样的顺序来查看返回的数据,将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生虚拟表VT10.
11. limit：限制查询结果返回的数量,取出指定行的记录，产生虚拟表VT11, 并将结果返回。

## on与where的用法区别：
1. on后面的筛选条件主要是针对的是关联表【而对于主表刷选条件不适用】。
2. 如果是想再连接完毕后才筛选就应把条件放置于where后面。对于关联表我们要区分对待。如果是要条件查询后才连接应该把查询件放置于on后。
3. 对于主表的筛选条件应放在where后面，不应该放在on后面

## having和where的用法区别：
1. having只能用在group by之后，对分组后的结果进行筛选(即使用having的前提条件是分组)。
2. where肯定在group by 之前，即也在having之前。
3. where后的条件表达式里不允许使用聚合函数，而having可以。

## count用法
统计某个列值的数量，也可以统计行数
count(*) 和count(1) 都是统计行数，
count(col) 是统计col列非null的行数





