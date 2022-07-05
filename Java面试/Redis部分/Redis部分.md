# Redis部分

## 一、Redis传统五大数据类型

### String

最常用的命令：

```
set k1 v1
get k1
```

同时设置 / 获取多个键值

```
mset k1 v1 k2 v2 ...
mget k1 k2 ...
```

数值的递增

```
set k1 100
incr k1  #+1
decr k1  #-1
increby k1 10  # +10
decrby k1 10  #-10
```

获取长度

```
strlen k1
```

应用场景：

+ 点赞，网站访问次数等可以使用`incr`命令，点击一次即+1



### List

双向链表，添加元素：

```
lpush key v1 v2 v3 ... #从链表的左边添加元素
rpush key v1 v2 v3 ... #从链表的右边添加元素
```

![image-20220627212818162](Redis部分.assets/image-20220627212818162.png)

list的一个特点就是一对多，应用场景：

现在几乎每个平台都会有订阅功能或者收藏功能，比如b站的关注up主

```
#表示xxxx号用户关注了3个up主
lpush like_up_loader:userId:xxxx upId1 upId2 upId2
#查看up主的消息
lrange like_up_loader:userId:xxxx 0 -1
```

### Hash

redis中的hash数据结构对应Java中的`Map<String, Map<String, Object>>`

设置字段

```
hset key field value  #一次设置一个
hmset key filed1 value1 field2 value2 ... #一次性设置多个
```

获取字段值

```
hget key field
hmget key field1 field2 ...
hgetall key  #一次性获取key的所有字段
```

应用场景：购物车（简单版）

```
#记录商品，表示xxxx用户将一个商品id为prodId的商品加入了购物车
hset shopping-cart-userId:xxxx prodId 1

#hincrby将指定的数值字段的值增加指定数目
#在这里表示用户xxxx又添加了一个prodId商品
hincrby shopping-cart-userId:xxxx prodId 1
```

![image-20220627211852169](Redis部分.assets/image-20220627211852169.png)

```
#全选
hgetall shopping-cart-userId:xxxx

#总计
hlen shopping-cart-userId:xxxx
```

![image-20220627212154930](Redis部分.assets/image-20220627212154930.png)

### **Set**

Redis中的Set相当于Java中的HashSet，无序，无重复

```
#如果有重复的值会被自动除去
sadd key v1 v2 v3 ...
#删除元素
srem key v1 v2 ...
#获取集合中所有元素
smembers key
#判断元素是否在集合中
sismember key v1
#获取集合中的所有元素
scard key
#从集合中【随机】弹出一个元素，【不删除】
srandmember key [count]   #如果不加count就默认随机弹出一个，如果加了count就是随机弹出count个
#从集合中随机删除一个元素
spop key [count]
```

示例：

![image-20220628210952778](Redis部分.assets/image-20220628210952778.png)

**set的集合运算**

```
#集合差运算 
sdiff set1 set2 [...]
#集合交运算
sinter set1 set2 [...]
#集合并运算
#sunion set1 set2 [...]
```

示例：

![image-20220628211520336](Redis部分.assets/image-20220628211520336.png)

**应用场景**

（1）抽奖程序

+ 直观的操作：`spop key` 和 `srandmember key`都是随机弹出一个或者多个，这就可以用来做抽奖程序，因为是随机的
+ 一般抽奖都会让用户决定是否参与，如果用户点击【立即参与】按钮，则后台调用`sadd choujiang:idxxxx userId`将用户加入抽奖名单
+ 一般页面也会显示有多少人参与抽奖，`scard key`统计总数

（2）点赞程序

+ 之前说过点赞统计可以用string去做，set也能做，并且能比string做的更好
+ 当在社交软件上发布文章或者其他的东西，其他用户点赞，`sadd userId:xxx:article:xxx otherUser1 otherUser2 ...`
+ 如果有用户发现误点了，取消点赞，`srem userId:xxx:article:xxx oneUser`
+ 展示所有点赞的用户头像，`smembers userId:xxx:article:xxx`然后去查数据库或者直接查缓存
+  判断某个用户是否给我点赞过，`sismember userId:xxx:article:xxx oneUser`

（3）共同兴趣

+ 比如微博共同关注的人，`sadd user1 interest1 interest2 ...` `sadd user2 interest1 interest2 ...`
+ 相同的关注，`sinter user1 user2`，可以做大数据的喜好推荐
+ 有谁跟我一起关注了某个人，`sismember user1 oneUser  sismember user2 oneUser`如果都为true，则user2也关注了onUser这个人

（4）可能认识的人

+ 比如我的好友有1 2 3 4 5 ==> `sadd user1 1 2 3 4 5`
+ 我的某一名好友有 3 4 5 6 7 ==> `sadd user2 3 4 5 6 7`
+ 我可能认识的人 ==> `sdiff user2 user1` 就是我的好友的好友，但不在我的好友列表里的人

### Zset（sorted set）

有序的集合，向集合中加入一个元素和该元素的分数，按照分数大小对元素进行排序

```
#向有序集合中添加元素，携带分数信息
zadd key score1 v1 score2 v2 ...
#获取集合中的指定下标的元素，可以选择是否携带分数信息
zrange key start stop [withscores]
#获取集合中某个元素的分数信息
zscore key member
#删除指定元素
zrem key v1 v2 ... 
#获取指定分数范围内的元素
zrangebyscore key min max [withscores]
#获取指定集合中元素的数量
zcard key
#获取指定分数范围内元素的数量
zcount key min max
#获取元素的排名，从小到大排列，（下标从0开始）
zrank key member
#获取元素的排名，从大到小排列
zrevrank key member
```

示例：

![image-20220629153520770](Redis部分.assets/image-20220629153520770.png)

应用场景：

（1）根据商品的销售量对商品进行排序显示

+ `zadd goods:sell:sort volume1 prodId1 volume2 prodId2 ...` 将商品的销售量作为分数，商品id作为元素加入zset中
+ 如果客户又买了number件商品prodId，`zincrby goods:sell:sort number prodId`
+ 获取销售量的前10名，`zrange goods:sell:sort 0 9 [withscroes]`，一般都要带上withscores参数

（2）热搜排名

+ 一条新闻加入zset，`zadd news:hot:sort number1 newsId1`，以点击量作为分数
+ 用户查看新闻（点赞，评论等），`zincrby news:hot:sort newsId`
+ 热搜前几名，`zrange news:hot:sort 0 rank withscores`

## 二、分布式锁

### 分布式锁的概念

+ 分布式微服务架构下，将各个服务拆分之后，为避免各个服务之间的冲突和数据故障而加上的一种锁
+ 与JVM层面上的锁（synchronized、cas，lock等）不同

**分布式锁的实现**

（1）MySQL

（2）zookeeper

（3）redis，现在的主流



**本地锁在分布式情况下的问题**

因为每个服务被部署到了多台服务器下，本地锁只能锁住转发到当前服务器的请求，但并不能锁住转发到其他服务器的请求，所以还是会有多个线程去访问数据库

### 分布式锁的引入

现在有多个服务要去查询数据库，可以让这些服务去竞争一把锁，这把锁需要是全局唯一的，这个全局唯一是指在微服务情况下全局唯一的

当某个服务竞争到了这把锁，即可执行后面的业务逻辑，否则就必须等待

这个锁可以是全局的MySQL数据库，也可以是redis缓存，也可以去每一个服务都能访问的地方

这里讨论使用redis作为锁的情况

![image-20220629164125085](Redis部分.assets/image-20220629164125085.png)

**分布式锁的第一阶段**

使用redis作为分布式锁，可以这样做，每一个服务先去访问redis，向redis中set一个key，比如说lock

只有第一个set成功的服务可以去访问数据库，其他服务都需要等待

分布式锁的基本思想就是基于这个的，使用redis的set lock value nx（nx表示如果不存在这个key，则set，是一个原子操作，如果多个客户端执行，只会执行成功一个，也可以直接使用setnx命令）命令去占锁，如果set成功，则表示获得了这把锁，把这些对应到代码上就是

```java
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "value");  //value可以任意

if(lock) {
    //表示加锁成功
    执行查询数据库的逻辑
    //解锁逻辑，可以直接把redis中lock键删除
    redisTemplate.delete("lock");    
} else {
    //加锁失败，重新尝试加锁，自旋的方式，不断尝试加锁
    
}
```

分布式锁阶段一存在的问题：死锁问题

![image-20220629165928600](Redis部分.assets/image-20220629165928600.png)

**分布式锁阶段二**

为解决阶段一的死锁问题，可以在获取到锁之后再设置一个过期时间，那么这样即使在业务逻辑执行期间出现了问题，也不会不释放锁

映射到代码上就是

```java
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "value");  //value可以任意

if(lock) {
    //表示加锁成功
    //设置锁的过期时间
    redisTemplate.expire("lock", 30, TimeUnit.SECONDS);
    执行查询数据库的逻辑
    //解锁逻辑，可以直接把redis中lock键删除
    redisTemplate.delete("lock");    
} else {
    //加锁失败，重新尝试加锁，自旋的方式，不断尝试加锁
    
}
```

![image-20220629170212902](Redis部分.assets/image-20220629170212902.png)



但这样还是不能解决问题，因为如果在设置锁和设置过期时间的操作是分开来的，如果在设置锁的过期时间之前，服务器发生了宕机，那锁的过期时间还是没有设置上，那还是会造成死锁的问题

**分布式锁阶段三**

为了解决上面的问题，可以使用`setnx expireTime lock value`这样的设置值和过期时间一体的命令

映射到业务代码上就是：

```java
//Boolean redisTemplate.opsForValue().setIfAbsent("lock", "value");  //value可以任意
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "value", 30, TimeUnit.SECONDS);
if(lock) {
    //表示加锁成功
    //设置锁的过期时间和加锁必须是同步的、原子的
    //redisTemplate.expire("lock", 30, TimeUnit.SECONDS);
    执行查询数据库的逻辑
    //解锁逻辑，可以直接把redis中lock键删除
    redisTemplate.delete("lock");    
} else {
    //加锁失败，重新尝试加锁，自旋的方式，不断尝试加锁
}
```



![image-20220629170933018](Redis部分.assets/image-20220629170933018.png)

但阶段三仍然有它的问题，由于业务逻辑的不确定性，如果随机设置一个锁的过期时间，很有可能执行业务逻辑的时间比锁的过期时间要长，比如，锁的过期时间是10s，但执行业务逻辑的时间是30s，那么这会出现几个问题：

+ 当业务逻辑执行到一半，锁过期了，redis自动把锁删了，那么其他正在等待锁的线程就能够获得这个锁，并再次进入到业务逻辑中，这就造成了，还是会有多个线程进入业务逻辑代码的情况
+ 然后，等到第一个服务把业务逻辑执行完了，已经过去30s了，现在redis内lock锁并不是第一个服务原来获取的锁，如果要执行删锁操作，删掉的就是其他服务持有的锁，那其余一些等待锁释放的服务也能够进入到业务逻辑代码中

**分布式锁阶段四**

为了解决上面的问题，可以在设置锁的时候，给lock键所对应的值附上一个唯一标识，比较常用的就是UUID标识，映射到Java代码上就是：

```java
//Boolean redisTemplate.opsForValue().setIfAbsent("lock", "value");  //value可以任意
String uuid = UUID.randomUUID().toString();
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 30, TimeUnit.SECONDS);
if(lock) {
    //表示加锁成功
    //设置锁的过期时间和加锁必须是同步的、原子的
    //redisTemplate.expire("lock", 30, TimeUnit.SECONDS);
    执行查询数据库的逻辑
    //解锁时，先判断lock中的值是否是自己的值
    String lockValue = redisTemplate.opsForValue().get("lock");
    if(uuid.equals(lockValue)) {
        //删除我自己的锁
        redisTemplate.delete("lock");
    }    
} else {
    //加锁失败，重新尝试加锁，自旋的方式，不断尝试加锁
    
}
```



![image-20220629172320283](Redis部分.assets/image-20220629172320283.png)

但这个阶段仍然存在问题，现在假设锁的过期时间是10s，某个服务执行业务逻辑的时间是9.5s，然后向redis发送一个get命令，由于网络传输也有时间，假设是0.3s，那么这个命令发到redis的时候，锁已经存在9.8s，还有0.2s就要过期了，但获取到值之后，仍然要传回服务器，假设还需要花掉0.5s，那么在这个返回结果的过程中，redis中lock的值就过期了，接下来获取到锁的就是另一个服务，但在之前服务判断中就发生了误判，还是会删掉别人的锁导致没锁柱的情况

**分布式锁最终阶段**

为了解决上面的问题，在redis的官方文档里给出了解决方案：https://redis.io/commands/set/

使用lua脚本解决删除的问题，下面是redis官方给的一段lua脚本

```lua
if redis.call("get",KEYS[1]) == ARGV[1]
then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

把这些映射到代码上就是：

```java
//Boolean redisTemplate.opsForValue().setIfAbsent("lock", "value");  //value可以任意
String uuid = UUID.randomUUID().toString();
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 30, TimeUnit.SECONDS);
if(lock) {
    //表示加锁成功
    //设置锁的过期时间和加锁必须是同步的、原子的
    //redisTemplate.expire("lock", 30, TimeUnit.SECONDS);
    执行查询数据库的逻辑
    //解锁时，使用lua脚本删除redis中的key
    String script = "if redis.call(\"get\",KEYS[1]) == ARGV[1] then return redis.call(\"del\",KEYS[1]) else return 0 end";
    //redis执行lua脚本的代码，泛型是返回值的类型
    redisTemplate.execute(new DefaultRedisScript<Integer>(script, Integer.class), Arrays.asList("lock"), uuid);
} else {
    //加锁失败，重新尝试加锁，自旋的方式，不断尝试加锁
}
```



![image-20220629173443333](Redis部分.assets/image-20220629173443333.png)





## 附录 Redis作为缓存

为了提升系统的性能，一般会将一些数据放到缓存中，加速访问，而数据库只承担数据持久化的功能做

**哪些数据适合放到缓存**

+ 即时性，数据一致性要求不高的数据
+ 访问量大且更新频率不高的数据（读多，写少）

缓存使用的一般流程

![image-20220629160145686](Redis部分.assets/image-20220629160145686.png)

最简单的缓存就是一个Map集合，称为本地缓存，本地缓存如果在单机模式下不会有任何问题（如果保证线程安全的话）

但在分布式情况下会有严重的问题，由于分布式微服务的架构，某一个服务会部署到多个服务器下

![image-20220629160745401](Redis部分.assets/image-20220629160745401.png)

（1）缓存失效的问题，因为如果查询数据，每一次数据都保存在本地缓存中，也就是某一台服务器的缓存中，但是如果下一次的请求被转发到了另一台服务器，那台服务器上的缓存并没有数据，所以还是要查询数据库

（2）数据一致性的问题，如果某次数据更新的请求被转发到某一台服务器上，这台服务器更新数据库并更新缓存数据，但下一次查询请求被转发到了另一台服务器上，那台服务器上的缓存并没有被更新，还是旧数据，所以会出现数据不一致的问题

**分布式缓存**

由于本地缓存会产生各种各样的问题，所以，在分布式的情况下就需要使用分布式缓存，各个服务器共享一个缓存中间件，一般是redis

分布式缓存还有一个好处就是可以设置集群，扩大了缓存容量，也可以达到高可用的目的	

![image-20220629161210343](Redis部分.assets/image-20220629161210343.png)



**redis缓存的问题**

（1）缓存穿透

指查询一个一定不存在的数据（指数据库中也不存在），由于缓存中没有这些数据，那么这些请求将会去访问数据库，但数据库中也没有这些数据，也就不能记录到缓存中，之后所有请求都会重复这样的步骤，全部打到数据库上，缓存就用不上了

如果有大量这样的请求，会导致数据库崩溃

解决方法：可以缓存空数据，并设置一个过期时间

![image-20220629161939773](Redis部分.assets/image-20220629161939773.png)

（2）缓存雪崩

指在设置缓存时key采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到数据库

解决方法：在原有的失效时间基础上增加一个随机值，降低缓存失效时间的重复率

![image-20220629162455930](Redis部分.assets/image-20220629162455930.png)

（3）缓存击穿

对于一些设置了过期时间的key，如果这些key可能会在某些时间被超高并发地访问，这些数据就被称为热点数据

如果这个key在大量请求同时进来的前正好失效，那么所有对这个key的数据查询都会落到数据库上，导致数据库崩溃

解决方法：加锁，让大量并发只有一个请求去查数据库，其他请求暂时等待，等那个请求查到数据之后，放到缓存中，其他人获取倒锁，查询缓存，并返回数据

![image-20220629162813613](Redis部分.assets/image-20220629162813613.png)
