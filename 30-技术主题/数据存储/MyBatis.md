# MyBatis


+ MyBatis是一款优秀的**持久层(Dao)框架**，它支持自定义SQL、存储过程以及高级映射。MyBatis免除了几乎所有的JDBC代码以及设置参数和获取结果集的工作。MyBatis可以通过简单的XML或注解来配置和映射原始类型、接口和Java POJO（Plain Old Java Objects，普通老式Java对象）为数据库中的记录。
+ MyBatis增强了JDBC的能力，降低了了JDBC的使用复杂度。



## 一、MyBatis提供的功能


1. 创建Connection、Statement、ResultSet对象，无需开发人员手动创建。
2. 执行SQL语句，无需开发人员手动执行。
3. 循环SQL语句，将记录转为Java对象、List集合。
4. 关闭数据库资源。无需开发人员手动关闭。



+  开发人员需要提供SQL语句。 
+  开发人员提供SQL语句 --> MyBatis执行SQL语句 --> MyBatis返回Java对象、List集合给开发人员。 



## 二、搭建MyBatis的开发环境


使用maven直接引入MyBatis的依赖即可：



```xml
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>x.x.x</version>
    </dependency>
```



## 三、MyBatis的几个主要的类、接口


1. Resources类：使用主配置文件的**类路径**作为参数，读取MyBatis的主配置文件。
2. SqlSessionFactoryBuilder类：  
作用：获取SqlSessionFactory类实例：build(主配置文件的输入流)
3. SqlSessionFactory接口：重量级对象（创建此类对象耗时、耗资源，整个项目中有一个就够用）  
作用：获取SqlSession类实例：openSession()；openSession(boolean var1)，true则创建一个能够自动提交事务的SqlSession对象。
4. SqlSession接口：定义了操作数据库的各个重载方法，线程不安全，需要在方法内部使用，记得关闭SqlSession对象。  
作用：执行SQL语句，返回执行结果。



## 四、基于xml文件的MyBatis的使用步骤


1. 搭建搭建MyBatis、Mysql的开发环境。
2. 创建对应数据表中的记录的实体类；创建持久层的Dao接口，用于定义操作数据库的方法。
3. 创建MyBatis的配置文件：sqlmapper文件（xml文件），用于写SQL语句，一般一个表对应一个sqlmapper文件。
4. 创建MyBatis的主配置文件：用于提供数据库的连接信息、sqlmapper文件的位置信息。一个项目一个主配置文件。



### 4.1、MyBatis主配置文件（xml文件）


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="database.properties"/>

    <settings>
        <!-- 控制日志；日志打印到控制台 -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <!-- 配置多个数据库的连接信息（数据库环境）。default：默认使用哪个数据库环境-->
    <environments default="development">
        <!-- 配置一个数据库的连接信息。id：数据库环境的唯一标识 -->
        <environment id="development">
            <!-- MyBatis的事务类型：JDBC -->
            <transactionManager type="JDBC"/>
            <!-- 表示数据源，用于连接数据库；type：表示数据源的类，POOLED：表示使用连接池 -->
            <dataSource type="POOLED">
                <!-- 配置数据库的具体信息。 -->
                <property name="driver" value="${driverClassName}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 全部sqlmapper文件的位置 -->
    <mappers>
        <!-- 单个sqlmapper文件的位置，为sqlmapper文件的classpath -->
        <mapper resource="GoodsDao.xml"/>
        <!-- 
            多个sqlmapper文件需要导入时，可以使用package标签
            条件：sqlmapper文件名需要与接口名相同且在同级目录中
         -->
        <package name="sqlmapper文件所在包的包名"/>
    </mappers>
</configuration>
```



### 4.2、MyBatis的sqlmapper文件（xml文件）


一般建在与Dao接口同级的位置，名称页与Dao接口保持一致。



sqlmapper文件：



```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!-- mybatis-3-mapper.dtd：约束文件，检查、限制当前文件中出现的标签、属性必须符合MyBatis的规范 -->
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!-- 当前文件的根标签。“namespace”属性一般使用Dao接口的全限定名称 -->
    <mapper namespace="com.bug.dao.GoodsDao">
        <!-- mapper中写一些特定标签，表示数据库的特定操作：CRUD -->
        <select id="queryAllGoods" resultType="com.bug.bean.Goods">
            select * from `goods`
        </selec t>
        <insert id="insertGoods">
            insert into `goods` values (null, #{name}, #{price}, #{image}, #{stock}, #{nums})
        </insert>
    </mapper>
```



+  使用特定标签表示数据库的特定操作：CRUD。  
<select>：查询  
1. id：将要执行的SQL语句的唯一标识，MyBatis会根据id来查找待执行SQL语句。一般为Dao接口中的方法名称。  
2. resultType：表示SQL语句执行后得到的结果集中的对象的类型。一般为类的全限定名称。  
<insert>：添加，#{实体属性名}想当于占位符，可以为insert语句中赋值实体的属性值。  
<delete>：删除  
<update>；更新 



### 4.3、MyBatis执行SQL语句


```java
    public static void main(String[] args) throws IOException {
        // 1. 定义MyBatis的主配置文件位置 --> 类路径classpath
        String config = "MyBatis.xml";

        // 2. 读取MyBatis.xml
        InputStream in = Resources.getResourceAsStream(config);

        // 3. 创建SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();

        // 4. builder利用in来获取SqlSessionFactory对象
        SqlSessionFactory factory = builder.build(in);

        // 5. 获取SqlSession对象
        SqlSession sqlSession = factory.openSession();

        // 6. 指定要执行的SQL语句：sqlmapper文件的命名空间 + “.” + 标签的id
        String sql = "com.bug.dao.GoodsDao.queryAllGoods";

        // 7. 执行SQL语句
        List<Goods> list = sqlSession.selectList(sql);

        // 8. 输出执行结果
        for (Goods goods : list) {
            System.out.println(goods);
        }

        // 9. 关闭sqlSession对象
        sqlSession.close();
    }
```



## 五、动态代理


通过调用**SqlSession对象.getMapper(DAO层接口.class)**，MyBatis会在内部创建Dao层接口的实现类并**实例化**，并将sql语句、sql语句的执行、结果集的返回封装到实现类的方法中。我们在进行JDBC操作时，就可以直接调用实现类的方法即可。（就是将DAO层接口的实现由开发人员转移到了MyBatis的内部）



MyBatis的**动态代理**就是为降低代码的冗余度、简化JDBC操作。



代码冗余原因：



1. SqlSession对象的重复获取
2. 待执行SQL语句的id重复度较高（同一个sqlmapper文件中定义的SQL语句）
3. SqlSession对象在使用完后，都要关闭



动态代理的前提条件：



+ 本身sqlmapper文件是与Dao接口毫无关系的，单独使用也能完成CRUD功能；也就是可以将Dao层去除，那么为什么将Dao层保留？基于JDBC的传统实现，开发人员发现保留Dao层接口，可以得到许多sql语句执行的**额外信息**。利用这些额外信息为动态代理的实现带来了希望。  
由此可知动态代理的前提条件：namespace值**必须为Dao层接口的全限制名**，数据库操作标签的id**必须为Dao中对应方法名**。



动态代理原理分析：



+ MyBatis利用反射技术获取Dao层的类的**全限制名**、**方法名**、**方法的返回值**。  
MyBatis内部创建Dao层接口的实现类，将利用额外信息得到的sql语句封装到实现类方法的内部。  
针对方法的不同返回值，MyBatis会为SqlSession对象选择不同的SQL语句执行方法进行封装。



### 5.1、动态代理的使用


SqlSession对象中由方法：getMapper(DAO接口.class)，用于获取DAO接口的实现类对象。DAO接口中不要出现重载方法，MyBatis是通过方法名称区别不同方法的。



```java
    //获取DAO接口的实现类对象
    Dao dao = SqlSession对象.getMapper(Dao.class);
    
    //直接调用实现类对象中的方法，得到sql语句的结果集
    List<Person> personList =  dao.selectAll();
```



## 六、sqlmapper文件中sql语句的传参


### 6.1、sql语句需要传入多个参数


1.  使用注解**@Param("形参名")**：在DAO实现类方法的形参前加上@Param("形参名")；qlmapper文件中的数据库操作标签使用占位符"#{形参别名}"。 
2.  使用Java对象作为形参，对象的属性值作为实参：#{属性名, 属性Java类型, 属性数据库数据类型}，一般简写为：#{属性名}。 



### 6.2、#和$的区别


1. #：占位符，MyBatis会按实际参数进行赋值，并使用PrepareStatement对象执行SQL语句。"#{}"代替了SQL语句中的"?"。效率高、避免SQL注入。
2. $：字符串替换，MyBatis会在"${}"位置处进行字符串的拼接。有SQL注入的风险



## 七、MyBatis封装**操作标签**的输出结果


### 7.1、数据库操作标签的resultType属性


+  结果类型： 
    1. "JavaBean的全限定名"：指SQL语句执行后得到的记录转为何种Java对象。
    2. "简单基本数据类型的全限定名"：也可使用简单基本数据类型在MyBatis中的别名代替全限定名，别名可在MyBatis的官网查看。结果返回简单类型。推荐使用**基本数据类型的全限定名**。
    3. Map<String, Object>：key是数据表的列名，value是该列对应的值。只能返回一条记录。
+  处理方式： 
    1. MyBatis执行SQL语句完毕后，使用类的无参构造器创建类实例。
    2. 通过setXxx()，给与列名同名的属性进行赋值。
+  定义**自定义类**的别名：方便在resultType中使用。一般还是使用全限定名称。 
    1. 在MyBatis的主配置文件中添加标签：typeAliases：用于添加多个类的别名；  
typeAlias标签：添加一个类的别名，属性type：类的全限定名 属性alias：别名

```xml
  <typeAliases>
      <typeAlias type="com.bug.bean.Goods" alias="Goods"/>
  </typeAliases>
```

    1. 在MyBatis的主配置文件中添加标签：package ：用于将一个包下的所有类的类名作为该类的别名。属性name：包路径。



### 7.2、数据库操作标签的resultMap属性


定义sql的执行结果和Java对象属性的映射关系，更加灵活的将列值赋值给属性。常用在**列名和属性名不同**的情况下。



resultMap属性和resultType属性只能二选其一。



+  使用resultMap：  
定义一个resultMap 

```xml
    <!-- 定义列名和Java对象属性的映射关系 -->
    <resultMap id="goodsMap" type="com.bug.bean.Goods">
        <!-- 主键列使用id标签 -->
        <id column="id" property="id"/>
        <!-- 非主键列使用result标签 -->
        <result column="name" property="name"/>
        <result column="price" property="price"/>
        <result column="image" property="image"/>
        <result column="stock" property="stock"/>
        <result column="nums" property="nums"/>
    </resultMap>
```

  
为数据库操作标签中的resultMap属性赋值已定义的resultMap的id，来引用该resultMap。 

```xml
    <select id="queryGoodsMap" resultMap="goodsMap">
        select * from `goods`
    </select>
```

 



### 7.3、模糊查询like


1. 在SQL语句中使用占位符#{}代替模糊查询的查询条件，然后在执行sql语句的Java代码中写入模糊查询条件，作为形参传入接口实现类的方法中。
2. 在SQL语句中拼接："%模糊查询条件%"，拼接格式："%" #{形参名称} "%"，注意在两个%与#{}之间是有空格的，不要忘记。



推荐使用：第一种，使用起来更加方便。



## 八、动态sql


动态sql：sql语句的内容是变化的，根据条件获取不同的sql语句，主要是where部分的不同。



### 8.1、动态sql的实现


MyBatis提供了实现动态sql的标签：if、where、foreach



+  if：条件判断 
    1. 语法格式：  
<if text = "判断条件">  
部分SQL语句  
</if>
    2. 当条件成立时，if标签中的部分sql语句会拼接到数据库操作标签的sql语句中。在拼接sql语句时一定要保证sql语句的正确，所以一般在where后面加上一个必定满足的条件，因为在多条件sql语句中，条件之间有and、or这样的逻辑符。
+  where：包含多个if标签 
    1. 语法格式：  
<where>  
<if text = "判断条件">部分SQL语句</if>  
<if text = "判断条件">部分SQL语句</if>  
</where>
    2. 当有if标签成立时，where标签会自动添加where关键字，并去除多余的and、or关键字，以保证sql语句的语法正确性。
    3. 当没有if标签成立时，where标签将什么都不做。
+  foreach：循环数组、集合，常用于sql语句的 "in(xx, xx, xx ...)" 关键字中 
    1. 语法格式：  
<foreach collection="" item="数组/集合对象" open="" close="" separator="" index="" nullable="">  
#{数组/集合对象} 或者 #{数组/集合对象.属性}  
</foreach>
    2. collection="array或list" item="自定义，代表数组或集合中的每一个对象" open="开头需拼接的字符" close="末尾需拼接的字符" separator="每一个对象间的分隔符" index="" nullable=""



### 8.2、动态sql的代码片段


一段sql语句、字段、表明等等经常被使用，可以将其定义为代码片段，来反复使用，提高了代码复用率。可以定义多个代码片段。



+  语法格式：  
<sql id="代码片段的唯一标识" databaseId="" lang="">  
复用的sql语句  
</sql> 
+  使用方法：  
<include refid="代码片段的id"/> 



## 九、MyBatis的通用分页功能：PageHelper


1.  maven引入PageHelper依赖： 

```xml
    <!-- MyBatis的通用分页插件：PageHelper -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>5.3.0</version>
    </dependency>
```

 

2.  在MyBatis的主配置文件中的**数据库环境配置前**加入plugin配置： 

```xml
     <plugins>
         <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
     </plugins>
```

 

3.  在执行DAO层方法的代码中加入：PageHelper.startPage(第x页,每页x条记录); 

