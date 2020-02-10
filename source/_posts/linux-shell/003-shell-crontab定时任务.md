---
title: 003-shell-crontab定时任务
categories:
  - linux-shell
abbrlink: a16096f7
date: 2020-02-08 15:10:01
tags:
---

摘要：有时需要操作系统统，定时做一些任务，解决一些问题。
通过crontab 命令，我们可以在固定的间隔时间执行指定的系统指令或 shell script脚本。时间间隔的单位可以是分钟、小时、日、月、周及以上的任意组合。这个命令非常适合周期性的日志分析或数据备份等工作。

<!-- more -->
# 简介

说明：
1. Linux和Mac下操作crontab都是一致的
2. 配置文件都在/etc/crontab下，如果没有就创建。默认系统级别存在，不需要额外安装定时任务
3. 测试直接使用crontab -e命令创建的定时任务是放在临时文件夹的，重启会删除，并且与/etc/crontab文件无关联。

## crontab服务的重启关闭，开启

系统级别，是linux下用来周期性的执行某种任务或等待处理某些事件的一个守护进程。所以下述命令不可用。
- Mac系统下
```bash
sudo /usr/sbin/cron start
sudo /usr/sbin/cron restart
sudo /usr/sbin/cron stop
```
- Ubuntu
```bash
sudo /etc/init.d/cron start
sudo /etc/init.d/cron stop
sudo /etc/init.d/cron restart
```

## 常用命令
查看定时任务：`crontab -l`  
列出用户test的所有调度任务:`crontab -l -u test`  
删除所有调度任务：`crontab -r`

# 使用

## 使用方式
- 方式一、系统级别-/etc/crontab 方式
```bash
sudo vim /etc/crontab
*/1 * * * * root /bin/date >> /tmp/time2.txt
```
保存使用：`:wq!`；默认超管只读权限，需要强制保存退出

- 方式二、用户级别-crontab -e 自定义脚本启动
1. 执行脚本编写
``` bash
# 每分钟执行一次date命令，输出时间到time.txt文本
*/1 * * * * /bin/date >> /tmp/time.txt
```
2. crontab命令调用crontab文件
``` bash
crontab testing_crontab
```
3. 查看 文件内容
```bash
tail -f  /tmp/time.txt
```

## /etc/crontab文件和crontab -e命令区别
1. 修改/etc/crontab这种方法只有root用户能用，这种方法更加方便与直接直接给其他用户设置计划任务，而且还可以指定执行shell等等，crontab -e这种所有用户都可以使用，普通用户也只能为自己设置计划任务。然后自动写入/var/spool/cron/usename
2. crontab -e是某个用户的周期计划任务；/etc/crontab是系统的周期任务
3. crontab -e与/etc/crontab修改语法格式不一样，后者多一个user指定
4. 不管用crontab -e或者/etc/crontab都不需要重新启动crond服务
5. crontab  -e是针对用户的cron来设计的，如果是系统的例行性任务，需要编辑/etc/crontab文件。需要注意的是：crontab -e的作用其实是/usr/bin/crontab这个执行文件，但是/etc/crontab是个纯文本文件，可以root的身份编辑这个文件。
6. cron服务的最低检测时间单位是分钟，所以cron会每分钟读取一次/etc/crontab与/var/spool/cron中的数据内容，因此，只要您编辑完/etc/crontab文件并且保存之后，crontab时设定就会自动执行。

注意：在Linux下的crontab会自动帮我们每分钟重新读取一次/etc/crontab的例行工作事项，但是某些原因或在其他的unix系统中，由于crontab是读到内存中，所以在您修改完/etc/crontab之后可能并不会马上执行，这时请重新启动crond服务。

/et

# crontab 命令
``` bash 
crontab [-u user] file [-u user] [ -e | -l | -r ]
*   -u user：用来设定某个用户的crontab服务；
*   file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab。
*   -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件。
*   -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容。
*   -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件。
*   -i：在删除用户的crontab文件时给确认提示。
```

# cron语法
## 语法
分钟 小时 日期 月份 周 命令  
如：数字范围 0-59 0-23 1-31 1-12 0-7 echo "hello" >> abc.log  
```text 特殊字符的含义
*(星号) 代表任何时刻都接受。
,(逗号) 代表分隔时段的意思。
-(减号) 代表一段时间范围内。
/n(斜线) 那个 n 代表数字，每隔 n 单位间隔。
```
## 示例
- 每年的五月一日 10:5 执行一次：`5 10 1 5 * command（要是执行网址（curl "http://网址"），或者执行其它的直接写路径）`
- 每天的三点，六点各执行一次：`00 3,6 * * * command`
- 每天的8:20, 9:20,10:20,11:20各执行一次：`20 8-11 * * * command`
- 每五分钟执行一次：`*/5 * * * * command`
- 每周一十点执行一次：`00 10 * * 1 command`
- 每天 02:00 执行任务：`0 2 * * * /bin/sh backup.sh`
- 每天 5:00和17:00执行任务：`0 5,17 * * * /scripts/script.sh`
- 每分钟执行一次任务：`* * * * * /scripts/script.sh`
- 每周日 17:00 执行任务：`0 17 * * sun /scripts/script.sh`
- 每 10min 执行一次任务：`*/10 * * * * /scripts/monitor.sh`
- 在特定的某几个月执行任务：`* * * jan,may,aug * /script/script.sh`
- 在特定的某几天执行任务：`0 17 * * sun,fri /script/scripy.sh（在每周五、周日的17点执行任务）`
- 在某个月的第一个周日执行任务：`0 2 * * sun [ $(date +%d) -le 07 ] && /script/script.sh`
- 每四个小时执行一个任务：`0 */4 * * * /scripts/script.sh`
- 每周一、周日执行任务：`0 4,17 * * sun,mon /scripts/script.sh`
- 每个30秒执行一次任务：我们没有办法直接通过上诉类似的例子去执行，因为最小的是1min。但是我们可以通过如下的方法。
`* * * * * /scripts/script.sh`
`* * * * * sleep 30; /scripts/script.sh`
- 多个任务在一条命令中配置
`* * * * * /scripts/script.sh; /scripts/scrit2.sh`
- 每年执行一次任务
`@yearly /scripts/script.sh`
@yearly 类似于“0 0 1 1 *”。它会在每年的第一分钟内执行，通常我们可以用这个发送新年的问候。
- 系统重启时执行：`@reboot /scripts/script.sh`

# 清除日志命令
主要目标：每日凌晨前10分钟，将catalina.out日志，copy重命名 catalina.out-2020-02-08.log，清空服务器上的catalina.out日志，

编译一个shell
``` bash
#!/bin/bash


```






