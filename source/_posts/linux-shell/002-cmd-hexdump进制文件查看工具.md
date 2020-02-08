---
title: 002-cmd-hexdump进制文件查看工具
categories:
  - linux-shell
abbrlink: 653046b
date: 2020-02-03 22:32:41
tags:
---

# 简介

hexdump是Linux下的一个二进制文件查看工具，它可以将二进制文件转换为ASCII、八进制、十进制、十六进制格式进行查看。

指令所在路径：/usr/bin/hexdump

# 语法
``` bash
hexdump: [-bcCdovx] [-e fmt] [-f fmt_file] [-n length] [-s skip] [file ...]
```
# 参数
此命令参数是Red Hat Enterprise Linux Server release 5.7下hexdump命令参数，不同版本Linux的hexdump命令参数有可能不同。
```text
-b              one-byte octal display           8进制显示
-c              one-byte character display       ASCII显示
-C              canonical hex+ASCII display       十六进制+ASCII显示
-d              two-byte decimal display        两字节计算，显示为10进制方式
-o              two-byte octal display         两字节计算，显示为8进制方式
-x              two-byte hexadecimal display    两字节计算，显示为16进制方式
-e format       format string to be used for displaying data   格式化输出
-f format_file  file that contains format strings
-n length       interpret only length bytes of input    输出多少个bytes的字符长度的内容
-s offset       skip offset bytes from the beginning    输出文件的开始偏移量  【注意：偏移量从0开始的！】
-v              display without squeezing similar lines    
-V              output version information and exit
```

# 使用
1. 帮助
```bash
hexdump --help
```
2. 案例
```bash
cat >test.txt
ABCDEF    
GHIJKM
123456
```
``` bash
$ hexdump -C test.txt
00000000  41 42 43 44 45 46 0a 47  48 49 4a 4b 4d 0a 31 32  |ABCDEF.GHIJKM.12|
00000010  33 34 35 36 0a                                    |3456.|
00000015
(base) 
```

跳过 7 个取6个
``` bash 
$ hexdump -C -s 7 -n 6 test.txt
00000007  47 48 49 4a 4b 4d                                 |GHIJKM|
0000000d
(base) 
```




