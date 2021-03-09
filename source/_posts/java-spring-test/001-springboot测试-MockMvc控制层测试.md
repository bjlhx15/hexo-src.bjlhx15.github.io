---
title: 001-springboot测试-MockMvc控制层测试
categories:
  - java-spring-test
abbrlink: 4f18d467
date: 2021-01-07 04:37:18
tags:
---

摘要：springboot测试-MockMvc
<!-- more -->

# 什么是Mock
    在面向对象的程序设计中，模拟对象（英语：mock object）是以可控的方式模拟真实对象行为的假对象。在编程过程中，通常通过模拟一些输入数据，来验证程序是否达到预期结果。

# 为什么使用Mock对象

使用模拟对象，可以模拟复杂的、真实的对象行为。如果在单元测试中无法使用真实对象，可采用模拟对象进行替代。

在以下情况可以采用模拟对象来替代真实对象：

- 真实对象的行为是不确定的（例如，当前的时间或温度）；
- 真实对象很难搭建起来；
- 真实对象的行为很难触发（例如，网络错误）；
- 真实对象速度很慢（例如，一个完整的数据库，在测试之前可能需要初始化）；
- 真实的对象是用户界面，或包括用户界面在内；
- 真实的对象使用了回调机制；
- 真实对象可能还不存在；
- 真实对象可能包含不能用作测试（而不是为实际工作）的信息和方法。
- 使用Mockito一般分三个步骤：1、模拟测试类所需的外部依赖；2、执行测试代码；3、判断执行结果是否达到预期；

# MockMvc

MockMvc是由spring-test包提供，实现了对Http请求的模拟，能够直接使用网络的形式，转换到Controller的调用，使得测试速度快、不依赖网络环境。同时提供了一套验证的工具，结果的验证十分方便。

接口MockMvcBuilder，提供一个唯一的build方法，用来构造MockMvc。主要有两个实现：StandaloneMockMvcBuilder和DefaultMockMvcBuilder，分别对应两种测试方式，即独立安装和集成Web环境测试（并不会集成真正的web环境，而是通过相应的Mock API进行模拟测试，无须启动服务器）。MockMvcBuilders提供了对应的创建方法standaloneSetup方法和webAppContextSetup方法，在使用时直接调用即可。

# SpringBoot中使用

## jar引入
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 测试代码
[github地址](https://github.com/bjlhx15/common.git) 

- demo简述待测试类

```java
@RestController
@RequestMapping("/user/postjson")
public class UserPostJsonController {
    @Autowired
    private IUserService userService;

    @ResponseBody
    @RequestMapping("/insert")
    public ResponseEntity insert(@RequestBody User record) {
        System.out.println(JSON.toJSONString(record));
        return ResponseEntity.ok(userService.insertUser(record));
    }
}
```

- 编写基础测试

```java
//SpringBoot1.4版本之前用的是SpringJUnit4ClassRunner.class
@RunWith(SpringRunner.class)
//SpringBoot1.4版本之前用的是@SpringApplicationConfiguration(classes = Application.class)
//@ContextConfiguration(locations = {"classpath*:spring-*.xml"})      //加载Spring配置文件 方式
@SpringBootTest(classes = DemoApplication.class)
//测试环境使用，用来表示测试环境使用的ApplicationContext将是WebApplicationContext类型的
@WebAppConfiguration
public class UserBaseControllerTest {

    @Autowired
    protected WebApplicationContext context;

    private MockMvc mockMvc;        //SpringMVC提供的Controller测试类

    @Before
    public void setUp() throws Exception {
        // 实例化方式一、对于controller有依赖的 无法注入
        // mockMvc = MockMvcBuilders.standaloneSetup(new UserPostJsonController()).build();
        // 实例化方式二、对于controller有依赖的 可以注入
         mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    }

    @Test
    public void insert() throws Exception {
        User user = new User();
        user.setName("lhx");
        user.setAge(20);
        String userStr = JSON.toJSONString(user);
        mockMvc
                .perform(MockMvcRequestBuilders.post("/user/postjson/insert")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .content(userStr)
                        .accept(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn()
                .getResponse().getContentAsString();  //将相应的数据转换为字符串
    }
}
```

- 说明
    - mockMvc.perform   执行一个请求。
    - MockMvcRequestBuilders.get("XXX") 构造一个请求，post等
    - ResultActions.param   添加请求传值
        - 请求为 get 可以使用
        - 请求为 post application/x-www-form-urlencoded 可以使用
        - 请求为 post application/json 需要使用 ResultActions.contentType(MediaType.APPLICATION_JSON_UTF8) 以及 ResultActions.content(userStr)
    - ResultActions.accept(MediaType.APPLICATION_JSON))  设置返回类型
    - ResultActions.andExpect   添加执行完成后的断言 如是状态码 200 MockMvcResultMatchers.status().isOk()
    - ResultActions.andDo       添加一个结果处理器，表示要对结果做点什么事情
        - 比如此处使用MockMvcResultHandlers.print()输出整个响应结果信息。
    - ResultActions.andReturn表示执行完成后返回相应的结果。
    
注意事项：如果使用DefaultMockMvcBuilder进行MockMvc实例化时需在SpringBoot启动类上添加组件扫描的package的指定，否则会出现404。
```
@ComponentScan(basePackages = "com.github.bjlhx15")
```

## 常用测试示例

- 通用头配置
```
//@Profile({"dev","dev-fesco"}) //环境变量
//SpringBoot1.4版本之前用的是SpringJUnit4ClassRunner.class
@RunWith(SpringRunner.class)
//SpringBoot1.4版本之前用的是@SpringApplicationConfiguration(classes = Application.class)
//@ContextConfiguration(locations = {"classpath*:spring-*.xml"})      //加载Spring配置文件 方式
@SpringBootTest(classes = DemoApplication.class)
//测试环境使用，用来表示测试环境使用的ApplicationContext将是WebApplicationContext类型的
@WebAppConfiguration
//配置事务的回滚,对数据库的增删改都会回滚, Spring框架中（Spring4.2以后），@TransactionConfiguration已经标注为过时的注解
@Rollback 
@Transactional
```
- 通用初始化构造
```
    @Autowired
    protected WebApplicationContext context;

    private MockMvc mockMvc;        //SpringMVC提供的Controller测试类

    @Before
    public void setUp() throws Exception {
        // 实例化方式一、对于controller有依赖的 无法注入
//        mockMvc = MockMvcBuilders.standaloneSetup(new UserPostJsonController()).build();
        // 实例化方式二、对于controller有依赖的 可以注入
         mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    }
```

- 常规测试
```
 .andExpect(model().attributeExists("user")) //验证存储模型数据  
 .andExpect(view().name("user/view")) //验证viewName  
 .andExpect(forwardedUrl("/WEB-INF/jsp/user/view.jsp"))//验证视图渲染时forward到的jsp  
 .andExpect(status().isOk())//验证状态码  
 .andDo(print()); //输出MvcResult到控制台
```

- 得到MvcResult自定义验证
```
MvcResult result = mockMvc.perform(get("/user/{id}", 1))//执行请求  
        .andReturn(); //返回MvcResult  
Assert.assertNotNull(result.getModelAndView().getModel().get("user")); //自定义断言
```

- 验证请求参数绑定到模型数据及Flash属性
```
mockMvc.perform(post("/user").param("name", "zhang")) //执行传递参数的POST请求(也可以post("/user?name=zhang"))  
            .andExpect(handler().handlerType(UserController.class)) //验证执行的控制器类型  
            .andExpect(handler().methodName("create")) //验证执行的控制器方法名  
            .andExpect(model().hasNoErrors()) //验证页面没有错误  
            .andExpect(flash().attributeExists("success")) //验证存在flash属性  
            .andExpect(view().name("redirect:/user")); //验证视图 
```

- 文件上传
```
byte[] bytes = new byte[] {1, 2};  
mockMvc.perform(fileUpload("/user/{id}/icon", 1L).file("icon", bytes)) //执行文件上传  
        .andExpect(model().attribute("icon", bytes)) //验证属性相等性  
        .andExpect(view().name("success")); //验证视图 
```

- json验证
```
    mockMvc.perform(post("/user")  
            .contentType(MediaType.APPLICATION_JSON).content(requestBody)  
            .accept(MediaType.APPLICATION_JSON)) //执行请求  
            .andExpect(content().contentType(MediaType.APPLICATION_JSON)) //验证响应contentType  
            .andExpect(MockMvcResultMatchers.jsonPath("$.id").value(1)); //使用Json path验证JSON
```

- 异步测试
```
//Callable  
    MvcResult result = mockMvc.perform(get("/user/async1?id=1&name=zhang")) //执行请求  
            .andExpect(request().asyncStarted())  
            .andExpect(request().asyncResult(CoreMatchers.instanceOf(User.class))) //默认会等10秒超时  
            .andReturn();  
      
    mockMvc.perform(asyncDispatch(result))  
            .andExpect(status().isOk())  
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))  
            .andExpect(jsonPath("$.id").value(1)); 
```
- post json
```
        String userStr = JSON.toJSONString(user);
        mockMvc.perform(MockMvcRequestBuilders.post("/user/postjson/insert")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(userStr)
                .accept(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn();
```

- post form
```
        mockMvc.perform(MockMvcRequestBuilders.post("/user/postform/insert")
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
//                        .content(userStr)   //这种无效
                .param("name", "lhx")
                .param("age", "30")
                .accept(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn();
```

- get
```
        mockMvc.perform(MockMvcRequestBuilders.get("/user/postform/insert")
                .param("name", "lhx")
                .param("age", "30")
                .accept(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn();
```

- 全局配置
```
mockMvc = webAppContextSetup(wac)  
            .defaultRequest(get("/user/1").requestAttr("default", true)) //默认请求 如果其是Mergeable类型的，会自动合并的哦mockMvc.perform中的RequestBuilder  
            .alwaysDo(print())  //默认每次执行请求后都做的动作  
            .alwaysExpect(request().attribute("default", true)) //默认每次执行后进行验证的断言  
            .build();  
      
    mockMvc.perform(get("/user/1"))  
            .andExpect(model().attributeExists("user"));
```




# 参看文章
[SpringBoot基础之MockMvc单元测试](https://blog.csdn.net/wo541075754/article/details/88983708)