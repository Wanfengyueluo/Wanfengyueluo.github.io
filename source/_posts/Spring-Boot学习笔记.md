---
title: Spring-Boot学习笔记
date: 2020-11-09 09:20:03
description: Spring Boot学习笔记--江南一点雨Spring Boot视频记录笔记
categories: Spring Boot
tags:
	- Spring Boot
---

# Spring Boot学习笔记

## Maven创建Spring Boot的方法

在某些情况下，start.spring.io无法访问时，可以直接使用Maven创建Spring Boot项目

1. 新建Maven项目

2. 在pom.xml文件中添加`parent`节点

   ```xml
   <parent>    
       <groupId>org.springframework.boot</groupId>    
       <artifactId>spring-boot-starter-parent</artifactId>    	        
       <version>2.3.0.RELEASE</version>
   </parent>
   ```

3. 添加`spring-boot-starter-web`和`spring-boot-starter-test`依赖

4. 添加`spring-boot-maven-plugin`插件

5. 创建启动类

   ```java
   @SpringBootApplication
   public class Application {
       public static void main(String[] args) {
           SpringApplication.run(Application.class, args);
       }
   }
   ```

## HttpMessageConverter

在Spring MVC中自动配置了`Jackson`和`Gson`的`HttpMessageConverter`

1. 将服务端返回的对象序列化成`JSON`字符串
2. 将前端传递的`JSON`字符串反序列化成`Java`对象

自定义`HttpMessageConverter`的方式

- Jackson

1. 提供`MappingJackson2HttpMessageConverter`的`bean`

   ```java
   @Bean
   MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {   
       MappingJackson2HttpMessageConverter converter = new 	
   MappingJackson2HttpMessageConverter();      
       ObjectMapper om = new ObjectMapper();   
       om.setDateFormat(new SimpleDateFormat("yyyy/MM/dd hh:mm:SSS"));  
       converter.setObjectMapper(om); 
       return converter;
   }
   //起主要作用的是ObjectMapper
   ```

2. 提供`ObjectMapper`

   ```java
   @Bean
   ObjectMapper objectMapper() {
   	ObjectMapper om = new ObjectMapper();
   	om.setDateFormat(new SimpleDateFormat("yyyy/MM/dd hh/mm/SSS"));
   	return om;
   }
   ```

- Gson
1. 修改`spring-boot-starter-web`中默认的`Jackson`依赖，添加`Gson`依赖

   ```xml
   <dependency>
   	<groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
       <exclusions>
   		<exclusion>
   			<groupId>org.springframework.boot</groupId>
   			<artifactId>spring-boot-starter-json</artifactId>
   		</exclusion>
   	</exclusions>
   </dependency>
   <dependency>
   	<groupId>com.google.code.gson</groupId>
   	<artifactId>gson</artifactId>
   </dependency>
   ```

2. 提供`GsonHttpMessageConverter`的`bean`

   ```java
   @Bean
   GsonHttpMessageConverter gsonHttpMessageConverter(){
   	GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
   	converter.setGson(new 
       GsonBuilder().setDateFormat("yyyy/MM/dd").create());
   	return converter;
   }
   //起主要作用的是Gson
   ```

3. 提供`Gson`

   ```java
   @Bean
   Gson gson(){
   	return new GsonBuilder().setDateFormat("yyyy/MM/dd").create();
   }
   ```


- FastJson

1. 修改`spring-boot-starter-web`中默认的`Jackson`依赖，添加`Fastjson`依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
       <exclusions>
           <exclusion>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-json</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>fastjson</artifactId>
       <version>1.2.68</version>
   </dependency>
   ```

2. 提供`FastJsonHttpMessageConverter`

   ```java
   @Bean
   FastJsonHttpMessageConverter fastJsonHttpMessageConverter(){
       FastJsonHttpMessageConverter converter = new 		
   FastJsonHttpMessageConverter();
       FastJsonConfig fastJsonConfig = new FastJsonConfig();
       fastJsonConfig.setDateFormat("yyyy/MM/dd");
       converter.setFastJsonConfig(fastJsonConfig);
       return converter;
   }
   ```

## 静态资源的访问

在Spring Boot的`WebMvcAutoConfiguration`中已经默认设置了五个静态资源的访问路径，按优先级顺序依次为`META-INF/resources`、`resources`、`static`、`public`、`/`。

自定义静态资源的位置

1. 在`application.properties`中添加

   ```prop
   spring.resources.static-locations=classpath:/xxx/
   # 也可以添加规则
   spring.mvc.static-path-pattern=/**
   ```

2. 在`Java`类中配置

   ```java
   @Configuration
   public class WebMVCConfig implements WebMvcConfigurer {
   	@Override
   	public void addResourceHandlers(ResourceHandlerRegistry registry) {
   		registry.addResourceHandler("/**").addResourceLocations("classpath:/xxx/");
   	}
   }
   ```

## 文件上传

主要使用`MultipartFile`类

1. 创建`Controller`

   ```java
   @RestController
   public class FileUploadController {
   	SimpleDateFormat sdf = new SimpleDateFormat("/yyyy/MM/dd/");
   
   	@PostMapping("/upload")
   	public String upload(MultipartFile file, HttpServletRequest req) {
   		String format = sdf.format(new Date());
   		String realPath = req.getServletContext().getRealPath("/img") + format;
   		File folder = new File(realPath);
   		if (!folder.exists()) {
   			folder.mkdirs();
   		}
   		String oldName = file.getOriginalFilename();
   		String newName = UUID.randomUUID().toString() + oldName.substring(oldName.lastIndexOf("."));
   		try {
   			file.transferTo(new File(realPath, newName));
   			String url = req.getScheme() + "://" + req.getServerName() + ":" + req.getServerPort() + "/img" + format + newName;
   			return url;
   		} catch (IOException e) {
   			e.printStackTrace();
   		}
   		return "error";
   	}
   }
   ```

2. 创建`index.html`

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
   <form action="/upload" method="post" enctype="multipart/form-data">
       <input type="file" name="file">
       <input type="submit" value="submit">
   </form>
   </body>
   </html>
   ```

3. 关于上传文件大小的限制需要在`application.properties`中进行修改

   ```properties
   spring.servlet.multipart.max-file-size=1MB
   ```

## @ControllerAdvice

- 全局异常处理

  1. 创建`MyCustomException`

     ```java
     @ControllerAdvice
     public class MyCustomException {
         //以response的方式返回
     	@ExceptionHandler(MaxUploadSizeExceededException.class)
     	public void myException(MaxUploadSizeExceededException e, HttpServletResponse res) throws IOException {
     		res.setContentType("text/html;charset=utf-8");
     		PrintWriter out = res.getWriter();
     		out.write("上传文件大小超出限制!");
     		out.flush();
     		out.close();
     	}
     
         //以视图的方式返回
     	@ExceptionHandler(MaxUploadSizeExceededException.class)
     	public ModelAndView myException(MaxUploadSizeExceededException e) throws IOException {
     		ModelAndView modelAndView = new ModelAndView("myerror");
     		modelAndView.addObject("error", "文件大小超过限制！");
     		return modelAndView;
     	}
     }
     ```

  2. 以`ModelAndView`的方式返回需要创建错误页面`myerror.html`以及`thymeleaf`的依赖

     ```html
     <!DOCTYPE html>
     <head xmlns:th="http://www.thymeleaf.org">
         <meta charset="UTF-8">
         <title>Title</title>
     </head>
     <body>
     	<h1 th:text="${error}"></h1>
     </body>
     </html>
     ```

- 预设全局数据

  1. 创建全局数据类，使用`Map`保存数据

     ```java
     @ControllerAdvice
     public class GlobalData {
     	@ModelAttribute(value = "person")
     	public Map<String,Object> mydata(){
     		Map<String,Object> map = new HashMap<>();
     		map.put("name","wan");
     		map.put("gender","male");
     		return map;
     	}
     }
     ```

  2. 在`Controller`中使用`Model`调用数据

     ```java
     @RestController
     public class HelloController {
     	@GetMapping("/hello")
     	public String hello(Model model){
     		Map<String, Object> map = model.asMap();
     		Set<String> keySet = map.keySet();
     		for (String s : keySet) {
     			System.out.println(s+":"+map.get(s));
     		}
     		return "hello";
     	}
     }
     ```

- 请求参数预处理

现在出现如下需求，前端向后端发送请求参数时出现命名冲突：

| key   | value    |
| ----- | -------- |
| name  | 三国演义 |
| price | 99       |
| name  | 吴冠中   |
| age   | 100      |

此时在服务器端接受的数据会有问题，解决方法如下：

1. 给接收参数重命名

   ```java
   @RestController
   public class BookController {
   	@PostMapping("/book")
   	public void addBook(@ModelAttribute("b") Book book, @ModelAttribute("a") Author author){
   		System.out.println(book);
   		System.out.println(author);
   	}
   }
   ```

2. 创建@ControllerAdvice类

   ```java
   @InitBinder(value = "a")
   public void initA(WebDataBinder binder){
       binder.setFieldDefaultPrefix("a.");
   }
   
   @InitBinder(value = "b")
   public void initB(WebDataBinder binder){
       binder.setFieldDefaultPrefix("b.");
   }
   ```

> @ModelAttribute
>
> WebDataBinder
>
> @InitBinder
>
> ControllerAdvice
>
> Model
>
> ModelAndView
>
> @ExceptionHandler

## 自定义异常

1. 在`classpath`下的`static`和`template`目录中创建`error`目录并创建静态错误页面和动态错误页面，按优先级依次访问：精确大于模糊，动态大于静态

2. 自定义错误数据

   ```java
   @Component
   public class MyErrorConfig extends DefaultErrorAttributes {
   	@Override
   	public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
   		Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, options);
   		errorAttributes.put("myerror","自定义异常信息！");
   		return errorAttributes;
   	}
   }
   ```

3. 自定义错误视图

   1. 自定义`ErrorViewResolver`继承自`DefaultErrorViewResolver`

   ```java
   @Component
   public class MyErrorViewResolver extends DefaultErrorViewResolver {
   	/**
   	 * Create a new {@link DefaultErrorViewResolver} instance.
   	 *
   	 * @param applicationContext the source application context
   	 * @param resourceProperties resource properties
   	 */
   	public MyErrorViewResolver(ApplicationContext applicationContext, ResourceProperties resourceProperties) {
   		super(applicationContext, resourceProperties);
   	}
   
   	@Override
   	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
   		ModelAndView modelAndView = super.resolveErrorView(request, status, model);
   		modelAndView.setViewName("myerror");
   		modelAndView.addObject(model);
   		return modelAndView;
   	}
   }
   ```

   2. 自定义视图`myerror.html`

> ErrorMvcAutoConfiguration
>
> DefaultErrorAttributes
>
> getErrorAttributes()
>
> DefaultErrorViewResolver
>
> resolveErrorView()

## CORS解决跨域问题

1. 在请求中添加注解`@CrosOrigin(origins = "http://xxxx:xxxx")`即可

2. 创建配置类

   ```java
   @Configuration
   public class MyMVCConfig implements WebMvcConfigurer {
   	@Override
   	public void addCorsMappings(CorsRegistry registry) {
   		registry.addMapping("/**")
   				.allowedOrigins("http://xxxx:xxxx")
   				.allowedHeaders("*")
   				.allowedMethods("*");
   	}
   }
   ```

> WebMvcConfigurer
>
> addCorsMappings(CorsRegistry registry)

## XML

1. 创建`beans.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean class="org.wan.xml.Hello" id="hello"/>
   </beans>
   ```

2. 创建`MyWebNvcConfig`

   ```java
   @Configuration
   @ImportResource(locations = "classpath:beans.xml")
   public class WebMvcConfig {
   }
   ```

## 注册拦截器

1. 自定义`Interceptor`

   ```java
   public class MyInterceptor implements HandlerInterceptor {
   	@Override
   	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
   		System.out.println("preHandle");
   		return true;
   	}
   
   	@Override
   	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
   		System.out.println("postHandle");
   	}
   
   	@Override
   	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
   		System.out.println("afterCompletion");
   	}
   }
   
   ```

2. 创建`WebMvcConfig`

   ```java
   @Configuration
   public class WebMvcConfig implements WebMvcConfigurer {
   	@Override
   	public void addInterceptors(InterceptorRegistry registry) {
   		registry.addInterceptor(myInterceptor()).addPathPatterns("/**");
   	}
   
   	@Bean
   	MyInterceptor myInterceptor(){
   		return new MyInterceptor();
   	}
   }
   ```

   

## 定义系统启动任务

1. CommandLineRunner

   - 创建MyCommanderLineRunner

     ```java
     @Component
     @Order(1)
     public class MyCommandLineRunner1 implements CommandLineRunner {
     	@Override
     	public void run(String... args) throws Exception {
     
     	}
     }
     
     @Component
     @Order(2)
     public class MyCommandLineRunner2 implements CommandLineRunner {
     	@Override
     	public void run(String... args) throws Exception {
     
     	}
     }
     //order表示优先级，值越小优先级越高
     ```

2. ApplicationRunner(获取参数的类型更丰富)

   - 创建MyApplicationRunner

     ```java
     @Component
     @Order(2)
     public class MyApplicationRunner1 implements ApplicationRunner {
     	@Override
     	public void run(ApplicationArguments args) throws Exception {
     
     	}
     }
     
     @Component
     @Order(1)
     public class MyApplicationRunner2 implements ApplicationRunner {
     	@Override
     	public void run(ApplicationArguments args) throws Exception {
     
     	}
     }
     ```

> CommandLineRunner
>
> ApplicationRunner

## 类型转换器

需求：前端向后端传递日期的字符串，后端使用Date对象无法接收，此时需要自定义转换器

```java
@Component
public class MyConverter implements Converter<String, Date> {
	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

	@Override
	public Date convert(String s) {
		if (s != null) {
			try {
				return sdf.parse(s);
			} catch (ParseException e) {
				e.printStackTrace();
			}
		}
		return null;
	}
}
```

## AOP

```java
@Component
@Aspect
public class MyLogger {
	@Pointcut("execution(* org.wan.aop.service.*.*(..))")
	public void pc1() {
	}

	@Before(value = "pc1()")
	public void before(JoinPoint jp) {
		String name = jp.getSignature().getName();
		System.out.println("before:" + name);
	}

	@After(value = "pc1()")
	public void after(JoinPoint jp) {
		String name = jp.getSignature().getName();
		System.out.println("after:" + name);
	}

	@AfterReturning(value = "pc1()",returning = "result")
	public void afterReturning(JoinPoint jp, Object result) {
		String name = jp.getSignature().getName();
		System.out.println("afterReturning:" + name+"----"+result);
	}

	@Around("pc1()")
	public Object around(ProceedingJoinPoint pjp) throws Throwable {
		return pjp.proceed();
	}

	@AfterThrowing(value = "pc1()",throwing = "e")
	public void afterThrowing(JoinPoint jp,Exception e) {
		String name = jp.getSignature().getName();
		System.out.println("afterThrowing:" + name+"----"+e.getMessage());
	}
}
```

> @Aspect
>
> @Pointcut("execution(* org.wan.aop.service.*\.\*(..))")
>
> @Before(value = "pc1()")
>
> @After(value = "pc1()")
>
> @AfterReturning(value = "pc1()",returning = "result")
>
> @Around("pc1()")
>
> @AfterThrowing(value = "pc1()",throwing = "e")



## JdbcTemplate

1. 依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-jdbc</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid-spring-boot-starter</artifactId>
       <version>1.1.10</version>
   </dependency>
   
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <scope>runtime</scope>
       <version>8.0.16</version>
   </dependency>
   ```

   

2. MySQL配置

   ```properties
   spring.datasource.one.url=jdbc:mysql://localhost:3306/test1?characterEncoding=utf8&useSSL=true&serverTimezone=UTC&zeroDateTimeBehavior=CONVERT_TO_NULL
   spring.datasource.one.username=root
   spring.datasource.one.password=123456
   spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
   
   
   spring.datasource.two.url=jdbc:mysql://localhost:3306/test2?characterEncoding=utf8&useSSL=true&serverTimezone=UTC&zeroDateTimeBehavior=CONVERT_TO_NULL
   spring.datasource.two.username=root
   spring.datasource.two.password=123456
   spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
   ```

3. 自定义`DataSource`

   ```java
   @Configuration
   public class DataSourceConfig {
   	@Bean
   	@ConfigurationProperties(prefix = "spring.datasource.one")
   	DruidDataSource dsOne(){
   		return DruidDataSourceBuilder.create().build();
   	}
   
   	@Bean
   	@ConfigurationProperties(prefix = "spring.datasource.two")
   	DruidDataSource dsTwo(){
   		return DruidDataSourceBuilder.create().build();
   	}
   }
   ```

4. 自定义`JdbcTemplate`

   ```java
   @Configuration
   public class JdbcTemplateConfig {
   
   	@Bean
   	JdbcTemplate jdbcTemplateOne(@Qualifier("dsOne") DataSource ds){
   		return new JdbcTemplate(ds);
   	}
   
   	@Bean
   	JdbcTemplate jdbcTemplateTwo(@Qualifier("dsTwo") DataSource ds){
   		return new JdbcTemplate(ds);
   	}
   }
   ```

> @ConfigurationProperties(prefix = "spring.datasource.one")
>
> DruidDataSource
>
> DruidDataSourceBuilder.create().build()
>
> JdbcTemplate
>
> @Qualifier("dsOne") DataSource ds

## MyBatis

1. 添加依赖以及配置`resource`

   ```xml
   <dependency>
       <groupId>org.mybatis.spring.boot</groupId>
       <artifactId>mybatis-spring-boot-starter</artifactId>
       <version>2.1.2</version>
   </dependency>
   
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid-spring-boot-starter</artifactId>
       <version>1.1.10</version>
   </dependency>
   
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <scope>runtime</scope>
       <version>8.0.16</version>
   </dependency>
   
   <!--添加resource的目的在于，将mapper.xml和对应的Mapper.java文件放在同一个package下-->
   <!--也可以在resource目录下创建和对应Mapper.java相同路径的mapper.xml-->
   <!--也可以将mapper.xml放于resource中的文件夹中，然后在application.properties中设置mybatis.mapper-locations=classpath:/mapper/*.xml-->
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

   

2. MySQL配置

   ```properties
   spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
   spring.datasource.username=root
   spring.datasource.password=123456
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   spring.datasource.url=jdbc:mysql://localhost:3306/test1?characterEncoding=utf8&useSSL=true&serverTimezone=UTC&zeroDateTimeBehavior=CONVERT_TO_NULL
    
   ```

3. 配置`mapper.xml`和`mapper.java`

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.example.mybatis.mapper.UserMapper">
       <select id="getAllUser" resultType="com.example.mybatis.bean.User">
           select * from user;
       </select>
   </mapper>
   ```

   ```java
   public interface UserMapper {
   	List<User> getAllUser();
   }
   ```

4. 配置多数据源

   - 配置数据源

     ```properties
     spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource
     spring.datasource.one.username=root
     spring.datasource.one.password=123456
     spring.datasource.one.url=jdbc:mysql://localhost:3306/test1?characterEncoding=utf8&useSSL=true&serverTimezone=UTC&zeroDateTimeBehavior=CONVERT_TO_NULL
     
     spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
     spring.datasource.two.username=root
     spring.datasource.two.password=123456
     spring.datasource.two.url=jdbc:mysql://localhost:3306/test2?characterEncoding=utf8&useSSL=true&serverTimezone=UTC&zeroDateTimeBehavior=CONVERT_TO_NULL
     ```

   - 创建`mapper1`和`mapper2`两个`package`，在各自包下创建`mapper.xml`和`mapper.java`

   - 创建自定义`DataSource`

     ```java
     @Configuration
     public class DataSourceConfig {
     
     	@Bean
     	@ConfigurationProperties(prefix = "spring.datasource.one")
     	DataSource dataSourceOne(){
     		return DruidDataSourceBuilder.create().build();
     	}
     
     	@Bean
     	@ConfigurationProperties(prefix = "spring.datasource.two")
     	DataSource dataSourceTwo(){
     		return DruidDataSourceBuilder.create().build();
     	}
     }
     ```

   - 创建配置类`MyBatisConfigOne`和`MyBatisConfigTwo`

     ```java
     @Configuration
     @MapperScan(basePackages = "com.example.mybatis.mapper1",sqlSessionFactoryRef = "sqlSessionFactory1",
     		sqlSessionTemplateRef = "sqlSessionTemplate1")
     public class MyBatisConfigOne {
     	@Resource(name = "dataSourceOne")
     	DataSource dsOne;
     
     	@Bean
     	SqlSessionFactory sqlSessionFactory1(){
     		SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
     		ssfb.setDataSource(dsOne);
     		try {
     			return ssfb.getObject();
     		} catch (Exception e) {
     			e.printStackTrace();
     		}
     		return null;
     	}
     
     	@Bean
     	SqlSessionTemplate sqlSessionTemplate1(){
     		return new SqlSessionTemplate(sqlSessionFactory1());
     	}
     }
     ```

     ```java
     @Configuration
     @MapperScan(basePackages = "com.example.mybatis.mapper2",sqlSessionFactoryRef = "sqlSessionFactory2",
     		sqlSessionTemplateRef = "sqlSessionTemplate2")
     public class MyBatisConfigTwo {
     	@Resource(name = "dataSourceTwo")
     	DataSource dsTwo;
     
     	@Bean
     	SqlSessionFactory sqlSessionFactory2(){
     		SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
     		ssfb.setDataSource(dsTwo);
     		try {
     			return ssfb.getObject();
     		} catch (Exception e) {
     			e.printStackTrace();
     		}
     		return null;
     	}
     
     	@Bean
     	SqlSessionTemplate sqlSessionTemplate2(){
     		return new SqlSessionTemplate(sqlSessionFactory2());
     	}
     }
     ```

> <build>
>  <resources>
>      <resource>
>          <directory>src/main/java</directory>
>          <includes>
>              <include>**/*.xml</include>
>          </includes>
>      </resource>
>      <resource>
>          <directory>src/main/resources</directory>
>      </resource>
>  </resources>
> </build>
>
> @MapperScan(basePackages = "com.example.mybatis.mapper1",sqlSessionFactoryRef = "sqlSessionFactory1",
> 		sqlSessionTemplateRef = "sqlSessionTemplate1")
>
> SqlSessionFactory
>
> SqlSessionTemplate

## JPA

1. 依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-jpa</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   
   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid-spring-boot-starter</artifactId>
       <version>1.1.10</version>
   </dependency>
   
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <scope>runtime</scope>
       <version>8.0.16</version>
   </dependency>
   ```

2. MySQL配置

   ```properties
   spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
   spring.datasource.url=jdbc:mysql://localhost:3306/test1?characterEncoding=utf8&useSSL=true&serverTimezone=UTC&zeroDateTimeBehavior=CONVERT_TO_NULL
   spring.datasource.username=root
   spring.datasource.password=123456
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   
   
   spring.jpa.show-sql=true
   spring.jpa.database=mysql
   spring.jpa.database-platform=mysql
   spring.jpa.hibernate.ddl-auto=update
   spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
   ```

3. 创建bean

   ```java
   @Entity(name = "t_book")
   public class Book {
   	@Id
   	@GeneratedValue(strategy = GenerationType.IDENTITY)
   	private Integer id;
   	private String name;
   	private String author;
   }
   ```

4. 创建dao

   ```java
   public interface BookDao extends JpaRepository<Book,Integer> {
   }
   ```

> spring.jpa.show-sql=true
> spring.jpa.database=mysql
> spring.jpa.database-platform=mysql
> spring.jpa.hibernate.ddl-auto=update
> spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
>
> @Entity(name = "t_book")
>
> @Id
> @GeneratedValue(strategy = GenerationType.IDENTITY)
>
> JpaRepository<Book,Integer>

## Redis

## Session

## Nginx

## MongoDB

## Docker

## RESTful


