## Redis入门

### 概述

> Redis是什么？

Remote Dictionary Server 远程服务字典

是一个基于开源使用ANSI C语言编写，支持网络，可基于内存亦可持久化的日志型，Key-Value数据库，并提供多种语言API。

免费和开源！当下最热门的NoSQL技术之一！也被人们称之为结构化数据库！



> Redis能干吗？

1. 内存存储，持久化（内存断电即失）（rdb，aof）

2. 效率高，可以用于高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器，计数器（浏览器！）

6. 。。。



> 特殊

1. 多样化的数据类型
2. 持久化
3. 集群
4. 事务
5. 。。。



> 资源

1. 官网：https://redis.io/
2. 中文网：http://www.redis.cn



### Linux安装

1. 先下载redis的tar包

2. 环境准备

   ```bash
   yum install gcc-c++
   ```

3. 进入解压后的redis目录

   ```bash
   make #编译（注意redis6.0以后的版本需要gcc5.3以上，centos7默认是4.8.5，需要升级）
   make install
   ```

4. 进入软件主目录

   ```bash
   cd /usr/local/bin
   ```

5. 复制配置文件

   ```bash
   mkdir myconf
   cp /opt/redis-6.0.3/redis.conf myconf
   ```

6. 修改配置文件（使其后台启动）

   ```bash
   daemonize yes
   ```

7. 启动服务（通过指定配置启动）

   ```bash
   redis-server myconf/redis.conf
   ```

8. 连接测试

   ```bash
   redis-cli -p 6379
   ```

9. 关闭

   ```bash
   # 在连接客户端执行
   shutdown #关闭服务
   exit	#退出客户端
   ```



### 基础知识

redis默认有十六个数据库！默认使用第0个

可以使用select进行切换！

```bash
127.0.0.1:6379> select 3	#切换数据库
OK
127.0.0.1:6379[3]> dbsize #查看库大小
(integer) 0
```

```bash
keys *	#查看所有key
flushdb #清空库
flushall #清空所有库
```



## 五大数据类型

### Redis-Key

```bash
set key value	#设置一个键值对
get key			#获取对应键的值
exists key		#判断某个键是否存在
move key 2		#移动某个键值对到另一个库
expire key 10   #该键值对十秒后过期
ttl key			#该键剩余时间
keys *			#当前数据库的所有键
type key		#查看当前键的值的数据类型
```



### String（字符串）

```bash
append key "字符串"		#追加字符串，不存在相当于set
strlen key				  #获取字符串的长度！
incr key				  #加一操作
decr key				  #减一操作
incrby key 10             #加十
decyby key 10			  #减十
getrange key 0 3		  #截取字符串0到2，0到-1查看全部
setrange key 1 xx		  #替换索引1位置及后面一个值，为xx
setex key 30 value		  #设置一个键值对并在30秒后过期 
setnx key value			  #不存在该key，才会设置（分布式锁）
mset k1 v1 k2 v2          #批量创建
mget k1 k2				  #批量获取
msetnx k1 v1 k2 v2        #批量，原子操作，有一个失败，全失败

set user:1 {name:zhangsan,age:3}
#对象，user:{id} {属性}

getset key 新值			#先返回当前值，在设置新值
```



### List

基本的数据类型，列表。

在redis里面，我们可以把list当作栈，队列，阻塞队列使用！

所有的list命令都是以l和r开头的！！！

```bash
lpush list value	#往list左边加入一个值
rpush list value	#在list右边加入一个值
lrange list 0 1     #获取第0个到第1个，最后进的为0
lpop list			#移除左边第一个
rpop list			#移除右边第一个
lindex list 2			#获取下标为2的值
llen list			#返回列表的长度

lrem list 1 one		#移除一个one
lrem list 2 one 	#移除两个one

ltrim list 1 2		#截器1到2，只剩下截取后的元素

rpoplpush list1 list2 #将第一个列表的最后一个值，移动到第二个列表的第一个位置

lset list 0 value	#给下标为0的设置一个值（要存在下标为0的值）

linsert list before value value1	#在value前插入一个value1
linsert list after value value1		#在value后插入一个value1
```



> 小结

* 他实际上是一个链表，before node after，left，right都可以插入值。
* 如果key不存在，创建新的链表
* 如果key存在，新增内容
* 如果移除所有值，空链表，也代表不存在
* 在两边插入或者改动值，效率最高！中间元素，效率相对来说较低！



消息队列！消息排队！栈，队列



### Set

set中的值是不能重复的！！

```bash
sadd list value		#往集合中添加元素
smembers list		#查看集合
sismember list value#查看集合是否有该值

scard list			#获取集合元素个数

srem list value		#移除某个元素

srandmember list 2	#随机获取两个元素，不写2就是一个

spop list			#随机删除一个

smove list1 list2 value #移动指定的元素

sdiff list1 list2	#list1有list2没有的
sinter list1 list2	#list1与list2共同有的
sunion list1 list2	#并集
```





### Hash（哈希）

Map集合，key-map！值是一个map集合！

命令多以h开头！！！

```bash
hset myhash field1 value1					#set一个具体的key-value

hget myhash field1							#获取map中的一个key的value

hmset myhash field1 value1 field2 value2	#设置多个key-value，有则覆盖

hmget myhash field1 field2					#获取多个

hgetall myhash								#获取全部

hdel myhash field1							#删除map中指定的key-value

hlen myhash									#map中有多少key-value

hexists	myhash field1						#判断map中指定的key是否存在

hkeys myhash								#获取map中所有的key

hvals myhash								#获取map中所有的value

hincrby myhash field3 1						#map中对应key的value加一

hsetnx myhash field3 value3					#不存在则可以创建
```



### Zset（有序集合）

在set的基础上，增加了一个值，set k1 v1    zset k1 score1 v1

```bash
zadd myset 1 one
zadd myset 2 two		#第一个位置添加one，第二个two

zrange meyset 0 -1		#遍历，从小到大
zrevrange myset 0 -1    #遍历，从大到小

zrangebyscore myset -inf +inf	#将负无穷到正无穷的数据进行从小到大排序

zrem myset k1			#移除k1

zcard myset				#获取集合中的个数

zcount myset 1 3		#获取指定区间的成员数量
```

案例：

成绩 工资表 消息级别 播放量排行



## 三种特殊数据类型

### geospatial 地理位置

朋友的定位，附件的人，打车距离计算

Redis的Geo在Redis3.2版本就推出了！这个功能可以推算出地理位置的信息，两地之间的距离，方圆几里的人！



只有六个命令！！

> geoadd
>
> 参数：key value（经度 维度 名称）

```bash
# geoadd 添加地理位置
geoadd china:city 116.40 39.90 beijing
geoadd china:city 121.47 31.23 shanghai
geoadd china:city 106.50 29.53 chongqin 114.05 22.52 shenzhen
```



> geopos

```bash
# 获取指定的城市的经度和维度
geopos china:city beijing
geopos china:city beijing chongqing
```



> geodist

```bash
# 绝对距离
# 单位 m km mi ft
geodist china:city beijing shanghai km
```



> georadius 以给定坐标的半径查找

```bash
georadius china:city 110 30 500 km #以110 30 为中心，查找500km以内的数据
georadius china:city 110 30 500 km withdist #显示直线距离
georadius china:city 110 30 500 km withcoord #显示他人的经纬度
georadius china:city 110 30 500 km count 2	#只显示两个
```



> georadiusbymember  以给定名称的半径查找

```bash
georadius china:city beijing 500 km #以北京为中心，查找500km以内的数据
```



> geohash 返回一个或者多个位置的hash表示

```bash
# 讲二维的经纬度转换成一维的字符串
geohash chain:city beijing
```



**底层使用zset实现，可以使用zset的一些命令**



### Hyperloglog

> 什么是基数？

基数就是指一个集合中不同元素的个数！！

例如：A{1,3,5,7,8,7}和B{1,3,5,7,8}的基数都是5，可以接受误差！

Hyperloglog 基数统计算法！



> 场景

**网站的UV（一个人访问一个网站多次，但是只算是一次！）**

传统方式，set保存用户id，然后统计！（用户量过大会占用过大存储空间）

基数统计占用的内存是固定的，2^64不同的元素的技术，只需要12kb内存！

如果从内存角度来比较的话Hyperloglog 首先！0.81%错误率！统计UV任务，可以忽略不计的！



> 使用

```bash
pfadd mykey a b c d e f g h i j				#创建一个基数统计
	
pfcount mykey								#计算基数统计的结果

pfmerge mykey3 mykey mykey2					#mykey3存mykey mykey2的并集
```

不允许容错的话，不建议使用！！！



### Bitmap

> 位存储

统计用户信息，活跃，不活跃！登录，未登录！打卡，未打卡！

两个状态的 ，都可以使用bitmap。

365天 = 365bit 1字节=8bit 46个字节左右！！

```bash
setbit sign 0 1			#在sign的第0个下标的位置存1
setbit sign 1 1			#在sign的第1个下标的位置存1
setbit sign 2 0			#在sign的第2个下标的位置存0

getbit sign 2			#获取sign第2个下标的值

bitcount sign			#统计1的个数
```



## 事务

原子性：要么同时成功，要么全部失败！

==Redis单条命令是保存原子性的，但是事务不保证原子性！==

事务的本质：一组命令的集合！一个事务中的所有命令都会被序列化，在事务执行的过程中，会按照顺序执行！

事务的特性：一次性，顺序性，排他性！

==Redis事务没有隔离级别的概念！==

所有命令都在事务中，并没有直接被执行！只有发起执行命令的时候才会执行！Exec

Redis的事务：

* 开启事务：multi
* 命令入队：。。。
* 执行事务：exec



> 正常执行事务！

```bash
multi				#开启事务
set k1 v1			#入队
set k2 v2			#入队
exec				#执行
```



> 放弃事务

```bash
multi				#开启事务
set k1 v1			#入队
set k2 v2			#入队
discard				#放弃事务
```



> 编译异常（命令有错）：事务中所有命令都不会被执行！

```bash
multi				#开启事务
set k1 v1			#入队
set k2   			#错误命令
exec				#执行
# 所有命令都没有执行
```





> ==运行异常（1/0）：如果事务中存在一条命令失败，不会影响其他命令的执行！没有原子性！==

```bash
multi				#开启事务
set k1 v1			#入队
incr k1				#自增（报错命令）
set k2 v2			#入队
exec				#执行

1) OK
2) (error) ERR value is not an integer or out of range
3) OK
```



> 监控

**悲观锁：**

* 很悲观，什么时候都会出问题，无论做什么都会加锁！（有性能问题）

**乐观锁：**

* 很乐观，任务什么时候都不会出问题，所有不会上锁！更新数据的时候去判断一下，在此期间是否有人修改过这个数据。
* 获取version
* 更新的时候比较version



> Redis监测（配合事务使用）

```bash
set money 100
set out 0
watch money				#监视money

# 正常情况
multi
decrby money 20
incrby out 20
exec

# 错误情况
watch money #获取“version”
multi
decrby money 20
incrby out 20
# 先不要执行
# 在另一个线程修改money的值
# 在执行exec发现失败！


unwatch money			# 放弃监视（解锁）
```



## Jedis

> 什么是Jedis

Redis官方推荐的java连接开发工具！使用Java操作Redis中间件！

1. 导入相应坐标

   ```xml
   <dependencies>
           <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
           <dependency>
               <groupId>redis.clients</groupId>
               <artifactId>jedis</artifactId>
               <version>3.2.0</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
           <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>fastjson</artifactId>
               <version>1.2.62</version>
           </dependency>
       </dependencies>
   ```

2. 编码测试

   ```java
   public class TestPing {
       public static void main(String[] args) {
           Jedis jedis = new Jedis("59.110.137.127", 6379);
           System.out.println(jedis.ping());
       }
   }
   ```

3. 常用Api跟命令类似



> 事务

```bash
public class TestPing {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("59.110.137.127", 6379);
        Transaction multi = jedis.multi();
        try {
            jedis.set("k1","v1");
            jedis.set("k2","v2");
            multi.exec();
        } catch (Exception e) {
            multi.discard();
            e.printStackTrace();
        } finally {
            jedis.close();
        }
    }
}
```



## SpringBoot整合

说明：在SpringBoot2.x 之后，原来使用的jedis被替换为了lettuce

jedis：采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全的，使用jedis pool连接池！BIO

lettuce：采用netty，实例可以再多个线程中共享，不存在线程不安全的情况！可以减少线程数据。NIO



1. 添加起步依赖

2. 查看自动配置类

   ![1590562331477](J:\桌面\文章\文章图片\1590562331477.png)

   ![1590562350566](J:\桌面\文章\文章图片\1590562350566.png)

3. 分析

   ![1590562403237](J:\桌面\文章\文章图片\1590562403237.png)

   第一个是自动配置类，可以查看相关配置信息。

   第二个是注入操作redis的模板对象，由注解可以看出，可以重写。	

   由于2.x之后底层是有Lettuce实现的，所有要序列化！！！（重新模板类）

   默认的序列化使用的是jdk的序列化！



### 自定义RedisTemplate

我们在redis中保存对象，需要在Java中将其序列化成json！！！

80%的人都不会去使用原生的RedisTemplate！！！

一般使用工具类！

```java
@Configuration
public class RedisConfig {
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 我们为了自己开发方便，一般直接使用 <String, Object>
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(factory);
        // Json序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // String 的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}
```





## Redis配置

启动的时候，就通过配置文件来启动！

> 网络

```bash
bind 127.0.0.1			# 绑定的ip，远程连接可以注释掉
protected-mode yes		# 保护模式，默认开启
port 6379				# 端口
```



> 通用

```bash
daemonize yes		# 以守护进程的方式运行，默认为no不开启

pidfile /var/run/redis_6379.pid			# 如果以后台的方式运行，需要指定一个pid文件

loglevel notice		# 日志基本，默认生产环境

logfile	""			# 日志的文件位置

databases 16		# 数据库数量，默认16

always-show-logo yes#是否显示logo
```



> 快照

持久化，在规定的时间内，执行了多少次操作，则会持久化到文件.rdb.aof

redis是内存数据库，没有持久化，那么数据断电即失！

```bash
save 900 1		# 如果900秒内至少有一个key进行了修改，我们就进行持久化
save 300 10		# 如果300秒内至少有十个key进行了修改，我们就进行持久化
save 60 10000   # 如果60秒内至少有一万个key进行了修改，我们就进行持久化


stop-writes-on-bgsave-error yes	# 持久化如果出错，是否需要停止工作

rdbcompression yes				# 是否压缩 rdb 文件，需要消耗一些cpu资源！

rdbchecksum	yes					# 保存 rdb 文件的时候，进行错误的检查校验！

dir ./							# rdb 保存目录
```



> 安全 SECURITY

```BASH
# 使用命令设置密码

config set requirepass "123456"

auth 123456			# 验证密码
```



> 限制 CLIENTS

```bash
maxclients 10000  # 设置能连接上redis的最大客户端的数量

maxmemory <bytes>  # redis 配置最大的内存容量

maxmemory-policy noeviction  # 内存到达上限之后的处理策略
  1、volatile-lru：只对设置了过期时间的key进行LRU（默认值）
  2、allkeys-lru ： 删除lru算法的key 
  3、volatile-random：随机删除即将过期key 
  4、allkeys-random：随机删除 
  5、volatile-ttl ： 删除即将过期的 
  6、noeviction ： 永不过期，返回错误
```



> APPEND ONLY 模式 aof配置

```bash
appendonly no   # 默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分所有的情况下，
rdb完全够用！

appendfilename "appendonly.aof"  # 持久化的文件的名字

# appendfsync always  # 每次修改都会 sync。消耗性能
appendfsync everysec  # 每秒执行一次 sync，可能会丢失这1s的数据！
# appendfsync no    # 不执行 sync，这个时候操作系统自己同步数据，速度最快！
```





## Redis持久化

==重点==

### RDB（Redis DataBase）

在指定的时间间隔内将内存中的数据集写入磁盘，也就是快照，他恢复时是将快照文件直接读到内存里。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。我们默认使用的就是RDB，一般情况下不需要修改这个配置！	

==rdb保存的文件是dump.rdb，生产环境会进行备份==

相关细节都在配置文件中设置！！！



> 触发机制

1. save的规则满足的情况下，会自动触发rdb规则！
2. 执行flushall命令，也会触发rdb规则！
3. 退出redis，也会产生rdb文件



> 如何恢复rdb文件！

1. 只需要将rdb文件放在我们redis配置中的指定目录就可以（默认启动目录），redis启动的时候就会自动检查dump.rdb恢复其中数据！

2. config get dir  #查看需要存在的位置



> 总结

[AOF]( https://www.jianshu.com/p/1e34fdc51e3b )

**优点：**

1. 适合大规模的数据恢复！
2. 如果你对数据完整性要求不高

**缺点：**

1. 需要一定的时间间隔进程操作！如果redis意外宕机了，这个最后一次修改数据就没有了
2. fork进程的时候，会占用一定的内存空间！





### AOF（Append Only File）

以日志的形式来记录每个写的过程，将Redis执行过的所有指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作！

==AOF保存的是appendonly.aof文件==

默认不开启，需要去配置文件中手动配置appen！！！



> 持久化文件损坏

如果aof文件损坏，可能导致一些错误！！（无法建立连接等）

使用：redis-check-aof --fix appendonly.aof

修复！！！（会将错误的命令删除了）

重启测试发现修复完成！



> 总结

**优点：**

1. 每一次修改都同步，文件的完整性会更加好！
2. 每秒同步一次，可能会丢失1秒的数据

**缺点：**

1. 相对于数据文件来说，aof远远大于rdb，修复的速度也比rdb慢。
2. aof运行效率也要比rdb慢，所以默认是关闭的！！





## Redis发布订阅

Redis发布订阅（pub/sub）是一种==消息通信模式==：发送者发送消息，订阅者接受消息。微信，微博，关注系统！

Redis客户端可以订阅任意数量的频道。

消息发送者发送消息到redis指定频道，订阅者接收

| 序号 | 命令                                           | 描述                               |
| :--- | :--------------------------------------------- | ---------------------------------- |
| 1    | [PSUBSCRIBE pattern [pattern ...\]]            | 订阅一个或多个符合给定模式的频道。 |
| 2    | [PUBSUB subcommand [argument [argument ...\]]] | 查看订阅与发布系统状态。           |
| 3    | [PUBLISH channel message]                      | 将信息发送到指定的频道。           |
| 4    | [PUNSUBSCRIBE [pattern [pattern ...\]]]        | 退订所有给定模式的频道。           |
| 5    | [SUBSCRIBE channel [channel ...\]]             | 订阅给定的一个或多个频道的信息。   |
| 6    | [UNSUBSCRIBE [channel [channel ...\]]]         | 指退订给定的频道。                 |



> 原理

Redis是使用c实现的，通过分析Redis源码里的 pubsub.c 文件，了解发布和订阅机制的底层实现，加深对Redis的理解。

Redis通过PUBLISH、SUBSCRIBE和PSUBSCRIBE等命令实现发布和订阅功能。



通过SUBSCRIBE命令订阅某频道后，redis-server里维护了一个字典，字典的键就是一个个频道，而字典的值则是一个链表，链表中保存了所有订阅这个频道的客户端。SUBSCRIBE命令的关键，就是将客户端添加到给定的频道的订阅链表中。



通过 PUBLISH 命令向订阅者发送消息，redis-server 会使用给定的频道作为键，在它所维护的 channel
字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。



Pub/Sub 从字面上理解就是发布（Publish）与订阅（Subscribe），在Redis中，你可以设定对某一个
key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应
的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。



使用场景：

1. 实时消息系统！
2. 实时聊天！
3. 订阅、关注系统

稍微复杂的场景我们就会使用消息中间件MQ！



## Redis主从复制

### 概念

主从复制，是将一台Redis服务器的数据，复制到其他Redis服务器。前者称为主节点（master/leader）， 后者称为从节点（slave/follower）；数据的复制是单向的，只能由主节点到从节点。

==Master以写为主，Slave以读为主。==

默认情况下，每个redis-server都是主节点；且一个主节点可以有多个从节点（或没有从节点），但一个从节点只能有一个主节点。

**主从复制的作用包括：**

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
4. 高可用基石：除了上诉作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

一般来说，要将Redis运用于工程项目中，只使用一台Redis是万万不能的（宕机），原因如下：

1. 从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较
   大；
2. 从容量上，单个Redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有。

内存用作Redis存储内存，一般来说，单台Redis最大使用内存不应该超过20G。
电商网站上的商品，一般都是一次上传，无数次浏览的，说专业点也就是"多读少写"。

==主从复制，读写分离，80%都是读操作，减缓服务器的压力！架构中经常使用！一主二从！==



### 环境配置

只配置从库，不用配置主库！（Redis默认就是主库）

```bash
127.0.0.1:6379> INFO replication	#查看当前库信息
# Replication
role:master							#角色（默认master）
connected_slaves:0					#没有从机
master_replid:eb8a91da4b3fded443de908c254d0030dbeb6edf
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
master_repl_meaningful_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

将配置文件复制三份，分别修改端口，pid文件名，rdb文件名，日志文件名！！！

然后以这三份配置启动！！！



### 一主二从

==默认情况下，每台Redis服务器都是主节点；==我们一般情况下只用配置从机就好了！

```bash
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379		#从机认谁做老大
OK
```

命令配置是暂时的，永久的需要在配置文件中配置！！！

```bash
replicaof 主机ip 主机端口
masterauth 主机密码
```

==此时主机可以读写，从机只能读==

主机宕机后，从机读功能正常使用。等待主机上线！！！



> 复制原理

Slave 启动成功连接到 master 后会发送一个sync同步命令
Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，==master将传送整个数据文件到slave，并完成一次完全同步。==
全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
增量复制：Master 继续将新的所有收集到的修改命令依次传给slave，完成同步
但是只要是重新连接master，一次完全同步（全量复制）将被自动执行！ 我们的数据一定可以在从机中看到！



> 层级模型

如同80是79的从节点，81是80的从节点。

80既是79的从节点，又是81的主节点。



> 谋朝篡位

```bash
SLAVEOF no one			# 将自己变成主节点
```



### 哨兵模式

自动选举主节点的模式！！！

> 概述

主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。Redis从2.8开始正式提供了Sentinel（哨兵） 架构来解决这个问题。
谋朝篡位的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。
哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独
立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

![1590851564977](J:\桌面\文章\文章图片\1590851564977.png)

这里的哨兵有两个作用

* 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
* 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。
各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

![1590851617510](J:\桌面\文章\文章图片\1590851617510.png)

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认
为主服务器不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一
定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover[故障转移]操作。
切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为
**客观下线**。



> 测试

我们目前的状态是 一主二从！

1、配置哨兵配置文件

```bash
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
```

后面的这个数字1，代表主机挂了，slave投票看让谁接替成为主机，票数最多的，就会成为主机！



2、启动哨兵

```bash
redis-sentinel myconfig/sentinel.conf
```

如果Master 节点断开了，这个时候就会从从机中随机选择一个服务器！ （这里面有一个投票算法！）

如果主机此时回来了，只能归并到新的主机下，当做从机，这就是哨兵模式的规则！



> 小结

优点：
1、哨兵集群，基于主从复制模式，所有的主从配置优点，它全有
2、 主从可以切换，故障可以转移，系统的可用性就会更好
3、哨兵模式就是主从模式的升级，手动到自动，更加健壮！
缺点：
1、Redis 不好啊在线扩容的，集群容量一旦到达上限，在线扩容就十分麻烦！
2、实现哨兵模式的配置其实是很麻烦的，里面有很多选择！



> 哨兵全部配置

```bash
# Example sentinel.conf

# 哨兵sentinel实例运行的端口 默认26379
port 26379

# 哨兵sentinel的工作目录
dir /tmp
# 哨兵sentinel监控的redis主节点的 ip port
# master-name 可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 配置多少个sentinel哨兵统一认为master主节点失联 那么这时客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2

# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000

# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1

# 故障转移的超时时间 failover-timeout 可以用在以下这些方面：
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。 
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则：
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
#通知脚本
# shell编程
# sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh

# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh # 一般都是由运维来配置！
```



## Redis缓存穿透和雪崩

