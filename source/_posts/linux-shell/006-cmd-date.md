---
title: cmd-data
categories:
  - linux-shell
abbrlink: f7a4df92
date: 2020-02-11 15:59:50
tags:
---

摘要：shell开发时，常用date控制展示时间

<!-- more -->

# date
date 可以用来显示或设定系统的日期与时间。

## 查看帮助语法说明
``` bash
date --help
```
```
用法：date [选项]... [+格式]
　或：date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
可选参数：
  -d, --date=STRING         显示字符串所指的日期与时间。字符串前后必须加上引号； not 'now'
  -f, --file=DATEFILE       like --date once for each line of DATEFILE
  -I[TIMESPEC], --iso-8601[=TIMESPEC]  output date/time in ISO 8601 format.
                            TIMESPEC='date' for date only (the default),
                            'hours', 'minutes', 'seconds', or 'ns' for date
                            and time to the indicated precision.
  -r, --reference=文件		显示文件指定文件的最后修改时间
  -R, --rfc-2822		以RFC 2822格式输出日期和时间，例如：2006年8月7日，星期一 12:34:56 -0600
      --rfc-3339=TIMESPEC   output date and time in RFC 3339 format.
                            TIMESPEC='date', 'seconds', or 'ns' for
                            date and time to the indicated precision.
                            Date and time components are separated by
                            a single space: 2006-08-07 12:34:56-06:00
  -s, --set=STRING          根据字符串来设置日期与时间。字符串前后必须加上引号；
  -u, --utc, --universal    print or set Coordinated Universal Time (UTC)
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出
```

``` text 日期格式字符串
# 如果需要以指定的格式显示日期，可以使用“+”开头的字符串指定其格式
  %%	一个文字的 %
  %a	当前locale 的星期名缩写(例如： 日，代表星期日)
  %A	当前locale 的星期名全称 (如：星期日)
  %b	当前locale 的月名缩写 (如：一，代表一月)
  %B	当前locale 的月名全称 (如：一月)
  %c	当前locale 的日期和时间 (如：2005年3月3日 星期四 23:05:25)
  %C	世纪；比如 %Y，通常为省略当前年份的后两位数字(例如：20)
  %d	按月计的日期(例如：01)
  %D	按月计的日期；等于%m/%d/%y
  %e	按月计的日期，添加空格，等于%_d
  %F	完整日期格式，等价于 %Y-%m-%d
  %g	ISO-8601 格式年份的最后两位 (参见%G)
  %G	ISO-8601 格式年份 (参见%V)，一般只和 %V 结合使用
  %h	等于%b
  %H	小时(00-23)
  %I	小时(00-12)
  %j	按年计的日期(001-366)
  %k   hour, space padded ( 0..23); same as %_H
  %l   hour, space padded ( 1..12); same as %_I
  %m   month (01..12)
  %M   minute (00..59)
  %n	换行
  %N	纳秒(000000000-999999999)
  %p	当前locale 下的"上午"或者"下午"，未知时输出为空
  %P	与%p 类似，但是输出小写字母
  %r	当前locale 下的 12 小时时钟时间 (如：11:11:04 下午)
  %R	24 小时时间的时和分，等价于 %H:%M
  %s	自UTC 时间 1970-01-01 00:00:00 以来所经过的秒数
  %S	秒(00-60)
  %t	输出制表符 Tab
  %T	时间，等于%H:%M:%S
  %u	星期，1 代表星期一
  %U	一年中的第几周，以周日为每星期第一天(00-53)
  %V	ISO-8601 格式规范下的一年中第几周，以周一为每星期第一天(01-53)
  %w	一星期中的第几日(0-6)，0 代表周一
  %W	一年中的第几周，以周一为每星期第一天(00-53)
  %x	当前locale 下的日期描述 (如：12/31/99)
  %X	当前locale 下的时间描述 (如：23:13:48)
  %y	年份最后两位数位 (00-99)
  %Y	年份
  %z +hhmm		数字时区(例如，-0400)
  %:z +hh:mm		数字时区(例如，-04:00)
  %::z +hh:mm:ss	数字时区(例如，-04:00:00)
  %:::z			数字时区带有必要的精度 (例如，-04，+05:30)
  %Z			按字母表排序的时区缩写 (例如，EDT)
```

## 示例
``` BASH
# 格式化输出：
date +"%Y-%m-%d" 
2020-02-11
```
``` BASH
date +%Y%m%d               #显示当前天年月日 
date -d "-1 day" +%Y%m%d   #显示前一天的日期 或 date -d "1 day ago" +%Y%m%d 
date -d "+1 day" +%Y%m%d   #显示后一天的日期 或 date -d "1 day" +%Y%m%d
# 其中：day 天；month 月；year 年；second 秒；
```







