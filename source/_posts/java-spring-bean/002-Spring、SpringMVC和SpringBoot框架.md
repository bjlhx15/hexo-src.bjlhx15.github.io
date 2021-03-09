---
title: 002-Spring、SpringMVC和SpringBoot框架
categories:
  - java-spring-bean
abbrlink: 509d5c14
date: 2021-01-09 09:28:39
tags:
---

摘要：
- Spring
Spring是一个开源容器框架，可以接管web层，业务层，dao层，持久层的组件，并且可以配置各种bean,和维护bean与bean之间的关系。其核心就是控制反转(IOC),和面向切面(AOP),简单的说就是一个分层的轻量级开源框架。
- SpringMVC
Spring MVC属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面。SpringMVC是一种web层mvc框架，用于替代servlet（处理|响应请求，获取表单参数，表单校验等。SpringMVC是一个MVC的开源框架，SpringMVC=struts2+spring，springMVC就相当于是Struts2加上Spring的整合。
- SpringBoot
Springboot是一个微服务框架，延续了spring框架的核心思想IOC和AOP，简化了应用的开发和部署。Spring Boot是为了简化Spring应用的创建、运行、调试、部署等而出现的，使用它可以做到专注于Spring应用的开发，而无需过多关注XML的配置。提供了一堆依赖打包，并已经按照使用习惯解决了依赖问题--->习惯大于约定。

spring mvc < spring < springboot
<!-- more -->

# spring的原理和组成

Spring为简化我们的开发工作，封装了一系列的开箱即用的组件功能模块，包括：Spring JDBC 、Spring MVC 、Spring Security、 Spring AOP 、Spring ORM 、Spring Test等。

![](00201.jpg)

# springmvc的原理和组成

SpringMVC是属于SpringWeb里面的一个功能模块（SpringWebMVC）。专门用来开发SpringWeb项目的一种MVC模式的技术框架实现。其原理如下：

![](00202.png)

MVC：Model（模型）、VIew（视图）、Controller（控制器）；

## 代码分析

```java
@HandlesTypes({WebApplicationInitializer.class})
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    public SpringServletContainerInitializer() {
    }

    public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
        List<WebApplicationInitializer> initializers = new LinkedList();
        Iterator var4;
        if (webAppInitializerClasses != null) {
            var4 = webAppInitializerClasses.iterator();

            while(var4.hasNext()) {
                Class<?> waiClass = (Class)var4.next();
                if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) && WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
                    try {
                        initializers.add((WebApplicationInitializer)ReflectionUtils.accessibleConstructor(waiClass, new Class[0]).newInstance());
                    } catch (Throwable var7) {
                        throw new ServletException("Failed to instantiate WebApplicationInitializer class", var7);
                    }
                }
            }
        }

        if (initializers.isEmpty()) {
            servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
        } else {
            servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
            AnnotationAwareOrderComparator.sort(initializers);
            var4 = initializers.iterator();

            while(var4.hasNext()) {
                WebApplicationInitializer initializer = (WebApplicationInitializer)var4.next();
                initializer.onStartup(servletContext);
            }

        }
    }
}
```

1. 通过SpringServletContainerInitializer来负责对容器启动时的相关组件的初始化，在web这个jar包下面有个meta-inf目录，下面有个services目录，下面有个javax.servlet.ServletContainerInitializer文件，里面指明了SpringServletContainerInitializer
2. 到底要初始化哪些组件是通过Servlet规范中所提供的注解handlesTypes来指定的
3. 在SpringServletCOntainerInitializer中，其HandlesTYpes注解则明确指定为了WebApplicationInitializer.class类型作为onStartup方法的第一个参数
4. 在SpringServletContainerIntializer的onStartup方法中，则主要是完成了一些验证与组件装配的工作
5. 在SpringServletCOntainerInitializer的onStartup方法中，由于某些容器并未遵循Servlet规范，导致虽然明确指定了HandlesTYpe注解的类型为webApplicationInitializer.class类型，但还是可能会存在将一些非法类型传递过来的情况，所以该方法还对传递过来的具体类型进行了细致的判断，只有符合条件的类型才会被纳入到List<webApplicationInitializer>集合中
6. 当以上判断完成之后，LIst<WebApplicationInitializer>就是接下来需要进行初始化的组件了，
7. 最后，通过遍历LIst<WebApplicationInitializer>列表，取出其中的每一个webApplicationInitializer对象，调用这些对象的onStartup方法，完成组件的启动初始化工作

小结：SpringServletContainerInitializer在整个初始化过程中，其扮演的角色实际上是委托或者是代理的角色，真正完成初始化工作的是一个个的webApplicationInitializer实现类


# springboot原理和特性

## springboot应用的容器初始化过程

```java
class TomcatStarter implements ServletContainerInitializer {

	private static final Log logger = LogFactory.getLog(TomcatStarter.class);

	private final ServletContextInitializer[] initializers;

	private volatile Exception startUpException;

	TomcatStarter(ServletContextInitializer[] initializers) {
		this.initializers = initializers;
	}

	@Override
	public void onStartup(Set<Class<?>> classes, ServletContext servletContext)
			throws ServletException {
		try {
			for (ServletContextInitializer initializer : this.initializers) {
				initializer.onStartup(servletContext);
			}
		}
		catch (Exception ex) {
			this.startUpException = ex;
			// Prevent Tomcat from logging and re-throwing when we know we can
			// deal with it in the main thread, but log for information here.
			if (logger.isErrorEnabled()) {
				logger.error("Error starting Tomcat context. Exception: "
						+ ex.getClass().getName() + ". Message: " + ex.getMessage());
			}
		}
	}

	public Exception getStartUpException() {
		return this.startUpException;
	}

}
```

1. 对于一个springboot应用来说，它并没有使用SpringServletContainerInitializer来进行容器的初始化，而是使用了TomcatStarter进行的。
2. TomcatStarter存在三点因素使得它无法通过SPI机制进行初始化：它没有不带参数的构造方法，它的声明并非public，其所在jar包并没有META-INF.services目录，当然也不存在名为javax.servlet.sertvletContainerInitialezer的文件了
3. 综上，TomcatStarter并非通过SPI机制进行的查找和实例化
4. 本质上，TOmcatStarter是通过spring boot框架new出来的
5. 与SpringServletContainerInitializer类似，TomcatStarter在容器的初始化过程中也是扮演着一个委托或者是代理的角色，真正执行的初始化动作实际上是由它所持有的ServletContextInitializer的onStartup方法来完成的

### 核心流程图

![](00205.jpg)

Spring-boot容器启动流程总体可划分为2部分：

1. 执行注解：扫描指定范围下的bean、载入自动配置类对应的bean加载到IOC容器。

2. main方法中具体SpringAppliocation.run()，全流程贯穿SpringApplicationEvent,有6个子类：
   - ApplicationEnvironmentPreparedEvent
   - ApplicationFailedEvent.class
   - ApplicationPreparedEvent.class
   - ApplicationReadyEvent.class
   - ApplicationStartedEvent.class
   - ApplicationStartingEvent.class
   
## Spring Boot中的一些特点：
- 创建独立的spring应用。
- 嵌入Tomcat, JettyUndertow 而且不需要部署他们。
- 提供的“starters” poms来简化Maven配置
- 尽可能自动配置spring应用。
- 提供生产指标,健壮检查和外部化配置
- 绝对没有代码生成和XML配置要求。


# 综合分析

![](00203.png)

springmvc和springboot应用组件之间的对应关系：
1. springServletContainerInitializer对应于TomcatStarter
2. webApplicationInitializer对应于ServletContextInitializer
