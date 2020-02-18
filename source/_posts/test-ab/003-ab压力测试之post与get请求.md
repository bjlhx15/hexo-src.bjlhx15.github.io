---
title: 003-ab压力测试之post与get请求
categories:
  - test-ab
abbrlink: 8dd4d804
date: 2020-02-14 14:05:44
---

摘要：使用ab检测指定地址 处理问题能力。
<!-- more -->

# 模拟get请求

直接在url后面带参数即可
``` BASH
ab -c 10 -n 10 http://www.test.api.com/?gid=2
```
# 模拟post请求

## application/x-www-form-urlencoded

在当前目录下创建一个文件post.txt,编辑文件post.txt写入
```
cid=4&status=1
```
相当于post传递cid,status参数
``` BASH
ab -n 100  -c 10 -p 'post.txt' -T 'application/x-www-form-urlencoded' 'http://test.api.com/ttk/auth/info/'
```
