---
title: 007-shell-编程注意事项、引号区别
categories:
  - linux-shell
abbrlink: b4490d2f
date: 2020-02-12 14:14:31
tags:
---

摘要：shell开发时，注意事项

<!-- more -->

# 单引号、双引号、反单引号的区别
```
单引号' '：包围变量的值时，原样输出。强引用。这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景。

双引号" "：包围变量的值时，解析里面的变量和命令。弱引。这种方式比较适合字符串中附带有变量和命令并且想将其解析后再输出的变量定义。

反引号` `：一般用于命令，执行的时候命令会被执行，相当于$()，赋值和输出都要用反引号引起来。
```

两种均可，后者，支持嵌套

# Shell编程注意事项
1. 变量赋值时‘=’两边不能有空格
2. 使用[]命令测试表达式的时候，在操作数和操作符或者方括号的前后都要至少留一个空格
3. 变量的引用： 使用$var 或者 ${var}
4. 命令的引用：【执行命令返回值给变量】
```
var2=`command` 或者 var3=$(command)
```




