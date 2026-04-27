# MyBatisPlus


## 框架结构


<!-- 这是一张图片，ocr 内容为： -->
![](https://img-blog.csdnimg.cn/49ff7211964244e99afe83a7dbdc3085.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBATXJfemhhbmd5ag==,size_20,color_FFFFFF,t_70,g_se,x_16)



## SpringBoot整合MyBatisPlus


### 引入依赖


```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3.4</version>
</dependency>
```



### 添加注解


@MapperScan("Mapper接口所在包的路径")（高版本的springboot好像可以不用添加此注解）



添加注解使得SpringBoot能够扫描到Mapper接口



## 配置MyBatisPlus


### 数据库（yaml格式）


```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/填自己的数据库?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT%2B8
    username: 用户
    password: 密码
```



### MyBatisPlus（yaml格式）


```yaml
#Mapper接口的映射文件的默认路径：/mapper/**/*.xml
mybatis-plus:
  configuration:
  	#mybatis-plus 日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```



## BaseMapper接口


MyBatisPlus有一个 **BaseMapper接口** ，这里面有针对 **单表** 的各种基础CRUD，所以在创建 **Mapper接口** 时需要 **extends BaseMapper** ，泛型T表示Mapper类操作的实体类。



当我们使用Mapper接口操作数据库时，不需要指定实体类与数据库表字段的映射关系，甚至不用指定实体类映射哪个数据库表！！！要弄明白这些原理，我们需要深入BaseMapper的底层源码！



## 常用注解


+  @TableName("表名")	在实体类类型上添加@TableName("表名")，标识实体类映射的数据库表 
    - 一般数据库表都有一个固定的前缀，全部用注解太麻烦，可以在配置mybatisplus时设置表前缀
+  [@TableId	](/TableId	) MyBatisPlus默认表中字段名=id的字段为主键，当主键名不为id时，可以在实体类中的某一属性上添加@TableId注解，将其标识为表主键。  



```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  #全局配置
  global-config:
    #数据库配置
    db-config:
      table-prefix: 表前缀
      #统一的主键生成策略
      id-type: auto
```



+  @TableField("字段名")	在实体类属性上使用，设置属性所对应的字段名 
    -  若实体类中的属性使用的是驼峰命名风格，而表中的字段使用的是下划线命名风格  
例如实体类属性userName，表中字段user_name  
此时MyBatis-Plus会自动将下划线命名风格转化为驼峰命名风格 



## 基础CRUD


### 插入insert


### 删除delete


仅列举几个常用的删除方法



1. deleteById(Serializable id)	根据 ID 删除
2. deleteByMap(Map<String, Object> columnMap)	根据 columnMap条件 删除，key对应字段名，value对应条件值，key与value构成一个完整的条件，map就是条件集合，通过map实现多条件删除，条件用 **AND** 连接。
3. deleteBatchIds(Collection<? extends Serializable> idList)	根据 ID 批量删除



### 修改update


修改方法就这两个



1. updateById(T entity)	根据 实体类ID 修改
2. update(T entity,  Wrapper updateWrapper)	根据 whereEntity条件 修改



### 查询select


仅列举几个常用的查询方法



1. selectById(Serializable id)	根据 ID 查询
2. selectBatchIds(Collection<? extends Serializable> idList)	根据ID 批量查询
3. selectByMap(Map<String, Object> columnMap)	根据 columnMap 条件



## 分页插件


### 添加配置类


```java
@Component
@MapperScan("com.bug.dao.mapper")
public class MybatisPlusConfig {

    /**
     * mybatis-plus的sql拦截器，用于配置mybatis-plus的分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();

        //添加分页内部拦截器并设置数据库类型（数据库类型不同，分页sql语句也不同）
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));

        return interceptor;
    }
}
```



### 测试类


```java
@Test
public void test() {
    Page<SysUser> page = new Page<>(2, 1);

    //查询出来的分页数据就在page中
    sysUserMapper.selectPage(page, null);
    System.out.println("page = " + page);

    //获取分页对象中的所有数据
    List<SysUser> userList = page.getRecords();
    for (SysUser sysUser : userList) {
        System.out.println("sysUser = " + sysUser);
    }

    //还有诸多方法，在此不再演示
}
```



### 自定义分页功能


仿照mybatis-plus的selectPage方法，来编写自定义分页查询方法



注意：mybatis-plus提供的Page对象必须位于分页查方法的参数列表中的**第一位**



## 乐观锁


### 配置类中配置乐观锁


```java
@Component
@MapperScan("com.bug.dao.mapper")
public class MybatisPlusConfig {

    /**
     * mybatis-plus的sql拦截器，用于配置mybatis-plus的分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();

        //添加分页内部拦截器并设置数据库类型（数据库类型不同，分页sql语句也不同）
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        //添加乐观锁内部拦截器
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());

        return interceptor;
    }
}
```



### 实体类属性添加@Version注解


[@Version ](/Version )  标识属性为实体的乐观锁版本号 



## 通用枚举


例如数据库表的sex字段，只有男、女、未知三种，就可以使用mybatis-plus的通用枚举（@EnumValue）来实现。



[@EnumValue ](/EnumValue )  标识属性，其值为将来存储到数据库中的某一字段的值。   
通常sex枚举类会有两个属性：性别代号，性别；数据库一般设置sex字段时数据类型为tinyint或int，所以一般将sex枚举类中的性别代号存储到数据库。



配置mybatis-plus扫描通用枚举类所在的包：**type-enums-package:通用枚举类的包路径**



## 代码生成器
