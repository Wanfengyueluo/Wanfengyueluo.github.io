---
title: Spring-Boot-Vue全栈开发实战笔记
date: 2020-11-09 08:54:12
description: Spring Boot+Vue全栈开发实战笔记
categories: Spring Boot
tags:
	- Spring Boot
	- Vue
---

# *Spring Boot+Vue全栈开发实战笔记*

# Spring Boot入门

## 不使用spring-boot-starter-parent

**spring-boot-starter-parent主要提供的默认配置**：

- Java版本默认使用1.8
- 编码格式默认使用UTF-8
- 提供Dependency Managerment进行项目依赖的版本管理
- 默认的资源过滤与插件配置

当不使用`spring-boot-starter-parent`时需要通过dependencyManagerment来实现项目依赖版本的统一管理

```xml
<dependencyManagerment>
	<dependencies>
    	<dependency>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.4.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagerment>
```

此时需要配置Java版本和编码格式

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
    	<source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>

<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
```

## 定制banner

在`resources`目录下创建一个banner.txt文件，网站参考：

- http://www.network-science.de/ascii/

也可以关闭banner,通过SpringApplicationBuilder来设置bannerMode为OFF。

```java
//修改main方法
public static void main(String[] args){
    SpringApplicationBuilder builder = new SpringApplicationBuilder(Application.class);
    builder.bannerMode(Banner.Mode.OFF).run(args);
}
```

## Web容器设置

### Tomcat配置

当添加`spring-boot-starter-web`依赖后，默认会使用Tomcat作为Web容器

### Jetty配置

### Undertow配置

## Properties配置

Spring Boot项目中的application.properties配置文件的位置优先级：

- 项目根目录下的config文件夹中
- 项目根目录下
- classpath(resources)下的config文件夹中
- classpath(resources)下

`application.properties`中的数据可以注入到Bean中，如：

```properties
book.name=xxx
book.author=yyy
```

```java
//将配置注入
@Component
@ConfigurationProperties(prefix = "book")
public class Book{
    private String name;
    private String author;
}
```

## YAML配置

```yaml
my:
	users:
		- name:xxx
		 address:China
		 favorite:
		 	- a
		 	- b
		 	- c
		- name:yyy
		  address:China
		  favorite:
		  	- a
		  	- b
		  	- c
```

```java
@Component
@ConfigurationProperties(prefix = "my")
public class Users {
    private List<User> users;
}

public class User {
    private String name;
    private String address;
    private List<String> favorites;
}
```

在Spring Boot中使用YAML有缺陷，例如无法使用`@PropertySource`注解加载YAML文件

## Profile配置

### 配置文件配置

1. 创建`application-dev.properties`和`application-prod.properties`
2. 在`application.properties`中配置`spring.profiles.active=xxx`

### 代码配置

```java
//修改main方法
public static void main(String[] args){
    SpringApplicationBuilder builder = new SpringApplicationBuilder(Application.class);
    builder.application().setAdditionProfiles("prod").run(args);
}
```

### 项目启动时配置

`java -jar xxx.jar --spring.profiles.active=prod`

# Spring Boot整合视图层

## Thymeleaf

### 添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### 配置Thymeleaf

Spring Boot 为Thymeleaf提供自动化配置类ThymeleafAutoConfiguration,相关配置属性在ThymeleafProperties类中，可以在`application.properties`中进行自定义配置：

```properties
#是否开启缓存，默认true
spring.thymeleaf.cache=true
#检查模板是否存在，默认true
spring.thymeleaf.cheak-template=true
#检查模板位置是否存在，默认true
spring.thymeleaf.cheak-template-location=true
#模板文件编码
spring.thymeleaf.encoding=UTF-8
#模板文件位置
spring.thymeleaf.prefix=classpath:/templates/
#Content-Type配置
spring.thymeleaf.servlet.content-type=text.heml
#模板文件后缀
spring.thymeleaf.suffix=.html
```

### 配置控制器

创建User实体类，然后在Controller中返回ModerAndView

```java
public class User {
    private Integer id;
    private String name;
}

@RestController
public class UserController {
    @GetMapping("/users")
    public ModelAndView users() {
        List<User> users = new ArrayList<>();
        User user = new User();
        user.setId(1);
        user.setName("wan");
        users.add(user);
        
        ModelAndView mv = new ModelAndView();
        mv.addObject("users",users);
        mv.setViewName("users");
        return mv;
    }
}
```

### 创建视图

在`resources/templates`目录下创建`users.html`

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>用户列表</title>
    </head>
    <body>
        <table border="1">
            <tr>
            	<td>用户编号</td>
                <td>用户姓名</td>
            </tr>
            <tr th:each="user:${users}">
            	<td th:text="${user.id}">用户编号</td>
                <td th:text="${user.name}">用户姓名</td>
            </tr>
        </table>
    </body>
</html>
```

首先导入Thymeleaf的命名空间，然后通过th:each进行遍历，通过th:text展示数据

## FreeMarker（非常古老的模板引擎...）

# Spring Boot整合Web

## 返回JSON数据

### 默认实现

`spring-boot-starter-web`默认将`jackson-databind`作为JSON处理器

此时对于字段忽略、日期格式化等可以通过注解实现

```java
public class Book {
    private String name;
    private String author;
    @JsonIgnore
    private Float price;
    @JsonFormat(pattern="yyyy-MM-dd")
    private Date publicationDate;
}
```

### 自定义转换器

#### Gson

首先，引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
    	<exclusion>
        	<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
	<groupId>com.google.code.gson</groupId>
	<artifactId>gson</artifactId>
</dependency>
```

然后提供一个GsonHttpMessageConverter

```java
@Configuration
public class GsonConfig {
    @Bean
    GsonHttpMessageConverter gsonHttpMessageConverter() {
        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        GsonBuilder builder = new GsonBuilder();
        builder.setDateFormat("yyyy-MM-dd");
        builder.excludeFieldsWithModifiers(Modifier.PROTECTED);
        Gson gson = builder.create();
        converter.setGson(gson);
        return converter;
    }
}
```

- 提供一个GsonHttpMessageConverter实例
- 设置Gson解析时日期的格式
- 设置Gson解析时修饰符为protected的字段被过滤
- 创建Gson对象放入GsonHttpMessageConverter的实例中并返回converter

```java
public class Book {
    private String name;
    private String author;
    protected Float price;//忽略
    private Date publicationDate;
}
```

#### fastjson...

## 静态资源访问

### 默认策略

`classpath`为`resources`

- `classpath:/META_INF/resources/`
- `classpath:/resources/`
- `classpath:/static/`(IDEA默认)
- `classpath:/public/`

### 自定义策略

#### 配置文件中定义

`application.properties`:

```properties
spring.mvc.static-path-pattern=/static/**
spring.resources.static-locations=classpath:/static/
```

过滤规则为`/static/**`，静态资源位置为`classpath:/static/`

#### Java编码定义

实现`WebMvcConfigurer`接口,然后实现`addResourceHandlers`方法

```java
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry
            .addResourceHandler("/static/**")
            .addResourceLocations("classpath:/static/");
    }
}
```

## 文件上传(...)

## @ControllerAdvice

`@ControllerAdvice` 是`@Controller`的增强版，主要用来处理全局数据，一般搭配`@ExceptionHandler`、`@ModelAttribute`以及`@Initbinder`使用

### 全局异常处理

```java
//返回ModelAndView
@ControllerAdvice
public class CustomExceptionHandler {
    @ExceptionHandler(Exception.class)
    public void exampleException(Exception e) throws Exception {
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg","xxx");
        mv.setViewName("error");
        return mv;
    }
}
```

在`resources/templates`目录下创建`error.html`:

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <div th:text="${msg}"></div>
    </body>
</html>
```

在定义的`CustomExceptionHandler`类上添加`@ControllerAdvice`注解。`exampleException`方法的参数可以有异常实例、HttpServletResponse以及HttpServletRequest、Model等，返回值可以是一段JSON、一个ModelAndView、一个逻辑视图名等。

### 添加全局数据

```java
@ControllerAdvice
public class GlobalConfig {
    @ModelAttribute(value="info")
    public Map<String,String> userInfo(){
     	HashMap<String,String> map = new HashMap<>();
   		map.put("username","w");
    	map.put("age",12);
   		return map;   
    }
}
```

- 在全局配置中添加`userInfo`方法，返回一个map。加入注解`@ModelAttribute`,其中的value表示这条返回数据的key，方法的返回值是返回数据的value。
- 此时在任意请求的Controller中，通过方法参数中的Model可以获取info的数据

```java
@RestController
public class MyController {
    @GetMapping("/hello")
    public void hello(Model model) {
        Map<String,String> map = model.asMap();
        Set<String> keySet = map.keySet();
        Iterator<String> iterator = keySet.iterator();
        while(iterator.hasNext()){
            String key = iterator.next();
            Object value = map.get(key);
            System.out.println(key+"---"+value);
        }
    }
}
```

### 请求参数预处理

`@ControllerAdvice`结合`@InitBinder`能实现参数预处理，即表单数据绑定到实体类上时进行一些额外处理

```java
public class Book {
    private String name;
    private String author;
}

public class Author {
    private String name;
    private int age;
}
```

此时Controller中同时接收两个实体类的数据，对于name属性会混淆，需要进行处理：

```java
@RestController
public class MyController {
    @GetMapping("/book")
    public String book(@ModelAttribute("b") Book book,@ModelAttribute("a") Author author) {
      	return book.toString()+"---"+author.toString();
    }
}
```

```java
@ControllerAdvice
public class GlobalConfig {
    @InitBinder("b")
    public void init1(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("b.");
    }
    @InitBinder("a")
    public void init2(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("a.");
    }
}
```

- `GlobalConfig`中，`@InitBinder("b")`表示该方法处理`@ModelAttribute("b")`对应的参数
- `@InitBinder("a")`表示该方法处理`@ModelAttribute("a")`对应的参数
- 每个方法中给相应的Field设置一个前缀，并且在浏览器请求中设置`a.name   b.name`即可区分
- `WebDataBinder`中还可以设置允许、禁止、必填的字段以及验证器等

## 自定义错误页(...)

Spring Boot中的错误默认由`BasicErrorController`类处理

## CORS

### 全局配置

```java
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedHeaders("*")
            .allowedMethods("*")
            .maxAge(1800)
            .allowedOrigins("http://localhost:8081");
    }
}
```

全局配置需要自定义类并实现`WebMvcConfigurer`接口，实现`addCorsMappings`方法

## 注册拦截器

```java
public class MyInterceptor1 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,HttpServletResponse response,Object handler) {
        System.out.println("MyInterceptor1  preHandle...");
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest request,HttpServletResponse response,Object handler,ModelAndView modelAndView) {
        System.out.println("MyInterceptor1  postHandle...");
    }
    @Override
    public void afterCompletion(HttpServletRequest request,HttpServletResponse response,Object handler,Exception ex) {
        System.out.println("MyInterceptor1  afterCompletion...");
    }
}
```

- 拦截器中的方法按`preHandle`->`Controller`->`postHandle`->`afterCompletion`的顺序执行

- 只有当`preHandle`方法返回`true`时后面的方法才会执行
- 当拦截器链中存在多个拦截器时，`postHandle`在拦截器链内所有拦截器返回成功时才会调用
- `afterCompletion`只有`preHandle`返回`true`才调用，**但**若拦截器内的第一个拦截器的`preHandle`返回`false`，后面的方法都不会执行

```java
//配置拦截器。定义配置类进行拦截器的配置
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor1())
            .addPathPatterns("/**")
            .excludePathPatterns("/xxx");
    }
}
//addPathPatterns表示拦截路径
//excludePathPatterns表示排除的路径
```

## 启动系统任务

Spring Boot对于像配置文件加载、数据库舒适化等操作提供了两种解决方案：

### 1. CommandLineRunner

Spring Boot项目在启动时会遍历所有`CommandLineRunner`的实现类并调用其中的`run`方法,当系统中有多个`CommandLineRunner`的实现类，使用`@Order`注解对这些实现类的调用顺序进行排序

```java
//添加两个CommandLineRunner
//Order()中的数字越小越先执行
//run方法的参数是系统启动时传入的参数，即入口类main方法的参数
//在调用SpringApplication.run方法时被传入
@Component
@Order(1)
public class MyCommandLineRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("Runner1"+Arrays.toString(args));
    }
}
@Component
@Order(2)
public class MyCommandLineRunner2 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("Runner2"+Arrays.toString(args));
    }
}
```

### 2. ApplicationRunner

`ApplicationRunner`的用法和`CommandLineRunner`基本一致，区别体现在`run`方法的参数上

```java
//添加两个ApplicationRunner
//Order()中的数字越小越先执行
//run方法的参数是ApplicationArguments对象
//getNonOptionArgs()获取参数
//getOptionNames()获取参数的key
//getOptionValues(key)获取key对应的value
@Component
@Order(2)
public class MyApplicationRunner1 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println("1-nonOptionArgs"+nonOptionArgs);
        Set<String> optionNames = args.getOptionNames();
        for(String optionName : optionNames) {
            System.out.println("1-		   key:"+optionName+";value:"+args.getOptionValues(optionName));
        }
    }
}
@Component
@Order(1)
public class MyApplicationRunner2 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        List<String> nonOptionArgs = args.getNonOptionArgs();
        System.out.println("2-nonOptionArgs"+nonOptionArgs);
        Set<String> optionNames = args.getOptionNames();
        for(String optionName : optionNames) {
            System.out.println("2-		   key:"+optionName+";value:"+args.getOptionValues(optionName));
        }
    }
}
```

## 路径映射

对于一些不需要在控制器中加载数据的页面，可以直接配置路径映射，提高访问速度

```java
//直接在MVC配置中重写addViewControllers方法
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login").setViewName("login");
        registry.addViewController("/index").setViewName("index");
    }
}
```

## 配置AOP

### AOP：在系统运行时动态添加代码的方式

- `Joinpoint`(连接点)：类里面可以被增强的方法为连接点。例如，想修改哪个方法的功能，那么该方法就是一个连接点
- `Pointcut`(切入点)：对`Joinpoint`进行拦截的定义为切入点。例如，拦截所有以`insert`开始的方法，这个定义即为切入点
- `Advice`(通知)：拦截到`Joinpoint`之后要做的事就是通知。例如，打印日志监控。通知分为前置通知、后置通知、异常通知、最终通知和环绕通知
- `Aspect`(切面)：`Pointcut`和`Advice`的结合
- `Target`(目标对象)：要增强的类称为`Target`

首先引入`spring-boot-starter-aop`依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

然后创建`UserService`类

```java
//com.wan.project.service
@Service
public class UserService {
    public String getUserById(Integer id) {
        System.out.println("get...");
        return "user";
    }
    public void deleteUserById(Integer id) {
        System.out.println("delete...");
    }
}
```

接下来创建切面

```java
@Component
@Aspect
public class LogAspect {
    @Pointcut("execution(* com.wan.project.service.*.*(...))")
    public void pc1() {
    }
    @Before(value="pc1()")
    public void before(JoinPoint jp) {
        String name = jp.getSignature().getName();
        System.out.println(name+"方法开始执行...");
    }
    @After(value="pc1()")
    public void after(JoinPoint jp) {
        String name = jp.getSignature().getName();
        System.out.println(name+"方法执行结束...");
    }
    @AfterReturning(value="pc1()",returning="result")
    public void afterReturning(JoinPoint jp,Object result) {
        String name = jp.getSignature().getName();
        System.out.println(name+"方法返回值为："+result);
    }
    @AfterThrowing(value="pc1()",throwing="e")
    public void afterThrowing(JoinPoint jp,Exception e) {
        String name = jp.getSignature().getName();
        System.out.println(name+"方法抛异常了，异常为："+e.getMessage());
    }
    @Around(value="pc1()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        return pjp.proceed();
    }
}
```

- `@Aspect`表明这是一个切面类
- `pc1()`方法使用`@Pointcut`注解，这是一个切入点，为`service`包下所有类的所有方法
  - `execution`中的第一个`*`表示方法返回任意值
  - `execution`中的第二个`*`表示`service`包下的任意类
  - `execution`中的第三个`*`表示类中的任意方法，括号中的`..`表示方法参数任意

- `@Before`表示这是前置通知，该方法在目标方法执行之前执行。通过`JoinPoint`参数可以获取目标方法的方法名、修饰符等信息
- `@After`表明这是后置通知，该方法在目标方法执行之后执行
- `@AfterReturning`表示这是返回通知，在该方法中可以获取目标方法的返回值
- `@AfterThrowing`表明这是异常通知，当目标方法发生异常时会调用
- `@Around`表明这是环绕通知。环绕通知功能最为强大，可以实现前置、后置、异常和返回通知的功能。目标方法进入环绕通知后，通过调用`ProceedingJoinPoint`对象的`proceed`方法使目标方法继续执行

配置完成后，在`Controller`中调用`UserService`的方法，`LogAspect`的代码就会动态嵌入目标方法中执行

```java
@RestController
public class UserController {
    @Autowired
    UserService userService;
    @GetMapping("/getUserById")
    public String getUserById(Integer id) {
      	return userService.getUserById(id);
    }
    @GetMapping("/deleteUserById")
    public String deleteUserById(Integer id) {
      	return userService.deleteUserById(id);
    }
}
```

# Spring Boot整合持久层

## 1. JdbcTemplate

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connecter-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid</artifactId>
    <version>1.1.9</version>
</dependency>
```

在`application.properties`中配置数据库基本连接信息：

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql:///test
spring.datasource.username=root
spring.datasource.password=123456
```

创建实体类`Book`

```java
public class Book {
    private Integer id;
    private String name;
    private String author;
}
```

创建数据库访问层`BookDao`

```java
public class BookDao {
    @Autowired
    JdbcTemplate jdbcTemplate;
    public int addBook(Book book) {
        return jdbcTemplate.update("INSERT INTO book(name,author) VALUES (?,?)",book.getName(),book.getAuthor());
    }
    public int updateBook(Book book) {
        return jdbcTemplate.update("UPDATE book SET name=?,author=? WHERE id=?",book.getName(),book.getAuthor(),book.getId());
    }
    public int deleteBook(Book book) {
        return jdbcTemplate.update("DELETE FROM book WHERE id=? ",book.getId());
    }
    public Book getBookById(Integer id) {
        return jdbcTemplate.queryForObject("SELECT * FROM book WHERE id=?",id);
    }
    public List<Book> getAllBooks() {
        return jdbcTemplate.query("SELECT * FROM book",new BeanPropertyRowMapper<>(Book.class));
    }
}
```

- 增删改的操作主要使用`update`和`batchUpdate`方法完成
- 查主要通过`query`和`queryForObject`方法完成
- 除此之外还有`execute`方法用来执行任意的SQL、`call`方法用来调用存储过程等
- 执行查询操作时，需要一个`RowMapper`将查询的列和实体类中的属性一一对应
  - 若列名与属性名相同，可以直接使用`BeanPropertyRowMapper`
  - 若不相同，则需要实现`RowMapper`接口，将其一一对应

创建`BookService`和`BookController`

```java
@Service
public class BookService {
    @Autowired
    BookDao bookDao;
    public int addBook(Book book) {
        return bookDao.addBook(book);
    }
    public int updateBook(Book book) {
        return bookDao.updateBook(book);
    }
    public int deleteBook(Integer id) {
        return bookDao.deleteBook(id);
    }
    public Book getBookById(Integer id) {
        return bookDao.getBookById(id);
    }
    public List<Book> getAllBooks() {
        return bookDao.getAllBooks();
    }
}
```

```java
@RestController
public class BookController {
    @Autowired
    BookService bookService;
    @GetMapping("/bookOps")
    public void bookOps() {
        Book b1 = new Book();
        b1.setName("aaa");
        b1.setAuthor("bbb");
        int id = bookService.addBook(b1);
        System.out.println("addBook:"+id);
        
        Book b2 = new Book();
        b2.setName("aaa1");
        b2.setAuthor("bbb1");
        b2.setId(1);
        int updateBook = bookService.updateBook(b2);
        System.out.println("updateBook:"+updateBook);
        
        Book b3 = bookService.getBookById(1);
        System.out.println("getBookById:"+b3);
        
        int deleteBook = bookService.deleteBook(2);
        System.out.println("deleteBook:"+deleteBook);
        
        List<Book> allBooks = bookService.getAllBooks();
        System.out.println("getAllBooks:"+allBooks);
    }
}
```

## 2. MyBatis

添加依赖

```xml
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connecter-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid</artifactId>
    <version>1.1.9</version>
</dependency>
```

在`application.properties`中配置数据库基本连接信息：

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql:///test
spring.datasource.username=root
spring.datasource.password=123456
```

创建实体类`Book`

```java
//org.wan.project.model.Book
public class Book {
    private Integer id;
    private String name;
    private String author;
}
```

创建`BookMapper`

```java
//com.wan.project.mapper
@Mapper
public interface BookMapper {
    int addBook(Book book);
    int deleteBookById(Integer id);
    int updateBookById(Integer id);
    Book getBookById();
    List<Book> getAllBooks();
}
```

指明该类是一个`Mapper`的方法：

- 在`BookMapper`上添加`@Mapper`注解，需要在每一个`Mapper`上都添加注解
- 在配置类上添加`@MapperScan("com.wan.project.mapper")`注解，表明扫描`com.wan.project.mapper`包下的所有接口作为`Mapper`

创建`BookMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.wan.project.mapper.BookMapper">
	<insert id="addBook" parameterType="org.wan.project.model.Book">
    	INSERT INTO book(name,author) VALUES (#{name},#{author});
    </insert>
    <delete id="deleteBookById" parameterType="int">
    	DELETE FROM book WHERE id=#{id};
    </delete>
    <update id="updateBook" parameterType="org.wan.project.model.Book">
    	UPDATE book SET name=#{name},author=#{author} WHERE id=#{id};
    </update>
    <select id="getBookById" parameterType="int",resultType="org.wan.project.model.Book">
    	SELECT * FROM book WHERE id=#{id};
    </select>
    <select id="getAllBooks" resultType="org.wan.project.model.Book">
    	SELECT * FROM book;
    </select>
</mapper>
```

创建`BookService`和`BookController`

```java
@Service
public class BookService {
    @Autowired
    BookMapper bookMapper;
    public int addBook(Book book) {
        return bookMapper.addBook(book);
    }
    public int updateBook(Book book) {
        return bookMapper.updateBook(book);
    }
    public int deleteBook(Integer id) {
        return bookMapper.deleteBook(id);
    }
    public Book getBookById(Integer id) {
        return bookMapper.getBookById(id);
    }
    public List<Book> getAllBooks() {
        return bookMapper.getAllBooks();
    }
}
```

```java
@RestController
public class BookController {
    @Autowired
    BookService bookService;
    @GetMapping("/bookOps")
    public void bookOps() {
        Book b1 = new Book();
        b1.setName("aaa");
        b1.setAuthor("bbb");
        int id = bookService.addBook(b1);
        System.out.println("addBook:"+id);
        
        Book b2 = new Book();
        b2.setName("aaa1");
        b2.setAuthor("bbb1");
        b2.setId(1);
        int updateBook = bookService.updateBook(b2);
        System.out.println("updateBook:"+updateBook);
        
        Book b3 = bookService.getBookById(1);
        System.out.println("getBookById:"+b3);
        
        int deleteBook = bookService.deleteBook(2);
        System.out.println("deleteBook:"+deleteBook);
        
        List<Book> allBooks = bookService.getAllBooks();
        System.out.println("getAllBooks:"+allBooks);
    }
}
```

配置`pom.xml`

在`Maven`工程中，`XML`配置文件建议写在`resources`目录下，但本项目的`Mapper.xml`写在包下，`Maven`在运行时会忽略包下的`XML`文件，因此需要在`pom.xml`中重新指明资源文件位置：

```xml
<build>
	<resources>
    	<resource>
        	<directory>src/main/java</directory>
            <includes>
            	<include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
        	<directory>src/main/resources</directory>
        </resource>
    </resources>
</build>
```

## 3. Spring Data JPA

创建数据库，不用创建表

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jpa</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connecter-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid</artifactId>
    <version>1.1.9</version>
</dependency>
```

在`application.properties`中配置数据库基本信息和JPA相关配置：

```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql:///jpa
spring.datasource.username=root
spring.datasource.password=123456
spring.jpa.show-sql=true
spring.jpa.database=mysql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```

创建实体类`Book`

```java
//org.wan.project.model.Book
@Entity(name = "t_book")
public class Book {
    @Id
    @GenerateValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    @Column(name = "book_name",nullable = false)
    private String name;
    private String author;
    @Transient
    private String description;
}
```

- `@Entity`:表示该类是一个实体类，项目启动时会根据该类自动生成一张表，表的名称为`@Entity`中`name`的值，如果不配置`name`，默认表名为类名
- `@Id`:表示该属性是一个主键，`@GenerateValue`表示主键自动生成，`strategy`表示主键的生成策略
- `@Column`定制生成字段的属性，`name`表示该属性对应的数据表中字段的名称，`nullable`表示该字段非空，默认生成表中字段名称就是实体类中属性的名称
- `@Transient`:表示在数据库中，该属性被忽略，不生成对应的字段

创建`BookDao`接口，继承`JpaRepository`

```java
public interface BookDao extends JpaRepository<Book,Integer> {
    List<Book> getBooksByAuthorStartingWith(String author);
    
    @Query(value = "select * from t_book where id=(select max(id) from t_book)",nativeQuery = true)
    Book getMaxIdBook();
    
    @Query("select b from t_book b where b.id>:id and b.author=:author")
    List<Book> getBookByIdAndAuthor(@Param("author") String author,@Param("id") Integer id);
    
    @Query("select b from t_book b where b.id>?2 and b.name like %?1%")
    List<Book> getBookByIdAndAuthor(String name,Integer id);//注意参数的顺序
}
```

如果`BookDao`中的方法涉及修改操作，需要添加`@Modifying`注解添加事务

Jpa的既定规范

| KeyWords            | 方法名举例                     | 对应的SQL                    |
| ------------------- | ------------------------------ | ---------------------------- |
| And                 | findByNameAndAge               | where name=? and age=?       |
| Or                  | findByNameOrAge                | where name=? or age=?        |
| Is                  | findByAgeIs                    | where age=?                  |
| Equals              | findIdEquals                   | where id=?                   |
| Between             | findByAgeBetween               | where age between ? and ?    |
| LessThan            | findByAgeLessThan              | where age<?                  |
| LessThanEquals      | findByAgeLessThanEquals        | where age<=?                 |
| GreaterThan         | findByAgeGreaterThan           | where age>?                  |
| GreaterThanEquals   | findByAgeGreaterThanEquals     | where age>=?                 |
| After               | findByAgeAfter                 | where age>?                  |
| Before              | findByAgeBefore                | where age<?                  |
| IsNull              | findByNameIsNull               | where name is null           |
| isNotNull,NotNull   | findByNameNotNull              | where name is not null       |
| Not                 | findByGenderNot                | where gender <>?             |
| In                  | findByAgeIn                    | where age in(?)              |
| NotIn               | findByAgeNotIn                 | where age not in(?)          |
| Like                | findByNameLike                 | where name like?             |
| NotLike             | findByNameNotLike              | where name not like?         |
| StartingWith        | findByNameStartingWith         | where name like '?%'         |
| EndingWith          | findByNameEndingWith           | where name like '%?'         |
| Containing,Contains | findByNameContaining           | where name like '%?%'        |
| OrderBy             | findByGreaterThanOrderByIdDesc | where age>? order by id desc |
| True                | findByEnabledTrue              | where enabled=true           |
| False               | findByEnabledFalse             | where enabled=false          |
| IgnoreCase          | findByNameIgnoreCase           | where UPPER(name) = UPPER(?) |

创建`BookService`

```java
@Service
public class BookService {
    @Autowired
    BookDao bookDao;
    //save方法由JpaRepository接口提供
    public void addBook(Book book) {
        bookDao.save(book);
    }
    
    //分页查询，返回值Page<Book>，该对象包含总记录数、总页数、每页记录数、当前页记录数等
    public Page<Book> getBookByPage(Pageable pageable) {
        return bookDao.findAll(pageable);
    }
    
    public List<Book> getBookByAuthorStartingWith(String author) {
        return bookDao.getBookByAuthorStartingWith(author);
    }
    
    public Book getMaxIdBook() {
        return bookDao.getMaxIdBook();
    }
    
    public List<Book> getBookByIdAndAuthor(String author,Integer id) {
        return bookDao.getBookByIdAndAuthor(author,id);
    }
    
    public List<Book> getBookByIdAndName(String name,Integer id) {
        return bookDao.getBookByIdAndName(name,id);
    }
}
```

创建`BookController`

```java
@RestController
public class BookController {
    @Autowired
    BookService bookService;
    
    @GetMapping("/findAll")
    public void findAll() {
        PageRequest pageable = PageRequest.of(2,3);
        Page<Book> page = bookService.getBookByPage(pageable);
        System.out.println("总页数："+page.getTotalPages());
        System.out.println("总记录数："+page.getTotalElements());
        System.out.println("查询结果："+page.getContent());
        System.out.println("当前页数："+page.getNumber()+1);
        System.out.println("当前页记录数："+page.getNumberOfElements());
        System.out.println("每页记录数："+page.getSize());
    }
    
    @GetMapping("/search")
    public void search() {
        List<Book> bs1 = bookService.getBookByIdAndAuthor("a",7);
        List<Book> bs2 = bookService.getBookByAuthorStartingWith("b");
        List<Book> bs3 = bookService.getBookByIdAndName("ab",6);
        Book book = bookService.getMaxIdBook();
        System.out.println("bs1："+bs1);
        System.out.println("bs2："+bs2);
        System.out.println("bs3："+bs3);
        System.out.println("book："+book);
    }
    
    @GetMapping("/save")
    public void save() {
        Book book = new Book();
        book.setAuthor("xxx");
        book.setName("aaa");
        bookSerevice.addBook(book);
    }
}
```

## 4. 多数据源

### 1. JdbcTemplate多数据源

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connecter-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

在`application.properties`中配置数据库连接信息

```properties
#数据源1
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.one.url=jdbc:mysql:///test-1
spring.datasource.one.username=root
spring.datasource.one.password=123456

#数据源2
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.two.url=jdbc:mysql:///test-2
spring.datasource.two.username=root
spring.datasource.two.password=123456
```

创建`DataSourceConfig`配置数据源，根据`application.properties`配置生成两个数据源

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties("spring.datasource.one")
    DataSource dsOne() {
        return DuridDataSourceBuilder.create().builder();
    }
    @Bean
    @ConfigurationProperties("spring.datasource.two")
    DataSource dsTwo() {
        return DuridDataSourceBuilder.create().builder();
    }
}
```

- `DataSourceConfig`提供两个数据源：`dsOne`和`dsTwo`，默认方法名即实例名
- `@ConfigurationProperties`注解表示使用不同前缀的配置文件来创建不同的`DataSource`实例

配置`JdbcTemplate`

```java
@Configuration
public class JdbcTemplateConfig {
    @Bean
    JdbcTemplate jdbcTemplateOne(@Qualifier("dsOne") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    @Bean
    JdbcTemplate jdbcTemplateTwo(@Qualifier("dsTwo") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

- `JdbcTemplateConfig`提供两个`JdbcTemplate`实例，每个实例需要提供`DataSource`，并且通过`@Qualifier`注解查找不同名称的`DataSource`实例注入

创建实体类`Book`

```java
public class Book {
    private Integer id;
    private String name;
    private String author;
}
```

创建`BookController`

```java
@RestController
public class BookController {
    @Resource(name = "jdbaTemplateOne")
    JdbcTemplate jdbaTemplateOne;
    
    @Autowired
    @Quailfier("jdbcTemplateTwo")
    JdbcTemplate jdbaTemplateTwo;
    //...
}
```

注入`JdbcTemplate`的两种方式

- `@Resource`，指明`name`属性
- `@Autowired`结合`@Qualifier`(效果等同与`@Resource`) 

### 2. MyBatis多数据源

依赖配置，`DataSourceConfig`，`application.properties`，实体类`Book`同上

配置`MyBatis`

```java
@Configuration
@MapperScan(value = "com.wan.project.mapper1",sqlSessionFactoryRef = "sqlSessionFactoryBean1")
public class MyBatisConfigOne {
    @Autowired
    @Qualifier("dsOne")
    DataSource dsOne;
    @Bean
    SqlSessionFactory sqlSessionFactoryBea1() throws Exception {
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setDataSource(dsOne);
        return ssfb.getObject();
    }
    @Bean
    SqlSessionTemplate sqlSessionTemplate1() throws Exception {
        return SqlSessionTemplate(sqlSessionFactoryBea1());
    }
}
```

```java
@Configuration
@MapperScan(value = "com.wan.project.mapper2",sqlSessionFactoryRef = "sqlSessionFactoryBean2")
public class MyBatisConfigOne {
    @Autowired
    @Qualifier("dsTwo")
    DataSource dsTwo;
    @Bean
    SqlSessionFactory sqlSessionFactoryBea2() throws Exception {
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        ssfb.setDataSource(dsTwo);
        return ssfb.getObject();
    }
    @Bean
    SqlSessionTemplate sqlSessionTemplate2() throws Exception {
        return SqlSessionTemplate(sqlSessionFactoryBea2());
    }
}
```

- `@MapperScan`注解指定`Mapper`接口所在位置，同时指定`SqlSessionFactory`的实例名，该位置下的`Mapper`使用`SqlSessionFactory`实例
- 创建`SqlSessionFactory`实例，并将`DataSource`实例设置给`SqlSessionFactory`，该`SqlSessionFactory`实例就是`@MapperScan`中`sqlSessionFactoryRef`参数指定的实例
- 提供`SqlSessionTemplate`实例，这是一个线程安全类，用于管理`MyBatis`中的`SqlSession`操作

创建`Mapper`及其相应的`Mapper`映射文件

```java
//com.wan.project.mapper1
public interface BookMapper1 {
    List<Book> getAllBooks();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.wan.project.mapper1.BookMapper1">
    <select id="getAllBooks" resultType="org.wan.project.model.Book">
    	SELECT * FROM book;
    </select>
</mapper>
```

```java
//com.wan.project.mapper2
public interface BookMapper2 {
    List<Book> getAllBooks();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.wan.project.mapper2.BookMapper2">
    <select id="getAllBooks" resultType="org.wan.project.model.Book">
    	SELECT * FROM book;
    </select>
</mapper>
```

创建`Controller`

```java
@RestController
public class BookController {
    @Autowired
    BookMapper1 bookMapper1;
    @Autowired
    BookMapper2 bookMapper2;
    @GetMapping("/test")
    public void test() {
        List<Book> books1 = bookMapper1.getAllBooks();
        List<Book> books2 = bookMapper2.getAllBooks();
        //...
    }
}
```

### 3. JPA多数据源

添加依赖并修改`application.properties`

```properties
#数据源1
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.one.url=jdbc:mysql:///test-1
spring.datasource.one.username=root
spring.datasource.one.password=123456

#数据源2
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.two.url=jdbc:mysql:///test-2
spring.datasource.two.username=root
spring.datasource.two.password=123456

spring.jpa.properties.show-sql=true
spring.jpa.properties.database=mysql
spring.jpa.properties.hibernate.hbm2ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```

`DataSourceConfig`同上

创建实体类`User`

```java
//com.wan.project.model.User
@Entity(name = "t_user")
public class User {
    @Id
    @GenerateValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private Integer age;
}
```

创建`JPA`配置

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = "com.wan.project.dao1",
                      entityManagerFactoryRef = "entityManagerFactoryBeanOne",
                      transactionManagerRef = "platformTranscationManagerOne")
public class JpaConfigOne {
    @Resource(name = "dsOne")
    DataSource dsOne;
    @Autowired
    JpaProperties jpaProperties;
    
    @Bean
    @Primary
    LocalContainerEntityManagerFactoryBean entityManagerFactoryBeanOne(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dsOne)
            .properties(jpaProperties.getProperties())
            .packages("com.wan.project.model")
            .persistenceUnit("pu1")
            .builder();
    }
    
    @Bean
    PlatformTransactionManager platformTransactionManagerOne(EntityManagerFactoryBuilder builder) {
        LocalContainerEntityManagerFactoryBean factoryOne = entityManagerFactoryBeanOne(builder);
        return new JpaTransactionManager(factoryOne.getObject());
    }
}
```

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = "com.wan.project.dao2",
                      entityManagerFactoryRef = "entityManagerFactoryBeanTwo",
                      transactionManagerRef = "platformTranscationManagerTwo")
public class JpaConfigOne {
    @Resource(name = "dsTwo")
    DataSource dsTwo;
    @Autowired
    JpaProperties jpaProperties;
    
    @Bean
    LocalContainerEntityManagerFactoryBean entityManagerFactoryBeanTwo(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dsTwo)
            .properties(jpaProperties.getProperties())
            .packages("com.wan.project.model")
            .persistenceUnit("pu2")
            .builder();
    }
    
    @Bean
    PlatformTransactionManager platformTransactionManagerTwo(EntityManagerFactoryBuilder builder) {
        LocalContainerEntityManagerFactoryBean factoryOne = entityManagerFactoryBeanTwo(builder);
        return new JpaTransactionManager(factoryOne.getObject());
    }
}
```

- `@EnableJpaRepositories`注解进行`JPA`配置，主要配置三个属性
  - `basePackages`:指定`Repository`的位置
  - `entityManagerFactoryRef`:指定实体类管理工厂`Bean`的名称
  - `transcationManagerRef`:指定事务管理器的引用名称，此处的引用名称就是`JpaConfigOne`类中注册的`Bean`的名称(默认`Bean`名称为方法名)

- 创建`LocalContainerEntityManagerFactoryBean`,该`Bean`用于提供`EntityManager`实例，该类的创建过程：
  - 首先配置数据源
  - 设置JPA相关设置(`JpaProperties`由系统自动加载)
  - 设置实体类所在位置
  - 配置持久化单元名(若项目中只有一个`EntityManagerFactory`，`persistenceUint`可以省略，否则必须明确指定)

- 项目中有多个`LocalContainerEntityManagerFactoryBean`实例时，`@Primary`表示该实例优先使用

- `JpaTransactionManage`提供对单个`EntityManagerFactory`的事务支持，专门用于解决`JPA`的事务管理

创建`Repository`

```java
//com.wan.project.dao1
public interface UserDao1 extends JpaRepository<User,Integer> {
    
}
```

```java
//com.wan.project.dao2
public interface UserDao2 extends JpaRepository<User,Integer> {
    
}
```

创建`Controller`

```java
@RestController
public class UserController {
    @Autowired
    UserDao1 userDao1;
    @Autowired
    UserDao2 userDao2;
    
    @GetMapping("/test")
    public void test() {
        User u1 = new User();
        u1.setAge(22);
        u1.setName("wan");
        userDao1.save(u1);
        
        User u2 = new User();
        u2.setAge(23);
        u2.setName("li");
        userDao2.save(u2);
    }
}
```

# Spring Boot整合NoSQL

## 1. Redis

### Redis安装

下载`Redis`:

`wget http://download.redis.io/releases/redis-4.0.10.tar.gz`

解压并编译：

```shell
tar -zxvf redis-4.0.10.tar.gz
cd redis-4.0.10
make MALLOC=libc
make install
```

配置`Redis`：

```conf
# 后台运行
daemonize yes
# 关闭该设置，使外网可以连接
# bind 127.0.0.1
# 关闭保护模式
protected-mode no
```

配置`CentOS`

```shell
# 关闭防火墙
systemctl stop firewalld.service
# 禁止开机启动
systemctl disable firewalld.service 
```

`Redis`启动与关闭

```shell
# 启动(指定redis.conf)
redis-server redis.conf
# 进入
redis-cli
# 关闭
redis-cli shutdown
```

### Redis整合

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

配置`Redis`和连接池

```properties
spring.redis.database=0
spring.redis.password=
spring.redis.port=6379
spring.redis.host=192.168.2.2
spring.redis.lettuce.pool.min-idle=5
spring.redis.lettuce.pool.max-idle=10
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=1ms
spring.redis.lettuce.shutdown-timeout=100ms
```

创建实体类`Book`

```java
public class Book implements Serializable {
    private Integer id;
    private String name;
}
```

创建`Controller`

```java
@RestController
public class BookController {
    @Autowired
    RedisTemplate redisTenplate;
    @Autowired
    StringRedisTemplate stringRedisTemplate;
    @GetMapping("/test")
    public void test() {
        ValueOperations<String,String> ops1 = stringRedisTemplate.opsForValue();
        ops1.set("name","www");
        System.out.println(ops1.get("name"));
        ValueOperations ops2 = redisTemplate.opsForValue();
        Book b1 = new Book();
        book.setId(1);
        book.setName("fff");
        ops2.set("b1",b1);
        Book b2 = (Book)ops.get("b1");
        System.out.println(b2;
    }
}
```

`StringRedisTemplate`是`RedisTemplate`的子类，`StringRedisTemplate`中的`key`和`value`都是`String`，采用的序列化方案是`StringSerializer`；而`RedisTemplate`可以操作对象，采用的序列化方案是`JdkSerializationRedisSerializer`

### Redis集群整合(...)

## 2. MongoDB

### MongoDB安装

下载`MongoDB`

`wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.0.tgz`

解压并重命名

`tar -zxvf mongodb-linux-x86_64-4.0.0.tgz`

`mv mongodb-linux-x86_64-4.0.0 mongodb`

配置`MongoDB`

```shell
cd mongodb
mkdir db
mkdir logs
```

在`bin`目录下创建`mongo.conf`

```conf
dbpath=/xxx/mongodb/db
logpath=/xxx/mogodb/logs
port=27017
# 以守护进程方式运行
fork=true
```

在`bin`目录下启动与关闭

```shell
# 启动
./mongod -f -config mongo.conf --bind ip_all
# 进入
./mongo
# 关闭
./mongod -shutdown -config mongo.conf
```

### MongoDB整合

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

配置`MongoDB`

```properties
spring.data.mongodb.database=test
spring.data.mongodb.host=192.168.2.2
spring.data.mongodb.port=27017
#验证登录信息的库 
#spring.data.mongodb.authentication-database=admin
# spring.data.mongodb.username=xxx
# spring.data.mongodb.password=123
```

创建实体类`Book`

```java
public class Book {
    private Integer id;
    private String name;
}
```

创建`Controller`，使用`MongoTemplate`(还可以使用`MongoRepository`)

```java
@RestController
public class BookController {
    @Autowired
    MongoTemplate mongoTemplate;
    @GetMapping("/test")
    public void test(){
        List<Book> books = new ArrayList<>();
        Book b1 = new Book();
        b1.setId(1);
        b1.setName("www");
        books.add(b1);
        mongoTemplate.insertAll(books);
        List<Book> list = mongoTemplate.findAll(Book.class);
        System.out.println(list);
        Book book = mongoTemplate.findById(1,Book.class);
        System.out.println(book);
    }
}
```

## 3. Session共享

`Session`共享配置

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

```properties
spring.redis.host=192.168.2.2
spring.redis.port=6379
spring.redis.password=
spring.redis.database=0
```

创建`Controller`，(配置完成后 ，就可以使用 `Spring Session` 了，其实就是使用普通的 `HttpSession` ，其他的 `Session` 同步到 `Redis` 等操作，框架已经自动帮你完成了)

```java
@RestController
public class HelloController {
    @Value("${server.port}")
    Integer port;
    @GetMapping("/set")
    public String set(HttpSession session) {
        session.setAttribute("user", "wan");
        return String.valueOf(port);
    }
    @GetMapping("/get")
    public String get(HttpSession session) {
        return session.getAttribute("user") + ":" + port;
    }
}
```

运行项目

```shell
# nohup表示不挂断程序运行，&表示后台运行
nohup java -jar xxx.jar --server.port=8080 &
nohup java -jar xxx.jar --server.port=8081 &
```

`Nginx`负载均衡

下载`Nginx`并解压

`wget https://nginx.org/download/nginx-1.14.0.tar.gz`

`tar -zxvf nginx-1.14.0.tar.gz`

进入解压目录执行编译安装 

```shell
cd nginx-1.14.0
./configure
make
make install
```

完成后，在`Nginx`安装目录下启动`Nginx`(默认端口80)

`./sbin/nginx`

然后修改`nginx.conf`

```conf
upstream wan.org{
	server 127.0.0.1:8080 weight=1;
	server 127.0.0.1:8081 weight=2;
}
server {
	listen       80;
	server_name  localhost;
	#...
	location / {
		proxy_pass     http://wan.org;
		proxy_redirect default;
	}
}
```

- `upstream`表示配置上游服务器
- `wan.org`表示服务器集群的名字，随意取
- `server`表示单独的服务
- `weight`表示服务权重，即有多少比例的请求从`Nginx`转发到该服务器
- `location`中`proxy_pass`表示请求转发的地址，`/`表示拦截所有请求，转发到配置的服务器集群中

重启`Nginx`

`./sbin/nginx -s reload`

# Spring Boot构建RESTful服务(...)

## 1. JPA实现REST

## 2. MongoDB实现REST

# 开发者工具与单元测试

## 1. devtools(...)

## 2. 单元测试

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

创建测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01ApplicationTests {
    @Test
    public void contextLoads() {
        
    }
}
```

- `@RunWith`，将`JUint`执行类修改为`SpringRunner`，`SpringRunner`是`SpringFramework`中测试类`SpringJUint4ClassRunner`的别名
- `@SpringBootTest`提供`SprinTestContext`中的常规测试功能外，还提供其他特性：
  - 默认的`ContextLoader`
  - 自动搜索`@SpringBootConfiguration`
  - 自定义环境属性
  - 支持不同的`webEnvironment`，主要有四种：
    - `MOCK`
    - `RANDOM_PORT`
    - `DEFINED_PORT`
    - `NONE`(一般不适于`Web`测试)
- `@Test`来自于`JUint`,`JUint`的其他注解也可以使用

`Service`测试

```java
@Service
public class HelloService {
    public String sayHello(String name) {
        return "Hello " + name + " !";
    }
}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01ApplicationTests {
    @Autowired
    HelloService helloService;
    @Test
    public void contextLoads() {
        String hello = helloService.sayHello("W");
        Assert.assertThat(hello,Matchers.is("Hello W !"));
    }
}
```

`Controller`测试

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(String name) {
        return "Hello "+name+" !";
    }
    @PostMapping("/book")
    public String addBook(@RequestBody Book book) {
        return book.toString();
    }
}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01ApplicationTests {
    @Autowired
    HelloController helloController;
    @Autowired
    WebApplicaionContext wac;
    @Autowired
    MockMvc mockMvc;
    @Before
    public void before() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }
    @Test
    public void test1() throws Exception {
        MvcResult mvcResult = mockMvc
            .perform( MockMvcRequestBuilders
            	.get("/hello")
       	    	.contentType(Media.APPLICATION_FORM_URLENCODED)
            	.param("name","W"))
            .addExpect(MockMvcResultMatchers.status().isOk())
            .addDo(MockMvcResultHandlers.print())
            .addReturn();
        System.out.println(mvcResult.getResponse().getContentAsString());
    }
    @Test
    public void test2() throws Exception {
        ObjectMapper om = new ObjectMapper();
        Book book = new Book();
        book.setId(1);
        book.setName("W");
        String s = om.writeValueAsString(book);
        MvcResult mvcResult = mockMvc
            .perform( MockMvcRequestBuilders
            	.post("/book")
       	    	.contentType(Media.APPLICATION_JSON)
            	.content(s))
            .addExpect(MockMvcResultMatchers.status().isOk())
            .addReturn();
        System.out.println(mvcResult.getResponse().getContentAsString());
    }
}
```

- 注入`WebApplicationContext`模拟`ServletContext`环境
- 声明`MockMvc`对象，并且在测试方法执行之前进行`MockMvc`的初始化
- `perform`方法开启一个`RequestBuilder`请求，具体请求通过`MockMvcRequestBuilders`构建

`JSON`测试(...)

# Spring Boot缓存

## 1. Ehcache 2.x缓存(...)

## 2. Redis单机缓存

添加依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

添加配置

```properties
spring.redis.database=0
spring.redis.password=
spring.redis.port=6379
spring.redis.host=192.168.2.2

spring.cache.cache-names=c1
```

开启缓存

```java
@SpringBootApplication
@EnableCaching//开启缓存
public class Application {
    public static void main(String[] args) {
 SpringApplication.run(Application.class, args);
    }
}
```

核心注解

- @CacheConfig:这个注解在类上使用，用来描述该类中所有方法使用的缓存名称

```java
@Service
@CacheConfig(cacheNames = "c1")
public class UserService {
    
}
```

- @Cacheable:一般加在查询方法上，表示将一个方法的返回值缓存起来，默认情况下，缓存的key就是方法的参数，缓存的value就是方法的返回值

```java
@Cacheable(key = "#id")
public User getUserById (Integer id,String username) {
    System.out.println("getUserById");
    return getUserFromDBById(id);
}
```

- @CachePut:一般加在更新方法上，当数据库中的数据更新后，缓存中的数据也要跟着更新，使用该注解，可以将方法的返回值自动更新到已经存在的key上

```java
@CachePut(key = "#user.id")
public User updateUserById(User user) {
    return user;
}
```

- @CacheEvict:一般加在删除方法上，当数据库中的数据删除后，相关的缓存数据也要自动清除，该注解在使用的时候也可以配置按照某种条件删除（condition属性）或者或者配置清除所有缓存（allEntries属性）

```java
@CacheEvict()
public void deleteUserById(Integer id) {
    //在这里执行删除操作， 删除是去数据库中删除
}
```

## 3. Redis集群缓存(...)

# Spring Boot安全管理

## 1. Spring Security

## 2. 基于数据库的认证

## 3. 高级配置

## 4. OAuth 2

## 5. Shiro

# Spring Boot整合WebSocket

# 消息服务

## 1. JMS

## 2. AMQP

# 企业开发

## 1. 邮件发送

## 2. 定时任务

## 3. 批处理

## 4. Swagger2

## 5. 数据校验

# 应用监控

## 1. 监控端点配置

## 2. 监控信息可视化

## 3. 邮件报警

# 项目构建与部署

## 1. 构建JAR

## 2. 构建WAR
