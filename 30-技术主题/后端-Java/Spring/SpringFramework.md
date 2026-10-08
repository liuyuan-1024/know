> Spring 框架是一个分层的、面向切面的 Java 应用程序的一站式轻量级解决方案，它是 Spring 技术栈的核心和基础，是为了解决企业级应用开发的复杂性而创建的。
>



## 一、Spring Farmework的两大核心概念


1. IoC: 指将创建对象的过程交给Spring。IoC 是 Inversion of Control 的简写，译为“控制反转”，它不是一门技术，而是一种设计思想，是一个重要的面向对象编程法则，能够指导我们如何设计出松耦合、更优良的程序。
2. AOP: Aspect Oriented Programming(面向切面编程)。AOP用来封装多个类的公共行为，将那些与业务无关，却为业务模块所共同调用的逻辑封装起来，减少系统的重复代码，降低模块间的耦合度。另外，AOP 还解决一些系统层面上的问题，比如日志、事务、权限等。



### 1.IOC


**能注入的数据有三类**  
a.基本类型和String  
b.其他bean类型(在配置文件中或注解配置过的bean)  
c.复杂类型/集合类型



**注入的方式：三种 ，前两种都是在xml中进行声明**



**第一种：使用构造函数提供（不经常变化的数据,会找定义变量的名称）**



1. **使用的标签：** constructor-arg
2. **标签出现的位置**：bean标签的内部
3. **标签中的属性：** 用于指定构造函数中那个参数赋值  
**type:** 用户指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型  
**index:** 用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引位置是从0开始  
**name:** 用于指定构造函数中指定名称的参数赋值（常用）  
**value：** 用于提供基本类型和String类型的数据  
**ref:** 用于指定其他的Bean类型数据。他指的就是在spring的IOC容器中出现过的bean对象



```yaml
<bean id="accountService" class="con.service.Impl.AccountServiceImpl">
	<constructor-arg name="name" value="test"></constructor-arg>
	<constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
<bean id="now" class="java.util.Data"></bean>
```



4. **优势：** 在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功  
**弊端：** 改变了bean对象的实例方式，是我们创建对象时，如果用不到这些数据，也必须提供。



**第二种：使用set方法提供 常用（因为在使用的service中，创建了setter方法，因此找的是setter方法名称）**



1. **涉及的标签：** property
2. **标签出现的位置：** bean标签的内部
3. **标签中的属性：**  
**name:** 用于指定注入时所调用的set方法名称  
**value:** 用于提供基本类型和String类型的数据  
**ref:** 用于指定其他的Bean类型数据。他指的就是在spring的IOC容器中出现过的bean对象
4. **优势：** 创建对象时没有明确的限制，可以直接使用默认构造函数  
**弊端：** 某个成员必须有值，则获取对象时有可能set方法没有执行



**servcie**



```java
public void setAccountDao(IAccountDao accountDao) {
    this.accountDao = accountDao;
}
```



**xml**



```yaml
    <bean id="accountService" class="com.***.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"></property>
    </bean>
```



**第三种** **使用注解提供的方法**  
**前提条件：**  
首先在recourse中的xml中，先进行注解说明，告知spring在创建容器时要扫描的包，配置所需要的标签不是在beans的约束中而是一个名称为context名称空间中。



```java
<!-- 配置容器扫描时，要扫描的包-->
<context:component-scan base-package="com.itheima"></context:component-scan>
```



1. **用于创建对象的**  
他们的作用就和在xml配置文件中编写一个`<bean>`标签实现的功能实现是一样的  
xml中： `<bean id="beanFactory" class="com.itheima.factory.BeanFactory"></bean>`  
**@Component：**  
作用：用于把当前类对象存入spring容器中  
属性： value：用于指定bean的id，当我们不写时，它的默认值是当前类名  
[**@Controller **](/Controller )** ** 一般用在表现层  
[**@Service **](/Service )** ** 一般用在业务层  
[**@Repository **](/Repository )** ** 一般用在持久层  
以上三个注解的作用和属性与Conponent一样。
2. **用于注入数据的**  
他们的作用就和在xml配置文件中的`<bean>`标签中写一个`<property>`标签的作用是一样  
xml中：`<property name="accountDao" ref="accountDao"></property>`  
**@Autowired:** 自动按照类型注入。  
`情况一` 只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功。  
`情况二` 如果ioc容器中没有任何bean的类型和要注入的变量类型匹配，则报错  
`情况三`如果ioc容器中多个类型匹配时，首先按照类型，先圈定一个范围，然后根据变量名称在这个范围中继续查找，找到了注入成功，否则失败  
**出现位置：** 成员变量上，方法上  
**细节：** 在使用注解注入时，set方法就不是必须的了  
**@Qualifier:**  
**作用**：在按照类中注入的基础之上再按照名称注入，它在给类成员注入时不能单独使用（和Autowired一起使用），但是在给方法参数注入时可以  
**属性：** value用于指定注入bean的id  
[**@Resource **](/Resource )** ** 作用直接按照bean的id注入，它可以独立使用  
**属性：** name用于指定注入bean的id  
**以上三个注入都只能注入其他bean类型的数据，而基本类型和string 类型无法使用上述注解实现。另外，集合类型的注入只能通过xml实现**  
[**@Value **](/Value )** **  
**作用**：用于注入基本类型和String类型的数据  
**属性**：value用于指定数据的值。它可以使用spring中SpEL（Spring的el表达式）SpEL的写法${表达式}
3. **用于改变作用范围的**  
他们的作用就和在bean中标签中使用scope属性实现的功能一样的  
[**@Scope **](/Scope )** **  
**作用**：用于指定bean的作用范围i  
**属性：** value 指定范围的取值，通常是：singleton（单例） prototype（多例）  
**xml中** `<bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype"></bean>`
4. **生命周期相关**  
他们的作用就和bean标签中使用init-method和destroy-methode的作用一样  
[**@PreDestory **](/PreDestory )** **  
**作用：** 用于指定销毁的方法  
[**@PostConstruct **](/PostConstruct )** **  
**作用：** 用于指定初始化方法
5. **复杂类型注入/集合类型注入**  
**xml种**  
用于给list结构集合注入的标签：list array set  
用于给map结构集合注入的标签：map props  
结构相同，标签可以互换



## 二、springframework的核心注解


[@Configuration ](/Configuration ) 所注解的类和 [@Bean ](/Bean ) 所注解的方法。 



+ @Configuration：标识该**类**为bean定义的地方。被标识的类等价于xml配置文件。  
此外，@Configuration类允许通过调用同一个类中的其他@Bean方法来定义bean间依赖，即引入第三方bean对象
+ @Bean：表明**方法**的实例化,、配置和初始化都是由 Spring IoC 容器管理的新对象。就是将方法的返回值作为bean。 
    1. bean的id就是方法名，bean的类型就是方法的返回值类型。
    2. bean如何自动注入外部bean? 在方法的形参列表中设置外部bean类型的形参即可，不需要@Autowired。
    3. bean如何自动注入内部bean? 直接在方法内部调用内部bean对应的方法即可
+ [@ComponentScan(basePackages ](/ComponentScan(basePackages ) = "路径名称")：配置spring的扫描路径。 
+ [@PropertySource(value ](/PropertySource(value ) = {"classpath:db.properties"})：引入外部属性资源文件，value是数组，可以引入多个外部属性资源文件。 
+ [@Import(value ](/Import(value ) = {SecondJavaConfig.class})：  
    1. 导入其他JavaConfig配置类
    2. 导入普通类，将其注册为bean
    3. 导入ImportSelector接口的实现类，直接注册多个bean，这些bean只能通过类型获取
    4. 导入ImportBeanDefinitionRegistrar接口的实现类，可以通过类型和名称获取bean



## 三、spring对bean的管理细节


### 1.创建bean的三种方式


**第一种：使用默认构造函数创建**  
在spring的配置文件中使用的是bean标签，配以id和class属性之后，且没有其他属性和标签时。  
采用的就是默认[构造函数](https://so.csdn.net/so/search?q=%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0&spm=1001.2101.3001.7020)创建bean对象，此时，如果类中没有没有默认构造函数，则对象无法创建  
**id和class**  
一般情况下,装配-个Bean时,通过指定一个id属性作为Bean的名称  
id属性在[IOC](https://so.csdn.net/so/search?q=IOC&spm=1001.2101.3001.7020)容器中必须是唯- -的  
一如果Bean的名称中含有特殊字符,就需要使用name属性  
**class**  
class用于设置一个类的完全路径名称,主要作用是IOC容器生成类的实例



```yaml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>
```



**第二种：使用普通工厂中的方法创建对象**（使用某个类中的方法创建对象，并存入spring容器）



```yaml
<bean id="instancefactory" class="com.itheima.factory.InstanceFactory"></bean>
<bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>
```



**第三种：使用工厂中的静态方法创建对象**（使用某个静态类中的静态方法创建对象，并存入spring容器）



```yaml
<bean id="instancefactory" class="com.itheima.factory.StaticFactory" factory-method="getAccountService"></bean>
```



### 2.bean对象的作用范围


<!-- 这是一张图片，ocr 内容为： -->
![](https://img-blog.csdnimg.cn/20200217233611324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNjYxNDA4,size_16,color_FFFFFF,t_70)global-session 作用于集群的会话范围（全局会话范围）当不是集群环境时，他就是session  
<!-- 这是一张图片，ocr 内容为： -->
![](https://img-blog.csdnimg.cn/20200217234000265.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNjYxNDA4,size_16,color_FFFFFF,t_70)



### 3.bean对象的生命周期


**单例对象**  
出生，当容器创建时对象出生  
活着，只要容器还在，对象一直活着  
死亡，容器销毁，对象消亡  
总结：单例对象的生命周期和容器相同  
**多例对象**  
出生，当我们使用对象时，spring框架为我们创建  
活着，对象在使用过程中就一直活着  
死亡，当对象长时间不用，且没有别的对象引用时，由Java的垃圾回收机制回收

