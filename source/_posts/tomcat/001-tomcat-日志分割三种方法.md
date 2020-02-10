---
title: 001-tomcat-日志分割三种方法
categories:
  - tomcat
abbrlink: '6998e040'
date: 2020-02-10 20:10:58
tags:
---

概述:tomcat-日志分割三种方法
三种方式均有优缺点，
- cronolog：需要在主机安装软件
- log4j：方便使用，但是不能删除
- crontab：需要有对应目录权限

采取cronolog、log4j缺点：已经做好对应主机镜像，大范围使用，在应用系统级别修改，所有使用者会参与修改；  
采用crontab，crontab不需要安装，检测linux默认自带安装。正好部署时使用shell脚本，此时只需在脚本中嵌入执行crontab脚本即可。

<!--more-->
# 方法一、用cronolog分割tomcat的catalina.out文件 
Linux 日志切割工具cronolog详解：https://blog.csdn.net/chenkeqin_2012/article/details/52670887
1. 编译安装cronolog
```bash
wget http://cronolog.org/download/cronolog-1.6.2.tar.gz 
tar zxvf cronolog-1.6.2.tar.gz 
cd cronolog-1.6.2
./configure 
make && make install 
```
2. 查看cronolog安装后所在目录（验证安装是否成功） 
which cronolog  
一般情况下显示为：/usr/local/sbin/cronolog 
3. 编辑tomcat目录bin下的catalina.sh文件
找到下面这行，类似这样的行有2处：
```
org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 &
```
- 第一处：tomcat是带“-security”参数的启动，
- 第二处：默认tomcat启动方式，也就是else下面的那部分，我们只修改这里。
- 另外还要把touch “$CATALINA_OUT"这行注释掉。
``` bash
#  touch "$CATALINA_OUT"
  if [ "$1" = "-security" ] ; then
    if [ $have_tty -eq 1 ]; then
      echo "Using Security Manager"
    fi
    shift
    "$_RUNJAVA" "$LOGGING_CONFIG" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
      -Djava.endorsed.dirs="$JAVA_ENDORSED_DIRS" -classpath "$CLASSPATH" \
      -Djava.security.manager \
      -Djava.security.policy=="$CATALINA_BASE"/conf/catalina.policy \
      -Dcatalina.base="$CATALINA_BASE" \
      -Dcatalina.home="$CATALINA_HOME" \
      -Djava.io.tmpdir="$CATALINA_TMPDIR" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 &

  else
    "$_RUNJAVA" "$LOGGING_CONFIG" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
      -Djava.endorsed.dirs="$JAVA_ENDORSED_DIRS" -classpath "$CLASSPATH" \
      -Dcatalina.base="$CATALINA_BASE" \
      -Dcatalina.home="$CATALINA_HOME" \
      -Djava.io.tmpdir="$CATALINA_TMPDIR" \
      org.apache.catalina.startup.Bootstrap "$@" start 2>&1 | /usr/local/sbin/cronolog /usr/local/tomcat/logs/catalina.%Y%m%d.out >> /dev/null &
#      >> "$CATALINA_OUT" 2>&1 &

  fi
```
4. 重启tomcat
查看日志目录是否生成catalina.yymmdd.out的日志文件

-rw-r--r-- 1 root root 10537 Jul 30 10:50 catalina.20140730.out

配置cronolog完成了，观察每天是否有一个新的catalina.yymmdd.out的日志文件生成，定期删除日期较旧的日志文件。

# 方法二、使用log4j成功使catalina.out文件实现分割
　　1、在tomcat根目录下建立common/classes/log4j.properties，内容如下
复制代码
log4j.rootLogger=INFO, R 
log4j.appender.R=org.apache.log4j.RollingFileAppender 
log4j.appender.R.File=${catalina.home}/logs/tomcat.newlog  #设定日志文件名
log4j.appender.R.MaxFileSize=100KB   #设定文件到100kb即分割
log4j.appender.R.MaxBackupIndex=10   #设定日志文件保留的序号数
log4j.appender.R.layout=org.apache.log4j.PatternLayout 
log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n
复制代码
　　2、在tomcat根目录下的common/lib下加入log4j.jar和commons-logging.jar
　　3、重新启动tomcat即可。

# 方法三、使用系统crontab
1. 编写一个crontab_log_work.sh文件,shell脚本如下:
```bash
#!/bin/bash

cd  `dirname $0`
d=`date +%Y%m%d`
d7=`date -d'7 day ago' +%Y%m%d`

cd  ../logs/

cp catalina.out catalina.out.${d}
echo "" > catalina.out 
rm -rf catalina.out.${d7}
```
2. 编写任务执行计划 crontab_log
```
55 23 * * * crontab_log_work.sh
```
3. 使用crontab 启动
```
crontab crontab_log
```






