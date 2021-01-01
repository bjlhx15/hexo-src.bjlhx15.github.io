---
title: 001-TICK集成一体化监控
categories:
  - devops
abbrlink: 95a952d4
date: 2020-04-10 16:27:05
tags:
---

通过TICK(Telegraf+Influxdb+Chronograf+Kapacitor)进行主机性能监控告警，职责描述如下：

Telegraf：数据采集，用于主机性能数据，包括主机CPU、内存、IO、进程状态、服务状态等
Influxdb的：时序数据库，用于存储Telegraf采集来的数据
Chronograf：数据可视化，用于将Influxdb数据库的性能数据时序展示
Kapacitor：规则告警，用于配置告警规则将Influxdb数据库查询触发规则的数据进行告警

其中，时序数据库可使用刚开源的TDEngine，可视化可以使用Grafana替代使用

<!-- more -->
## 概述
 数据采集：Telegraf 和 数据存储：InfluxDB

下载地址：
https://portal.influxdata.com/downloads/

环境：CentOS7.4 64位

在平台监控系统中，可以使用 Telegraf 采集多种组件的运行信息，而不需要自己手写脚本定时采集，大大降低数据获取的难度；且 Telegraf 配置极为简单，只要有基本的 Linux 基础即可快速上手。Telegraf 按照时间序列采集数据，数据结构中包含时序信息，时序数据库就是为此类数据设计而来，使用 Influxdb 可以针采集得到的数据完成各种分析计算操作。

### 为什么要用telegraf和influxdb？

①、在数据采集和平台监控系统中，Telegraf 可以采集多种组件的运行信息，而不需要自己手写脚本定时采集，降低数据获取的难度；

②、Telegraf 配置简单，只要有基本的 Linux 基础即可快速上手；

③、Telegraf 按照时间序列采集数据，数据结构中包含时序信息，influxdb就是为此类数据设计而来，使用 Influxdb 可以针采集得到的数据完成各种分析计算操作；

## 数据采集：Telegraf
Telegraf 是一个用 Go 编写的代理程序，可收集系统和服务的统计数据，并写入到 InfluxDB 数据库。内存占用小，通过插件系统可轻松添加支持其他服务的扩展。

Telegraf 是收集和报告指标和数据的代理。

Telegraf是TICK Stack的一部分，是一个插件驱动的服务器代理，用于收集和报告指标。

Telegraf 集成了直接从其运行的容器和系统中提取各种指标，事件和日志，从第三方API提取指标，甚至通过StatsD和Kafka消费者服务监听指标。

它还具有输出插件，可将指标发送到各种其他数据存储，服务和消息队列，包括InfluxDB，Graphite，OpenTSDB，Datadog，Librato，Kafka，MQTT，NSQ等等。

常用的输入插件（mysql、redis、prometheus）配置可参见 附录说明

Telegraf由4个独立的插件驱动

Input Plugins：输入插件，收集系统、服务、第三方组件的数据
Processor Plugins：处理插件，转换、处理、过滤数据
Aggregator Plugins：聚合插件，数据特征聚合
Output Plugins：输出插件，写metrics数据

相比zabbix，对主流开源应用的探测支持的更好，并且无需安装agent。

### 安装使用
```
wget https://dl.influxdata.com/telegraf/releases/telegraf-1.14.1-1.x86_64.rpm
rpm -ivh telegraf-1.14.1-1.x86_64.rpm
systemctl start telegraf 
```

## 数据存储：InfluxDB

Influxdb 是一个开源的分布式时序、时间和指标数据库，使用 Go 语言编写，无需外部依赖。Influxdb 有如下三大特性：

①、基于时间序列(Time Series)，支持与时间有关的相关函数（如最大，最小，求和等）；

②、可度量性（Metrics）：你可以实时对大量数据进行计算；

③、基于事件（Event）：它支持任意的事件数据；

### 安装使用
```bash
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.0.x86_64.rpm
rpm -ivh influxdb-1.8.0.x86_64.rpm
systemctl start influxd 
# 检测
curl "http://localhost:8086/query?q=show+databases"
```

### 创建 Influxdb 用户和数据库
上述服务启动后，
```bash
# 进入数据库
influx
# Connected to http://localhost:8086 version 1.8.0
# InfluxDB shell version: 1.8.0
create user "telegraf" with password 'password'
show users
# user     admin
# ----     -----
# telegraf false
create database telegraf
show databases
# name: databases
# name
# ----
# telegraf
# _internal
```

### 配置Telegraf 监听本机cpu信息

```bash
vim /etc/telegraf/telegraf.conf
   ## 修改内容如下： 
   [[outputs.influxdb]]
     urls = ["http://localhost:8086"] # required 
     database = "telegraf" # required
     retention_policy = ""
     precision = "s"
     timeout = "5s"
     username = "telegraf"
     password = "password" 
```
systemctl restart telegraf

### 配置信息
vim /etc/influxdb/influxdb.conf

https://docs.influxdata.com/influxdb/v1.8/administration/config/

```
全局配置

reporting-disabled = false  # 该选项用于上报influxdb的使用信息给InfluxData公司，默认值为false
bind-address = ":8088"  # 备份恢复时使用，默认值为8088

1、meta相关配置

[meta]
dir = "/var/lib/influxdb/meta"  # meta数据存放目录
retention-autocreate = true  # 用于控制默认存储策略，数据库创建时，会自动生成autogen的存储策略，默认值：true
logging-enabled = true  # 是否开启meta日志，默认值：true

2、data相关配置

[data]
dir = "/var/lib/influxdb/data"  # 最终数据（TSM文件）存储目录
wal-dir = "/var/lib/influxdb/wal"  # 预写日志存储目录
query-log-enabled = true  # 是否开启tsm引擎查询日志，默认值： true
cache-max-memory-size = 1048576000  # 用于限定shard最大值，大于该值时会拒绝写入，默认值：1000MB，单位：byte
cache-snapshot-memory-size = 26214400  # 用于设置快照大小，大于该值时数据会刷新到tsm文件，默认值：25MB，单位：byte
cache-snapshot-write-cold-duration = "10m"  # tsm引擎 snapshot写盘延迟，默认值：10Minute
compact-full-write-cold-duration = "4h"  # tsm文件在压缩前可以存储的最大时间，默认值：4Hour
max-series-per-database = 1000000  # 限制数据库的级数，该值为0时取消限制，默认值：1000000
max-values-per-tag = 100000  # 一个tag最大的value数，0取消限制，默认值：100000

3、coordinator查询管理的配置选项

[coordinator]
write-timeout = "10s"  # 写操作超时时间，默认值： 10s
max-concurrent-queries = 0  # 最大并发查询数，0无限制，默认值： 0
query-timeout = "0s  # 查询操作超时时间，0无限制，默认值：0s
log-queries-after = "0s"  # 慢查询超时时间，0无限制，默认值：0s
max-select-point = 0  # SELECT语句可以处理的最大点数（points），0无限制，默认值：0
max-select-series = 0  # SELECT语句可以处理的最大级数（series），0无限制，默认值：0
max-select-buckets = 0  # SELECT语句可以处理的最大"GROUP BY time()"的时间周期，0无限制，默认值：0

4、retention旧数据的保留策略

[retention]
enabled = true  # 是否启用该模块，默认值 ： true
check-interval = "30m"  # 检查时间间隔，默认值 ："30m"

5、shard-precreation分区预创建

[shard-precreation]
enabled = true  # 是否启用该模块，默认值 ： true
check-interval = "10m"  # 检查时间间隔，默认值 ："10m"
advance-period = "30m"  # 预创建分区的最大提前时间，默认值 ："30m"

6、monitor 控制InfluxDB自有的监控系统。 默认情况下，InfluxDB把这些数据写入_internal 数据库，如果这个库不存在则自动创建。 _internal 库默认的retention策略是7天，如果你想使用一个自己的retention策略，需要自己创建。

[monitor]
store-enabled = true  # 是否启用该模块，默认值 ：true
store-database = "_internal"  # 默认数据库："_internal"
store-interval = "10s  # 统计间隔，默认值："10s"

7、admin web管理页面[1.3界面已删除使用 1：chronograf]

[admin]
enabled = true  # 是否启用该模块，默认值 ： false
bind-address = ":8083"  # 绑定地址，默认值 ：":8083"
https-enabled = false  # 是否开启https ，默认值 ：false
https-certificate = "/etc/ssl/influxdb.pem"  # https证书路径，默认值："/etc/ssl/influxdb.pem"

8、http API

[http]
enabled = true  # 是否启用该模块，默认值 ：true
bind-address = ":8086"  # 绑定地址，默认值：":8086"
auth-enabled = false  # 是否开启认证，默认值：false
realm = "InfluxDB"  # 配置JWT realm，默认值: "InfluxDB"
log-enabled = true  # 是否开启日志，默认值：true
write-tracing = false  # 是否开启写操作日志，如果置成true，每一次写操作都会打日志，默认值：false
pprof-enabled = true  # 是否开启pprof，默认值：true
https-enabled = false  # 是否开启https，默认值：false
https-certificate = "/etc/ssl/influxdb.pem"  # 设置https证书路径，默认值："/etc/ssl/influxdb.pem"
https-private-key = ""  # 设置https私钥，无默认值
shared-secret = ""  # 用于JWT签名的共享密钥，无默认值
max-row-limit = 0  # 配置查询返回最大行数，0无限制，默认值：0
max-connection-limit = 0  # 配置最大连接数，0无限制，默认值：0
unix-socket-enabled = false  # 是否使用unix-socket，默认值：false
bind-socket = "/var/run/influxdb.sock"  # unix-socket路径，默认值："/var/run/influxdb.sock"

9、subscriber 控制Kapacitor接受数据的配置

[subscriber]
enabled = true  # 是否启用该模块，默认值 ：true
http-timeout = "30s"  # http超时时间，默认值："30s"
insecure-skip-verify = false  # 是否允许不安全的证书
ca-certs = ""  # 设置CA证书
write-concurrency = 40  # 设置并发数目，默认值：40
write-buffer-size = 1000  # 设置buffer大小，默认值：1000

10、graphite 相关配置

[[graphite]]
enabled = false  # 是否启用该模块，默认值 ：false
database = "graphite"  # 数据库名称，默认值："graphite"
retention-policy = ""  # 存储策略，无默认值
bind-address = ":2003"  # 绑定地址，默认值：":2003"
protocol = "tcp"  # 协议，默认值："tcp"
consistency-level = "one"  # 一致性级别，默认值："one
batch-size = 5000  # 批量size，默认值：5000
batch-pending = 10  # 配置在内存中等待的batch数，默认值：10
batch-timeout = "1s"  # 超时时间，默认值："1s"
udp-read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。 该配置的默认值：0
separator = "."  # 多个measurement间的连接符，默认值： "."

11、collectd

[[collectd]]
enabled = false  # 是否启用该模块，默认值 ：false
bind-address = ":25826"  # 绑定地址，默认值： ":25826"
database = "collectd"  # 数据库名称，默认值："collectd"
retention-policy = ""  # 存储策略，无默认值
typesdb = "/usr/local/share/collectd"  # 路径，默认值："/usr/share/collectd/types.db"
auth-file = "/etc/collectd/auth_file"
batch-size = 5000
batch-pending = 10
batch-timeout = "10s"
read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。默认值：0

12、opentsdb

[[opentsdb]]
enabled = false  # 是否启用该模块，默认值：false
bind-address = ":4242"  # 绑定地址，默认值：":4242"
database = "opentsdb"  # 默认数据库："opentsdb"
retention-policy = ""  # 存储策略，无默认值
consistency-level = "one"  # 一致性级别，默认值："one"
tls-enabled = false  # 是否开启tls，默认值：false
certificate= "/etc/ssl/influxdb.pem"  # 证书路径，默认值："/etc/ssl/influxdb.pem"
log-point-errors = true  # 出错时是否记录日志，默认值：true
batch-size = 1000
batch-pending = 5
batch-timeout = "1s"

13、udp

[[udp]]
enabled = false  # 是否启用该模块，默认值：false
bind-address = ":8089"  # 绑定地址，默认值：":8089"
database = "udp"  # 数据库名称，默认值："udp"
retention-policy = ""  # 存储策略，无默认值
batch-size = 5000
batch-pending = 10
batch-timeout = "1s"
read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。 该配置的默认值：0
　
14、continuous_queries

[continuous_queries]
enabled = true  # enabled 是否开启CQs，默认值：true
log-enabled = true  # 是否开启日志，默认值：true
run-interval = "1s"  # 时间间隔，默认值："1s"
```

telegraph会和influxDB的HTTP APi通信来写入数据。

## 可视化-1：chronograf

influxdb 在1.3之后取消web 界面后 使用的一个新的管理界面
Chronograf是InfluxData的TICK堆栈的用户界面组件。它使您的基础架构的监控和警报易于设置和维护。
### 安装
```
wget https://dl.influxdata.com/chronograf/releases/chronograf-1.8.2.x86_64.rpm
rpm -ivh chronograf-1.8.2.x86_64.rpm
systemctl start chronograf 
```
修改配置启动：
vim /etc/influxdb/influxdb.conf
基础配置
http://116.198.1.1:8888/

查看界面输入基础数据库连接即可

一个操作界面。


## 规则告警：Kapacitor
Kapacitor是TICK堆栈的数据处理平台。Kapacitor负责在Chronograf中创建和发送警报。

### 安装：
```bash
wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.4-1.x86_64.rpm
yum localinstall kapacitor-1.5.4-1.x86_64.rpm
systemctl start kapacitor
kapacitor list tasks
# ID Type      Status    Executing Databases and Retention Policies
```

对于Kapacitor URL，输入运行Kapacitor的计算机的主机名或IP，并确保包含Kapacitor的默认端口：9092。
接下来，命名连接字符串; 这可以是你想要的任何东西。由于在Kapacitor的默认配置中禁用了授权，因此无需为Username和Password输入输入任何信息。最后，点击Connect。