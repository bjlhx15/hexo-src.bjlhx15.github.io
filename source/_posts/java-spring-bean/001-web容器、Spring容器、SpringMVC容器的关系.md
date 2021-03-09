---
title: 001-web容器、Spring容器、SpringMVC容器的关系
categories:
  - java-spring-bean
abbrlink: 1c366cf8
date: 2021-01-08 22:45:05
tags:
---

摘要：
- springmvc和spring都是容器，容器就是管理对象的地方，例如Tomcat，就是管理servlet对象的。
- springMVC容器和spring容器，就是管理bean对象的地方，springmvc就是管理controller对象的容器，spring就是管理service和dao的容器。
- 一般在springmvc的配置文件里配置的扫描路径就是controller的路径，而spring的配置文件里自然配的就是service和dao的路径。
<!-- more -->

# Servlet

Servlet是JavaEE规范的一种，主要是为了扩展Java作为Web服务的功能，统一接口。由其他内部厂商如tomcat，jetty内部实现web的功能。如一个http请求到来：
容器将请求封装为servlet中的HttpServletRequest对象，调用init（），service（）等方法输出response,由容器包装为httpresponse返回给客户端的过程。

![](00103.webp)

在Servlet规范中，提供了ServletContext,ServletRequest,ServletResponse,Filter等诸多接口。
基本类图和调用关系如下：
![](00104.webp)

![](00105.webp)

接口的作用，生命周期和使用：

## Servlet
- 作用：用于处理请求（service方法）
- 生命周期：加载实例化、初始化、处理客户端请求、销毁。
   加载实例化主要是交由web容器完成，而其他三个阶段则对应Servlet的init、service和destroy方法。
   Servlet对象被创建出来后需要对其进行初始化操作，初始化工作可以放在以ServletConfig类型为参数的ini方法中，
   ServletConfig为web.xml配置文件中配置的对应的初始化参数，由web容器完成web.xml配置读取并封装成ServletConfig对象；
   当Servlet初始化完成后，开始接受客户端的请求，这些请求被封装成ServletRequest类型的请求对象和ServletResponse类型的响应对象，
   通过service方法处理请求并响应客户端；当一个Servlet需要从web容器中移除时，就会调用对应的destroy方法用于释放所有的资源，
   并且调用destroy方法之前要保证所有正在执行service方法的线程都完成执行。
- 使用：servlet规范中定义了GenericServlet接口，定义了通用，协议独立的servlet,他们的子接口HttpServlet就是用来处理http请求的Servlet,根据http协议扩展了不同方式的请求处理方法，如doPost,doGet.

## ServletContext
Servlet与Servlet容器之间直接通信的接口,一个web应用只独有一个ServletContext.
- 作用：
   - 用于在web应用范围内存取共享数据,如setAttribute(String name, Object object)，getAttribute()
   - 获取当前Web应用的资源，如getContextPath()
   - 获取服务器端的文件系统资源，如getResourceAsStream()
   - 输出日志，如log(String msg) ： 向Servlet的日志文件中写日志
   - 在具体ServletContext 实现中，提供了添加Servlet，Filter,Listener到ServletContext里面的方法
- 生命周期：和web应用的生命周期一样
- 使用：一般由web容器实现，如tomcat

## Filter
- 作用：用于Web容器对请求和响应做统一处理，例如统一改变HTTP请求内容和响应内容，它可以作用在某个Servlet或一组Servlet
- 生命周期：加载实例化、初始化（init）、处理客户端请求(doFilter)、销毁(destroy)
- 使用：在doFilter方法中调用chain.doFilter(request, response)之前的代码可用来做一些请求校验，之后代码可用来做一些响应包装。

## ServletRequest
封装了客户端请求的所有信息，如果使用HTTP协议通信则包括HTTP协议的请求行和请求头。HTTP协议对应请求对象类型是HttpServletRequest类
- 作用：
   - 获取HTTP协议请求头部，如getHeader、getHeaders
   - 获取请求路径，如getContextPath、getServletPath
   - 获取cookie的方法，如getCookies
   - 获取session的方法，如getSession,session是存储在服务器内存中，返回响应的时候会写入浏览器一个sessionId的cookie，用来标示这一个会话
- 生命周期：只在servlet的service方法或过滤器的doFilter方法作用域内有效，除非启用了异步处理调用了ServletRequest接口对象的startAsync方法，此时request对象会一直有效，直到调用AsyncContext的complete方法。另外，web容器通常会为了性能而不销毁ServletRequest接口的对象，而是重复利用ServletRequest接口对象。

## ServletResponse
Servlet通过ServletResponse对象来生成响应结果。
- 作用：定义了一系列与生成响应结果相关的方法，如:
   - setCharacterEncoding() —— 设置相应正文的字符编码。响应正文的默认字符编码为ISO-8859-1；
   - setContentLength() —— 设置响应正文的长度；
   - setBufferSize() —— 设置用于存放响应正文数据的缓冲区的大小
   - getBufferSize() —— 获得用于存放响应正文数据的缓冲区的大小；
   - reset() —— 清空缓冲区内的正文数据，并且清空响应状态代码及响应头
   - resetBuffer() —— 仅仅清空缓冲区的正文数据，不清空响应状态代码及响应头；
   - flushBuffer() —— 强制性地把缓冲区内的响应正文数据发送到客户端；
   - isCommitted() —— 返回一个boolean类型的值，如果为true，表示缓冲区内的数据已经提交给客户，即数据已经发送到客户端；
   - getOutputStream() —— 返回一个ServletOutputStream对象，Servlet用它来输出二进制的正文数据；
   - getWriter() —— 返回一个PrinterWriter对象，Servlet用它来输出字符串形式的正文数据；
   
为了提高输出数据的效率，ServletOutputStream和PrintWriter首先把数据写到缓冲区内。当缓冲区内的数据被提交给客户后，ServletResponse的isComitted方法返回true。
- 生命周期：ServletResponse接口只在Servlet的service方法或过滤器的doFilter方法作用域内有效，除非它关联的ServletResponse接口调用了startAsync方法启用异步处理，此时ServletResponse接口会一直有效，直到调用AsyncContext的complete方法。另外，web容器通常会为了性能而不销毁ServletResponse接口对象，而是重复利用ServletResponse接口对象。

## Listener
当触发某个事件，如servlet context初始化完成时，需要做一些事情，servlet规范中定义了若干个Listener用于监听这些事件。
- 作用：用于对特定对象的生命周期和特定事件进行响应处理，主要用于对Session,request,context等进行监控。
   - 监听域对象自身的创建和销毁的事件监听器
      - ServletContextListener：ServletContext的创建和销毁：contextInitialized方法和contextDestroyed方法，作为定时器、加载全局属性对象、创建全局数据库连接、加载缓存信息等
      - HttpSessionListener：HttpSession的创建和销毁：sessionCreated和sessionDestroyed方法，可用于统计在线人数、记录访问日志等
      - ServletRequestListener： ServletRequest的创建和销毁：requestInitialized和requestDestroyed方法
   - 监听域对象中的属性的增加和删除的事件监听器
      - ServletContextAttributeListener、HttpSessionAttributeListener、ServletRequestAttributeListener接口。
      - 实现方法：attributeAdded、attributeRemoved、attributeReplaced
   - 监听绑定到HttpSeesion域中的某个对象的状态的事件监听器(创建普通JavaBean)
      - HttpSession中的对象状态：绑定→解除绑定；钝化→活化
      - 实现接口及方法：HttpSessionBindingListener接口(valueBound和valueUnbound方法)、HttpSessionActivationListener接口(sessionWillPassivate和sessionDidActivate方法)

# web容器-tomcat

## web容器使用spring管理bean

  web容器是管理servlet，以及监听器(Listener)和过滤器(Filter)的。这些都是在web容器的掌控范围里。但他们不在spring和springmvc的掌控范围里。
  因此，我们无法在这些类中直接使用Spring注解的方式来注入我们需要的对象，是无效的，web容器是无法识别的。

  但我们有时候又确实会有这样的需求，比如在容器启动的时候，做一些验证或者初始化操作，这时可能会在监听器里用到bean对象；又或者需要定义一个过滤器做一些拦截操作，也可能会用到bean对象。
  那么在这些地方怎么获取spring的bean对象呢？下面我提供两个方法：
```java
public void contextInitialized(ServletContextEvent sce) {
　　ApplicationContext context = (ApplicationContext) sce.getServletContext().getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE); 
　　UserService userService = (UserService) context.getBean("userService");
}
```
```java
public void contextInitialized(ServletContextEvent sce) {
　　WebApplicationContext webApplicationContext = WebApplicationContextUtils.getWebApplicationContext(sce.getServletContext()); 
　　UserService userService = (UserService) webApplicationContext.getBean("userService"); 
}
```
注意：以上代码有一个前提，那就是servlet容器在实例化ConfigListener并调用其方法之前，要确保spring容器已经初始化完毕！而spring容器的初始化也是由Listener（ContextLoaderListener）完成，
因此只需在web.xml中先配置初始化spring容器的Listener，然后在配置自己的Listener。

## Servlet、spring核心关系图
![](00101.jpg)

- web容器中有servlet容器，spring项目部署后存在spring容器和springmvc容器。其中spring控制service层和dao层的bean对象。springmvc容器控制controller层bean对象。
- servlet容器控制servlet对象。项目启动是，首先 servlet初始化，初始化过程中通过web.xml中spring的配置加载spring配置，初始化spring容器和springmvc容器。待容器加载完成。servlet初始化完成，则完成启动。
- HTTP请求到达web容器后，会到达Servlet容器，容器通过分发器分发到具体的spring的Controller层。执行业务操作后返回结果。

![](00102.jpg)   

小结：

- Tomcat在启动时给每个Web应用创建一个全局的上下文环境，这个上下文就是ServletContext，其为后面的Spring容器提供宿主环境。
- Tomcat在启动过程中触发容器初始化事件，Spring的ContextLoaderListener会监听到这个事件，它的contextInitialized方法会被调用，在这个方法中，Spring会初始化全局的Spring根容器，这个就是Spring的IoC容器，IoC容器初始化完毕后，Spring将其存储到ServletContext中，便于以后来获取。
- Tomcat在启动过程中还会扫描Servlet，一个Web应用中的Servlet可以有多个，以SpringMVC中的DispatcherServlet为例，这个Servlet实际上是一个标准的前端控制器，用以转发、匹配、处理每个Servlet请求。
- Servlet一般会延迟加载，当第一个请求达到时，Tomcat&Jetty发现DispatcherServlet还没有被实例化，就调用DispatcherServlet的init方法，DispatcherServlet在初始化的时候会建立自己的容器，叫做SpringMVC 容器，用来持有Spring MVC相关的Bean。同时，Spring MVC还会通过ServletContext拿到Spring根容器，并将Spring根容器设为SpringMVC容器的父容器，请注意，Spring MVC容器可以访问父容器中的Bean，但是父容器不能访问子容器的Bean， 也就是说Spring根容器不能访问SpringMVC容器里的Bean。说的通俗点就是，在Controller里可以访问Service对象，但是在Service里不可以访问Controller对象。   

tomcat等容器其实就是web服务的实现，暴露端口，按照特定资源URL找到处理的servlet。然后处理请求。
web.xml其实tomcat在启动时候需要加载的配置欢迎页、Filter、Listener、Servlet等类的定义。当然不止加载这些东西，这些东西是需要加载到JVM堆内存中实例化的对象。
## Tomcat启动时加载资源三个阶段
- 第一阶段：JVM相关资源
   - $JAVA_HOME/jre/lib/ext/*.jar
   - 系统classpath环境变量中的*.jar和*.class 
- 第二阶段：Tomcat自身相关资源
   - $CATALINA_HOME/common/classes/*.class  
   - $CATALINA_HOME/commons/endorsed/*.jar   
   - $CATALINA_HOME/commons/i18n/*.jar   
   - $CATALINA_HOME/common/lib/*.jar   
   - $CATALINA_HOME/server/classes/*.class   
   - $CATALINA_HOME/server/lib/*.jar   
   - $CATALINA_BASE/shared/classes/*.class   
   - $CATALINA_BASE/shared/lib/*.jar 
- 第三阶段：Web应用相关资源
   - 具体应用的webapp目录: /WEB-INF/classes/*.class   
   - 具体应用的webapp: /WEB-INF/lib/*.jar
在tomcat目录${CATALINA_HOME}/conf下和web应用目录${CATALINA_HOME}/webapps/WebDemo(WebDemo为web应用名)下都有web.xml这个文件，但是内容不一样。

Tomcat在激活、加载、部署web应用时，会解析加载${CATALINA_HOME}/conf目录下所有web应用通用的web.xml，然后解析加载web应用目录中的WEB-INF/web.xml。
其实根据他们的位置，我们就可以知道，conf/web.xml文件中的设定会应用于所有的web应用程序，而某些web应用程序的WEB-INF/web.xml中的设定只应用于该应用程序本身。

如果没有WEB-INF/web.xml文件，tomcat会输出找不到的消息，但仍然会部署并使用web应用程序，servlet规范的作者想要实现一种能迅速并简易设定新范围的方法，以用作测试，因此，这个web.xml并不是必要的，不过通常最好还是让每一个上线的web应用程序都有一个自己的WEB-INF/web.xml。

web.xml中可以配置web应用名称，图标，描述，ServletContext上下文参数，Fliter配置，Listener配置，Servlet配置，会话超时配置，MIME类型配置等等。

# spring和springmvc

## springmvc配置
spring-mvc.xml
```xml
<context:component-scan base-package="com.github.bjlhx15.controller" />
```
## spring配置
applicationContext-service.xml

```xml
<!-- 扫描包加载Service实现类 -->
<context:component-scan base-package="com.github.bjlhx15.service"></context:component-scan>
<!--或者-->
<context:component-scan base-package="com.github.bjlhx15">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

## spring容器和springmvc容器关系
    spring容器和springmvc容器的关系是父子容器的关系。   
   - spring容器是父容器，springmvc是子容器。在子容器里可以访问父容器里的对象，但是在父容器里不可以访问子容器的对象，
   - 说的通俗点就是，在controller里可以访问service对象，但是在service里不可以访问controller对象

   所有的bean，都是被spring或者springmvc容器管理的，他们可以直接注入。
   然后springMVC的拦截器也是springmvc容器管理的，所以在springmvc的拦截器里，可以直接注入bean对象。如
   ```xml
   <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/employee/**" ></mvc:mapping>
            <bean class="com.smart.core.shiro.LoginInterceptor" ></bean>
        </mvc:interceptor>
    </mvc:interceptors>
   ```

# web容器和spring容器结合

## web容器servlet到springmvc的servlet

![](00106.webp)

其中FrameworkServlet会和Spring的ApplicationContext联系起来，它实现了ApplicationContextAware接口。
    
![](00107.webp) 
    
    
