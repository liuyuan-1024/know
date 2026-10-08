# MySQL的基本语句


## MySQL存储引擎


MySQL数据库提供了多种存储引擎，用户可以根据不同的需求为数据表选择不同的存储引擎，用户也可以根据自己的需要编写自己的存储引擎，**MySQL的核心就是存储引擎**。



数据库的存储引擎决定了表在计算机中的存储方式。**存储引擎就是如何存储数据、如何为存储的数据建立索引和如何更新、查询数据等技术的实现方法**。因为在关系数据库中数据的存储是以表的形式存储的，所以存储引擎简而言之就是指表的类型。



**show engines;**，即可查看当前MySQL服务实例支持的存储引擎。



MySQL 5.7支持的存储引擎有：InnoDB、MRG_MyISAM、Memory、BLACKHOLE、MyISAM、CSV、Archive、Federated和PERFORMANCE schema等9种。不同存储引擎都有各自的特点，以适应不同的需求。

| 功  能 | InnoDB | MyISAM | Memory |
| --- | --- | --- | --- |
| 存储限制 | 64TB | 256TB | RAM |
| 支持事务 | 支持 | 无 | 无 |
| 空间使用 | 高 | 低 | 低 |
| 内存使用 | 高 | 低 | 高 |
| 支持数据缓存 | 支持 | 无 | 无 |
| 插入数据速度 | 低 | 高 | 高 |
| 支持外键 | 支持 | 无 | 无 |




MySQL的默认存储引擎是InnoDB，如果想设置其他存储引擎，可以使用MySQL命令：**set  default_storage_engine=MyISAM;**



MySQL 5.7选择InnoDB作为默认存储引擎。InnoDB是事务型数据库的首选引擎，是具有提交、回滚和崩溃恢复能力的事务安全存储引擎，支持行锁定和外键约束。



## 设置字符集为UTF8


1. 设置**客户端**字符集：set character_set_server = utf8;
2. 设置**客户端**字符集：set character_set_client = utf8;
3. 设置**连接**字符集：set character_set_connection = utf8;
4. 设置**数据库**字符集：set character_set_database = utf8;
5. 设置**结果集**字符集：set character_set_results = utf8;



## 一、有关database的操作
| 命令 | 作用 |
| --- | --- |
| create database `数据库`<br/> ; | 创建一个默认字符集的数据库 |
| create database if not exists `数据库`<br/> default charset utf8 collate utf8_general_ci; | 创建一个不存在的数据库并指定字符集 |
| drop database `数据库`<br/>; | 删除数据库 |
| drop database if exists `数据库`<br/>; | 如果数据库存在就删除 |




## 二、有关table的操作
| 命令 | 作用 |
| --- | --- |
| create table `表`<br/> values(column_name column_type 约束, [column_name column_type 约束,] column_name column_type 约束)engine=InnoDB default charset=utf8; | 创建一个具有指定字段的数据表并指定存储引擎和字符集 |
| drop table if exists `表` | 如果该表存在，就删除 |
| alter table `表`<br/> add [column] 列 | 新增列（column可省） |
| alter table `表`<br/> drop [column] 列 | 删除列（column可省） |
| alter table `表`<br/> modify `列`<br/> 类型[+约束]; | 修改列类型（和约束） |
| alter table 表名 add constraint 约束名 约束类型 (字段名) | 添加约束 |




如果完整性约束条件涉及该表的多个属性列，则必须定义在表级上，其他情况则既可以定义在列级上也可以定义在表级上。 这些约束条件主要包括not null、primary key、unique（唯一性约束）、（以及check（检查约束）。

| 约束条件 | 作用 |
| --- | --- |
| not null（非空约束） | 字段不能为空，为空报错 |
| unique（外键参照完整性约束） | 设置字段唯一，不能重复 |
| auto_increment | 字段自增，一般用于int类型的字段 |
| primary key（主键约束） | 用于字段后的约束，只能指定一个单独的主键 |
| primary key (列, 列...)（主键约束） | 用于创建table的末尾，可以指定一个组合主键 |
| foreign key(字段) references 主表(字段)（外键参照完整性约束） | 建表阶段，用于字段约束，设置字段的外键 |




外键约束：constraint foreign_key_name  foreign key (col_name1 [,col_name2….]) references table_name(col_name1[,col_name2...)])



删除表的主键：alter table 表名 drop primary key



删除表的外键约束时，需指定外键约束名称，**alter table 表名 drop foreign key 约束名**（如果没有定义外键名可以用 show create table <表名>查看外键名）



删除表字段的唯一性约束，实际上只需删除该字段的唯一性索引即可，**show keys  from 表名；alter table 表名 drop index 约束名**



修改表名：alter table 旧表名 rename  to 新表名



## SQL基本的执行方法（两种）：


+ . 文件路径
+ source 文件路径



## 三、表的增删改查


1.  增（insert） 

| 增（insert） | 增加表数据 |
| --- | --- |
| insert into `表`<br/> * values(v1,v2,v3...); | 指定全字段的值并添加此记录 |
| insert into `表`<br/> (field1,field2,field3...) values(v1,v2,v3...); | 为指定字段赋值并添加此纪录 |
| insert into `表`<br/> (field1,field2,field3...) values(v1,v2,v3...), values(v1,v2,v3...)...; | 一次插入多条记录 |


2.  
2.  删（delete） 

| 删（delete） | 删除表数据 |
| --- | --- |
| delete from `表` | 删除表中全部数据 |
| delete from `表`<br/> where 条件 | 删除表中满足条件的记录 |


3.  
3.  改（update） 

| 改（update） | 修改表数据 |
| --- | --- |
| update `表`<br/> set field1=newV1, field2=newV2...; | 修改表中全部数据的指定字段 |
| update `表`<br/> set field1=newV1, field2=newV2... where 条件; | 修改表中满足条件的记录的指定字段 |


4.  
4.  查（select） 

| 查（select） | 查询表数据 |
| --- | --- |
| select * from `table` | 查询表中全部数据的全部字段 |
| select * from `table`<br/> where 条件 | 查询表中满足条件记录的全部字段 |
| select (field1,field2...) from `table` | 查询表中全部数据的指定字段 |
| select field1,field2... from `table`<br/> where 条件 | 查询表中满足条件记录的指定字段 |
| select * from `table`<br/> limit M N | 跳过前M条记录，并查询N条记录 |
| select * from `table`<br/> limit N offset M | 跳过前M条记录，并查询N条记录 |
| select * from 表 [group  by  <字段>[having<条件表达式>]] | 查询数据并按字段排序 |
| select * from 表 [order  by  <字段>] [ asc/desc ] | 查询数据并按asc/desc排序 |


5.  



## 四、where 条件 的编写


1.  like模糊查询  
like 'xx%xx'	%代表任意数量的字符，其他字符都是严格匹配的 



## 五、多表查询


### 1、内连接查询


表1 inner join 表2 on 连接条件 where 条件，inner可省略



### 2、外连接查询


只能连接两个表



1. 左外连接：左表 left outer join 右表 on 连接条件，outer可省，返回满足条件的记录以及左表的全部记录
2. 右外连接：左表 right outer join 右表 on 连接条件，outer可省，返回满足条件的记录以及右表的全部记录
3. 全外连接：mysql中并没有全外连接，但可以使用**union**关键字实现类似全外连接的作用，返回满足条件的记录及左右表的全部记录



### 3、交叉连接


表1 cross join 表2，不能使用where关键字，相当于广义笛卡尔积，即表1中每一行与表2中每一行连接。



### 4、嵌套连接


在where语句、having语句中嵌套查询语句；外层查询为父查询，内层查询为子查询；内层查询独立存在，先执行内层查询，再执行外层查询。



+ where xxx in (子查询语句)，in表示在某一范围内
+ where xxx = (子查询语句)，=表示确切为某一值，一般用于子查询只有一条记录时。
+ where xxx 比较运算符 any/all (子查询语句)，any表示任一，all表示所有。
+ where exists (子查询语句)，带有exists的子查询会返回逻辑真或逻辑假，故子查询一般直接查询*，用列名无实际意义；这个嵌套连接是先执行外层查询，但是否取出来，取决于子查询语句的返回值是否为真。
+ where not exists (子查询语句)



## 六、索引


合理的索引可以极大提高数据信息的查询速度和应用程序的性能。MySQL在查询数据表时会默认扫描整个数据表，当数据表的数据量非常大时，查询速度就会很慢。这时使用索引就不会查询整个数据表，而是查询索引，从而提高查询速度。索引只是提供一种高速查询数据表的方法。



### 索引的优点
| 优点 | 解释 |
| --- | --- |
| 加速数据检索 | 索引能够以一列或多列为基础实现快速查询找数据行 |
| 优化查询 | 查询优化器是依赖索引起作用的，索引能加速连接、排序、分组等操作 |
| 强制实施记录的唯一性 | 通过给列添加唯一索引，可以保证数据表中数据的不重复 |




索引并不是越多越好，索引会占据一定的存储空间



### 索引的分类
| 分类 | 解释 |
| --- | --- |
| 普通索引（index） | 基本索引类型，允许列中插入重复值和空值 |
| 唯一索引（unique） | 列值必须唯一，允许有空值；若是组合索引，组合必须唯一；一个表个有多个unique索引 |
| 主键（primary） | 不允许有空值，一个表只能有一个主键索引 |
| 全文索引（fulltext） | 列支持值的全文查找，允许该列插入重复值、空值；该索引只对char、varchar、text类型列创建，MySQL中只有MyISAM存储引擎支持全文索引 |
| 空间索引（spatial） | |




### 设置索引的原则


+  一个表创建大量索引会影响insert、update、delete语句的性能；故针对需要经常更新的数据表，应避免创建大量索引。 
+  在包含大量重复值的列上创建索引，查询时间会较长。 
+  经常需要排序、分组、联合操作的列一定要创建索引，即用于join、where、order by的列要创建索引。 



### 创建索引的方式


+ 建表时，在表的约束条件中添加索引：index/fulltext/unique/spatial 索引名 (字段名 [长度] [asc | desc])
+ 建表后： 
    - create index/fulltext/unique/spatial 索引名 on 表 (字段名 [长度] [asc | desc])
    - alter table 表 add  index/fulltext/unique/spatial 索引名 (字段名 [长度] [asc | desc])

