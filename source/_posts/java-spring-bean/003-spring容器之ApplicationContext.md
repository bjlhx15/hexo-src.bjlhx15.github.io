---
title: 003-spring容器之ApplicationContext
categories:
  - java-spring-bean
abbrlink: ae0ddb33
date: 2021-01-09 13:11:35
tags:
---

摘要：
ApplicationContext：是spring继BeanFactory之外的另一个核心接口或容器，允许容器通过应用程序上下文环境创建、获取、管理bean。为应用程序提供配置的中央接口。在应用程序运行时这是只读的，但如果实现支持这一点，则可以重新加载。

一个ApplicationContext提供:
- 访问应用程序组件的Bean工厂方法。从org.springframework.beans.factory.ListableBeanFactory继承。
- 以通用方式加载文件资源的能力。继承自org.springframe .core.io。ResourceLoader接口。---beanXML
- 向注册侦听器发布事件的能力。继承自ApplicationEventPublisher接口。
- 解析消息的能力，支持国际化。继承自MessageSource接口。
- 从父上下文继承。后代上下文中的定义总是优先级。例如，这意味着单个父上下文可以被整个web应用程序使用，而每个servlet都有自己独立于任何其他servlet的子上下文。
<!-- more -->
# 接口依赖关系

如果说BeanFactory是Spring的心脏，那么ApplicationContext就是完整的身躯了。ApplicationContext由BeanFactory派生而来，提供了更多面向实际应用的功能。

![](00302.jpg)

# ApplicationContext子类接口

主要是：ConfigurableApplicationContext、WebApplicationContext

## ConfigurableApplicationContext
该接口提供了根据配置创建、获取bean的一些方法，其中主要常用的实现包括：ClassPathXmlApplicationContext、FileSystemXmlApplicationContext等。提供了通过各种途径去加载实例化bean的手段。

### ClassPathXmlApplicationContext

独立的XML应用程序上下文，从类路径中获取上下文定义文件，将普通路径解释为包含包路径的类路径资源名(例如，“mypackage / myresource.txt”)。
适用于测试工具以及嵌入在jar中的应用程序上下文。配置位置的默认值可以通过getConfigLocations重写，配置位置可以表示具体的文件，比如“/myfiles/context”。xml“或ant样式的模式，比如”/myfiles/*-context。参见org.springframework.util。模式细节的AntPathMatcher javadoc)。

注意：对于多个配置位置，后面的bean定义将覆盖前面加载的文件中定义的配置位置。可以利用这一点，通过额外的XML文件故意覆盖某些bean定义。

### FileSystemXmlApplicationContext:

独立的XML应用程序上下文，从文件系统或url获取上下文定义文件，将普通路径解释为相对的文件系统位置(例如，“mydir / myfile.txt”)。
适用于测试线束以及独立环境。
注意：普通路径总是被解释为相对于当前VM工作目录，即使它们以斜杠开头。(这与Servlet容器中的语义一致。)使用显式的“file:”前缀强制执行绝对文件路径。
配置位置的默认值可以通过getConfigLocations重写，配置位置可以表示具体的文件，比如“/myfiles/context”。xml“或ant样式的模式，比如”/myfiles/*-context。参见org.springframework.util。
模式细节的AntPathMatcher javadoc)。注意:对于多个配置位置，后面的bean定义将覆盖前面加载的文件中定义的配置位置。
可以利用这一点，通过额外的XML文件故意覆盖某些bean定义。这是一个简单的一站式应用程序上下文。考虑将GenericApplicationContext类与org.springframework.bean .factory.xml结合使用。用于更灵活的上下文

- ClassPathXmlApplicationContext  FileSystemXmlApplicationContext 
  一个是从类路径获取配置文件并进一步实例化bean，一个是从文件系统或url加载文件并进一步实例化bean。

## WebApplicationContext

### AnnotationConfigWebApplicationContext是WebApplicationContext实现，
- 它接受带注释的类作为输入—特别是@Configuration-annotated类，
- 也接受普通的@Component类和使用javax兼容JSR-330的类。注入注解。
- 允许逐个注册类(指定类名作为配置位置)以及类路径扫描(指定基本包作为配置位置)。
- 对于多个@Configuration类，后面的@Bean定义将覆盖前面加载的文件中定义的类。
   - 可以利用这一点，通过额外的配置类故意覆盖某些bean定义。提供了注册注解类和扫描注解类等操作：
    ```
    public void register(Class<?>... annotatedClasses) {---注册一个或多个要处理的带注释的类。
        Assert.notEmpty(annotatedClasses, "At least one annotated class must be specified");
        this.annotatedClasses.addAll(Arrays.asList(annotatedClasses));
    }

    public void scan(String... basePackages) {---在指定的包中扫描类
        Assert.notEmpty(basePackages, "At least one base package must be specified");
        this.basePackages.addAll(Arrays.asList(basePackages));
    }
    ```
#### 核心方法：loadBeanDefinitions

![](00301.jpg)

### XmlWebApplicationContext

是WebApplicationContext实现，它从XML文档获取配置，默认情况下，配置将取自“/WEB-INF/applicationContext.xml" for the root context, and "/WEB-INF/test-servlet。对于具有名称空间“test-servlet”的上下文(类似于对于具有servlet-name“test”的DispatcherServlet实例)。

# 在springboot中获取Bean

## 直接注入
使用@Autowired，当然通过context也可以getbean操作
```java
@Controller
@RequestMapping("/a")
public class Demo4Controller {
	//将容器注入进来
    @Autowired
    private ApplicationContext context;
    @RequestMapping("/c")
    public String m1() {
    	//获取所有的bean名称，并打印输出
        String[] beanDefinitionNames = this.context.getBeanDefinitionNames();
        System.out.println(Arrays.asList(beanDefinitionNames));
        //返回逻辑视图名称
        return "demo2";
    }
}
```

## 实现 ApplicationContextAware接口
实现 ApplicationContextAware接口并重写setApplicationContext方法来进行容器获取
```java
@Component
public class SpringBeanUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext = null;

    // 获取ApplicationContext对象
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringBeanUtil.applicationContext = applicationContext;
    }

    /**
     * @Title: getBeanByName
     * @Description: TODO  通过bean的名字来获取Spring容器中的bean
     * @param beanName
     * @return
     * @return: Object
     */
    public static Object getBeanByName(String beanName) {
        if (applicationContext == null){
            return null;
        }
        return applicationContext.getBean(beanName);
    }

    public static <T> T getBean(Class<T> type) {
        return applicationContext.getBean(type);
    }

    public static void getAllBean(){
        String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            System.out.println(beanDefinitionName);
        }
    }
}

```

# 普通bean加入springboot容器

springboot也是一个spring容器。以下是把Bean注入到容器

示例注入类：
```java
public class TestJoinBean {

    @Autowired
    IUserService userService;

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public IUserService getUserService() {
        return userService;
    }

    public void setUserService(IUserService userService) {
        this.userService = userService;
    }
}
```

## @ImportResource 导入xml
在resources下像往常一样编写xml配置文件，在springboot启动类加上@importResource注解并在value数组里指明配置文件的位置，即可把spring容器和springboot容器整合一起。

```java
@ImportResource({
        "classpath:spring-config.xml"
})
```

## @Configuration和@Bean

    上一种方法的弊端是如果存在多个xml配置文件则需要在@ImportResource中引入多个注解。所以springboot推荐使用基于@Configuration注解的形式去替代xml文件来向spring容器添加bean。

    从Spring3.0起@Configuration开始逐渐用来替代传统的xml配置文件。@Configuration继承自@Component，spring会把标注了@Configuration的对象管理到容器中。除此之外还要在某一个方法加上@Bean，该方法的返回对象被当做bean交给容器，该bean的名称为方法的名称。
```java
@Configuration
public class myAppConfig {

    @Bean
    public helloService helloService(){
        System.out.println("给容器添加组件");
        helloService helloService = new helloService();
        return helloService;
    }
}
```

## 使用注解组件-类级别注解,加包扫描的方式注入Bean
如果使用组件注解需要搭配@ComponentScan，该注解会指定需要扫描的包，扫描以后，路径内所有带有组件注解的类都将被注册进IOC容器当中

@Component：定义一个Bean

以下几个注解功能都和@Component相同，一般用于特定位置，便于区分。

@Controller/RestController：定义一个Bean，用于标识控制器，其中@RestController包含了@ResponseBody注解，用于Rest风格接口的开发

@Service等

## 使用@Import注解

SpringBoot的@Import注解可以用于为容器中注册Bean

有三种使用方法：
1. 直接@Import某个类，将这个类添加到IOC容器中,@Import注解需要和@Configuration结合使用才能生效
```java
@Configuration
@Import( { A.class,B.class } )
```

2. 实现了ImportSelector接口的实现类
   - 如果想根据某些属性进行判断是否导入某个Bean，则可以在@Import中传入实现了@ImportSelector的实现类
   - 在执行时，@Import如果判断出传入的是实现了@ImportSelecor接口的实现类，就会调用该类中的selectImport方法，判断需要导入的configuration
```
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        //返回值为String数组类型，实际值应该是注入Bean类的全类名。
        return new String[]{"com.github.bjlhx15.common.springdemo.springcontainer.domain.auto.TestJoinBean"};
    }
}
```
```java
@Configuration
@Import( { MyImportSelector.class } )
```

3. 导入实现了@ImportBeanDefinitionRegistrar接口的实现类
   - 在执行时，会调用该类中的registerBeanDefinitions方法，通过参数中的BeanDefinitionRegistry可以进行Bean的注册
```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //可以从importingClassMetadata和registry对象获取一些上下文信息进行其他业务逻辑的判断
        boolean b1 = registry.containsBeanDefinition("com.github.bjlhx15.common.springdemo.springcontainer.domain.auto.TestJoinBeann");

        if(b1){
            registry.registerBeanDefinition("Class", new RootBeanDefinition(Class.class));
        }
    }
}
```
```java
@Configuration
@Import( { MyImportBeanDefinitionRegistrar.class } )
```

## 手工装载

Spring容器初始化时，从资源中读取到bean的相关定义后，保存在BeanDefinitionMap，在实例化bean的操作就是依据这些bean的定义来做的，而在实例化之前，Spring允许我们通过自定义扩展来改变bean的定义，定义一旦变了，后面的实例也就变了，而beanFactory后置处理器，即BeanFactoryPostProcessor就是用来改变bean定义的。

### 手工装载-实现BeanDefinitionRegistryPostProcessor接口
BeanDefinitionRegistryPostProcessor继承了BeanFactoryPostProcessor接口，BeanFactoryPostProcessor的实现类在其postProcessBeanFactory方法被调用时，可以对bean的定义进行控制，因此BeanDefinitionRegistryPostProcessor的实现类一共要实现以下两个方法：

BeanFactoryPostProcessor可以修改各个注册的Bean，BeanDefinitionRegistryPostProcessor可以动态将Bean注册：
- void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException：
    - 该方法的实现中，主要用来对bean定义做一些改变。

- void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException：
    - 该方法用来注册更多的bean到spring容器中。BeanDefinitionRegistry提供了如下操作BeanDefinition，判断、注册、移除等方法

```java
@Component
public class ManualRegistBeanUtil2 implements BeanDefinitionRegistryPostProcessor {

    private BeanDefinitionRegistry beanDefinitionRegistry;
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry beanDefinitionRegistry1) throws
            BeansException {
        // 创建一个bean的定义类的对象
//        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(clazz);
        // 将Bean 的定义注册到Spring环境
        beanDefinitionRegistry=beanDefinitionRegistry1;
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        // bean的名字为key, bean的实例为value
        Map<String, Object> beanMap = configurableListableBeanFactory.getBeansWithAnnotation(AutoDiscoverClass.class);
    }

    public void registerBean(Class clazz){
        // 创建一个bean的定义类的对象
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(clazz);
        // 将Bean 的定义注册到Spring环境
        beanDefinitionRegistry.registerBeanDefinition("testService", rootBeanDefinition);
    }
}
```

使用
```java
    @Autowired
    private ManualRegistBeanUtil2 registerBeanUtil2;
    //代码中
    registerBeanUtil2.registerBean(TestJoinBean.class);

    TestJoinBean bean = context.getBean(TestJoinBean.class);
    System.out.println(bean);
    System.out.println(bean.getUserService());
```



### 手工装载-ConfigurableApplicationContext获取工厂

```java
public class ManualRegistBeanUtil {
    public static <T> T registerBean(ConfigurableApplicationContext applicationContext, String name, Class<T> clazz, Object... args) {
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(clazz);
        for (Object arg : args) {
            beanDefinitionBuilder.addConstructorArgValue(arg);
        }
        BeanDefinition beanDefinition = beanDefinitionBuilder.getRawBeanDefinition();

        BeanDefinitionRegistry beanFactory = (BeanDefinitionRegistry) applicationContext.getBeanFactory();
        beanFactory.registerBeanDefinition(name, beanDefinition);
        return applicationContext.getBean(name, clazz);
    }
}
```

测试：
```java
        ManualRegistBeanUtil.registerBean(applicationContext, "test", TestJoinBean.class);

        TestJoinBean bean = context.getBean(TestJoinBean.class);
        System.out.println(bean);
        System.out.println(bean.getUserService());
```

### 手工装载-GenericApplicationContext

有的时候在有类型后需要将现有某个类装载到spring容器托管中
```java
    @Autowired
    private GenericApplicationContext gac;
    //代码中
    gac.registerBean(TestJoinBean.class);
    TestJoinBean bean = context.getBean(TestJoinBean.class);
```
注入后的名称为 类全名：com.github.bjlhx15.common.springdemo.springcontainer.domain.auto.TestJoinBean

如果使用这种注入 TestJoinBean ，这里如有有依赖注入应该怎么使用，经测试使用Autowired,可以注入使用，如有初始化的信息要卸载哪里

## 类初始化顺序-依赖注入使用时需要在PostConstruct注解中使用
```java
@Component
public class CtorAutoPostSort {
    public CtorAutoPostSort() {
        System.out.println("构造函数:" + iUserService);

    }

    @Autowired(required = false)
    IUserService iUserService;

    @PostConstruct
    public void init() {

        System.out.println("PostConstruct:" + iUserService);
    }

    {
        System.out.println("构造代码块:" + iUserService);
    }

    static {
        System.out.println("静态代码块:不能获取普通属性");
    }
}
```
类实例化过程：静态代码块>构造代码块>构造函数>@Autowired>@PostConstruct

```
静态代码块:不能获取普通属性
构造代码块:null
构造函数:null
PostConstruct
PostConstruct:com.github.bjlhx15.common.springdemo.event.service.impl.UserServiceImpl@146dcfe6
```
