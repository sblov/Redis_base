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

​	通过让系统放松对某一时刻数据一致性的要求来换取系统整体伸缩性和性能上改观。由于在于大型系统往往由于地域分布和极高性能的要求，不可能采用分布式事务来完成这些指标，要想获得这些指标，必须采用另一种方式来完成，BASE就能解决这个问题。

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

#### Units单位

> 配置大小单位，开头定义了一些基本的度量单位，只支持bytes，不支持bit；对大小写不敏感

#### Includes包含

> 通过includes包含其他的配置文件

#### Modules模块

> 加载其他模块

#### Network网络

> bind、port、tcp-backlog、timeout.....

#### General通用

> 通用设置，daemonize、pidfile、loglevel、logfile、databases

#### Snapshotting快照

> RDB持久化，dbfilename....

#### Replication复制

> 主从复制

#### Security安全

> 安全设置，requirepass

```shell
config set requirepass "..."	#redis中设置密码
#在之后的访问操作时，都必须加上`auth "..."`
```

#### Clients客户端

> 客户端连接数，maxclients默认10000

#### Memory内存

> 内存管理设置，maxmemory、maxmemory-policy（内存清理策略：lru，ttl...）、maxmemory-samples

#### Append Only Mode追加

> AOF持久化

### Redis的持久化

#### RDB

##### Redis DataBase

​	在指定的时间间隔内将内存中的数据集快照写入磁盘，恢复时将快照文件直接读到内存

​	Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程结束，再将这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能，如果需要进行大规模数据的恢复，且对于数据恢复完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失

> **Fork**
>
> ​	Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据数值都与原进程一致，是一个全新的进程，并作为原进程的子进程

##### 配置

​	对应Snapshotting快照的配置

```shell
save 60 10000 #一分钟内10000数据更改时，备份一次
stop-writes-on-bgsave-error yes	#在bgsave发生错误时停止主线程数据的写入
rdbcompression yes	#采用LZF压缩dump.rdb文件
rdbchecksum yes	#CRC64校验
```

##### RDB快照触发

> **配置文件中快照配置**

> **save或bgsave命令**
>
> save：只管保存，其他不管，全部阻塞
>
> bgsave：redis会在后台异步进行快照操作，同时还可以相应客户端请求。可以通过lastsave命令获取最后移除成功执行快照的时间

> **执行flushall命令**也会产生dump.rdb文件，因为该命令，所有rdb文件是空的

##### 恢复

​	将备份文件（dump.rdb）移到redis安装目录并启动服务即可

​	`config get dir` 	获取目录

##### 优势

​	适合大规模的数据恢复，对数据完整性和一致性要求不高

##### 劣势

​	在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改

​	fork时，内存中的数据被克隆一份，大致2倍的膨胀性需要考虑

##### 停止

​	动态停止所有RDB保存规则：`redis-cli config set save “”`

#### AOF

##### Append Only File

​	以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据。redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

##### 配置

​	当Append Only Mode开启，会先加载appendfilename的文件，再是dump.rdb文件。当appendfile文件读取错误时，通过redis-check-aof进行语法修复（rdb对应redis-check-rdb）

```shell
appendonly no	#开启aof,默认关闭
appendfilename "appendonly.aof"	#aof默认保持文件
# appendfsync always	#每有一次写操作，触发一次
appendfsync everysec	#每秒触发一次
# appendfsync no	#不追加
no-appendfsync-on-rewrite no	#重写时是否运行appendfsync，用默认no即可，保证数据安全性
auto-aof-rewrite-percentage 100	#设置重写的基准值	
auto-aof-rewrite-min-size 64mb	#设置重写的基准值
aof-load-truncated yes
aof-use-rdb-preamble yes
```

##### 恢复

正常恢复

> ​	将appendonly开启，将有数据的aof文件复制一份到对应目录，重启redis将重新加载

异常恢复

> ​	备份被写坏的aof文件，恢复redis-check-aof --fix进行修复，重启redis

##### 重写

​	AOF采用文件追加 方式，文件会越来越大，为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阀值时，redis就会启动aof文件的内容压缩，只保留可以恢复数据的最小指令集，可以使用命令bgrewriteaof

> **原理**

​	AOF文件持续增大而过大时，会fork出一条新进程来将文件重写（先临时写文件再rename），遍历新进程的内存中数据，每条记录有一条的set语句。重写aof文件的操作，并没有读取旧aof文件，而是将整个内存中的数据库内容使用命令的方式重写了一个新的aof文件，这与快照类似

> 触发机制

​	redis会记录上次重写时的aof大小，默认配置是当前aof文件大小是上次rewrite后大小的一倍，且文件大于64MB时触发

```shell
auto-aof-rewrite-percentage 100	#设置重写的基准值	
auto-aof-rewrite-min-size 64mb	#设置重写的基准值
```

##### 优势

> 追加策略

##### 劣势

>​	相同数据集的数据而言，aof文件要远大于rdb文件，恢复速度慢于rdb
>
>​	aof运行效率慢于rdb，每秒同步策略效率较好，不同步效率与rdb相同

#### 选择

​	RDB持久化方式能够在指定的时间间隔，对数据进行快照存储

​	AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾

​	redis还能对aof文件进行后台重写，使aof文件的体积不至于过大

**只做缓存**

> ​	如果只希望数据在服务器运行的时候存在，可以不使用任何持久化方式

**同时开启**

> ​	在该情况下，当redis重启时会优先载入aof文件来恢复原始的数据，因为在通常情况下aof文件保存的数据要比rdb文件保存的数据要完整

> ​	rdb数据不实时，同时使用两者时，服务器重启只会找aof文件。但不能只使用aof，由于rdb更时候用于备份数据库（aof在不断变化不好备份），快速重启，而且不会右aof可能存在的bug，留着以防万一

#### 性能建议

​	由于RDB文件用于后备，建议只在slave上持久化RDB文件，而且只有15分钟备份一次即可，只保留 `save 900 1`  这条规则

​	如果enable aof，好处是在最恶劣情况下也只会丢失不超过两秒的数据，启动脚本比较简单，只load自己的aof文件即可。代价：带来持续的IO；aof rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少aof rewrite的频率，aof重写的基础大小默认值64m太小，可以设置5G以上。默认超过原大小100%大小时重写可以改到适当的数值

​	如果不enable aof，仅靠master-slave replication实现高可用性也可以。能省掉大量io，也减少rewrite时带来的系统波动。代价是如果master/slave同时挂掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。该方式 为新浪微博架构

### Redis的事务

​	可以一次执行多个命令，本质是一组命令的集合。一个事务中所有命令都会序列化，按顺序的串行执行而不会被其他命令插入

​	一个队列中，一次性、顺序性、排他性的执行一系列命令

#### 命令

```shell
127.0.0.1:6379> multi	#开启事务	discard 取消事务 
OK
127.0.0.1:6379> set k1 v1	#入队列
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> exec	#执行事务
1) OK
2) OK
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> watch k1	#监视k1，当该值在其他线程被更改时，当前线程需要取消watch(unwatch)，重新watch
OK
#----------------正常-----------------------------
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set k1 value
QUEUED
127.0.0.1:6379> exec	#执行exec会直接取消watch
1) OK
127.0.0.1:6379> get k1
"value"
#--------------其它线程 set k1 oth ---------------
127.0.0.1:6379> watch k1
OK
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set k1 vvv
QUEUED
127.0.0.1:6379> exec
(nil)	#表示执行失败
127.0.0.1:6379> get k1
"oth"
#127.0.0.1:6379> unwatch	#取消所有watch
#OK
#-----------------incr会失败------------------
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> incr k1	#执行时判断
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR value is not an integer or out of range
#---------------全部执行失败-------------------
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> set k1 xxx
QUEUED
127.0.0.1:6379> getset k2 xxx
QUEUED
127.0.0.1:6379> setget k2 xxx	#入队时即判断
(error) ERR unknown command `setget`, with args beginning with: `k2`, `xxx`, 
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k1
"oth"
```

悲观锁

> ​	考虑每次拿数据时，有其它进程会修改，则在拿数据时上锁。传统关系型数据库的这种锁机制如：行锁、表锁、读写锁.....

乐观锁

> ​	 与悲观锁相反，但在更新时会判断在此期间其它进程是否更新该数据，可以使用版本号等机制。

CAS（check and set）

#### 总结

​	watch指令，类似乐观锁，事务提交时，如果key的值已经被其它进程修改，整个事务队列都不会被执行

​	通过watch在事务执行前监控多个keys，若watch后任何key的值发生改变，exec执行的事务都将被放弃，同时返回 nullmulti-bulk应答以通知调用事务者执行失败

​	==单独的隔离操作（执行不会被其他命令打断）；没有隔离级别的概念（提交前不会被实际执行）；不保证原子性（没有回滚）==

### Redis的发布订阅

​	进程间的一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接受消息

#### 实例

sub

```shell
127.0.0.1:6379> subscribe a b c	#订阅a,b,c
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "a"
3) (integer) 1
1) "subscribe"
2) "b"
3) (integer) 2
1) "subscribe"
2) "c"
3) (integer) 3	#当订阅的有新的推送时，显示
1) "message"
2) "a"
3) "a1"
1) "message"
2) "b"
3) "b1"
            
lov@lov:/opt/redis-5.0.3/src$ sudo ./redis-cli -p 6379
[sudo] password for lov: 
127.0.0.1:6379> psubscribe new*	#通过*通配符匹配
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "new*"
3) (integer) 1
1) "pmessage"
2) "new*"
3) "new1"
4) "new1-value"

```

pub

```shell
127.0.0.1:6379> publish a a1	#推送消息
(integer) 1
127.0.0.1:6379> publish b b1
(integer) 1
127.0.0.1:6379> publish b b1
(integer) 0
127.0.0.1:6379> publish new1 new1-value
(integer) 1
```

### Redis的主从	复制（Master/Slave）

​	主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，master以写为主，slave以读为主

​	读写分离，容灾恢复

#### 配置

```shell
拷贝多个redis.conf文件
开启daemonize
pid文件名
指定端口
log文件名
dump.rdb文件名
```

#### 实例

```shell
#配置三个redis服务，redis6379.conf，redis6380.conf，redis6381.conf
#开启连接
```

master-6379

```shell
lov@lov:/opt/redis-5.0.3/src$ sudo ./redis-cli -p 6379
127.0.0.1:6379> INFO replication	#查看主从信息
# Replication
role:master		#默认为master
connected_slaves:0
.............................
127.0.0.1:6379> INFO replication	#当有从声明时，连接管理
# Replication
role:master
connected_slaves:2	#连接的slave数
slave0:ip=127.0.0.1,port=6380,state=online,offset=28,lag=1		#6380端口
slave1:ip=127.0.0.1,port=6381,state=online,offset=28,lag=1	#6381端口
...........................
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3	#当在这里设置，slaves自动同步，单默认只有master有写权限
OK
127.0.0.1:6379> SHUTDOWN	#当master挂掉后其他的slaves会等待，直到master重启；当slave挂掉后，重启需要再配置一次
not connected> 
```

slave-6380

```shell
lov@lov:/opt/redis-5.0.3/src$ sudo ./redis-cli -p 6380127.0.0.1:6380> INFO replication
.......................
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379	#配置为本地6379的slave
OK
127.0.0.1:6380> mget k1 k2 k3	#获取同步的数据
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6380> INFO replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up	#master连接
.......................
127.0.0.1:6380> INFO replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down	#当master关闭后，连接关闭，等待master开启
..........................
127.0.0.1:6380> mget k1 k2 k3	#之前同步的数据依然保存
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6380> INFO replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up	#master连起后恢复
.............................
127.0.0.1:6380> get k4
"v4"
127.0.0.1:6380> SLAVEOF no one	#将该redis服务设为master
OK
127.0.0.1:6380> INFO replication
# Replication
role:master
connected_slaves:0
master_replid:e89cd7cfabf0ae269ee97e3f682c179da3fb5f5e
master_replid2:06b3d527e02e830773f819585b47fd0b669e0282
..........................
127.0.0.1:6380> mget k1 k2 k3	#数据存在
1) "v1"
2) "v2"
3) "v3"
```

slave-6381

```shell
lov@lov:/opt/redis-5.0.3/src$ sudo ./redis-cli -p 6381
127.0.0.1:6381> SLAVEOF 127.0.0.1 6379
OK
127.0.0.1:6381> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6381> INFO replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
................................
127.0.0.1:6381> get k4
"v4"
127.0.0.1:6381> SLAVEOF 127.0.0.1 6380	#当master挂掉后，以6380为master
OK
127.0.0.1:6381> INFO replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
..............................
127.0.0.1:6381> get k4
"v4"
127.0.0.1:6381> set k5 v5	#默认只有读的权限
(error) READONLY You can't write against a read only replica.
```

#### 复制原理

​	slave启动成功连接到master后，会发送一个sync命令，master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕后，master将传送整个数据文件到slave，以完成一次完全同步

**全量复制**

> ​	slave服务在接收到数据库文件数据后，将其存盘并加载到内存中，只要重新连接master，一次完全同步（全量复制）将被自动执行

**增量复制**

> ​	master继续将新的所有收集到的修改命令依次传给slave，完成同步

#### 哨兵模式（sentinel）

​	能够后台监控主机是否故障，如果故障了根据投票数自动将slave转为master

##### 配置

1、新建sentinel.conf文件

```shell
sentinel monitor host6379 127.0.0.1   6379 		1
		(redis库名自定义)  (ip地址)	(端口号)  (票数大于1的即可被选为master)
```

2、启动哨兵

```shell
sudo ./redis-sentinel /opt/myredis/sentinel.conf  
```

> 当master挂掉后，会投票选取出新的master；
>
> 当挂掉的master重启后，会直接作为当前master的slave；
>
> 对于一组sentinel，能同时监控多个master

#### 复制延迟

​	由于所有的写操作都是先在master上操作，然后同步更新到slave上，所以从master到slave有一定的延迟，当系统繁忙时，延迟会更加严重，slave机器数量的增加也会使这个问题加重

### Jedis

#### 测试

Pom.xml

```xml
<dependency>
	    <groupId>org.apache.commons</groupId>
	    <artifactId>commons-pool2</artifactId>
	    <version>2.4.2</version>
	</dependency>
  	<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
	<dependency>
	    <groupId>redis.clients</groupId>
	    <artifactId>jedis</artifactId>
	    <version>2.9.0</version>
	</dependency>
```

Test

```java
Jedis	jedis = new Jedis("127.0.0.1", 6379);
System.out.println(jedis.ping());
		
jedis.slaveofNoOne();
		
jedis.set("k1", "v1");
jedis.set("k2", "v2");
jedis.set("k3", "v3");
System.out.println(jedis.mget("k1","k2","k3"));
```

#### API

##### 事务

模拟消费

```java
public class JedisTest {
	
	//模拟消费
	public boolean transMethod() throws InterruptedException {
		//连接redis服务器
		Jedis jedis = new Jedis("127.0.0.1",6379);
		
		int balance;	//可借用
		int debt;	//贷款
		int amount = 10;
		
		jedis.watch("balance");		//监控balance
		//当其他进程 set balance
		Thread.sleep(3000);		//模拟网络延迟
		
		balance = Integer.parseInt(jedis.get("balance"));
		if (balance < amount) {		//判断balance是否够用
			jedis.unwatch();
			System.out.println("unenough");
			return false;
		}else {
			System.out.println(".......transaction");
			Transaction transaction = jedis.multi();
			transaction.decrBy("balance", amount);	
			transaction.incrBy("debt", amount);
			
			List<Object> exec = transaction.exec();	//事务执行
			
			if (exec.size() == 0) {	//，其他线程干扰，事务执行失败
				System.out.println("transaction error");
				return false;
			}
			//System.out.println(exec.size());
			
			balance = Integer.parseInt(jedis.get("balance"));
			debt = Integer.parseInt(jedis.get("debt"));
				System.out.println(".........."+balance);
			System.out.println(".........."+debt);
			return true;
		}		
	}
	public static void main(String[] args) throws InterruptedException {
		
		boolean flag = new JedisTest().transMethod();
		System.out.println(flag);	
	}	
}
```

##### 主从

##### JedisPool

jedisPoolUtil

```java
public class JedisPoolUtil {

	private static volatile JedisPool jedisPool = null;
	
	private JedisPoolUtil() {	}

	//获得连接池
	public static JedisPool	getJedisPoolInstance() {
		if (null == jedisPool) {
			synchronized (JedisPoolUtil.class) {
				if (jedisPool == null) {
						
					JedisPoolConfig config = new JedisPoolConfig();
					config.setMaxIdle(32);
					config.setMaxWaitMillis(100*1000);
					config.setTestOnBorrow(true);
					
					jedisPool = new JedisPool(config,"127.0.0.1",6379);
				}
			}
		}
		
		return jedisPool;
	}

	//释放当前连接
	public static void release(JedisPool jedisPool,Jedis jedis) {
		if (null !=jedis) {
			jedisPool.returnResourceObject(jedis);
		}
	}
	
}
```

main

```java
JedisPool jedisPool = JedisPoolUtil.getJedisPoolInstance();
		Jedis jedis = jedisPool.getResource();
		Set<String> string = jedis.keys("*");
		System.out.println(string);
		JedisPoolUtil.release(jedisPool, jedis);
```

