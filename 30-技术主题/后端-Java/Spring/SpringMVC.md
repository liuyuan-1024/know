# Sping MVC基础


+  就是SpringWeb 
+  Spring的model-view-controller框架是围绕一个DispatcherServlet来设计的，这个Servlet会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染等，甚至还能支持文件上传。 



## 一、Spring MVC的优点


1. 客户端请求提交到DispatcherServlet
2. DispatcherServlet控制器查询一个或多个HandleMaping, 找到处理请求的Controller
3. DispatcherServlet将请求提交到Controller
4. Controller调用业务逻辑处理后, 返回ModelAndView
5. DispatcherSerlvet查询一个或多个ViewResoler(视图解析器), 找ModelAndView指定的视图
6. 视图负责将结果显示到客户端



## 二、SpringMVC的web.xml的配置


```xml
        <!-- 配置DispatcherServlet-->
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 设置SpringMVC核心配置文件的名称和位置 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 自定义的核心配置文件 -->
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <!-- 将DispatcherServlet的初始化提前到服务器启动时 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```



## 三、SpringMVC的标识


+ 各大组件在SpringMVC中就是一个普通的类，所以需要**注解**标识类为一个组件（只有标识还不够，还需要扫描才能将类作为bean）



1. @Component：标识类为普通的组件
2. @Controller：标识类为控制层组件
3. @Service：标识类为业务层组件
4. @Repository：标识类为持久层组件
5. @RequestMapping("访问路径")：构建请求与控制器方法之间的映射关系



## 四、SpringMVC的扫描


+ 在自定义核心中定义扫描路径，扫描该路径下的类是否为Spring的组件



```xml
    <!-- 配置扫描路径-->
    <context:component-scan base-package="com.ashes.mvc.controller"></context:component-scan>
```



## 五、Thymeleaf视图处理器


+ 在自定义核心配置文件中配置Thymeleaf视图处理器



```xml
     <!-- 配置Thymeleaf视图解析器-->
    <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <!-- 视图解析器的优先级 -->
        <property name="order" value="1"/>
        <!-- 视图解析时使用的编码-->
        <property name="characterEncoding" value="UTF-8"/>
        <!-- 模板引擎 -->
        <property name="templateEngine">
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <!-- 模板处理器 -->
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                        <!-- 视图前缀 -->
                        <property name="prefix" value="/WEB-INF/templates/"/>
                        <!-- 视图后缀 -->
                        <property name="suffix" value=".html"/>
                        <!-- 模板的模型 -->
                        <property name="templateMode" value="HTML5"/>
                        <property name="characterEncoding" value="UTF-8"/>
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
```



## 六、@RequestMapping注解


+ 将请求与处理请求的控制器方法联系起来, 建立映射关系当SpringMVC收到请求时, 会通过映射关系找到对应的控制器方法处理请求



### 6.1、@RequestMapping注解的位置


1. 标识类：设置映射请求的请求路径的初始信息
2. 标识方法：设置映射请求的请求路径的具体信息



+ 当Controller既标识了类, 又标识了方法, 那么需要先访问初始信息再访问具体信息



### 6.2、@RequestMapping如何匹配请求?


+ 这与@RequestMapping的属性有关，一个属性对应一个匹配方式。value属性是必须设置的，也就是至少需要请求地址来匹配请求。多个属性同时使用，那么请求就要同时满足多个属性的条件才能匹配到控制器方法



常用的属性：



1. value(String [])：通过**请求地址**匹配请求，参数是String数组，故多个请求地址能对应同一个控制器方法。请求地址不匹配报404。
2. method(RequestMethod [])：通过请求的**请求方式**（get/post/put等等）匹配请求，故请求的请求方式必须匹配RequestMethod数组中的某一个值。请求方式不匹配报405。
3. params(String [])：通过请求**参数**匹配请求，必须满足参数数组的所有值。请求参数不匹配报404。
4. headers(String [])：通过请求的**请求头**匹配请求，必须满足参数数组的所有值。请求头不匹配报404。



### 6.3、SpringMVC注解的value属性支持ant风格的路径格式


+ ant风格



1.  ?：单个任意字符 
2.  *：0个或多个任意字符 
3.  **：一层或多层任意目录  
语法格式: /**/xxx 必须这样写，前后不能有除“/”外的其他字符 



### 6.4、SpringMVC注解的value属性支持路径中的占位符（{}）


**{}**表示占位符，其中可以定义name，用于表示参数名。只能使用@PathVariable获取占位符值，并将值赋值给形参。故在控制器方法的形参列表中需要使用一个注解：[**@PathVariable(value **](/PathVariable(value )** = "id或者password") **，在跟上与value值同名的形参。使用了占位符就必须要传入参数，否则或报404。



eg：



```java
    @RequestMapping("/test/{id}/{password}")
    public String test(@PathVariable("id") Integer id, @PathVariabl("password")String password){
        方法体
    }
```



## 七、SpringMVC获取请求参数


### 7.1、通过ServletAPI获取参数（不常用）


HttpServletRequest request，使用request的getParameter()获取单值请求参数、getParameterValues()获取多值请求参数（同一个参数名，多个参数值，例如：多选框）



### 7.2、通过控制器形参来获取请求参数（常用）


控制器形参需要与请求参数同名，这样才能正确地获取请求参数的值。对于多值参数：可以使用String参数来获取，也可以使用String数组参数来获取。



#### 7.2.1、@RequestParam注解


+ value属性：请求参数名。
+ required属性：设置请求参数是否必须传输，默认为true。
+ defaultValue属性：无论请求参数是否必须传输，当请求参数为空字符串或者不传输时，会使用默认值为请求参数赋值。



若控制器形参与请求参数不同名，则需要为**控制器形参**添加一个注解：[**@RequestPara(value **](/RequestPara(value )** = "请求参数名"， required = "true"，defaultValue="默认值") **，将请求参数与控制器形参建立映射关系。



#### 7.2.2、@RequestHeader注解


属性与用法同@RequestParam注解一样，但@RequestHeader注解是建立请求头信息和控制器形参之间的映射关系。



#### 7.2.3、@CookieValue注解


属性与用法同@RequestParam注解一样，但@CookieValue注解是建立Cookie数据和控制器形参之间的映射关系。



### 7.3、通过POJO（实体类）获取控制器形参（常用）


使用实体类型的形参代替控制器原本的形参，Spring会将请求中的参数与实体类的参数相对应（请求参数与实体类的属性同名）并将请求参数值注入属性中（底层：反射调用set()）



### 7.4、CharacterEncodingDFilter解决获取请求参数的乱码问题


1. get请求的乱码是由tomcat造成的，需要到tomcat的核心配置文件：server.xml中的Connector标签中添加“URIEncoding="UTF-8"”。get请求乱码在tomcat8及以后的版本已被解决，只有tomcat7会出现get请求乱码。
2. post请求乱码：在web.xml中配置SpringMVC中的CharacterEncodingFilter来修改浏览器post请求的默认编码和服务器响应的编码



```xml
        <!-- 配置修改post请求的编码的过滤器-->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!-- 设置请求的编码 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <!-- 设置响应的编码 -->
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```



## 八、域对象共享数据


+ 三大域对象：request域、session域、servletContext域



### 8.1、向request域对象中共享数据


#### 8.1.1、使用ServletAPI向request域对象中共享数据(不常用)


request.setAttribute("name","value")，将value以键值对形式共享到request域中



#### 8.1.2、使用ModelAndView向request域对象中共享数据(建议使用)


1. ModelAndView：包括model和view。model用于向request域中共享数据，view用于设置视图，实现页面跳转。
2. 无论是哪种数据共享方式，模型数据和视图信息都会封装成一个ModelAndView对象



```java
    @RequestMapping("/testModelAndView")
    public ModelAndView testModelAndView() {
        ModelAndView mav = new ModelAndView();

        //向reqyest域中共享数据
        mav.addObject("name","value");

        //设置视图，实现页面跳转
        mav.setViewName("viewName");

        return mav;
    }
```



#### 8.1.3、使用Model向request域对象中共享数据


```java
    @RequestMapping("/testModel")
    public String testModel(Model model) {
        //向request域中共享数据
        model.addAttribute("hello", "world");
        return "视图名称";
    }
```



#### 8.1.4、使用Map向request域对象中共享数据


```java
    @RequestMapping("/testMap")
    public String testMap(Map<String, Object> map) {
        //向request域中共享数据
        map.put("hello", "world");
        return "视图名称";
    }
```



#### 8.1.5、使用ModelMap向request域对象中共享数据


```java
    @RequestMapping("/testModelMap")
    public String testModelMap(ModelMap modelMap) {
        //向request域中共享数据
        map.put("hello", "world");
        return "视图名称";
    }
```



#### 8.1.6、Model接口、Map接口、ModelMap类之间的关系


Model接口、Map接口、ModelMap类，三者类型的参数其最终本质是BindingAwareModel类的实例。因为BindingAwareModel类继承了ExtendedModelMap类，ExtendedModelMap类实现了Model接口、继承了ModelMap类。而ModelMap类是Model接口、Map接口的实现子类，



### 8.2、向session域对象中共享数据


```java
    @RequestMapping("/testSession")
    public String testSession(HttpSession session) {
        //向request域中共享数据
        session.setAttribute("hello", "world");
        return "视图名称";
    }
```



### 8.3、向ServletContext域(application域)对象中共享数据


```java
    @RequestMapping("/testApplication")
    public String testApplication(HttpSession session) {
        ServletContext application = session.getServletContext();
        //向request域中共享数据
        session.setAttribute("hello", "world");
        return "视图名称";
    }
```



## 九、SpringMVC的视图


1. SpringMVC视图是View接口，视图的作用是渲染数据，将Model中的数据展示给用户
2. SpringMVC视图的种类有很多，默认有“转发视图”、“重定向视图”。



### 9.1、ThymeleafView(转发常用)


当控制器方法中所设置的视图名称**没有任何前缀**时，此时视图名称会被SpringMVC核心配置文件中所配置的视图解析器解析，视图名称拼接视图前缀、视图后缀所得的最终路径，再通过**转发**进行页面跳转。无法转发请求：请求也会被ThymeleafResolver解析，会得到一个错误的地址。



### 9.2、转发视图(不常用，Thymeleaf也是转发的形式进行页面跳转)


+  SpringMVC的默认转发视图：InternalResourceView 
+  当控制器方法中所设置的视图名称**以“forward”为前缀**时，创建InternalResourceView视图，此时视图名称**不会**被SpringMVC核心配置文件中所配置的视图解析器解析，而是将前缀“forward:”去除，剩余部分作为最终路径，再通过**转发请求**进行页面跳转。无法直接进行页面跳转。eg："forward:/请求地址"，利用处理该请求地址的Controller进行页面跳转。 



### 9.2、重定向视图


+  SpringMVC的默认转发视图：RedirectView 
+  当控制器方法中所设置的视图名称**以“redirect”为前缀**时，创建RedirectView视图，视图名称**不会**被SpringMVC核心配置文件中所配置的视图解析器解析，而是将前缀“redirect:”去除，剩余部分作为最终路径，再通过**重定向请求**进行页面跳转。重定向无法访问WEB-INF下的目录，所以一般重定向也是重定向请求，通过请求进行页面跳转。eg："redirect:/请求地址"，浏览器以此请求地址再此发送请求，利用处理该请求地址的Controller进行页面跳转。 



## 十、视图控制器


在控制器方法中没有其他任何的请求处理，即只有本身的请求处理时，可以使用视图控制器来简化Controller。



在SpringMVC的核心配置文件中添加两个标签：



1. 简化Controller：<mvc:view-controller path="请求路径" view-name="视图名称"></mvc:view-controller>
2. 开启MVC的注解驱动(还有其他功能)：<mvc:annotation-driven/>



当我们使用了标签：mvc:view-controller，控制器中的所有请求映射(RequestMaping)将全部失效。需要单标签：mvc:annotation-driven来开启MVC的注解驱动，使控制器中的所有请求映射有效。



## 十一、RESTFul


Representational State Transfer：表现层(视图页面、控制器)资源状态转移，是一种软件架构风格



1. 资源：以名词为核心来组织，一个资源可以有一个或多个URL来标识。URL既是资源的名称，也是资源在Web上的地址。对某个资源感兴趣的客户端可以通过资源的URL来与其进行交互。
2. 状态：资源的表现形式，html、jsp、xml、JSON、图片等等。
3. 状态转移：资源在客户端和服务器端之间转移。



### RESTFul的实现


在HTTP协议中，有四种操作方式(请求方式)：GET(获取资源)、POST(添加资源)、PUT(更新资源)、DELETE(删除资源)。在RESTFul中，操作资源的方式是由**请求方式**来决定的，请求方式不同，操作资源的方式也不同。



RESTFul提倡URL地址使用统一的风格。从前到后各个单词使用“/”分隔，不使用“?name=value”的形式来携带请求参数，而是将要发送给服务器的数据作为URL地址的一部分，以保证风格的一致性。



一般请求是GET请求、POST请求，无法发送PUT请求、DELETE请求。需要在web.xml中配置一个filter：HiddenHttpMethodFilter



```xml
    <!-- 配置HiddenHttpMethodFilter -->
    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```



```java
/**
 * 使用RESTFul模拟user资源的增删改查
 * /user        GET     查询所有用户信息
 * /user/{id}   GET     查询指定id用户信息
 * /user        POST    添加用户信息
 * /user        PUT     更新用户信息
 * /user/{id}   DELETE  删除用户信息
 */
@Controller
public class UserController {
    @RequestMapping(value = "/user", method = RequestMethod.GET)
    public String queryAllUser() {
        System.out.println("查询所有user信息");
        return "success";
    }

    @RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
    public String queryUserById(@PathVariable("id") Integer id) {
        System.out.println("根据id=" + id + "查询user信息");
        return "success";
    }

    @RequestMapping(value = "/user", method = RequestMethod.POST)
    public String insertUser(String username, String password) {
        System.out.printf("添加user信息：username=%s password=%s", username, password);
        return "success";
    }

    @RequestMapping(value = "/user", method = RequestMethod.PUT)
    public String updateUser(String username, String password) {
        System.out.printf("修改user信息：username=%s password=%s", username, password);
        return "success";
    }

    @RequestMapping(value = "/user/{id}", method = RequestMethod.DELETE)
    public String updateUser(@PathVariable("id")Integer id) {
        System.out.printf("删除id=%d的user的信息",id);
        return "success";
    }
}
```



```html
    <a th:href="@{/user}">GET请求查询所有uesr信息</a><br>
    <a th:href="@{/user/666}">GET请求根据id查询uesr信息</a><br>
    <!-- 添加用户信息 -->
    <form th:action="@{/user}" method="post">
        用户名：<input type="text" name="username"><br>
        密码：<input type="password" name="password"><br>
        <input type="submit" value="添加用户信息">
    </form>
    <!-- 修改用户信息 -->
    <form th:action="@{/user}" method="post">
        <input type="hidden" name="_method" value="PUT">
        用户名：<input type="text" name="username"><br>
        密码：<input type="password" name="password"><br>
        <input type="submit" value="修改用户信息">
    </form>
      <!-- 删除用户信息 -->
    <form th:action="@{/user/666}" method="post">
        <input type="hidden" name="_method" value="DELETE">
    </form>
```



### 开放静态资源的访问


DispatcherServlet无法访问静态资源，需要使用DefaultServlet(默认Servlet)来访问。  
当没有找到请求对应的请求映射时，请求就会被DisfaultServlet处理。都处理不了，就报404。



在SpringMVC.xml中配置标签：<mvc:default-servlet-handler/>。  
此标签需要与标签：<mvc:annotation-driven/>(开启SpringMVC注解驱动)一起使用。否则，所有请求都会由DefaultServlet处理。这会导致无法访问请求映射。



## 十二、HttpMessageConverter


报文信息转换器，将请求报文转为Java对象或将Java对象转为响应报文。  
HttpMessageConverter提供了两个注解和两个类型：@RequestBody、@ResponseBody、RuquestEntity、ResponseEntity。



### 12.1、@RequestBody(不常用)


将请求报文中的请求体转为Java对象。  
可以获取请求体，需要在控制器方法中设置一个形参，并使用@RequestBody标识，那么当前请求的请求体就会为此注解所标识的形参赋值。



### 12.2、RuquestEntity<泛型>(不常用)


请求实体：接受请求头、请求体，用于封装整个请求报文的类。  
需要在控制器方法中设置一个该类型的形参，那么当前请求的请求报文就会赋值给该形参。



+ getHeaders()：获取请求头信息
+ getBody()：获取请求体信息



### 12.3、@ResponseBody(常用)


将Java对象转为响应报文中的响应体。  
用于标识**控制器方法**，可以将方法的**返回值**作为响应报文的响应体响应给浏览器



由于在微服务中@ResponseBody注解使用非常多，所以产生了复合注解：@RestController，此注解用于标识一个类为Controller，并且为每一个控制器方法添加@ResponseBody注解。



#### 12.3.1、JSON


json是JavaScript中的一种数据交互格式(xml也是)



```xml
    <!-- json所需的依赖 -->
    <!-- jackson-databind -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.13.1</version>
    </dependency>
```



json的类型：



1. json对象：{name : value}
2. json数组：[一个一个的数据]



### 12.4、ResponseEntity(常用)


响应实体：接受响应头、响应体。用于控制器的**返回值类型**，使得用户可以自定义响应报文。



#### 12.4.1、文件上传和下载


1.  上传文件所需的依赖： 

```xml
     <!-- commons-fileupload -->
     <dependency>
         <groupId>commons-fileupload</groupId>
         <artifactId>commons-fileupload</artifactId>
         <version>1.4</version>
     </dependency>
```

 



## 十三、SpringMVC的拦截器


拦截器类似与过滤器，不同的是：拦截器是在控制器方法的前后起作用的。



拦截器的三个方法：



1. boolean preHandle：控制器方法执行前执行。此方法是拦截请求的方法，此方法的返回值决定了是否放行请求，true放行、false拦截。
2. void postHandle：控制器方法执行后执行。
3. void afterCompletion：渲染视图完毕后执行。



配置拦截器：



```xml
    <!-- 配置拦截器，有三种方式，使用其中一种就行，此处就写了两种 -->
    <mvc:interceptors>
        <!-- 第一种配置方式 bean标签表示当前类的对象就是一个拦截器，所有请求都会被拦截，因为没设置拦截规则 -->
        <bean class="com.ashes.mvc.interceptor.MyInterceptor"></bean>
        <!-- 第二种配置方式 -->
        <mvc:interceptor>
            <!-- 拦截请求路径，/**会拦截所有请求，而不是/* -->
            <mvc:mapping path="拦截路径"/>
            <!-- 放行请求路径 -->
            <mvc:exclude-mapping path="放行路径"/>
            <!-- 配置具体拦截器对象 -->
            <bean class="com.ashes.mvc.interceptor.MyInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```



## 十四、异常处理器


### 14.1、基于配置的异常处理


在SpringMVC.xml中配置异常处理：



```xml
    <!-- 配置异常处理 -->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <prop key="所设置的出现的异常的全类名">视图名称</prop>
            </props>
        </property>
        <!-- 设置一个键值对，在请求域中存储异常信息 -->
        <property name="exceptionAttribute" value="请求域的键"></property>
    </bean>
```



### 14.2、基于注解的异常处理


1. @ControllerAdvice：将类标识为异常处理的组件
2. @ExceptionHandler：将方法标识为异常处理，方法的形参需要是Exception类



两者搭配在一起使用，像@Controller、@RequestMapping一样。



## 十五、注解配置SpringMVC


## 十六、SpringMVC的执行流程


### 16.1、SpringMVC的常用组件


1. Handler：**处理器**，即**Controller中的方法**，由工程师开发。  
作用：对请求进行具体处理，但Handler的执行是由HandlerApater决定的。
2. DispatcherServlet：**前端控制器**，由框架提供。  
作用：统一处理请求和响应，整个流程控制的中心，由前端控制器调用其他组件处理请求。
3. HandlerMapping：**处理器映射器**，由框架提供。  
作用：根据请求的URL、method等信息查询Handler，即**控制器方法**。
4. HandlerApater：**处理器适配器**，由框架提供。  
作用：执行Handler。
5. ViewResolver：**视图解析器**，由框架提供。  
作用：进行视图解析，得到相应的视图。
6. View：**视图**，由框架提供。  
作用：将model数据通过页面展现给用户，页面还是要自己写。



### 16.2、DispatcherServlet的初始化流程


DispatcherServlet就是一个Servlet，天然遵循Servlet的生命周期。



可以自己看源码。



DisparcherServlet的初始化策略：初始化DispatcherServlet的各个组件。eg：springMVC.xml中配置的组件。



### 16.3、DispatcherServlet调用组件处理请求


### 16.4、SpringMVC的执行流程
