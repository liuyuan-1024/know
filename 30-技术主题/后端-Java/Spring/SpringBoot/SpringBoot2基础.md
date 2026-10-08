# SpringBoot2基础入门


## 一、自动配置原理


### 1.1、依赖管理


```xml
<!--依赖管理-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.4</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<!--
	上面父项目的父项目
	声明了开发中常用的依赖的版本号，所以在导入默认依赖时，无需关注依赖的版本号。自动版本仲裁。
	在导入非默认依赖时，需要声明版本号。
-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.6.4</version>
</parent>
```



+  自定义版本： 
    -  
        1.  在父项目**“spring-boot-dependencies”**中，查看规定当前依赖的版本号的key值。 
        2.  在项目中的pom.xml中的重写配置如下即可。 

```xml
<properties>
    <java.version>11</java.version>
    <mysql.version>5.7.19</mysql.version>
</properties>
```

 

+  场景启动器Starter 
    -  官方给出的场景启动器一般为：spring-boot-starter-_， _代表某种场景。 
    -  只要引入了某种场景的Starter，SpringBoot就会自动引入此场景所需的常规依赖。 
    -  SpringBoot支撑的所有场景：[SpringBoot的所有Starter](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters) 
    -  第三方的场景启动器：*-spring-boot-starter 
    -  所有的场景启动器的最底层依赖（SpringBoot自动配置功能的核心依赖）： 

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.6.4</version>
  <scope>compile</scope>
</dependency>
```

 



### 1.2、自动配置


+  自动配置Tomcat 
    - 引入Tomcat依赖
    - 配置Tomcat
+  自动配置SpringMVC 
    - 引入SpringMVC全套组件
    - 自动配置SpringMVC的常用组件（功能）
+  自动配置Web常用功能：字符编码问题 
+  默认包结构 
    - 主程序所在的包会被默认扫描。因此，我们无需配置包扫描
    - 自主配置扫描路径： 
        1. [@SpringBootApplication(scanBasePackages ](/SpringBootApplication(scanBasePackages ) = "xx.xx") 
        2. [@ComponentScan(value ](/ComponentScan(value ) = "xx.xx") 
+  按需加载所需自动配置项 
    -  引入了哪些Starter，就会自动配置哪些配置项 
    -  SpringBoot的所有自动配置功能都依赖于： 

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-autoconfigure</artifactId>
  <version>2.6.4</version>
  <scope>compile</scope>
</dependency>
```

 



## 二、SpringBoot容器


### 2.1、添加组件（注解方式）


#### 2.1.1、[@Configuration(proxyBeanMethods ](/Configuration(proxyBeanMethods ) = true) 


+ 将类注册为配置类，相当于Spring中的xml配置文件
+ Full(全)模式、Lite(轻量级)模式 
    - Full模式：proxyBeanMethods = true，配置类会代理bean方法。即在使用bean方法时，会检查SpringBoot容器中是否存在此bean对象。当然，这会降低执行速度。
    - LIte模式：proxyBeanMethods = false，配置类不会启用代理bean方法。这样执行速度会加快。
    - 如何选择模式：仅是向SpringBoot容器中注入组件，而组件之间也并无依赖，一般调整成false。反之，调整成true。



#### 2.1.2、@Bean、@Controller、@Service、[@Repository ](/Repository ) 


+ @Bean：将方法的**返回值**作为bean对象（组件）注入SpringBoot容器中。bean对象一般为单实例
+ 其余注解相似：控制层、业务逻辑层、持久层



#### 2.1.3、@ComponentScan、[@Import ](/Import ) 


+ @ComponentScan("包路径")：配置扫描路径
+ @Import({xx.class, ...})：导入多个其他组件，自定义或第三方；需要用在SpringBoot容器中组件对应的类上，被导入组件的名称就是它的全类名



#### 2.1.4、@Conditional：按需装配


+ 条件装配：满足conditional的指定条件，则进行组件注入



#### 2.1.5、[@ImportResource ](/ImportResource ) 


+ @ImportResource("classpath:***.xml")：导入Spring配置文件



### 2.2、配置绑定


+  使用Java读取properties文件中的内容，并将其封装到JavaBean中，以供随时取用。 
+  核心注解[**@ConfigurationProperties(prefix **](/ConfigurationProperties(prefix )** = "前缀") **：在类上使用，为该类读取properties配置文件，并绑定以该前缀开头的属性 
    1.  [@ConfigurationProperties(prefix ](/ConfigurationProperties(prefix ) = "前缀") + [@Component ](/Component )   
只有在SpringBoot容器中注册了的组件，才能享受SpringBoot提供的强大功能，@Component就是将类注册为SpringBoot容器中的组件。 
    2.  [@ConfigurationProperties(prefix ](/ConfigurationProperties(prefix ) = "前缀") + @EnableConfigurationProperties(xxx.class)、   
第三方类我们无法为题添加@Component注解。因此，我们为**SpringBoot配置文件**添加**@EnableConfigurationProperties(第三方类.class)**，开启某个类的属性配置功能，并将该类注入SpringBoot容器。 



## 三、自动配置


### 3.1、@SpringBootApplication


@SpringBootApplication的三个核心注解：



```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```



#### 3.1.1、@SpringBootConfiguration


+ @Configuration：说明@SpringBootConfiguration就是标记了一个类是配置类



#### 3.1.2、@EnableAutoConfiguration


+  @AutoConfigurationPackage：自动配置包 
    1. @Import({Registrar.class})：导入Registrar类，此类批量注册了**@SpringBootApplication所注解的Application类所在的包**中所有的组件。
+  @Import({AutoConfigurationImportSelector.class})： 
    1.  AutoConfigurationImportSelector类： 

```java
this.getAutoConfigurationEntry(annotationMetadata)//为容器批量注册一些组件。
List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes)//获取所有需要导入到容器中的组件(配置类)
List<String> configurations = pringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());//利用工厂加载一些东西，加载啥呢？看下面
Map<String, List<String>> loadSpringFactories(ClassLoader classLoader)//真正得到所有组件
```

 



### 3.2、自动配置流程


+ 查看自动配置了哪些东西：在application.properties文件中



```java
#开启DEBUG模式，可以在控制台看到哪些自动配置生效了：Negative(不生效)、Positive(生效)
debug=true
```



+ 修改配置项： 
    1. 参考官网：[Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties)
    2. 自己加入或替换组件：利用注解@Bean、@Component等等（就近配置）
    3. 自定义组件：xxxxCustomizer



## 四、常用小技巧


+  Lombok依赖 
+  热更新 

```xml
<!-- 热更新 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

 

+  Spring Initailizr：IDEA创建SpringBoot项目 

