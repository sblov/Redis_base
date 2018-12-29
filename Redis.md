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