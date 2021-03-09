---
title: 002-springboot测试-Mockito服务层测试
categories:
  - java-spring-test
abbrlink: 1a42b9be
date: 2021-01-07 18:37:22
tags:
---

摘要：002-springboot测试-Mockito服务层测试
- 使用实际真实接口数据测试
- 使用Mock接口数据测试
- 使用Mock接口数据、以及真实接口测试

<!-- more -->

# 什么是Mockito
    Mockito是一个java测试框架，它提供mock的创建，验证，打桩。在测试中，可能会依赖外部资源，比如数据库等，mockito可以模拟这些数据，进行测试。

## 打桩

就是模拟一些函数调用的反应。

```
    Mockito.when(list.get(1)).thenReturn(3); // 含义遇到 list.get(1) 等于 3 不执行内部
    Assert.assertEquals(list.get(1),3);
```
- 可以进行多次打桩，但是以最后一次为准。
- 一次打桩，可多次调用

## 参数匹配

```
    Mockito.when(list.get(Mockito.anyInt())).thenReturn("hi");//只要传入任何int，返回hi
    Assert.assertEquals("hi",list.get(1));//hi
    Assert.assertEquals("hi",list.get(999));//hi
    Mockito.when(list.contains(Mockito.isA(One.class))).thenReturn(true);//只要传入String类型，就返回hello
    Assert.assertTrue(list.contains("hello"));//hello
    Assert.assertTrue(list.contains(1));//发生错误
```

参数匹配器是为了更加灵活的进行验证和打桩，可以自定义

## Mockito工具常用注解
有些函数需要处理某个服务的返回结果，而在对函数单元测试的时候，又不能启动那些服务

- @Mock：对函数的调用均执行mock（即虚假函数），不执行真正部分。
- @Spy：对函数的调用均执行真正部分。
- @InjectMocks：创建一个实例，简单的说是这个Mock可以调用真实代码的方法，其余用@Mock（或@Spy）注解创建的mock将被注入到用该实例中。
    - @Mock出的对象会被注入到@InjectMocks对象中
    - 它会把上下文中你标记为@Spy和@Mock的对象都自动注解进去。相当于把实现类中的私有成员属性给偷梁换柱
    - 注意当前没有问题了 但是service包含了多个既有真正实现也有mock的需要使用反射注入 
    ```
    ReflectionTestUtils.setField(service, "departService", departService);
    ```

Mockito中的Mock和Spy都可用于拦截那些尚未实现或不期望被真实调用的对象和方法，并为其设置自定义行为。二者的区别在于Mock不真实调用，Spy会真实调用。

# 使用示例

## 真实数据

```java
@RunWith(SpringRunner.class)//调用Spring单元测试类
@SpringBootTest(classes = DemoApplication.class) //加载Spring配置文件
public class UserServiceImplTest01RealData {

    @Autowired
    private IUserService service;

    @Test
    public void insertUser() {

        User user = new User();
        user.setName("lhx");
        user.setAge(202);

        service.insertUser(user);
    }
}
```

## 全部mock数据
```java
@RunWith(SpringRunner.class)//调用Spring单元测试类
@ContextConfiguration(locations = {"classpath*:spring-config-mybatis-jd-dev.xml"})
public class UserServiceImplTest02MockData {

    @Mock
    private IDepartService departService;

    @Mock
    private UserMapper mapper ;

    @InjectMocks
    private UserServiceImpl service;


    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void insertUserAndDepart() {
        User user = new User();
        user.setName("lhx");
        user.setAge(202);
        String userStr = JSON.toJSONString(user);
        when(service.insertUser(user))
                .thenReturn(1);
        when(departService.insert(user))
                .thenReturn(1);
        int i = service.insertUserAndDepart(user);
        Assert.assertEquals(2,i);
    }
}
```

## 部分mock数据、部分真实接口

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = DemoApplication.class) //加载Spring配置文件
@Transactional
@Rollback
public class UserServiceImplTest03PartMockPartReal {

    @Mock
    private IDepartService departService;

    @Autowired
    private UserMapper mapper;

    @Autowired
    @InjectMocks
    private UserServiceImpl service;


    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
        ReflectionTestUtils.setField(service, "departService", departService);
    }

    @Test
    public void insertUserAndDepart() {
        User user = new User();
        user.setName("lhx");
        user.setAge(202);

        when(departService.insert(user))
                .thenReturn(1);
//        when(mapper.insert(user))
//                .thenReturn(1);
        int i = service.insertUserAndDepart(user);
        Assert.assertEquals(2, i);
    }
}
```

# 常用方法
```
方法名	描述
Mockito.mock(classToMock)	模拟对象
Mockito.verify(mock)	验证行为是否发生
Mockito.when(methodCall).thenReturn(value1).thenReturn(value2)	触发时第一次返回value1，第n次都返回value2
Mockito.doThrow(toBeThrown).when(mock).[method]	模拟抛出异常。
Mockito.mock(classToMock,defaultAnswer)	使用默认Answer模拟对象
Mockito.when(methodCall).thenReturn(value)	参数匹配
Mockito.doReturn(toBeReturned).when(mock).[method]	参数匹配（直接执行不判断）
Mockito.when(methodCall).thenAnswer(answer))	预期回调接口生成期望值
Mockito.doAnswer(answer).when(methodCall).[method]	预期回调接口生成期望值（直接执行不判断）
Mockito.spy(Object)	用spy监控真实对象,设置真实对象行为
Mockito.doNothing().when(mock).[method]	不做任何返回
Mockito.doCallRealMethod().when(mock).[method] //等价于Mockito.when(mock.[method]).thenCallRealMethod();	调用真实的方法
reset(mock)	重置mock
```

# MockitoJUnitRunner 与 SpringRunner(SpringJUnit4ClassRunner)说明

## MockitoJUnitRunner
- 特别适用于Mockito测试框架
- 当您希望将测试集中在单个类上并避免在依赖项上调用方法时（而是调用易于配置的模拟/虚拟），Mockito框架可以帮助模拟依赖项。 
- 以上是mockito的用途，但有关此运行的更多内容 - 来自文档：“保持测试清洁并改善调试体验”。“Runner是完全可选的 - 还有其他方法可以让@Mock工作”。来源 - https://static.javadoc.io/org.mockito/mockito-core/2.6.8/org/mockito/junit/MockitoJUnitRunner.html

## 基于SpringJUnit4ClassRunner
- 特别适用于弹簧框架
- 当需要加载spring上下文（创建spring bean，执行依赖注入等）时，用于集成测试。  
- 在集成测试中，您可能无法对依赖项进行尽可能多的模拟，但您可以在同一测试中执行这两项操作。  
- 当您想要测试加载弹簧上下文或者可能从服务/高级别一直测试到较低级别（如使用单个测试的数据访问）时，集成测试非常有用。

在某些情况下，您可能希望同时使用它们 - 比如集成测试，您还希望模拟某些依赖项（可能是它们进行远程调用）。不幸的是你不能使用两个@RunWiths
可以参看上述第三种：部分mock数据、部分真实接口

# @RunWith(MockitoJUnitRunner.class) vs MockitoAnnotations.initMocks(this)

MockitoJUnitRunner 和 initMocks(this) 都可以为UT提供框架使用的自动验证

- 在单元测试中使用@Mock, @Spy, @InjectMocks等注解时，需要进行初始化后才能使用
- 若在单元测试类中使用了@RunWith(SpringJUnit4ClassRunner.class) 就不能再使用@RunWith(SpringJUnit4ClassRunner.class)，可以使用 MockitoAnnotations.initMocks(this) 来代替
- MockitoAnnotations.initMocks(this)，其中this就是单元测试所在的类，在initMocks函数中Mockito会根据类中不同的注解（如@Mock, @Spy等）创建不同的mock对象，即初始化工作

# 参看文章
[1](https://blog.csdn.net/gentlezuo/article/details/108293961)