---
title: 002-dubbo的四种配置方式
categories:
  - rpc-dubbo
abbrlink: be7f604f
date: 2020-02-26 15:24:01
---

摘要：后续文章主要以2.7.5 版本说明
<!-- more -->

# xml配置【最为常用】
代码级别：https://github.com/bjlhx15/rpc-dubbo.git 的 rpc-dubbo-001-sample
使用的既是 xml配置
## 示例

### provider.xml 示例
```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="org.apache.dubbo.samples.basic.impl.DemoServiceImpl"/>
    <dubbo:service interface="org.apache.dubbo.samples.basic.api.DemoService" ref="demoService"/>
</beans>
```

### consumer.xml 示例
```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry group="aaa" address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.samples.basic.api.DemoService"/>
</beans>
```

### 标签支持自定义参数
所有标签都支持自定义参数，用于不同扩展点实现的特殊配置，如：
如
```xml
<dubbo:protocol name="jms">
    <dubbo:parameter key="queue" value="your_queue" />
</dubbo:protocol>
```
或
```xml
<dubbo:protocol name="jms" p:queue="your_queue" />  
```

## 配置说明
```
标签	用途	解释
<dubbo:service/>	服务配置	用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心
<dubbo:reference/>	引用配置	用于创建一个远程服务代理，一个引用可以指向多个注册中心
<dubbo:protocol/>	协议配置	用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受
<dubbo:application/>	应用配置	用于配置当前应用信息，不管该应用是提供者还是消费者
<dubbo:module/>	模块配置	用于配置当前模块信息，可选
<dubbo:registry/>	注册中心配置	用于配置连接注册中心相关信息
<dubbo:monitor/>	监控中心配置	用于配置连接监控中心相关信息，可选
<dubbo:provider/>	提供方配置	当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选
<dubbo:consumer/>	消费方配置	当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选
<dubbo:method/>	方法配置	用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息
<dubbo:argument/>	参数配置	用于指定方法参数配置
```

## 不同粒度配置的覆盖关系
以 timeout 为例，下图显示了配置的查找顺序，其它 retries, loadbalance, actives 等类似：
- 方法级优先，接口级次之，全局配置再次之。
- 如果级别一样，则消费方优先，提供方次之。

其中，服务提供方配置，通过 URL 经由注册中心传递给消费方。

（建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置）。

理论上 ReferenceConfig 中除了interface这一项，其他所有配置项都可以缺省不配置，框架会自动使用ConsumerConfig，ServiceConfig, ProviderConfig等提供的缺省配置。

# 其他配置
属性配置、API配置、注解配置
参看：http://dubbo.apache.org/zh-cn/docs/user/configuration/properties.html








