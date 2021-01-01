---
title: 003-dubbo的服务注册与发现
categories:
  - rpc-dubbo
abbrlink: 8c67eec6
date: 2020-02-26 16:07:53
---

摘要：后续文章主要以2.7.5 版本说明，推荐使用zookeeper注册中心。
<!-- more -->

参看官方源码，支持一下方案：
```xml
    <modules>
        <module>dubbo-registry-api</module>
        <module>dubbo-registry-default</module>
        <module>dubbo-registry-multicast</module>
        <module>dubbo-registry-zookeeper</module>[推荐]
        <module>dubbo-registry-redis</module>
        <module>dubbo-registry-consul</module>
        <module>dubbo-registry-etcd3</module>
        <module>dubbo-registry-nacos</module>[推荐]
        <module>dubbo-registry-multiple</module>
        <module>dubbo-registry-sofa</module>
        <module>dubbo-registry-eureka</module>
    </modules>
```

# Multicast 注册中心
# zookeeper 注册中心

## 需要安装zookeeper

## 配置

### 单机
```xml
<dubbo:registry address="zookeeper://127.0.0.1:12181"/>
<!-- 或 -->
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181" />
```

### 集群
```xml
<dubbo:registry address="zookeeper://10.20.153.10:2181?backup=10.20.153.11:2181,10.20.153.12:2181" />
<!-- 或 -->
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181,10.20.153.11:2181,10.20.153.12:2181" />
```

### zk客户端
dubbo 2.6 以前的版本引入 zkclient 操作 zookeeper
dubbo 2.6 及以后的版本引入 curator 操作 zookeeper
2.7.5版本查看代码：curator

# Nacos 注册中心
https://nacos.io/zh-cn/docs/quick-start-docker.html

参看git：[https://github.com/bjlhx15/shell.git](https://github.com/bjlhx15/shell.git)
中:docker/nacos


# Redis 注册中心







