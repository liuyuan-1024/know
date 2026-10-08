# SpringBoot2核心功能


## 一、配置文件


### 1、properties配置文件


### 2、yaml（yml）配置文件


非常适合用来做以数据为中心的配置文件



1.2.1、导入yaml配置文件



在需要yaml配置文件的类上使用：@ConfigurationProperties(prefix="前缀")



#### 2.1、yaml的基本语法


+ key: value，**key:**和**value**之间有空格
+ 大小写敏感
+ 使用缩进表示层级关系
+ 缩进只能用**空格**，不能使用**Tab**，在IDEA中使用Tab也没有问题。
+ 缩进的空格数不重要，只要同层级的元素**左对齐**即可
+ **#**表示注释
+ **单引号、双引号**表示字符串，可被转义或不转义。字符串不添加引号，**默认是双引号**；双引号不会转义**转义字符**，单引号会转义。



#### 2.2、数据类型


+  字面量：单个的、不可分割的值，String、null、number等 

```yaml
key: value
```

 

+  对象：就是键值对的集合，也就是Map集合 

```yaml
#行内写法
object(attr: value, attr1: value1, attr2: value2)
#普通写法
object:
	attr: value
	attr1: value1
	attr2: value2
```

 

+  数组 

```yaml
#行内写法
array[value, value1, value2]
#普通写法
array:
	- value
	- value1
	- valuen
```

 



## 二、Web开发


### 1、静态资源


#### 1.1、静态资源的目录


+ 默认访问：static（or /public、/META-INF/resources、/resources、/called）



#### 1.2、静态资源的访问


+  原理：静态资源的访问映射“/resources/**”，请求进来会先查看Controller能不能处理，不能处理的所有请求会交给静态资源处理器处理 
+  静态资源的访问前缀：默认是无前缀的，当我们利用拦截器拦截请求时就会误拦截静态资源请求，可以在application.yaml文件中设置前缀 

```yaml
spring: 
  mvc: 
    static-path-pattern: /前缀/**
```

 

+  修改静态资源的默认访问路径： 

```yaml
web:
    resources:
      static-locations: classpath:/文件路径
```

 



#### 1.3、欢迎页面WelcomePage


+ 静态资源的欢迎页面：index.html 
    - 可以配置静态资源路径，但设置了静态资源的访问前缀，就无法默认访问index页面为欢迎页面了
+ Controller处理“index”请求，显示欢迎页面



### 2、请求参数的处理


#### 2.1、注解获取参数


使用在方法形参上



1.  @PathVariable("参数名")：获取请求地址上的参数值。 
2.  @RequestHeader： 
    1. @RequestHeader：获取所有的请求头信息，封装成Map集合。
    2. @RequestHeader("key/name")：获取keyd对应的value。
3.  @RequestParam： 
    1. @RequestParam：获取所有的请求参数值，封装成Map集合。
    2. @RequestParam("参数名")：获取请求参数值，多选框的值可以使用list集合接收。
4.  CookieValue("key/name")：获取Cookie或者Cookie的值。 
5.  RequestAttribute("key/name")：获取请求域中name对应的value。 
6.  RequestBody：获取请求体的数据，POST请求有请求体。 
7.  MarixVariable(value="key/name", pathVar="指定路径变量上的矩阵变量")：矩阵变量，其绑定在路径中的所以一定要在求情路径中使用**" {} "**,这样才能将矩阵变量解析到请求中。 

```java
@RequestMapping(:"/xxx/{path}")
//利用“{}”将矩阵变量加入进来，当然矩阵变量需要使用@MarixVariable注解的参数来接收。
```

 

    1.  请求地址形式一：/xxx/{xxx;key=value;key1=value1;key2=value2} 
    2.  请求地址形式二：/xxx/{xxx;key=value;key1=value1;key2=value2,value3,value4} 
    3.  " ; "将各个矩阵变量隔开，" , "将矩阵变量的各个值隔开。 
    4.  SpringBoot默认关闭了矩阵变量，手动开启：UrlPathHelper会对路径进行解析，removeSemicolonContent（移除分号内容），默认是开启的 



```java
 /**
  * 实现WebMvcConfigurer接口，并重写configurePathMatch方法
  */
 public class WebConfig implements WebMvcConfigurer {
	 @Override
	 public void configurePathMatch(PathMatchConfigurer configurer) {
		 UrlPathHelper urlPathHelper = new UrlPathHelper();
 
		 //不移除分号内容
		 urlPathHelper.setRemoveSemicolonContent(false);
 
		 configurer.setUrlPathHelper(urlPathHelper);
	 }
 }
```



```java
	 /** 
	  * 向容器中注入一个WebMvcConfigurer类型的bean对象，并重写configurePathMatch方法
	  */
	 public class WebConfig {
		 @Bean
		 public WebMvcConfigurer webMvcConfigurer() {
			 return new WebMvcConfigurer() {
				 @Override
				 public void configurePathMatch(PathMatchConfigurer configurer) {
					 UrlPathHelper urlPathHelper = new UrlPathHelper();
	 
					 //不移除分号内容
					 urlPathHelper.setRemoveSemicolonContent(false);
	 
					 configurer.setUrlPathHelper(urlPathHelper);
				 }
			 };
		 }
	 }
```



#### 2.2、ServletAPI


在Controller方法的形参列表中添加Request、Session类型参数，在方法体中使用参数的API即可获取请求参数。



### 3、视图解析器与模板引擎Thymeleaf


SpringBoot的视图解析默认不支持JSP。SpringBoot的默认打包方式是jar包，也会就是一个压缩包，JSP不支持在压缩包内编译。可以引入第三方模板引擎技术实现页面渲染。



Thymeleaf是现代化、服务端Java模板引擎，效率不是很高，无法适用于高并发。



#### 3.1、Thymeleaf的基本语法
| 表达式名称 | | 语法 | 作用 |
| :---: | --- | --- | --- |
| 变量取名 | | ${...} | 获取Request域、Session域、对象 |
| 选择变量 | | *{...} | 获取上下文对象（application域对象） |
| 消息 | | #{...} | |
| 链接 | | @{...} | 构造超链接，自动拼接项目路径 |
| 片段表达式 | | ~{...} | |




#### 3.2、Thymeleaf的使用


1.  引入Starter 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

 

2.  自动配置好了Thymeleaf 
    1. 所有Thymeleaf配置值都在ThymeleafProperties配置文件中
    2. 已配置**SpringTemplateEngine**
    3. 已配置**ThymeleafViewResolver**
3.  开发人员只需要按照Thymeleaf给出的规则开发页面即可 

```java
//页面要放在“classpath:/templates/”路径下
public static final String DEFAULT_PREFIX = "classpath:/templates/";
//页面是html页面
public static final String DEFAULT_SUFFIX = ".html";
```

 



### 4、拦截器HandlerInterceptor接口


SpringBoot框架的拦截器，作用于servlet的拦截器Fitler一样。



+ HandlerInterceptor接口的三个方法 
    1. preHandler()：预处理。在请求被处理之前，进行一些操作。
    2. postHandler()：在目标方法完成之后，转发到目标页面之前，进行一些操作。
    3. afterCompletion()：请求处理全部完成后，进行一些操作。
+ 实现拦截器 
    1. 实现HandlerInterceptor接口，重写HandlerInterceptor接口的三个方法。
    2. 在自定义Config类（实现WebMvcConfigurer接口，能够自定义配置）中，重写addInterceptors方法来注册拦截器并配置拦截器的拦截规则。



### 5、文件上传


```html
<!-- 表单上传文件必须要有的属性 -->
<form role="form" th:action="@{/upload}" method="post" enctype="multipart/form-data"></form>
```



2. 文件上传的请求处理：



```java
@PostMapping("/upload")
public String upload(@RequestParam("email") String email, @RequestParam("password") String password, @RequestParam("headerImg") MultipartFile headerImg,
				 @RequestParam("headerImg") MultipartFile[] images) throws IOException {

log.info("邮箱={}，密码={}，头像的大小={}，图片数量={}", email, password, headerImg.getSize(), images.length);

   if (!headerImg.isEmpty()) {
	   String originalFilename = headerImg.getOriginalFilename();
	   //保存到服务器
	   headerImg.transferTo(new File("E:\\" + originalFilename));
   }
   
   for(MultipartFile image : images){
	   if (!image.isEmpty()) {
	   String originalFilename = image.getOriginalFilename();
	   //保存到服务器
	   headerImg.transferTo(new File("E:\\" + originalFilename));
	}
   }

   return "redirect:main";
}
```



### 6、SpringBoot默认错误处理机制


+  错误处理机制 
    1. 机器(其他)客户端发送请求产生错误：响应一个JSON数据，包含错误、响应状态码、异常的详细信息。
    2. 浏览器端发送请求产生错误：响应一个拥有以上相同错误信息的HTML页面。
+  自定义错误页面 
    1. 放在**静态资源路径**下的**error文件夹**会被默认为存放错误页面的文件夹，其中可以存放4xx、5xx页面会被自动解析。注意：5xx页面的文件名可以就叫**5xx**，这样所有5开头的错误就会被这一个页面响应。
    2. 放在**templates文件夹（模板文件夹）**下的**error文件夹**，其中可以存放4xx、5xx页面会被自动解析。



### 7、Web原生组件（Servlet、Filter、Listener）的注入


> SpringBoot默认是不支持Web原生组件的注入。
>



#### 7.1、使用ServletAPI注入


+ [@ServletConponentScan(basePackages ](/ServletConponentScan(basePackages ) = "待扫描包")：需要在主配置类上添加，配置Servlet、Filter、Listener的包扫描路径。 
+ [@WebServlet(urlPatterns ](/WebServlet(urlPatterns ) = "请求地址")：标识类为一个Servlet组件，这里面的请求不会被Interceptor的拦截。 
+ [@WebFilter(urlPatterns ](/WebFilter(urlPatterns ) = "请求地址")：/* 是原生Servlet的拦截全部请求，/** 是Spring的拦截全部请求。 
+ @WebListener：



**DispatcherServlet是如何注入容器中的？**



+ SpringBoot通过ServletRegistrationBean< DispatcherServlet >将DispatcherServlet注入容器。
+ 容器自动配置DispatcherServlet，而且DispatcherServlet的属性也绑定到在WebMvcProperties中前缀为“spring.mvc”的属性值。DispatcherServlet默认拦截：“/”请求



**为什么原生Servlet的请求不会被Interceptor拦截？**



+ 根据Servlet的精准匹配原则，Servlet会根据请求地址来匹配处理请求，当请求直接被原生Servlet匹配到时，就不会经过Interceptor，因此也无法被拦截。当然，请求经过DispatcherServlet会被拦截。



#### 7.2、使用RegistrationBean注入


+ ServletRegistrationBean类、FilterRegistrationBean类、ServletListenerRegistrationBean类：使用@Bean注解将其注册为容器中的组件



```java
//未添加属性：proxyBeanMethods = false，保证依赖的组件始终是单实例的。
@Configuration
public class MyRegistration {
      @Bean
      public ServletRegistrationBean myServlet() {
          return new ServletRegistrationBean(Servlet对象, 请求路径);
      }
  
      @Bean
      public FilterRegistrationBean myFilter() {
          //1.拦截Servlet对象处理的请求
          return new FilterRegistrationBean(Filter对象, Servlet对象);
          
          //2.自定义拦截路径
          FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
          List<String> list = Arrays.asList("/filter", "/filter1", ...);
          filterRegistrationBean.setUrlPatterns(list);
          return filterRegistrationBean;
      }
  
      @Bean
      public ServletListenerRegistrationBean myListener() {
          return new ServletListenerRegistrationBean(Listener对象);
      }
  }
```



### 8、定制化原理


#### 8.1、常见的定制化方式


1.  修改application.properties配置文件。 
2.  [**@Bean **](/Bean )** **替换、增加容器中的默认组件、视图解析器 + **@Configuration自定义配置类xxxConfiguration** 
3.  xxxCustomizer：定制化器，可以修改xxx的默认规则。 
4.  Web应用实现**WebMvcConfigurer**即可实现定制化功能：也能加[@Bean ](/Bean )  



```java
//@EnableWebMvc：全面接管WebMVC,这会导致SpringBoot针对资源处理的自动配置全部失效，此注解不常用
@Configuration
public class MyWebConfig implements WebMvcConfigurer
```



#### 8.2、原理分析套路


Starter场景 --> xxxAutoConfiguration --> 导入xxx组件 --> 绑定xxxProperties --->绑定application.properties配置文件



### 9、数据访问


#### 9.1、SQL（包含配置过程）


1. 配置数据库操作场景：一般为MyBatis
2. 导入数据库驱动：一般为MySQL
3. 配置Druid连接池



##### 9.1.1、数据源的自动配置


1.  导入JDBC的starter场景（一般使用MyBatis框架） 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

 

2.  数据库驱动，官方并未自动配置数据库驱动，我们需要配置与自己数据库对应的相同版本的驱动 



```xml
<!-- 第一种方法：依赖引入具体版本 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.49</version>
</dependency>
<!-- 第二种方法：重新声明仲裁版本 -->
<properties>
    <java.version>11</java.version>
    <mysql.version>5.1.49</mysql.version>
</properties>
```



##### 9.1.2、Druid连接池


1.  导入Druid的场景启动器 

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.8</version>
</dependency>
```

 

2.  配置Druid连接池 

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/blog
    username: root
    password: root_ly

    druid:
      #监控页的配置
      stat-view-servlet:
        enabled: true
        reset-enable: false

      filters: stat, wall, slf4j

      web-stat-filter: #监控web
        enabled: true

      filter: #对上面的filters的各个项进行详细的配置
        stat: #防止sql注入
          slow-sql-millis: 1000 #毫秒
          log-slow-sql: true
        wall: #防火墙
          enabled: true
          config:
            drop-table-allow: false
```

 



##### 9.1.3、MyBatis


1. 导入MyBatis的场景启动器

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
```

 

2. 只要我们写的Mapper接口添加了 [**@Mapper **](/Mapper )** ** 就会被MyBatis自动扫描进来。



+ **纯配置文件方式**



```yaml
mybatis:
  config-location: classpath:mybatis/mybatis_config.xml #全局配置文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml #sqlmpper文件位置
```



```xml
<!--
	编写Mapper接口并绑定xml配置文件
	Mapper接口需要添加@Mapper
-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="Mapper接口的全类名">
	sql语句
</mapper>
```



+ **注解+配置文件**  
在mapper接口中，使用 **“@Select、@Insert、@Delete、**[**@Update **](/Update )** ” ** 可以实现sql语句的编写，每个数据库操作标签都有 [**@Option **](/Option )** **，即配置项，可以配置增删改查标签的属性。当sql语句比较复杂时，可以使用Mapper配置文件的方式



#### 9.1.4、MyBatisPlus


#### 9.2、NoSQL


Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http:_www.redis.cn_commands_geoadd) 索引半径查询。 Redis 内置了 [复制（replication）](http:_www.redis.cn_topics_replication)，[LUA脚本（Lua scripting）](http:_www.redis.cn_commands_eval)， [LRU驱动事件（LRU eviction）](http:_www.redis.cn_topics_lru-cache)，[事务（transactions）](http:_www.redis.cn_topics_transactions) 和不同级别的 [磁盘持久化（persistence）](http:_www.redis.cn_topics_persistence)， 并通过 [Redis哨兵（Sentinel）](http:_www.redis.cn_topics_sentinel)和自动 [分区（Cluster）](http:_www.redis.cn_topics_cluster-tutorial)提供高可用性（high availability）。



### 10、单元测试


#### 10.1、Junit5


详细注解：[JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)



常用注解：



+ [@Test ](/Test ) 
+ @ParameterizedTest：参数化测试
+ @RepeatedTest：可重复测试
+ @RepeatedTest：为测试方法赋予一个名称
+ @BeforeEach：在测试方法执行之前自动执行
+ @BeforeEach：在测试方法执行之后自动执行
+ @BeforeAll：在所有测试方法执行之前自动执行，一般为static
+ @AfterAll：在所有测试方法执行之后自动执行，一般为static
+ @Tag：标识测试的类别
+ @Disabled：禁用测试
+ @Timeout：超时，可以设置超时时间，当测试超时，报异常
+ @ExtendWith：为测试类提供扩展测试功能



#### 10.2、断言（assertions）


断言的方法都在：**org**.**junit**.**jupiter**.**api**.**Assertions**这个包里



前面的断言失败，后面的代码就不会执行



+ 简单断言
+ 数组断言
+ 组合断言
+ 异常断言
+ 超时断言



### 11、指标监控（SpringBoot Actuator）


引入Actuator



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



使用Actuator



> 所有的Actuator相关的信息都在 **项目名/actuator/** 下查看。
>



## 三、SpringBoot的高级特性


### 1、profile功能


为方便多环境适配，springboot简化lprofile功能



1. 创建各个环境的配置文件，eg：application-test.yaml、application-prod.yaml等等，“-xxx”是标识一个文件是什么环境，xxx可以自定义书写。
2. 在主配置文件中说明当前的环境：spring.profile.active=环境标识后缀



## 主程序


```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        //启动SpringBoot应用
        SpringApplication.run(Application.class, args);
    }
}
```



+  @SpringBootApplication：标注类为SpringBoot的一个应用 
    -  @SpringBootConfiguration：SpringBoot的配置  
	@Configuration：Spring配置类 
    -  @EnableAutoConfiguration：自动配置  
	@AutoConfigurationPackage：自动配置包  
		@AutoConfigurationPackage：自动配置**包注册**  
	@Import({AutoConfigurationImportSelector.class})：自动配置导入选择器 

