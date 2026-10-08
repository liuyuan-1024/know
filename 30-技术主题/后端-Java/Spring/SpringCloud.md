# 微服务


+ 服务按照业务来划分，每个服务通常只专注于某一个特定的业务、所需代码量小，复杂度低、易于维护。
+ 每个微服都可以独立开发、部署和运行，且代码量较少，因此启动和运行速度较快。
+ 每个服务从设计、开发、测试到维护所需的团队规模小，一般 8 到 10 人，团队管理成本小。
+ 采用单体架构的应用程序只要有任何修改，就需要重新部署整个应用才能生效，而微服务则完美地解决了这一问题。在微服架构中，某个微服务修改后，只需要重新部署这个服务即可，而不需要重新部署整个应用程序。
+ 在微服务架构中，开发人员可以结合项目业务及团队的特点，合理地选择语言和工具进行开发和部署，不同的微服务可以使用不同的语言和工具。
+ 微服务具备良好的可扩展性。随着业务的不断增加，微服务的体积和代码量都会急剧膨胀，此时我们可以根据业务将微服务再次进行拆分；除此之外，当用户量和并发量的增加时，我们还可以将微服务集群化部署，从而增加系统的负载能力。
+ 微服务能够与容器（Docker）配合使用，实现快速迭代、快速构建、快速部署。
+ 微服务具有良好的故障隔离能力，当应用程序中的某个微服发生故障时，该故障会被隔离在当前服务中，而不会波及到其他微服务造成整个系统的瘫痪。
+ 微服务系统具有链路追踪的能力。



# Java 微服务框架


市面上的 Java 微服务框架主要有以下 5 种：



+ Spring Cloud：它能够基于 REST 服务来构建服务，帮助架构师构建出一套完整的微服务技术生态链。
+ Dropwizard：用于开发高性能和 Restful 的 Web 服务，对配置、应用程序指标、日志记录和操作工具都提供了开箱即用的支持。
+ Restlet： 该框架遵循 RST 架构风格，可以帮助 Java 开发人员构建微服务。
+ Spark：最好的 Java 微服务框架之一，该框架支持通过 Java 8 和 Kotlin 创建微服务架构的应用程序。
+ Dubbo：由阿里巴巴开源的分布式服务治理框架。



# SpringCloud


Spring Cloud 被称为构建分布式微服务系统的“全家桶”，它并不是某一门技术，而是一系列微服务解决方案或框架的有序集合。它将市面上成熟的、经过验证的微服务框架整合起来，并通过 Spring Boot 的思想进行再封装，屏蔽掉其中复杂的配置和实现原理，最终为开发人员提供了一套简单易懂、易部署和易维护的 **分布式系统开发工具包** 。



Spring Cloud 中包含了 spring-cloud-config、spring-cloud-bus 等近 20 个子项目，提供了服务治理、服务网关、智能路由、负载均衡、断路器、监控跟踪、分布式消息队列、配置管理等领域的解决方案。



Spring Cloud 并 **不是一个拿来即可用** 的框架，它是一种 **微服务规范** ，共有以下 2 代实现：



+ 第一代实现：Spring Cloud Netflix
+ 第二代实现：Spring Cloud Alibaba



<!-- 这是一张图片，ocr 内容为： -->
![](assets/SpringCloud%E6%9C%8D%E5%8A%A1.png)<!-- 这是一张图片，ocr 内容为：CLOUD升级 服务总线 服务网关 服务配置 服务降级 服务注册中心 服务调用2 服务调用 CONFIG ZUUL HYSTRIX BUS FEIGN RIBBON EUREKA RESILIENCE4J OPENFEIGN ZUUL2 ZOOKEPER LOADBALANCER NACOS NACOS SENTIENL CONSUL GATEWAY NACOS -->
![](https://cdn.nlark.com/yuque/0/2024/png/43050341/1710504489349-c90303f9-1d43-4676-a7cc-059e06b299a5.png)

## 服务之间的调用(RestTemplate)


RestTemplate提供了多种便捷访问远程HTTP服务的方法, 是一种简单便捷的访问restful服务模板类, 是Spring提供的用于访问Rest服务的客户端模板工具集



### RestTemplate的使用


在模块的配置类中注入RestTemplate实例



```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```



消费者模块依据自身业务逻辑使用RestTemplate实例调用其他服务模块的接口即可



## 服务注册中心(最基础, 最重要)


### Eureka基础知识


什么是服务治理?



> 在传统的rpc远程调用框架中, 管理每个服务与服务之间依赖关系比较复杂, 所以需要使用服务治理, 管理服务与服务之间的关系, 可以实现 **服务调用, 负载均衡/容错, 服务发现与注册**.
>

