---
title: SpringBoot
date: 2020-02-17 15:46:49
tags:
---

# 入门

- `spring-boot-starter`:核心模块，包括自动配置支持、日志和YAML
- `spring-boot-starter-test`:测试模块，包括JUnit、Hamcrest、Mockitso
- `spring-boot-starter-web`:Web模块

```properties
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```java
//创建HelloController
@RestController
@RequestMapping(value="/users")//下面的映射都在/users下
public class HelloController{
    @GetMapping("/hello")
    public String index(){
        return "Hello World";
    }
}

//创建单元测试
@RunWith(SpringJUint4ClassRunner.class)
@SoringApplicationConfiguration(classes = MockServletContext.class)
@WebAppConfiguration
public class HelloTest{
    private MockMvc mvc;
    
    @Before
    public void setUp() throws Exception{
        mvc = MockMvcBuliders.standaloneSetup(new HelloController()).builder();
    }
    
    @Test
    public void getHello throws Exception{
        mvc.perform(MockMvcRequestBuilders.get("/hello")
                   .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOK())
            .andExcept(content().string(equalTo("Hello World")));
    }
}

/*
使用MockServletContext来构建一个空的WebAppliactionContext,然后创建的HelloController就可以再@Before中创建并传递到MockMvcBuliders.standaloneSetup()中。
*/
```

- `@Controller`: 修饰class，用来创建http请求的对象
- `@RestController`: Spring4后加入的注解，相当于使用`@Controller`和`@ResponseBody`(用于返回json格式)
- `@RequestMapping`: 配置url映射

`application.properties`

```properties
com.wan.example.name=wanfeng
com.wan.example.age=22
```

```java
//再application.properties中定义然后通过@Value("${属性名}")注解来加载对应配置属性
@Component
public class MyProperties{
    @Value("${com.wan.example.name}")
    private String name;
    @Value("${com.wan.example.age}")
    private int age;
    //getter，setter
}
```

```java
//通过单元测试验证属性是否根据配置文件加载
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(Application.class)//启动类
public class ApplicationTest{
    @AutoWired
    private MyProperties myProperties;
    
    @Test
    public void getHello() throws Exception{
        Assert.assertEquals(myProperties.getName(),"wanfneg");
        Assert.assertEquals(myProperties.getAge(),22);
    }
}
```

#### 通过命令行设置属性值

`java -jar xxx.jar --server.port=8888`

#### 屏蔽命令行访问属性设置

`SpringApplication.setAddCommandLineProperties(false)`

#### 多环境配置

在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中

- `application-dev.properties`:开发环境
- `application-test.properties`:测试环境
- `application-prod.properties`:生产环境

在`application.properties`中设置`spring.profiles.active=xxx`就会加载对应配置文件

# 构建RESTful API

| 请求类型 | URL       | 功能说明           |
| -------- | --------- | ------------------ |
| GET      | /users    | 查询用户列表       |
| POST     | /users    | 创建一个用户       |
| GET      | /users/id | 根据id查询一个用户 |
| PUT      | /users/id | 根据id更新一个用户 |
| DELETE   | /users/id | 根据id删除一个用户 |

```java
//User
public class User {
    private Integer id;
    private String name;
    private int age;
	
    //getter,setter
}

//UserController
@RestController
@RequestMapping(value = "/users")
public class UserController {
    static Map<Integer,User> userMap = Collections.synchronizedMap(new HashMap<Integer,User>());

    @RequestMapping(value = "/",method = RequestMethod.GET)
    public List<User> getUserList(){
        // 处理"/users/"的GET请求，用来获取用户列表
        // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递
        List<User> users = new ArrayList<User>(userMap.values());
        return  users;
    }

    @RequestMapping(value = "/",method = RequestMethod.POST)
    public String postUser(@ModelAttribute User user){
        // 处理"/users/"的POST请求，用来创建User
        // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数
        userMap.put(user.getId(),user);
        return "success";
    }

    @RequestMapping(value = "/{id}",method = RequestMethod.GET)
    public User getUser(@PathVariable Integer id){
        // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息
        // url中的id可通过@PathVariable绑定到函数的参数中
        return userMap.get(id);
    }

    @RequestMapping(value = "/{id}",method = RequestMethod.PUT)
    public String putUser(@PathVariable Integer id,@ModelAttribute User user){
        // 处理"/users/{id}"的PUT请求，用来更新User信息
        User u = userMap.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        userMap.put(id,u);
        return "success";
    }

    @RequestMapping(value = "/{id}",method = RequestMethod.DELETE)
    public String deleteUser(@PathVariable Integer id){
        // 处理"/users/{id}"的PUT请求，用来更新User信息
        userMap.remove(id);
        return "success";
    }
}
//Test
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
@WebAppConfiguration
public class ApplicationTest {

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception{
        mvc = MockMvcBuilders.standaloneSetup(new UserController()).build();
    }

    @Test
    public void testUserController() throws Exception{

        RequestBuilder request = null;

        // 1. get
        request = get("/users/");
        mvc.perform(request)
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("[]")));

        // 2. post
        request = post("/users/")
                    .param("id","1")
                    .param("name","wan")
                    .param("age","22");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));

//        // 3. get{id=1}
//        request = get("/users/1");
//        mvc.perform(request)
//                .andExpect(content().string(equalTo("{" +
//                        "id:" + 1+
//                        "name:" + "wan"+
//                        "age:" +22+
//                        "}")));

        // 4. pust
        request = put("/users/1")
                .param("name","feng")
                .param("age","23");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));

        // 5. delete
        request = delete("/users/1");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));
    }
}

//Application
@SpringBootApplication
public class Application {
  public static void main(String[] args) {
      SpringApplication.run(Application.class,args);
  }
}
```

# 使用swagger构建RESTful API文档

Swagger2的依赖

```properties
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
```

在`Application.java`同级创建Swagger2的配置类

```java
//Swagger2
@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.wan"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("more....")
                .termsOfServiceUrl("wan")
                .contact("wan")
                .version("1.0")
                .build();
    }
}
/**
 * 通过@Configuration注解，让Spring来加载该类配置。再通过@EnableSwagger2注解来启用Swagger2。
 *
 * 再通过createRestApi函数创建Docket的Bean之后，apiInfo()用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。
 * select()函数返回一个ApiSelectorBuilder实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被@ApiIgnore指定的请求）。
 */
```

添加文档内容

```java
/**
* 在完成了上述配置后，其实已经可以生产文档内容，
* 但是这样的文档主要针对请求本身，而描述主要来源于函数等命名产生，对用户并不友好，
* 我们通常需要自己增加一些说明来丰富文档内容。
* 我们通过@ApiOperation注解来给API增加说明、通过@ApiImplicitParams、@ApiImplicitParam注解来	 给参数增加说明。
*/
@RestController
@RequestMapping(value = "/users")
public class UserController {
    static Map<Integer,User> userMap = Collections.synchronizedMap(new HashMap<Integer,User>());

    @ApiOperation(value = "获取用户列表",notes = "")
    @RequestMapping(value = {""},method = RequestMethod.GET)
    public List<User> getUserList(){
        // 处理"/users/"的GET请求，用来获取用户列表
        // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递
        List<User> users = new ArrayList<User>(userMap.values());
        return  users;
    }

    @ApiOperation(value = "创建用户",notes = "根据User对象创建用户")
    @ApiImplicitParam(name = "user",value = "用户实体user",required = true,dataType = "User")
    @RequestMapping(value = "/",method = RequestMethod.POST)
    public String postUser(@ModelAttribute User user){
        // 处理"/users/"的POST请求，用来创建User
        // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数
        userMap.put(user.getId(),user);
        return "success";
    }

    @ApiOperation(value="获取用户详细信息", notes="根据url的id来获取用户详细信息")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Integer")
    @RequestMapping(value = "/{id}",method = RequestMethod.GET)
    public User getUser(@PathVariable Integer id){
        // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息
        // url中的id可通过@PathVariable绑定到函数的参数中
        return userMap.get(id);
    }

    @ApiOperation(value="更新用户详细信息", notes="根据url的id来指定更新对象，并根据传过来的user信息来更新用户详细信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Integer"),
            @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    })
    @RequestMapping(value = "/{id}",method = RequestMethod.PUT)
    public String putUser(@PathVariable Integer id,@ModelAttribute User user){
        // 处理"/users/{id}"的PUT请求，用来更新User信息
        User u = userMap.get(id);
        u.setName(user.getName());
        u.setAge(user.getAge());
        userMap.put(id,u);
        return "success";
    }

    @ApiOperation(value="删除用户", notes="根据url的id来指定删除对象")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Integer")
    @RequestMapping(value = "/{id}",method = RequestMethod.DELETE)
    public String deleteUser(@PathVariable Integer id){
        // 处理"/users/{id}"的PUT请求，用来更新User信息
        userMap.remove(id);
        return "success";
    }
}
```

# Web应用的统一异常处理

# Java 8时间日期API（LocalDate等）的序列化问题

# 使用Spring Security进行安全控制

添加依赖

```properties
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Spring Security配置

```java
//创建Spring Security配置类WebSecurityConfig
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/","/hello").permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
                .logout()
                .permitAll();

    }
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception{
        auth.inMemoryAuthentication()
                .withUser("user").password("password").roles("USER");
    }
}
```

- 通过`@EnableWebSecurity`注解开启Spring Security功能
- 继承`WebSecurityConfigurerAdapter`，并重写方法设置Web安全的细节
- `configure(HttpSecurity http)`方法
  - 通过`authorizeRequests()`定义哪些URL需要被保护、哪些不需要保护
  - 通过`loginPage()`定义当需要登录时，转到的登录页面
- `configureGlobal(AuthenticationManagerBuilder auth)`方法，在内存中创建了一个用户，该用户的名称为user，密码为password，用户角色为USER。

# 使用JdbcTemplate访问数据库

# 使用Spring-data-jpa让数据访问更简单、更优雅

# 多数据源配置与使用

# 使用Flyway来管理数据库版本

# 使用LDAP来统一管理用户信息

# Spring Boot中的事务管理

# [使用Redis做集中式缓存](http://blog.didispace.com/springbootcache2/)

