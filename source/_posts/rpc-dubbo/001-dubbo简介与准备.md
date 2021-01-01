---
title: 001-dubbo简介与准备
categories:
  - rpc-dubbo
abbrlink: 47f3ff77
date: 2020-02-26 14:49:08
---

摘要：阿里出品，开源。目前持续支持。
<!-- more -->

# 代码示例
dubbo个人示例代码：https://github.com/bjlhx15/rpc-dubbo.git

docker安装：https://github.com/bjlhx15/shell.git

dubbo代码：https://github.com/apache/dubbo.git

dubbo参看文档：http://dubbo.apache.org/zh-cn/docs/user/quick-start.html

nacos文档：https://nacos.io/zh-cn/docs/what-is-nacos.html


# 主要功能
远程接口调用、负载均衡和容错、自动服务注册和发现。

# 简单使用

需要安装注册中心、开发提供者、消费者

## 注册中心安装-mac机器docker安装zk
参看git：[https://github.com/bjlhx15/shell.git](https://github.com/bjlhx15/shell.git)
中docker/zookeeper/mac

## 安装dubbo-admin对服务监控功能
看：https://github.com/apache/dubbo-admin

生产环境配置 步骤：
```
下载代码: git clone https://github.com/apache/dubbo-admin.git

在 dubbo-admin-server/src/main/resources/application.properties中指定注册中心地址

构建

mvn clean package -Dmaven.test.skip=true
启动

mvn --projects dubbo-admin-server spring-boot:run
或者
cd dubbo-admin-distribution/target; java -jar dubbo-admin-0.1.jar
访问 http://localhost:8080
```

## 代码开发

代码级别：https://github.com/bjlhx15/rpc-dubbo.git 的 rpc-dubbo-001-sample
```
id-generator-sample-dubbo-consumer
id-generator-sample-dubbo-intf
id-generator-sample-dubbo-provider

```
1. 修改配置
在 provider、consumer 项目配置中：dubbo-XX.xml 修改配置中心地址
2. 分别启动对应的main方法
3. 测试
服务端服务：http://localhost:8082/get

消费者服务：http://localhost:8083/get

参看项目：https://github.com/apache/dubbo/tree/master/dubbo-demo/dubbo-demo-xml