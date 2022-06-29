# Redis

## 一、NoSQL数据库

### 1. NoSQL数据库概述

NoSQL = **Not Only SQL**，意思是不仅仅是SQL，泛指非关系型的数据库。

NoSQL不依赖业务逻辑方式存储，而以简单的key-value模式存储，因此大大的增加了数据库的扩展能力 

+ 不遵循SQL标准
+ 不支持ACID(原子性，隔离性，一致性和持久性的数据库事务)
+ 远超SQL的性能

NoSQL数据库打破了传统关系型数据库以业务逻辑为依据的存储模型，而针对不同数据结构类型改为以性能为最优先的存储方式

### 2. NoSQL使用场景

+ 对数据高并发的读写
+ 海量数据的读写
+ 对数据高可扩展性的

**NoSQL不适用的场景**

+ 需要事务支持
+ 基于sql的结构化查询存储，处理复杂的关系，需要即席查询
+ 用不着SQL的以及SQL不能处理的情况，可以考虑使用NoSQL

### 3. 常见的NoSQL数据库

**Memcache**：很早出现的NoSQL数据库，数据都存在内存中，不支持持久化

**Redis**：几乎覆盖了Memcache的绝大部分功能，数据都在内存中，**支持持久化**，主要用作备份恢复

**MongoDB**：文档性数据库，数据都在内存中，如果内存不足，把不常用的数据存放至硬盘

### 4. 行式存储数据库

#### 4.1 行式数据库

把每行数据作为一个数据结构进行存储

![](C:\Users\86198\Desktop\JavaEE\Redis\image\rowSaveDB.jpg)

#### 4.2 列式数据库

把一列作为一种数据结构进行存储

![](C:\Users\86198\Desktop\JavaEE\Redis\image\columnSaveDB.jpg)

## 二、Redis概述

Redis是一个开源的**key-value**存储系统

和Memcache类似，它支持存储的value类型相对更多，包括string，list，set，zset(sorted set --- 有序集合)和hash

这些数据类型都支持push/pop，add/remove及取交集，并集和差集或者更丰富的操作，并且这些操作都是**原子性**的

在此基础上，Redis支持各种不同方式的排序

与Memcache一样，为保证效率，数据都是缓存在内存中

区别是Redis会**周期性**的把更新的**数据写入磁盘**或者把修改操作写入追加的记录文件

并且在此基础上实现了master-salve(主从)同步

### 1. 应用场景

#### 1.1 配合关系型数据库做高速缓存

+ 高频次，热门访问的数据，降低数据库的I/O
+ 分布式架构，做session共享

![](C:\Users\86198\Desktop\JavaEE\Redis\image\RedisUseForm1.jpg)

#### 1.2 多样的数据结构存储持久化数据

![](C:\Users\86198\Desktop\JavaEE\Redis\image\RedisUseForm2.jpg)

### 2. Redis的启动

#### 2.1 前台启动

在Linux下，将Redis的压缩文件放在/usr/local/redis目录下，解压安装，默认将将一些redis的配置放在/user/local/bin目录下，前台启动只需要在bin目录下，在终端输入redis-server命令，即可启动redis

![](C:\Users\86198\Desktop\JavaEE\Redis\image\redisStartBefore.jpg)

退出直接输入ctrl + c即可退出

#### 2.2 后台启动

将redis的配置文件redis.conf复制至/etc目录下：cp redis.conf /etc/redis.conf

跳转至etc目录下，使用vi命令编辑redis.conf文件，将其中的daemonize no修改为daemonize yes

退出vim模式，进入/usr/local/bin目录下，输入redis-server /etc/redis.conf启动redis

![](C:\Users\86198\Desktop\JavaEE\Redis\image\redisStartBehin.jpg)

#### 2.3 关闭

可以在启动的命令行中直接输入redis-cli shutdown关闭redis

也可以退出，使用kill命令杀掉redis进程

### 3. Redis相关知识简介

Redis默认有16个数据库，类似数组下标从0开始，初始**默认使用0号库**

使用命令select \<dbid>来切换数据库，比如select 2

统一密码管理，所有库同样的密码

Redis是单线程 + 多路IO复用技术

+ 多路复用是指使用一个线程来检查多个文件描述符(Socket)的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时，得到就绪状态后进行正真的操作可以在同一个线程里执行，也可以启动线程执行(比如使用线程池)

与Memcache三点不同：支持多数据类型，支持持久化，单线程 + 多路IO复用

## 三、Redis常用五大数据类型

### 1. Redis键操作

**keys \***: 查看当前库中的所有key

**exists key**: 判断某个key是否存在

**type key**: 查看key的类型

**del key**: 删除指定的key的数据

**unlink key**: 根据value选择非阻塞删除(仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作)

**expire key second**: 为指定的key设置过期时间，单位为秒

**ttl key**: 查看key还有多少秒过期，-1表示永不过期，-2表示已经过期



**操作库命令**

**select dbid**切换数据库

**dbsize**查看当前数据库中有多少数量的key

**flusdb**清空当前库

**flushall**通杀全部库

### 2. Redis字符串(String)

#### 2.1 基本介绍

String是Redis中最基本的类型

String类型是二进制安全的，意味着Redis的String可以包含任何数据，比如jpg图片或者序列化对象

String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M

#### 2.2 常用命令

**set \<key>\<value>** 添加键值对，当数据库中已经有key值时，新设置的key值会覆盖原来的key值

**get \<key>**查询对应键值

**append \<key>\<value>**将给定的\<value>追加到原值的末尾，当原来key不存在时，会新建一个key

**strlen \<key>**获得值的长度

**setnx \<key>\<value>**只有在key不存在时，设置key的值；如果key存在，则设置失败

**incr \<key>**: 将key中存储的数字值增1(数字类型)，只能对数字值操作，如果为空，新增值为1

**decr \<key>**将key中存储的数字值减1，如果为空，则新增值为-1

**incrby/decrby \<key>\<步长>**将key中存储的数字值增减，自定义步长

#### 2.3 原子性

原子操作：指**不会被线程调度机制打断**的操作；这种操作一旦开始，就一直运行结束，中间不会有任何的上下文切换

+  在单线程中，能够在单条指令中完成的操作都可以认为是原子操作，因为中断只能发生于指令之间
+ 多线程中，不能被其他线程打断的操作就叫原子操作

*注意：java中的 i++不是原子操作*

**mset \<key1>\<value1>\<key2>\<value2>...**同时设置一个或这多个key-value对

**mget \<key1>\<key2>...**同时获取一个或者多个key所对应的value值

**msetnx \<key1>\<value1>\<key2>\<value2>...**同时设置一个或多个key-value对，当且仅当所有给定的key都不存在时，设置成功 **原子操作，如果有一个key存在，导致所有设置的值都失败**

**getrange \<key><起始位置><结束位置>**获得值的范围,**包括首位**

**setrange \<key><起始位置>\<value>**用 \<value> 覆写\<key>所储存的字符串值，从<起始位置>开始(**索引从0开始**)。

**setex \<key><过期时间>\<value>**设置键值的同时，设置过期时间，单位秒。

**getset \<key>\<value>**以新换旧，设置了新值同时**获得旧值**，但实际上当前key已经被替换为新值。

#### 2.4 String所使用的底层数据结构

String的数据结构为简单动态字符串(Simple Dynamic String,缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.

![](C:\Users\86198\Desktop\JavaEE\Redis\image\String.jpg.png)

如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是**字符串最大长度为512M**。

### 3. Redis列表(List)

#### 3.1 基本介绍

Redis列表时简单的字符串列表，按照插入的顺序排序，可以在列表的头部或者尾部添加一个元素

底层是一个**双向链表**，对两端的操作性能很高，通过索引下标的操作中间的节点性能会比较差

#### 3.2 常用命令

**lpush/rpush \<key>\<value1>\<value2>\<value3> ....** 从左边/右边插入一个或多个值。

**lpop/rpop \<key>**从左边/右边吐出一个值，如果键所对应的列表中还有值，那么键就存在，如果键所对应的列表中没有值，那么键就不存在

**rpoplpush \<key1>\<key2>**从\<key1>列表右边吐出一个值，插到\<key2>列表左边。

**lrange \<key>\<start>\<stop>**按照索引下标获得元素(从左到右)

**lrange \<key> 0 -1**  0左边第一个，-1右边第一个，（0  -1表示获取所有）

**lindex \<key>\<index>**按照索引下标获得元素(从左到右)

**llen \<key>**获得列表长度 

**linsert \<key> before \<value>\<newvalue>**在\<value>的前面插入\<newvalue>插入值

**lrem \<key>\<n>\<value>**从左边删除n个value(从左到右)

**lset\<key>\<index>\<value>**将列表key下标为index的值替换成value

#### 3.3 数据结构

List的数据结构为快速链表quickList。

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。

它将所有的元素紧挨着一起存储，分配的是一块连续的内存。

当数据量比较多的时候才会改成quicklist。

因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。

![](C:\Users\86198\Desktop\JavaEE\Redis\image\list.png)

Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

### 4. Redis集合(Set)

#### 4.1 基本介绍

Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以**自动排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的Set是string类型的无序集合。它**底层其实是一个value为null的hash表**，所以添加，删除，查找的**复杂度都是O(1)**。

一个算法，随着数据的增加，执行时间的长短，如果是O(1)，数据增加，查找数据的时间不变

#### 4.2 常用命令

**sadd \<key>\<value1>\<value2> .....** 将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略

**smembers \<key>**取出该集合的所有值。

**sismember \<key>\<value>**判断集合\<key>是否为含有该\<value>值，有1，没有0

**scard\<key>**返回该集合的元素个数。

**srem \<key>\<value1>\<value2> ....** 删除集合中的某个元素。

**spop \<key>随机从该集合中吐出一个值(顺便删除)。**

**srandmember \<key>\<n>**随机从该集合中取出n个值。不会从集合中删除 。

**smove \<source>\<destination>value**把集合中一个值从一个集合移动到另一个集合

**sinter \<key1>\<key2>**返回两个集合的交集元素。

**sunion \<key1>\<key2>**返回两个集合的并集元素。

**sdiff \<key1>\<key2>**返回两个集合的**差集**元素(key1中的，不包含key2中的)

#### 4.3 数据结构

Set数据结构是dict字典，字典是用哈希表实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

### 5. Redis哈希(Hash)

#### 5.1 基本介绍

Redis hash 是一个键值对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

类似Java里面的Map<String,Object>

用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储

主要有一下两种存储方式

| 方式一                                                       | 方式二                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![](C:\Users\86198\Desktop\JavaEE\Redis\image\HashSaveOne.jpg)每次修改用户某个属性，需要先反序列化，改好后再序列化回去，开销较大 | ![](C:\Users\86198\Desktop\JavaEE\Redis\image\HashSaveTwo.jpg)用户ID数据冗余 |

结合上述两种方式的优点，演变出方式三：

![](C:\Users\86198\Desktop\JavaEE\Redis\image\HashSaveThree.jpg)

通过**key(用户ID) + field(属性标签)** 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题

#### 5.2 常用命令

**hset \<key>\<field>\<value>**给\<key>集合中的\<field>键赋值\<value>

**hget \<key1>\<field>**从\<key1>集合\<field>取出 value 

**hmset \<key1>\<field1>\<value1>\<field2>\<value2>...** 批量设置hash的值，hset也能够做到

**hexists\<key1>\<field>**查看哈希表 key 中，给定域 field 是否存在。 

**hkeys \<key>**列出该hash集合的所有field

**hvals \<key>**列出该hash集合的所有value

**hincrby \<key>\<field>\<increment>**为哈希表 key 中的域 field 的值加上增量 

**hsetnx \<key>\<field>\<value>**将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .

#### 5.3 数据结构

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

### 6. Redis有序集合(Zset)

#### 6.1 基本介绍

Redis有序集合zset与普通集合set非常相似，是一个**没有重复元素**的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分（score）**,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。**集合的成员是唯一的，但是评分可以是重复** 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

#### 6.2 常用命令

**zadd \<key>\<score1>\<value1>\<score2>\<value2>…**

将一个或多个 member 元素及其 score 值加入到有序集 key 当中。

**zrange \<key>\<start>\<stop> [WITHSCORES]**  

返回有序集 key 中，下标在\<start>\<stop>之间的元素

带WITHSCORES，可以让分数一起和值返回到结果集。

**zrangebyscore key minmax [withscores] [limit offset count]**

返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 

**zrevrangebyscore key maxmin [withscores] [limit offset count]**        

同上，改为从大到小排列。 

**zincrby \<key>\<increment>\<value>**   为元素的score加上增量

**zrem \<key>\<value>**删除该集合下，指定值的元素

**zcount \<key>\<min>\<max>**统计该集合，分数区间内的元素个数 

**zrank \<key>\<value>**返回该值在集合中的排名，从0开始。

#### 6.3 数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

**跳表**

1. 简介：有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

2. 实例：在下图所示的跳表中，查询元素51

   ![](C:\Users\86198\Desktop\JavaEE\Redis\image\jumpForm.png)

从第2层开始，1节点比51节点小，向后比较。

21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层

在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下

在第0层，51节点为要查找的节点，节点被找到，共查找4次。

## 四、Redis新数据类型

### 1. Bitmaps



### 2. HyperLogLog



### 3. Geospatial



## 五、Jedis

### 1. Jedis所需要的jar包

Jedis是java连接redis数据库的一种方式

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 2. 连接Redis注意事项

+ 需要禁用Linux的防火墙，或者不禁用防火墙，只开放6379端口
+ 在Redis配置文件redis.conf中，注释掉bind 127.0.0.1以及修改protected-mode为no，禁用保护模式

### 3. Jedis常用操作

#### 3.1 创建测试程序

```java
public class Demo01 {
    public static void main(String[] args) {
        //第一个参数为主机ip地址，6379为Redis固定端口号
        Jedis jedis = new Jedis("192.168.137.3",6379);
        String pong = jedis.ping();
        //如果连接成功会返回一个PONG
        System.out.println("连接成功："+pong);
        //操作完成之后关闭连接
        jedis.close();
    }
}
```

#### 3.2 常用的命令

**set/get/keys \*/mset/mget**

```java
jedis.set("k1", "v1");
jedis.set("k2", "v2");
jedis.set("k3", "v3");
Set<String> keys = jedis.keys("*");
String k1 = jedis.get("k1");
jedis.mset("str1","v1","str2","v2","str3","v3");
List<String> keys = jedis.mget("k1", "k2", "k3");
```

命令以方法的形式出现，使用方式与在Redis命令行中的形式相同，在方法中传入想要输入的数据的字符串形式

#### 3.3 Jedis使用实例

完成一个手机验证码功能

1、输入手机号，点击发送后随机生成6位数字码，2分钟有效

2、输入验证码，点击验证，返回成功或失败

3、每个手机号每天只能输入3次



## 六、Redis事务

### 1. Redis的事务定义

Redis事务是一个单独的隔离操作：事务中的**所有命令都会序列化、按顺序地执行**。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

Redis事务的主要作用就是**串联多个命令**，防止别的命令插队

### 2. Redis事务命令

**Multi**：进入事务模式，将之后输入的命令一次放入命令队列中，但不会执行

**Exec**：按顺序执行命令队列中的命令

**discard**：放弃组队，将之前加入命令队列中的命令舍弃

![](C:\Users\86198\Desktop\JavaEE\Redis\image\Multi-Exec-discard.png)

| 案例：                                                       |
| ------------------------------------------------------------ |
| <img src="C:\Users\86198\Desktop\JavaEE\Redis\image\RedisTX_1.jpg" style="zoom:150%;" /> |
| <img src="C:\Users\86198\Desktop\JavaEE\Redis\image\RedisTX_2.jpg" style="zoom:150%;" />组队阶段报错，提交失败 |
| <img src="C:\Users\86198\Desktop\JavaEE\Redis\image\RedisTX_3.jpg" style="zoom:150%;" />组队成功，但是其中命令有执行失败的情况，不影响其他正确命令的执行 |

### 3. 事务的错误处理

+ 组队时某个命令出现了报告错误，执行时整个的所有队列都会被取消

  ![](C:\Users\86198\Desktop\JavaEE\Redis\image\MultiError.jpg)

+ 如果执行阶段某个命令执行失败报错，则只有报错的命令不会被执行，其他命令都会执行，不会回滚(与关系数据库中不同)

  ![](C:\Users\86198\Desktop\JavaEE\Redis\image\ExecError.jpg)

### 4. 事务冲突问题

场景：账户中总共有10000元，有三个请求同时发出：

一个请求想给金额减8000

一个请求想给金额减5000

一个请求想给金额减1000

![](C:\Users\86198\Desktop\JavaEE\Redis\image\RedisTXConflict.jpg)

如果不采用锁机制，就会出现最后余额成为负数的情况

#### 4.1 悲观锁

**悲观锁(Pessimistic Lock)**，正如其名，具有强烈的独占和排他特性。它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此，**在整个数据处理过程中，将数据处于锁定状态**。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。效率比较低

![](C:\Users\86198\Desktop\JavaEE\Redis\image\Pessimistic Lock.jpg)

#### 4.2 乐观锁

**乐观锁(Optimistic Lock),** 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis就是利用这种check-and-set机制实现事务的。

![](C:\Users\86198\Desktop\JavaEE\Redis\image\Optimistic Lock.jpg)

#### 4.3 watch命令

Redis中，在Multi命令之前可以输入watch \<key1>[key2]...命令监视一个或多个key，如果**在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。**

**unwatch**命令，取消watch命令对所有key的监视，如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。

#### 4.4 Redis事务三特性

+ **单独的隔离操作**

  事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

+ **没有隔离级别的概念**

  队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

+ **不能保证原子性**

  事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

#### 4.5 Redis事务案例----秒杀



## 七、Redis持久化

由于Redis的数据都存放在内存中，如果没有配置持久化，redis重启后数据就全丢失了，于是需要开启redis的持久化功能，将数据保存到磁盘上，当redis重启后，可以从磁盘中恢复数据。

Redis提供了2个不同形式的持久化方式

+ **RDB(Redis DataBase)**
+ **AOF(Append Only File)**

### 1. RDB

#### 1.1 RDB简介

在**指定时间间隔内**将内存中的数据集快照写入磁盘，快照，可以理解为将当前数据集拍成一张照片保存下来

实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。

#### 1.2 RDB备份操作

Redis会单独创建（fork）一个子进程来进行持久化，**会先将数据写入到一个临时文件中**，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**RDB的缺点是最后一次持久化后的数据可能丢失**。

![](C:\Users\86198\Desktop\JavaEE\Redis\image\RDB.png)

*补充：fork*

+ Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程
+ 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了**“写时复制技术”**
+ 一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

#### 1.3 dump.rdb文件

RDB操作时默认的临时文件名，持久化数据，在redis的安装文件夹下

**RDB快照触发方式**

redis.conf中的默认快照配置

*save 900 1*      #如果在900s内有一个key被修改了，就进行持久化操作

*save 300 10*  #如果在300s内有10个key被修改了，就进行持久化操作

*save 60 10000*  #如果在60s内有10000个key被修改了，就进行持久化操作

save / bgsave命令触发

save：该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，直到RDB过程完成为止。

![](C:\Users\86198\Desktop\JavaEE\Redis\image\save.jpeg)

bgsave：执行该命令时，Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。

![](C:\Users\86198\Desktop\JavaEE\Redis\image\bgsave.jpeg)

#### 1.4 RDB总结

![](C:\Users\86198\Desktop\JavaEE\Redis\image\RDB.jpg)

### 2. AOF

#### 2.1 AOF简介

AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，**读操作不会记录**，以文本的方式记录，可以打开文件看到详细的操作记录，**只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

![](C:\Users\86198\Desktop\JavaEE\Redis\image\AOF.png)

#### 2.2 AOF持久化流程

（1）客户端的请求写命令会被append追加到AOF缓冲区内；

（2）AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；

（3）AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；

（4）Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的

*注意：在redis的默认配置中 AOF持久化操作不开启，只开启 RDB持久化操作，当 AOF和 RDB同时开启时，AOF的优先级高*

#### 2.3 AOF的启动和修复

AOF的启动：修改配置文件

修改默认的appendonly no，改为yes，并重启Redis

AOF的修复：

如遇到**AOF文件损坏**，通过/usr/local/bin/**redis-check-aof--fix appendonly.aof**进行恢复，并重启Redis

