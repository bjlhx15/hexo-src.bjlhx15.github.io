---
title: 008-cmd-文件cat、touch、echo
categories:
  - linux-shell
abbrlink: 341c10ca
date: 2020-02-24 14:30:44
tags:
---

摘要：008-cmd-文件cat、touch、echo

<!-- more -->

# cat
cat命令是linux下的一个文本输出命令，通常是用于观看某个文件的内容

- cat主要有三大功能：
1. 一次显示整个文件。
```bash
cat filename
```
2. 从键盘创建一个文件。
``` BASH
cat  >  filename
```
只能创建新文件,不能编辑已有文件.
3. 将几个文件合并为一个文件。
``` BASH
cat   file1   file2  > file
```
两种均可，后者，支持嵌套

## 语法
cat具体命令格式为 : cat [-AbeEnstTuv] [--help] [--version] fileName

说明：把档案串连接后传到基本输出(屏幕或加 > fileName 到另一个档案)

参数：
> -n 或 –number 由 1 开始对所有输出的行数编号
> -b 或 –number-nonblank 和 -n 相似，只不过对于空白行不编号
> -s 或 –squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行
> -v 或 –show-nonprinting
## 示例
1. 把 linuxfile1 的档案内容加上行号后输入 linuxfile2 这个档案里
``` BASH
cat -n linuxfile1 > linuxfile2
```
2. 把 linuxfile1 和 linuxfile2 的档案内容加上行号(空白行不加)之后将内容附加到 linuxfile3 里。
``` BASH
cat -b linuxfile1 linuxfile2 >> linuxfile3
cat /dev/null > /etc/test.txt 此为清空/etc/test.txt档案内容
```

# touch 

一是用于把已存在文件的时间标签更新为系统当前的时间（默认方式），它们的数据将原封不动地保留下来；

二是用来创建新的空文件。

``` text
 -a或--time=atime或--time=access或--time=use  只更改存取时间。
 -c或--no-create  不建立任何文件。
 -d<时间日期>  使用指定的日期时间，而非现在的时间。
 -f  此参数将忽略不予处理，仅负责解决BSD版本touch指令的兼容性问题。
 -m或--time=mtime或--time=modify  只更改变动时间。
 -r<参考文件或目录>  把指定文件或目录的日期时间，统统设成和参考文件或目录的日期时间相同。
 -t<日期时间>  使用指定的日期时间，而非现在的时间。
 --help  在线帮助。
 --version  显示版本信息。
```

## 示例
1. 创建file1—file10共10个文件
``` BASH
touch file{1..10}
```
2. 设定文件的时间戳
``` BASH
touch -t 201810121230 hh.sh 【-t用十进制数】
```
3. 更新log.log的时间和log2012.log时间戳相同
``` BASH
touch -r hh hh.sh    【touch -r目标文件  源文件】
```




