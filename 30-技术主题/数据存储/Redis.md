# Redis6


Redis6采用**单线程 + 多路IO复用**



## 一、启动Redis


+  前台启动（不推荐）：在cmd中输入“redis-server”，直接回车。但是redis启动后，这个cmd窗口就无法再关闭以及运行其他东西，否侧redis服务会关闭。 
+  后台启动（推荐）：关闭此cmd窗口，redis依旧会后台运行 
    1. cmd中**"cd /opt/redis-6.0.6"**，进入redis目录，**cp redis.conf /etc/redis.conf**（复制**“redis.conf”**到**/etc/redis.conf**）
    2. 修改**"daemonize no"为yes**（支持后台启动）
    3. **cd /usr/local/bin**，启动redis：**redis-server /etc/redis.conf**，通过**ps -ef | grep redis**查看redis的启动情况
    4. 在设置了Redis密码(redis_ly)后，可以通过**AUTH "root_ly"**验证身份
+  关闭Redis： 
    1. 单实例redis关闭：在 **/usr/local/bin**路径下，cmd中直接键入**shutdown**；也可以客户端连接后（进入redis服务终端后）键入shutdown进行关闭
    2. 多实例redis关闭：直接关闭指定进程



## 二、Redis的常用五大数据类型


### 1、String字符串


+  原子操作：指不会被线程调度机制打断的操作 
+  String类型是一个二进制安全的类型，意味着Redis的string可以包含任何数据，eg：jpg图片、序列化对象等等。 
+  一个Redis字符串的value值最大是512M 
+  字符串的数据结构是**简单动态字符串**，就是可以修改内容的字符串，类似Java中的StringBuffer。采用预分配空间的方式来减少内存的频繁分配 

| 指令（增、改） | 作用 |
| --- | --- |
| set  | 当key不存在时，设置一个键值对；当key存在时，修改key对应的value |
| setnx  | 只有当key不存在时，才会创建键值对 |
| mset   | 一次性创建多个键值对 |
| msetnx   | 只有当key不存在时，才会一次性创建多组键值对 |
| setex  | 设置键值对及其过期时间（秒） |
| getset  | 获取旧值的同时设置新值 |
| append  | 将xxx追加到key的value末尾 |
| setrange  | 将xxx插入到index索引位置 |
| incr/decr  | 将数字型value的值+1或-1，只能对数字值操作 |
| incrby/dectby  | 将数字型value的值+step或-step，只能对数字值操作 |


| 指令（删） | 作用 |
| --- | --- |
| del  | 直接删除key |
| unlink  | 选择非阻塞删除，仅将key从keyspace元数据中删除，后续异步操作会真正删除 |


| 指令（查询） | 作用 |
| --- | --- |
| get  | 获取key对应的value |
| mget   | 一次性获取多个value |
| getrange  | 获取value**[index1, index2]**索引位置上的字符 |
| keys * | 查看当库的所有key |
| dbsize | 查看当前库中有多少个key |
| type  | 查看key的类型 |
| strlen  | 获取value的长度 |
| exists  | 判断某个key是否存在 |


| 指令 | 作用 |
| --- | --- |
| expire  | 设置key的过期时间（key在n秒后过期） |
| ttl  | 查看key什么时候过期，-1代表永不过期，-2代表已过期 |
| select n | 选择index=n的库，一共有16个库，index从0-15 |
| flushdb | 清空当前库的key |
| flushall | 清空所有库的key |




### 2、List集合


+ 单值多键
+ List是简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部（左侧）、尾部（右侧）。
+ 底层数据结构：quickList快速链表（双向链表）。当列表元素较少时，使用zipList压缩列表结构，即使用一块连续内存空间存储元素；当列表元素较多时，使用quickList快速链表结构（通过指针将多个zipList连接起来，既满足了快速插入删除，也不会出现太大的冗余）。

| 指令 | 作用 |
| --- | --- |
| lpush/rpush  | 从左/右侧插入一个或多个值 |
| linsert  | 按照左插入规则（左为前），在value的前或后插入newValue |
| | |
| lpop/rpop  | 从左/右侧删除一个value，值在键在，值亡键亡 |
| rpoplpush  | k1右侧删除一个value，k2左侧插入该value |
| lrem  | 从左侧删除n个指定value |
| | |
| lset  | 将index索引位置value替换成指定value |
| | |
| lindex  | 获取index索引位置的元素 |
| lrange  | 从左侧开始查询**[index1, index2]**索引位置的value，**0 -1 代表查询所有** |
| | |
| llen  | 获取列表的长度 |




### 3、Set集合


+ Set集合是无序无重复集合
+ 底层是dict字典，而字典是value=null的hash表，所以set集合添加、删除、查询的复杂度都是O(1)

| 指令 | 作用 |
| --- | --- |
| sadd  | 添加一个或多个值，已存在值会被自动忽略 |
| | |
| srem  | 删除集合中的指定成员 |
| spop  | 集合中随机移除一个成员 |
| | |
| smembers  | 获取set集合的所有成员 |
| sismember  | 查看集合中是否具有成员v；1有、0没有 |
| srandmember  | 随机取出一组由n个成员组成的序列 |
| scard  | 集合的成员个数 |
| | |
| smove  | 将source中的value移动到destination |
| sinter  | 返回两集合的交集 |
| sunion  | 返回两集合的并集 |
| sdiff  | 返回两集合的差集，k1 - k2 |




### 4、Zset有序集合


+ 类似Set集合(Set集合：无序无重复集合)，Zset是有序无重复集合；Zset集合中的每一个成员都有一个**score(评分)**，依据score的从低到高排序，成员无重复但score可重复。
+ 底层数据结构： 
    1. Redis中的hash：保证value的唯一性，通过value查找score
    2. 跳跃表：为元素排序，通过score的范围获取对应的元素列表

| 指令 | 作用 |
| --- | --- |
| zadd  | 设置集合中的一个或多个value及其score |
| zincrby  | 增加value的score的值 |
| | |
| zrem  | 删除该集合中指定元素 |
| | |
| zrange  [withscores] | 查询集合中**[index1, index2]索引位置的value**，可选是否携带score |
| zrangebyscore  [withscores, limit offset count] | 查询集合中**score处于[min max]的value**，可选是否携带score |
| zrevrangebyscore  [withscores, limit offset count] | 同zrangebyscore，只是将排序改为从大到小 |
| zcount  | 统计集合中score处于[min max]的元素的个数 |
| zrank  | 返会该元素在集合中的排名，从0开始 |
| | |




### 5、Hash哈希


+ Redis中的hash是一个键值对的集合，其中value部分由**filed、value**构成（key对应filed，filed对应value），类似Java中的Map<String, Object>；eg：key = user ，value = [{filed=id，value=1}, {filed=name，value=刘源} .......]；那么**key + filed**可以锁定一个value
+ 底层数据结构：zipList/hashTable，当filed-value长度较短且元素较少时，使用zipList；否则，使用hashTable。

| 指令 | 作用 |
| --- | --- |
| hset  | 设置集合中一个或多个filed的value |
| hmset  | 设置集合中多个filed的value |
| hsetnx  | 当filed不存在时，创建filed并赋值value |
| hincrby  | 使filed的value增加increment(只对数字有效) |
| | |
| hget  | 获取集合中filed字段的value |
| | |
| hexists  | 查看集合中filed是否存在 |
| hkeys  | 查询集合中所有的filed |
| hvals  | 查询集合中所有的value(字段值) |




## 三、Redis的配置文件


1、NotWork部分



主要设置了Redis的一些配置项，



1. **bing: ip** 设置Redis的访问者，不设置就允许所有ip访问
2. **protected-mode yes/no** 是否开启保护模式，在未设置密码时只允许本机访问
3. tcp-backlog 511 tcp-backlog是一个连接队列，tcp-backlog总和=未完成三次握手队列 + 已完成三次握手队列，在高并发下需要一个较大的 tcp-backlog值来避免慢连接
4. timeout 0 超时时间，单位是秒，0代表永不超时
5. tcp-keepalive 300 每个300秒检测一次连接是否还在操作Redis，存活就继续提供服务，否侧就释放连接
6. daemonize yes/no 是否开启后台启动
7. loglevel 日志级别，debug、verbose(冗长的)、默认notice(注意)、warning(警告)
8. logfile “” 设置日志输出文件，默认为空



## 四、Redis的发布和订阅


Redis的发布和订阅（pub/sub）是一种消息通信模式，pub（消息发布者）发送消息，sub（消息订阅者）接收消息。



Redis客户端可以订阅任意数量的频道（pub发送消息的渠道），订阅者只能获取在订阅频道后，发布者发送的消息。

| 发布 | 订阅 |
| --- | --- |
| publish 频道 消息 | subscribe 频道 |




## 五、Redis6的新数据类型


### 1、Bitmaps


+ 本质上不是数据类型，而是字符串，但它可以对字符串的位进行操作
+ Bitmaps单独提供了一套命令来操作字符串的位。可以把Bitmaps想象成一个以位为单位的数组，数组的每个单元都是0或1，**数组的下标在Bitmaps中叫偏移量(offset)**。

| 命令 | 作用 |
| --- | --- |
| setbit  | 设置对应偏移量的value（0或1） |
| getbit  | 获取对应偏移量的value |
| bitcount  [start end] | 统计整个字符串(或[start, end]之间的字符串)为1的bit数量 |
| bitop and/or/not/xor destkey k1 k2 | 对k1、k2进行与(交)、或(并)、非、异或操作，并将结果保存在destkey中 |




### 2、HyperLogLog


基数：集合中不重复元素



基数问题：统计集合中不重复元素的数量



+ HyperLogLog用来做基数统计的算法，优点：在输入元素的数量或体积很大时，计算基数所需的空间总是固定的且很小（只需花费12kb的空间就能计算接近2^64个不同元素的基数）；HyperLogLog只能根据输入元素来计算基数，而不会储存输入元素。

| 指令 | 作用 |
| --- | --- |
| pfadd  | 在HyperLogLog中添加一个或多个指定元素 |
| pfcount | 统计HLL中的基数的数量 |
| pfmerge  | 将一个或多个HLL的元素合并到目标HLL中 |




### 3、GEO（geographic）地理信息


元素的二维坐标，在地图上就是经纬度



两极无法添加；-180 < lon < 180，-85.05112878< lat < 85.05112878；已添加的元素无法再次添加

| 指令 | 作用 |
| --- | --- |
| geoadd  | 添加一个或多个成员的经度、纬度 |
| geopos  | 获取成员的坐标值 |
| geodist  | 获取两成员间的直线距离，单位：米(默认)，千米，英里，英尺 |
| georadius  | 已指定经纬度位中心，找出指定半径内的成员 |




## 六、Jedis操作Redis6


### 1、修改设置，保证其他主机可以连接Redis


	在虚拟机centos上根据ifconfig命令确定虚拟机ip



	注释掉 # bind 127.0.0.1



	关闭保护模式 protected-mode no



	开启守护线程模式 daemonize no



	使用修改好的redis.conf文件启动Redis



	关闭虚拟机的防火墙：systemctl stop firewalld.service；查看虚拟机防火墙状态：systemctl status firewalld.service



### 2、引入Jedis所需依赖


```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.1.1</version>
</dependency>
```



### 3、编写Java代码


```java
//连接某主机某端口的Redis
Jedis jedis = new Jedis("192.168.19.128", 6379);
//Redis身份验证
jedis.auth("redis_ly");
```



## 七、SpringBoot整合Redis


### 1、引入依赖


```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- 操作Redis的连接池 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```



2、在application.properties文件中配置Redis的配置



```properties
#Redis服务器的地址
spring.redis.host=192.168.19.128
#Redis服务器的端口号
spring.redis.port=6379
#使用的Redis库
spring.redis.database=0
#超时时间
spring.redis.timeout=1800000
#连接池最大连接数，负数代表没有限制
spring.redis.lettuce.pool.max-active=20
#最大阻塞时间，负数代表没有限制
spring.redis.lettuce.pool.max-wait=-1
#连接池最大空闲连接数
spring.redis.lettuce.pool.max-idle=5
#连接池最小空闲连接数
spring.redis.lettuce.pool.min-idle=0
```



### 3、controller


```java
@RestController
public class RedisController {
    
    @Autowired
    private RedisTemplate<Object, Object> redisTemplate;

    @GetMapping("/testRedis")
    public String testRedis() {
        ValueOperations<Object, Object> valueOperations = redisTemplate.opsForValue();

        valueOperations.set("name", "liuyuan");

        return (String) valueOperations.get("name");
    }
}
```



## 八、Redis事务


### 1、Redis事务介绍


+ Redis事务：一个单独的隔离操作；事务中的所有命令都被序列化并按顺序执行；Redis执行事务期间，不会被其它客户端发送的命令打断。
+ Redis事务的主要作用：串联多个命令，防止其他命令插队。

| 指令 | 作用 |
| --- | --- |
| multi | 开启Reids事务，之后输入Redis操作指令会存放在队列中，暂不执行 |
| exec | 执行Redis事务队列中的操作，Redis事务结束 |
| discard | 放弃执行事务队列中的操作(要在exec前执行才有效) |




### 2、Redis事务的错误处理


+ 组队过程命令出错：在执行事务队列时，队列中所有命令全部取消
+ 执行过程命令出错：取消报错命令，执行其他正确命令，不会回滚



### 3、事务冲突


产生事务冲突一般由两种解决方案：乐观锁、悲观锁



+ 悲观锁（Pessimistic Lock）：就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，当其他线程想要访问数据时，都需要阻塞挂起（效率低）。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁、表锁，读锁，写锁等，都是在操作之前先上锁。
+ 乐观锁（Optimistic Lock）：就是很乐观，每次去拿数据的时候都认为别人不会修改，所以每次在拿数据的时候都不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用**版本号**等机制。乐观锁适用于**多读**的应用类型，这样可以提高吞吐量。**乐观锁策略：提交版本必须大于记录当前版本才能执行更新**

| 指令 | 作用 |
| --- | --- |
| watch <k...> | 监视一个或多个key |
| unwatch <k...> | 取消监视一个或多个key |




在事务开启(multi)前，使用watch能够监视key，一旦key在事务执行前被修改，事务将被打断。



在事务被执行或放弃时，自动取消监视key。



### 4、Redis事务三特性


1. **单独的隔离操作:**事务中的所有命令都会序列化,按顺序的执行,事务在执行的过程中,不会呗其他客户端发送来的命令请求所打断。
2. **没有隔离级别的概念:**队列中的命令没有提交之前都不会实际被执行,因为事务提交前任何指令都不会被实际执行。
3. **不保证原子性:**事务中如果有一条命令执行失败,其后的命令仍然会被执行,没有回滚。



### 5、并发模拟工具ab工具


虚拟机里安装ab工具插件：yum install httpd-tools



查看ab工具的使用：ab -help



ab工具中的几个重要参数：



1. -n：请求次数
2. -c：并发请求次数
3. -p：post请求的参数发放在postfile这个文件中
4. -T：如果使用POST/PUT请求提交，需要设置参数类型**application/x-www-form-urlencoded**



eg：ab -n 22000 -c 3456 -p postfile文件地址 -T application/x-www-form-urlencoded 请求地址



### 6、商品秒杀案例


+  连接超时问题：Redis无法同时处理更多连接请求，当请求等待时间过长时，就会出现连接超时问题。 
    -  解决：连接池解决连接超时问题，Java编写连接池工具类(网上搜一个) 

```java
public   class   JedisPoolUtil  {
    //被volatile修饰的变量不会被本地线程缓存，对该变量的读写都是直接操作共享内存。 
    private   static   volatile  JedisPool  jedisPool  =  null ;
    
    private  JedisPoolUtil() {} 
    
    public   static  JedisPool getJedisPoolInstance() {
        if ( null  ==  jedisPool ) 
        { 
            synchronized  ( JedisPoolUtil . class ) 
            { 
                if ( null  ==  jedisPool ) 
                { 
                    JedisPoolConfig poolConfig =  new  JedisPoolConfig(); 
                    poolConfig.setMaxActive(1000); 
                    poolConfig.setMaxIdle(32); 
                    poolConfig.setMaxWait(100*1000); 
                    poolConfig.setTestOnBorrow( true ); 
                    jedisPool  =  new  JedisPool(poolConfig, "127.0.0.1" ); 
                } 
            } 
        } 
        return   jedisPool ; 
    } 

    public static void release(JedisPool jedisPool,Jedis jedis) 
    { 
        if ( null  != jedis) 
        { 
            jedisPool.returnResourceObject(jedis); 
        } 
    }
 }
```

 

+  超卖问题 
    - 通过乐观锁为商品库存上锁，防止商品超卖
+  少买问题（库存遗留） 
    - 乐观锁造成库存遗留，因版本号不一致导致后续其他用户秒杀时，程序无法继续执行秒杀步骤
    - LUA脚本解决库存遗留问题，利用**LUA脚本的原子性**弥补了**乐观锁的不保证原子性**



## 九、Redis持久化


Redis持久化：将Redis中的数据写入硬盘中



Redis提供了两种不同形式的持久化方式：RDB、AOF



### 1、RDB


RDB：在指定的时间间隔内将内存中的数据集快照写入硬盘，也就是Snapshot快照，它恢复时是将快照文件直接读取到内存中。



Redis进行RDB持久化会默认将持久化数据存储到**dump.rdb**文件中，dump.rdb文件在Redis启动目录中



+ Redis会创建一个子进程（fork）进行持久化，先将数据写入一个临时文件中，待整个持久化过程结束，再用这个临时文件替换以前的持久化文件。
+ 整个过程中，主线程是不进行任何IO操作的，这保证了极高的性能。
+ 如果需要进行**大规模**的数据恢复，且对数据完成性不是非常敏感，那么RDB是比AOF更加高效的。RDB占据磁盘空间小
+ **RDB缺点就是最后一次持久化的数据可能丢失**。未满足持久化操作的条件（eg：在20秒内操作了5个及以上的key，会自动持久化）



fork：复制一个与当前进程一摸一样的新进程，并作为原进程的子进程。



+ 在Linux系统中，fork会产生一个与父进程完全相同的子进程，但子进程在此后多会exec系统调用，处于效率考虑，Linux引入了**写时复制技术**



rdb文件的备份



+ cp 待备份文件位置 目标文件位置：将文件复制到另一个位置，完成备份



rdb文件的恢复



+ mv 备份文件 待恢复文件：移动文件
+ cp 备份文件 待恢复文件：复制文件



### 2、AOF（Append Only File）


以日志的形式来保存写操作（增量保存）。将Redis执行过的写操作记录下来（读操作不记录），只许追加文件不许改写文件；redis在启动之初会读取该文件内容来重新构建数据。



AOF默认不开启，开启后会将写操作存入**appendonly.aof**中，这个文件在Redis的启动目录下；当AOF和RDB同时开启，Redis默认使用AOF。



+  AOF的开启：配文件中开启AOF 
+  AOF的恢复：重启Redis，重新加载数据 
+  AOF的异常恢复：如果AOF文件损坏，通过**/usr/local/bin/redis-check-aof --fix appendonly.aof** 



#### 2.1、AOF的同步频率


+ appendfsync always  ：每次Redis的写操作都会立即记入日志，性能较差但数据完整性较高。
+ appendfsync everysec  ：每秒记入日志一次，如果宕机，本秒数据可能丢失。
+ appendfsync no  ：不主动进行同步，将同步时机交给操作系统。



#### 2.2、Rewrite压缩


AOF采用了追加文件的方式，为避免文件越来越大，新增了重写机制，当文件大小超过了某个阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用



重写条件：AOF文件大于基准值的一倍，才会重写



#### 2.3、AOF的持久化流程


1. 客户端的写命令会被追加到AOF缓存区中
2. AOF缓存区会根据持久化策略(always everysec no)将写操作同步到磁盘的AOF文件中
3. 当AOF文件大小满足重写条件或手动重写时，Redis就会启动rewrite，压缩AOF文件



## 十、Redis的主从复制


主机数据更新后，根据配置和策略，自动同步到备机的master/slaver机制；master(主人)以写为主，slaver(奴隶)以读为主。一般为一主多从。



好处：



+ 读写分离
+ 容灾的快速恢复：当一台从服务器挂掉了，能够快速地切换到另一台从服务器



### 1、建立一主多从（一台电脑上）


1. 先复制**/etc/redis.conf**文件到myRedis文件夹中，关闭redis.conf的AOF功能
2. 创建多个redis配置文件，一般以redis+端口号命名，eg：redis6379.conf
3. 填写redis6379.conf文件内容， 
    1. **include /myRedis/redis.conf**  引入redis.conf文件内容作为新增redis文件的公有内容
    2. pidfile /var/run/redis6379.pid
    3. port 6379  修改端口
    4. dbfilename dump6379.rdb  rdb文件的名称
4. redis-server /myRedis/redis6379.conf  使用各个配置文件启动不同端口的Redis服务器
5. redis-cli -p 端口号  客户端连接不同端口的服务器
6. **slaveof 主机IP 端口号**  在服务器上执行，使此服务器成为该主机的从机
7. info replication  查看本服务器的主从信息



+ 当从服务器挂掉后，从服务器就会变成主服务器，再次将服务器设为主服务器的从服务器，从服务器会利用主服务器的rdb文件将主服务器的数据全部拷贝过来
+ 当主服务器执行写操作之后，都会与从服务器进行数据同步



### 2、薪火相传


从服务器可以由从服务器，不断迭代



### 3、反客为主


如果主服务器宕机了，从服务器就可以晋升为主服务器，使用**slaveof no one**指令即可。这需要运维人员手动实现，不是十分便捷。后面介绍**哨兵模式**，可实现从服务器的自动化晋升。



### 4、哨兵模式


反客为主的自动版，后台监视主服务器是否故障；如果故障，会根据投票数自动将从库晋升为主库。主机会变成挂掉的从机



1. 自定义的myReids目录中新建**sentinel.conf**文件，名字绝对不能错。
2. **sentinel monitor 自定义被监视主机名称 被监视主机IP 端口号 同意迁移票数下限**， 
    1. 同意迁移票数下限：至少需要多少个哨兵同意，才能迁移主机
3. 启动哨兵：redis-sentinel /myRedis/sentinel.conf



复制延时问题：主机的写操作同步到从机上，会有延迟；系统繁忙时、slacer数量较多时，延迟会更严重。



replica-priority 100：这个数值越小，优先级越高，主机宕机时，从机就优先晋升。



## 十一、Redis的集群


Reids集群实现了对Redis的水平扩容，即启动N个Redis节点，将整个数据库分布存储在N个节点中，每个节点存储总数据的1/N。



Redis集群通过分区（partition）来提供一定程度的可用性（availability）：即使集群中有一部分的节点失效或无法进行通讯，集群也可以继续处理命令请求。



### 1、Redis集群实现


Redis配置文件：



1. cluster-enabled yes  开启集群模式
2. cluster-config-file nodes-6379.conf  设置节点配置文件
3. cluster-node-timeout 15000  设置节点失联时间，单位毫秒，超过该时间，集群自动进行主从切换



进入Redis的安装路径：**opt/redis.6.0.6/src**，在src文件夹下执行：



+ **redis-cli --clusted --cluster -replicas 1 集群的服务器的IP:port** 
    - -replicas 1  代表以最简单的方式配置集群
    - IP地址需要真实的IP地址
+ **redis-cli -c -p 端口号**  使用集群的方式连接Redis服务器
+ **cluster nodes**  查看所有集群节点的状态
+ 每个节点都会由1/N个插槽，插槽用于存储key



向集群节点添加key，Redis会依据某种算法计算处该key将放在哪个节点中；一次性添加多个key时，需要将key分组，Redis会按照组来进行插槽计算，eg：mset k1{组} v1 k2{组} v2.......。

| 指令 | 作用 |
| --- | --- |
| cluster keyslot  | 计算该key所在插槽号 |
| cluster countkeysinslot 4847 | 返回4847号插槽中的key的数量，无法查询到其他节点的插槽中存储的key |
| cluster getkeysinslot 4847 10 | 返回4847号插槽中的10个key |




### 2、集群的Jedis开发


```plain
public class JdeisClusterDemo {
    public static void main(String[] args) {
        HostAndPort hostAndPort = new HostAndPort("124.221.13.90", 6379);

        JedisCluster jedisCluster = new JedisCluster(hostAndPort);

        //一些读写操作
    }
}
```



### 3、Redis集群的不足


1. 不支持多键操作
2. 不支持多键的Redis事务、LUA脚本



## 十二、Redis应用问题解决


### 1、缓存穿透


缓存穿透：用户不断地请求**缓存和数据库中都不存在的数据**，导致每次请求都会转到数据库，从而压垮数据库。



解决方案：



1. 对null值进行短时间缓存：如果一个查询结果为null，对null进行缓存并设置较短的过期时间（最长不超过5 min）。
2. 设置白名单：使用bitmaps定义一个白名单，名单id作为偏移量，访问的id存在于名单中就放行，否侧拦截。
3. 布隆过滤器：对2的优化
4. 实时监视：当发现Redis的命中率急速下降，排查访问对象和访问数据，并与运维人员配合设置黑名单限制访问。



### 2、缓存击穿


缓存击穿：Redis中**一个热点key在失效**的同时，大量的请求过来，从而会全部到达数据库，压垮数据库。



解决方案：



1. 预先设置热门数据
2. 实时调整
3. 使用锁



### 3、缓存雪崩


缓存雪崩：Redis中缓存的数据**大面积同时失效**，或者Redis宕机，从而会导致大量请求直接到数据库，压垮数据库。



解决方案：



1. 构建多级缓存架构：nginx + redis + 其他缓存（encache）
2. 使用锁或队列：使用锁或队列保证不会有大量线程对数据库一次性进行读写，从而避免失效时大量并发请求落到底层存储系统上，不适用高并发情况。
3. 设置过期标志更新缓存：记录缓存数据是否过期（设置提前量），如果过期会，通知其他线程到后台更新实际key的缓存。
4. 将缓存失效时间分散开：为缓存失效时间添加一个随机数，这样缓存失效时间会隔开，很难引发集体失效。



### 4、分布式锁


普通锁一次只能对分布式集群系统的一个节点上锁（针对单机），采用分布式锁可以对所有节点上锁（所有节点共享同一把锁）。



分布式锁的实现：



1. 基于数据库的分布式锁
2. 基于Redis的分布式锁
3. 基于Zookeepr的分布式锁



#### 基于Redis的分布式锁
| 命令 | 作用 |
| --- | --- |
| setnx  | 对此key设置时，就会上锁 |
| expire  | 设置锁的过期时间，到期，锁自动释放 |
| set  nx ex  | 在设置key时就上锁并设置过期时间 |
| del <> | |




分布式锁存在问题：



+  由于网路延迟等问题，可能导致a的锁被b释放，这当然不行。 
    -  解决方案： 
        *  UUID防止误释放：UUID标识不同的操作，在释放锁时，先判断当前UUID与待释放锁的UUID是否一致，一致再释放锁。 

```plain
//获取一个随机的UUID
String uuid = UUID.randomUUID().toString();
//获取锁并设置锁的UUID、过期时间
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 3, TimeUnit.SECONDS);
//释放锁时比较UUID是否一致
```

 

+  删除缺少原子性：a比较完UUID相同后，准备释放锁，此时恰好锁过期，自动释放掉了；锁到了b上，那么a就会释放b的锁 
    - 解决方案： 
        * LUA脚本弥补删除缺少原子性



为确保分布式锁的可用，需要确保同时满足以下四个条件：



1. 互斥性：在同一时间，只有一个客户端能够持有锁。
2. 不会发生死锁：即使有一个客户端在持锁期间崩溃而没有主动解锁，也能保证后续客户端能够加锁。
3. 解铃还须系铃人：加锁和解锁的客户端必须是同一个客户端，不能释放其他客户端的锁。
4. 加锁和解锁必须具有原子性。



## 十三、Redis6的新功能


### 1、ACL（Access  Controll  List）访问控制列表
| 命令 | 作用 |
| --- | --- |
| acl list | 显示用户权限列表 |
| acl whoami | 当前使用的用户 |
| acl setuser <username on/off  > pwd ~操作范围 +操作权限> | 添加用户并设置密码以及相应的权限，*代表全部 |




### 2、IO多线程


Redis执行命令依旧是单线程，多线程是用来处理网络数据的读写和协议的解析。



多线程默认不开启，开启需要在配置文件中添加:



1. io-threads-do-reads yes  开启IO多线程
2. io-threads 4  设置IO线程的数量



### 3、工具支持Cluster


Redis5以前的版本需要单独安装ruby环境，才能使用Redis集群；Redis5以后，官方将ruby集成到了redis-cli。

