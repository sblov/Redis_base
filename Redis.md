## NoSQL

​	数据库优化

 - 单机mysql（数据库）

 - Memcached（缓存）+ mysql（数据库）+ 垂直拆分（对不同模块存不同数据库）

 - mysql（数据库）主从读写分离

 - 分表分库 + 水平拆分 + mysql（数据库）集群

 - .....优化瓶颈

​	同时由于web应用数据的发展，导致NoSQL出现。

​	NoSQL（Not Only SQL）数据库，非关系型数据库最早出现在1998年，在2009年被空前讨论，发展和繁荣。主要由于web2.0的发展，是的重新对数据存储模型进行思考。

#### 数据模型

​	聚合模型：kv键值、Bson、列族、图形

#### 数据库分类

​	键值对数据库：redis...

​	列存储数据库：Cassandra、HBase...

​	文档型数据库：MongoDB、CouchDB...

​	图形数据库：Neo4j、InfoGrid...​	

#### 分布式数据库CAP原理

##### 传统ACID

- Atomicity 原子性

- Consistency 一致性

- Isolation 独立性

- Durability 持久性

  传统数据库必须满足整个ACID

##### 	CAP

- Consistency 强一致性

- Availability 可用性

- Partition tolerance 分区容错性

​	Nosql理论CAP只能满足其中两个

​	CAP的核心理论是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性三个需求，最多只能较好的满足两个

​	CA：单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大（传统型数据库，mysql，oracle）

​	CP：满足一致性，分区容错性的系统，通常性能不是特别高（Redis，MongoDB）

​	AP：满足可用性，分区容错性的系统，通常可能对一致性要求低一些，造成弱一致性（大多数网站架构的选择）

​	在分布式存储系统中，做多只能实现上面两点。由于当前网络硬件肯定会出现延迟丢包等问题，所以**分区容错性是必须需要实现的**。所以只能在一致性与可用性间进行权衡。
##### BASE

​	BASE是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案。

​	基本可用（Basically Available）

​	软状态（Soft state）

​	最终一致性（Eventually consistent）

​	通过让系统放松对某一时刻数据一致性的要求来换取系统整体伸缩性和性能上改观。由于在于大型系统往往由于地域分布和极高性能的要求，不可能采用分布式事物来完成这些指标，要想获得这些指标，必须采用另一种方式来完成，BASE就能解决这个问题。

​	（例如淘宝双十一）





---

## Redis

### 简介

​	Remote Dictionary Server（Redis），一个开源的由Salvatore Sanfilippo使用ANSI C语言编写的key-value数据存储服务器。

​	Redis属于NoSQL

NoSQL：

- key-value存储：Berkeley DB，Memcache DB，Redis
- 文档存储：MongoDB，CouchDB
- 列存储：Hbase，Cassandra

Redis-github：https://github.com/antirez/redis

### 特点

> - Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用
> - Redis不仅仅支持简单的key-value类型的数据，还提供list、set、zset、hash等数据结构的存储
> - Redis支持数据的备份，即master-slave模式的数据备份

### 作用

> - 内存存储和持久化：Redis支持异步将内存中的数据写入磁盘，同时不影响继续服务
>
> - 取最新的数据：可以将最新的几条评论的id放在list集合中
>
> - 模拟类似HttpSession这种需要设定过去时间的功能
>
> - 发布、订阅消息系统
>
> - 定时器、计数器

### 安装

```shell
sudo mv redis-5.0.3.tar.gz  /opt/
sudo tar -zxvf redis-5.0.3.tar.gz 
cd redis-5.0.3/
sudo make
```

### 使用

> `./redis-server /opt/myredis/redis.conf`

​	安装目录的src下，指定conf文件启动redis

> `./redis-cli  -p 6379`

​	连接指定端口的redis

> `ps -ef|grep redis`

​	查看redis进程

#### 启动后

**单进程**

> 单进程模型来处理客户端的请求。对读写等事件的相应是通过对linux中epoll函数的包装来做到的。redis的实际处理速度完全依赖于主进程的执行效率

**默认16个数据库，从0开始，默认使用0号库**

> redis.conf
>
> ```shell
> # Set the number of databases. The default database is DB 0, you can select
> # a different one on a per-connection basis using SELECT <dbid> where
> # dbid is a number between 0 and 'databases'-1
> databases 16
> ```

**Select命令切换数据库**

**Dbsize查看当前数据库的key的数量**

> ```shell
> 127.0.0.1:6379> DBSIZE
> (integer) 3
> 127.0.0.1:6379> keys *
> 1) "k2"
> 2) "k1"
> 3) "a"
> ```

**Flushdb清空当前库**

**Flushall清空全部库**

**统一密码管理，16个库都是同样的密码**

**Redis下标从0开始**

**默认端口6379**

> 原因：`http://oldblog.antirez.com/post/redis-as-LRU-cache.html`

### Redis数据类型

#### 键（key）关键字

```shell
127.0.0.1:6379> keys *	#当前库所以key
1) "k2"
2) "k1"
3) "a"
127.0.0.1:6379> get k1	#获取一个key的value
"v1"
127.0.0.1:6379> EXISTS k1	#判断当前key是否存在
(integer) 1
127.0.0.1:6379> move k2 1	#将当前库下的key-value移到1号库下
(integer) 1
127.0.0.1:6379> select 1	#选择1号库
OK
127.0.0.1:6379[1]> get k2	
"v3"
127.0.0.1:6379[1]> expire k2 10	#给key指定过期时间，到时间自动从内存中销毁
(integer) 1
127.0.0.1:6379[1]> ttl k2	#查看当前key的有效时间，-2为失效，-1为永久
(integer) 2
127.0.0.1:6379[1]> ttl k2
(integer) -2
127.0.0.1:6379[1]> keys *
(empty list or set)
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> type k1	#判断当前key类型
string
```

==**五大数据类型**==

#### 字符串（String）

> String是二进制安全的，是Redis最基本的数据类型，一个Redis字符串value最多可以是512M

```shell
单值单value
127.0.0.1:6379> keys *
1) "k1"
2) "a"
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> append k1 value	#追加
(integer) 7
127.0.0.1:6379> get k1
"v1value"
127.0.0.1:6379> del a	#删除
(integer) 1
127.0.0.1:6379> get a
(nil)
127.0.0.1:6379> set k2 1
OK
127.0.0.1:6379> incr k2	#自增1
(integer) 2
127.0.0.1:6379> incr k2
(integer) 3
127.0.0.1:6379> incr k2
(integer) 4
127.0.0.1:6379> decr k2	#自减1
(integer) 3
127.0.0.1:6379> decr k2
(integer) 2
127.0.0.1:6379> incrby k2 2	#自增2
(integer) 4
127.0.0.1:6379> incrby k2 2
(integer) 6
127.0.0.1:6379> decrby k2 2	#自减2
(integer) 4
127.0.0.1:6379> decrby k2 2
(integer) 2
127.0.0.1:6379> getrange k1 0 3	#获取范围值
"v1va"
127.0.0.1:6379> setrange k1 0 xxx	#从0开始设置值
(integer) 7
127.0.0.1:6379> get k1
"xxxalue"
127.0.0.1:6379> setex k3 6 v4	#设值时指定失效时间（6s）
OK
127.0.0.1:6379> ttl k3
(integer) -2
127.0.0.1:6379> setnx k1 v1	#当改key不存在时设值
(integer) 0
127.0.0.1:6379> setnx k3 v3
(integer) 1
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3	#map赋值
OK
127.0.0.1:6379> mget k1 k2 k3	#map获取
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v1 k4 v4	#if not exist，必须全部满足
(integer) 0
127.0.0.1:6379> get k4
(nil)
127.0.0.1:6379> msetnx k4 v4 k5 v5
(integer) 1
127.0.0.1:6379> get k4
"v4"
```

#### 列表（List）

> Redis列表是最简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部或尾部，底层为链表

```shell
如果值全移除，对应的键就消失
链表的头尾操作效率高，中间元素效率低
127.0.0.1:6379> lpush list_1 1 2 3 4	#从左插入值
(integer) 4
127.0.0.1:6379> rpush list_1 1 2 3 4	#从右插入值
(integer) 8
127.0.0.1:6379> lrange list_1 0 -1	#获取该范围值
1) "4"
2) "3"
3) "2"
4) "1"
5) "1"
6) "2"
7) "3"
8) "4"
127.0.0.1:6379> lpop list_1	#从左出栈
"4"
127.0.0.1:6379> rpop list_1	#从右出栈
"4"
127.0.0.1:6379> lrange list_1 0 -1	
1) "3"
2) "2"
3) "1"
4) "1"
5) "2"
6) "3"
127.0.0.1:6379> lindex list_1 0	#获取下标为0的值
"3"
127.0.0.1:6379> llen list_1	#获取list的长度
(integer) 6
127.0.0.1:6379> lrem list_1 2 3	#移除list中两个3
(integer) 2
127.0.0.1:6379> lrange list_1 0 -1	
1) "2"
2) "1"
3) "1"
4) "2"
127.0.0.1:6379> ltrim list_1 0 2	#截取0到2下标的值
OK
127.0.0.1:6379> lrange list_1 0 -1
1) "2"
2) "1"
3) "1"
127.0.0.1:6379> rpoplpush list_1 list_2 #list_1右出栈的值，直接左入栈list_2
"1"
127.0.0.1:6379> lrange list_2 0 -1
1) "1"
127.0.0.1:6379> rpoplpush list_1 list_2
"1"
127.0.0.1:6379> rpoplpush list_1 list_2
"2"
127.0.0.1:6379> lrange list_2 0 -1
1) "2"
2) "1"
3) "1"
127.0.0.1:6379> lset list_2 1 x	#设置下标为1的值为x
OK
127.0.0.1:6379> lrange list_2 0 -1
1) "2"
2) "x"
3) "1"
127.0.0.1:6379> linsert list_2 before x jav	#在x前插入值
(integer) 4
127.0.0.1:6379> lrange list_2 0 -1
1) "2"
2) "jav"
3) "x"
4) "1"
```

#### 集合（Set)

```shell
127.0.0.1:6379> sadd  set_1 1 1 2 2 3 3 #添加值
(integer) 3
127.0.0.1:6379> smembers set_1	#获取set中所有值
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> sismember set_1 1	#判断是否是成员值
(integer) 1
127.0.0.1:6379> scard set_1	#获取set中的成员数
(integer) 3
127.0.0.1:6379> srem set_1 3	#移除指定的值
(integer) 1
127.0.0.1:6379> smembers set_1
1) "1"
2) "2"
127.0.0.1:6379> sadd set_1 3 4 5 6 7
(integer) 5
127.0.0.1:6379> smembers set_1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
127.0.0.1:6379> srandmember  set_1 3	#随机获取三个值
1) "4"
2) "5"
3) "6"
127.0.0.1:6379> srandmember  set_1 3
1) "3"
2) "5"
3) "1"
127.0.0.1:6379> spop set_1 	#随机出栈
"3"
127.0.0.1:6379> spop set_1 
"5"
127.0.0.1:6379> sadd set_2 x y z
(integer) 3
127.0.0.1:6379> smove set_1 set_2 4	#将set_1中的4移入set_2
(integer) 1
127.0.0.1:6379> smembers set_2
1) "4"
2) "y"
3) "z"
4) "x"
127.0.0.1:6379> clear

127.0.0.1:6379> del set_1
(integer) 1
127.0.0.1:6379> del set_2
(integer) 1
127.0.0.1:6379> sadd set_1 1 2 3 4 5
(integer) 5
127.0.0.1:6379> sadd set_2  1 2 3 a b
(integer) 5
127.0.0.1:6379> sdiff set_1  set_2	#差集
1) "4"
2) "5"
127.0.0.1:6379> sinter set_1  set_2	#交集
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> sunion set_1  set_2	#并集
1) "3"
2) "5"
3) "4"
4) "1"
5) "2"
6) "b"
7) "a"
```

#### 哈希（Hash）

> Redis hash是一个键值对集合，类似Map<String,Object>

```shell
value为键值对
127.0.0.1:6379> hset user id 1	#设值
(integer) 1
127.0.0.1:6379> hget user id	#获取
"1"
127.0.0.1:6379> hmset user name lov age 22	#多个赋值
OK
127.0.0.1:6379> hmget user id name age	#多个获取
1) "1"
2) "lov"
3) "22"
127.0.0.1:6379> hgetall user	#获取所有可以key-value
1) "id"
2) "1"
3) "name"
4) "lov"
5) "age"
6) "22"
127.0.0.1:6379> hdel user name	#删除其中的key—value
(integer) 1
127.0.0.1:6379> hgetall user
1) "id"
2) "1"
3) "age"
4) "22"
127.0.0.1:6379> hlen user	#key-value个数
(integer) 2
127.0.0.1:6379> hexists user id	#判断是否存在当前key
(integer) 1
127.0.0.1:6379> hkeys user	#获取所有key
1) "id"
2) "age"
127.0.0.1:6379> hvals user	#获取所有value
1) "1"
2) "22"
127.0.0.1:6379> hincrby user age 2	#自增
(integer) 24
127.0.0.1:6379> hincrby user age 2
(integer) 26
127.0.0.1:6379> hset user score 91.5
(integer) 1
127.0.0.1:6379> hincrbyfloat user score 0.5	#浮点数自增
"92"
127.0.0.1:6379> hsetnx user email lov@gemail.com	#不存在则设值
(integer) 1
```

#### 有序集合（Zset）

> Sorted set，和set一样是String类型元素的集合，且不允许重复的成员。
>
> 不同的是每个元素都会关联一个double类型的分数
>
> Redis通过分数来为集合中的成员进行排序，zset成员是唯一的，但分数可以重复	

```shell
127.0.0.1:6379> zadd z_1 60 v1 70 v2 80 v3 90 v4	#添加
(integer) 4
127.0.0.1:6379> zrange z_1 0 -1	#获取
1) "v1"
2) "v2"
3) "v3"
4) "v4"
127.0.0.1:6379> zrange z_1 0 -1 withscores	#同时获取score
1) "v1"
2) "60"
3) "v2"
4) "70"
5) "v3"
6) "80"
7) "v4"
8) "90"
127.0.0.1:6379> zrangebyscore z_1 60 80	#获取score在该范围的
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> zrangebyscore z_1 60 (90	# ( 表示不包含
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> zrangebyscore z_1 (60 (90
1) "v2"
2) "v3"
127.0.0.1:6379> zrangebyscore z_1 60 90 limit 2 2	#取该范围中值，从下标2开始取两条
1) "v3"
2) "v4"
127.0.0.1:6379> zrem z_1 v4	#移除
(integer) 1
127.0.0.1:6379> zrange z_1 0 -1
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> zcard z_1	#个数
(integer) 3
127.0.0.1:6379> zcount z_1 60 80	#该范围的个数
(integer) 3
127.0.0.1:6379> zrank z_1 v1	#获取该value的下标
(integer) 0
127.0.0.1:6379> zscore z_1 v1	#获取该value的score
"60"
127.0.0.1:6379> zrevrank z_1 v3	#反转获取value的下标
(integer) 0
127.0.0.1:6379> zrevrange z_1 0 -1	#反转获取
1) "v3"
2) "v2"
3) "v1"
127.0.0.1:6379> zrevrangebyscore z_1 80 60	#反转通过score获取
1) "v3"
2) "v2"
3) "v1"
```

### 解析配置文件

Units单位

> 配置大小单位，开头定义了一些基本的度量单位，只支持bytes，不支持bit；对大小写不敏感

Includes包含

> 通过includes包含其他的配置文件

General通用

Snapshotting快照

Replication复制

Security安全

Limits限制

Append Only Mode追加

常见配置redis.conf

### Redis的持久化

### Redis的事物

### Redis的发布订阅

### Redis的复制（Master/Slave）

### Jedis