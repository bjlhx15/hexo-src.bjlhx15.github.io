---
title: 004-java-事件-spring事件
categories:
  - java-base
abbrlink: 57a8108a
date: 2021-01-09 21:20:10
---

摘要：java事件模型
主要有四种
1. 观察者模式
2. java事件
3. spring事件-java事件的包装
4. 分布式事件驱动-jms

事件监听机制更关注于特定的事件,观察者模式更关注于变化,再根据具体变化作出不同的响应
<!-- more -->

# 观察者模式
[https://www.cnblogs.com/bjlhx/p/11545163.html](https://www.cnblogs.com/bjlhx/p/11545163.html)

# java事件处理机制（自定义事件）

## 概念

java中的事件机制的参与者有3种角色：

1. event object：事件状态对象，用于listener的相应的方法之中，作为参数，一般存在与listerner的方法之中

2. event source：具体的事件源，比如说，你点击一个button，那么button就是event source，要想使button对某些事件进行响应，你就需要注册特定的listener。

3. event listener：对每个明确的事件的发生，都相应地定义一个明确的Java方法。这些方法都集中定义在事件监听者（EventListener）接口中，这个接口要继承 java.util.EventListener。 实现了事件监听者接口中一些或全部方法的类就是事件监听者。

伴随着事件的发生，相应的状态通常都封装在事件状态对象中，该对象必须继承自java.util.EventObject。事件状态对象作为单参传递给应响应该事件的监听者方法中。发出某种特定事件的事件源的标识是：遵从规定的设计格式为事件监听者定义注册方法，并接受对指定事件监听者接口实例的引用。

具体的对监听的事件类，当它监听到event object产生的时候，它就调用相应的方法，进行处理。

先看看jdk提供的event包：

public interface EventListener：所有事件侦听器接口必须扩展的标记接口。

public class EventObject extends Object implements Serializable ：所有事件状态对象都将从其派生的根类。 所有 Event 在构造时都引用了对象 "source"，在逻辑上认为该对象是最初发生有关 Event 的对象。

## 示例

发邮件，会触发 写数据库，写日志等操作

### 常规逻辑
1. 实现发送逻辑
```java
public class MailSenderJdk {

    public void sendMail(String to){
        System.out.println("MailSender开始发送邮件");

        //写DB、写日志
        System.out.println("over");
    }
}
```
2. 实际发送
```java
public class MailSenderJdkTest {

    @Test
    public void sendMail() {
        MailSenderJdk mailSenderJdk=new MailSenderJdk();
        mailSenderJdk.sendMail("bjlhx15@163.com");
    }
}
```
代码耦合高，不易扩展

### jdk事件逻辑
- 增加事件源类，继承EventObject
```java
public class MailSendJdkEvent extends EventObject {
    private String to;  //目的地

    public MailSendJdkEvent(Object source, String to) {
        super(source);
        this.to = to;
    }

    public String getTo() {
        return this.to;
    }
}

```
- 扩展事件监听接口
```java
public interface MailSendJdkEventListener extends EventListener {
    void MailSendJdkEventListener(MailSendJdkEvent source);
}
```
- 实现事件监听接口
写DB
```java
public class MailSendJdkEventListenerDb implements MailSendJdkEventListener {
    @Override
    public void MailSendJdkEventListener(MailSendJdkEvent source) {
        System.out.println("write DB:"+source.getTo());
    }
}
```
写日志
```java
public class MailSendJdkEventListenerLog implements MailSendJdkEventListener {
    @Override
    public void MailSendJdkEventListener(MailSendJdkEvent source) {
        System.out.println("write log:"+source.getTo());
    }
}
```
- 事件管理类
```java
public class MailManager {
    private Vector listeners;

    /**
     * 添加事件
     *
     * @param listener
     */
    public void adListener(MailSendJdkEventListener listener) {
        if (listeners == null) {
            listeners = new Vector();
        }
        listeners.add(listener);
    }

    /**
     * 移除事件
     *
     * @param listener
     */
    public void removeListener(MailSendJdkEventListener listener) {
        if (listeners == null)
            return;
        listeners.remove(listener);
    }

    /**
     * 通知所有的Listener
     */
    public void notifyListeners(MailSendJdkEvent event) {
        if (listeners == null)
            return;
        Iterator iter = listeners.iterator();
        while (iter.hasNext()) {
            MailSendJdkEventListener listener = (MailSendJdkEventListener) iter.next();
            listener.MailSendJdkEventListener(event);
        }
    }
}
```
- 发送逻辑
```java
public class MailSenderJdk {

    public void sendMail(String to) {
        System.out.println("MailSender开始发送邮件");

        MailManager manager = new MailManager();
        manager.adListener(new MailSendJdkEventListenerDb());// 给增加监听器
        manager.adListener(new MailSendJdkEventListenerLog());// 给增加监听器
        // 触发
        MailSendJdkEvent event = new MailSendJdkEvent(this, to);
        manager.notifyListeners(event);
        System.out.println("over");
    }
}
```
- 输出
```
MailSender开始发送邮件
write DB:bjlhx15@163.com
write log:bjlhx15@163.com
over
```

# spring事件机制

## 概念
spring事件发送监听涉及3个部分
- ApplicationEvent：继承EventObject，表示事件本身，自定义事件需要继承该类,可以用来传递数据,比如上述操作,我们需要将用户的邮箱地址传给事件监听器.
- ApplicationEventPublisherAware：事件发送器,通过实现这个接口,来触发事件.也可以委托ApplicationContext 发送事件
- ApplicationListener：实现了EventListener，事件监听器接口,事件的业务逻辑封装在监听器里面.也可以使用事件注解EventListener

## 异步配置
springboot启动类注解：@EnableAsync   
执行事件监听方法上：@Async

## 示例
- 事件源，继承ApplicationEvent
```java
public class MailSendEvent extends ApplicationEvent {

    private String to;  //目的地
    public MailSendEvent(Object source,String to) {
        super(source);
        this.to = to;
    }

    public String getTo() {
        return this.to;
    }
}
```
- 事件监听
```java
@Component
public class MailSendEventListenerDb implements ApplicationListener<MailSendEvent> {
    @Override
    public void onApplicationEvent(MailSendEvent source) {
        System.out.println("write DB:" + source.getTo());
    }
}
```
```java
@Component
public class MailSendEventListenerLog {
    @EventListener
    public void onApplicationEvent(MailSendEvent source) {
        System.out.println("write log:" + source.getTo());
    }
}
```
- 事件发送器
接口方式
```java
@Service
public class MailSenderIface implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher applicationEventPublisher;

    public void sendMail(String to) {
        System.out.println("MailSender开始发送邮件");
        MailSendEvent event = new MailSendEvent(this, to);
        applicationEventPublisher.publishEvent(event);
        System.out.println("over");
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
```
委托ApplicationContext
```java
@Service
public class MailSender {
    @Autowired
    private ApplicationContext applicationContext;  //容器事件由容器触发

    public void sendMail(String to) {
        System.out.println("MailSender开始发送邮件");
        MailSendEvent event = new MailSendEvent(applicationContext,to);
        applicationContext.publishEvent(event);
        System.out.println("over");
    }
}
```
- 测试
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MailSenderTest {

    @Autowired
    MailSender mailSender;
//    @Autowired
//    MailSenderIface mailSender;
    @Test
    public void sendMail() {
        mailSender.sendMail("bjlhx15@163.com");
    }
}
```

