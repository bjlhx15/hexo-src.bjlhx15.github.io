---
title: 005-cmd-stat显示与touch修改文件的各种时间
categories:
  - linux-shell
abbrlink: ced46313
date: 2020-02-11 09:01:06
tags:
---

摘要：stat查看具体，touch操作具体
主要是测试log4j2日志删除策略时候使用。
<!-- more -->

# stat显示指定文件的状态信息

## 查看帮助
- mac
``` bash
stat -?
# stat: illegal option -- ?
# usage: stat [-FlLnqrsx] [-f format] [-t timefmt] [file ...]
```
- centos
``` BASH
stat --help
# 用法：stat [选项]... 文件...
# Display file or file system status.

#   -L, --dereference     follow links
#   -Z, --context         print the SELinux security context 
#   -f, --file-system     display file system status instead of file status
#   -c --format=格式	使用指定输出格式代替默认值，每用一次指定格式换一新行
#       --printf=格式	类似 --format，但是会解释反斜杠转义符，不使用换行作
# 				输出结尾。如果您仍希望使用换行，可以在格式中
# 				加入"\n"
#   -t, --terse		使用简洁格式输出
#       --help		显示此帮助信息并退出
#       --version		显示版本信息并退出
```

## 查看具体时间
- mac
一次尝试一下其中含义，发现 -x 比较容易理解各种时间

``` BASH
stat -x text.txt
#   File: "text.txt"
#   Size: 6            FileType: Regular File
#   Mode: (0644/-rw-r--r--)         Uid: (545858136/lihongxu6)  Gid: (699739227/(699739227))
# Device: 1,4   Inode: 15213285    Links: 1
# Access: Tue Feb 11 08:53:33 2020
# Modify: Tue Feb 11 08:53:32 2020
# Change: Tue Feb 11 08:53:32 2020
```

Access是文件访问时间，Modify是文件内容最后修改时间，Change是文件属性最后修改时间，分别对应时间戳atime/mtime/ctime。  
Change时间比较特殊，当改变文件的名称，大小和权限的时候Change时间才会改变。

通过上述发现并没有包含文件的创建时间，即crtime。查看源码可知，这是因为inode结构体中并没有crtime。

- centos
``` BASH
stat  test.txt 
#   File: "test.txt"
#   Size: 0         	Blocks: 0          IO Block: 4096   普通空文件
# Device: fc01h/64513d	Inode: 1579964     Links: 1
# Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
# Access: 2020-02-11 10:07:53.343752608 +0800
# Modify: 2020-02-11 10:08:14.709608664 +0800
# Change: 2020-02-11 10:08:14.709608664 +080
```

### 查看文件的创建时间。
- mac
Mac OS X上没有debugfs（8）。Debugfs（8）是用于调试Linux文件系统ext2 / ext3的Linux程序。  
可以使用： HFS+ try fsck(8) or use Disk Utility. 或者自带命令： GetFileInfo
``` BASH
GetFileInfo text.txt 
# file: "/Users/lihongxu6/IdeaProjectsGit/shell/test/fileop/text.txt"
# type: "\0\0\0\0"
# creator: "\0\0\0\0"
# attributes: avbstclinmedz
# created: 02/11/2020 08:53:27
# modified: 02/11/2020 08:53:32
```

- centos
1. 查看文件的inode号
``` BASH
stat test.txt
# 或
stat -x text.txt
```
inode:15213285
2. 输出分区
``` BASH
df test.txt
```
``` BASH
# df text.txt                         
# Filesystem   512-blocks      Used Available Capacity iused               ifree %iused  Mounted on
# /dev/disk1s1  489620264 215965128 243191936    48% 2289359 9223372036852486448    0%   /
```

3. 通过debugfs就可以查询到文件的完整信息
linux
``` BASH
debugfs -R 'stat <15213285>' /dev/disk1s1
```

# linux修改文件各种时间
查看下
``` BASH
stat  test.txt 
#   File: "test.txt"
#   Size: 0         	Blocks: 0          IO Block: 4096   普通空文件
# Device: fc01h/64513d	Inode: 1579964     Links: 1
# Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
# Access: 2020-02-11 10:07:53.343752608 +0800
# Modify: 2020-02-11 10:08:14.709608664 +0800
# Change: 2020-02-11 10:08:14.709608664 +080
```

## 修改修改时间
``` BASH
# 文件修改时间设置为：2020年02月11日09:17:52
touch -t 202002110917.52 test.txt
# or
touch -d "2020-02-11 09:18:08" test.txt
# 查看实际是否修改
stat test.txt
```

