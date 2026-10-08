# knife4j


增强版的swagger



## Knifej和swagger-bootstrap-ui对比


1. Knife4j在更名之前，原来的名称是叫swagger-bootstrap-ui，这是两种不一样风格的ui显示，将原来的蓝色变成炫酷的黑色模式；
2. Knifej是使用knife4j-spring-boot-starter的风格来编写的，可以将配置项写在配置文件中，这些配置项提供了许多增强功能，可以更好的整合springboot、springcloud；
3. Knifej执行更新，为了更平滑的演进，而swagger-bootstrap-ui已停更；



## 版本分析


1. **Knife4j底层依赖springfox，因此无需再单独引入Springfox的具体版本，且两者有对应的版本要求，否则会产生很多冲突；**
2. **使用Knife4j2.0.6及以上的版本，Spring Boot的版本必须大于等于2.2.x；**



## 访问方式


+ http://{ip}:{port}/doc.html，访问方式和之前的保持一致，如果项目中配置拦截器等，需要放开doc.html静态资源



## springBoot整合knife4j-2x版本


### 引入knife4j依赖


```xml
<!-- knife4j 底层整合了spring-fox 默认访问地址是：http://${host}:${port}/doc.html -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.9</version>
</dependency>
```



### knife4j-2x的配置类


```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2WebMvc;

/**
 * knife4j-2X的配置类
 */
@Configuration
@EnableSwagger2WebMvc
public class Knife4jConfig {

    @Bean(value = "defaultApi")
    public Docket defaultApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .useDefaultResponseMessages(false)
                .apiInfo(apiInfo())
                //分组名称
                .groupName("smallStove")
                .enable(true)
                .select()
                //方式一: 配置扫描 所有想在swagger界面的统一管理接口。都必须在此包下
                .apis(RequestHandlerSelectors.basePackage("com.bug.api"))
                //方式二: 只有当方法上有  @ApiOperation 注解时才能生成对应的接口文档
//                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
//                .apis(RequestHandlerSelectors.withClassAnnotation(RestController.class))
                //所有路径
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("开源小灶API接口文档")
                .description("开源小灶的 RESTFul风格 APIs")
                .termsOfServiceUrl("http://localhost:8080/")
                .contact(new Contact("BugOS-ly", "http://localhost:8080/", "admin@qq.com"))
                .version("1.0")
                .build();
    }
}
```



### 释放knife4j-2x的静态资源


如果配置了拦截器（spring Security、shiro等）需要释放knife4j-2x的静态资源，否则访问doc.html会出现空白页。



#### springSecurity释放knife4j-2x静态资源


```java
// 在springSecurity的配置类中重写一下方法，并释放资源
@Override
public void configure(WebSecurity web) {
    web.ignoring().antMatchers("/swagger/**")
        .antMatchers("/swagger-ui.html")
        .antMatchers("/webjars/**")
        .antMatchers("/v2/**")
        .antMatchers("/v2/api-docs-ext/**")
        .antMatchers("/swagger-resources/**")
        .antMatchers("/doc.html");
}
```

