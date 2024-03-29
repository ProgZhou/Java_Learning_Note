# MySQL部分

### SQL执行流程

**MySQL查询执行流程**

+ `查询缓存`，服务器如果在查询缓存中发现了这条SQL语句，就会直接将结果返回给客户端；如果没有，就进入到解析器阶段

  + 缓存的命中率很低，在MySQL8.0之后抛弃了这个功能
  + 因为两个查询请求完全相同才会命中，如果在任何字符串上的不同（比如加空格，注释，大小写）都会导致缓存不命中

+ `解析器`，在解析器当中，对SQL语句做语法，语义分析；SQL分析分为词法分析和语法分析

  + 分析器先做词法分析，MySQL首先需要识别出SQL语句中的字符串
  + 接着做语法分析，根据词法分析的结果，判断当前SQL语句是否符合MySQL定义的语法
  + 如果通过了上面的两个分析，则会生成一棵语法树

  <img src="MySQL部分.assets/image-20220720200647521.png" alt="image-20220720200647521" style="zoom:80%;" />

+ `优化器`，在优化器中确定SQL语句的执行路径，比如是全表检索还是索引检索等

  + 一条查询可以有多种执行方式，最后返回的结果都相同，优化器的作用就是找到其中最好的执行计划
  + 比如，表中有多个索引，决定使用哪个索引；或者在一个语句有多表关联的时候，决定各个表的连接顺序；还有表达式简化，子查询转换为连接等等
  + 最后会生成一个执行计划

+ `执行器`，负责SQL语句的最终执行，在执行之前会首先判断一下用户是否具备权限，如果没有则报错；如果有，就执行SQL语句并返回结果

  + 如果有相应的执行权限，就会打开表继续执行，此时会调用存储引擎的API对表进行读写

<img src="MySQL部分.assets/image-20220720200632041.png" alt="image-20220720200632041" style="zoom:80%;" />





### 存储引擎

存储引擎简而言之就是表的类型，主要功能是接收上层传下来的指令，然后对表中的数据进行提取或写入操作

> 在MySQL8.0之后，默认的存储引擎为InnoDB，最大的特点就是支持数据库事务和分布式事务
>
> MySQL8.0之前是MyISAM

**InnoDB存储引擎**

+ 具备外键支持功能的`事务存储引擎`，可以确保事务的完整提交和回滚
+ 是MySQL5.5版本之后默认的事务型存储引擎，可以确保事务的完整提交和回滚
+ 对比MyISAM，InnoDB写的处理效率较差，并且会占用更多的磁盘空间以保存数据和索引
+ InnoDB存储引擎使用的索引与数据在同一个文件中，即有一个聚簇索引
+ InnoDB使用二级索引的时候的回表操作会比MyISAM的回表慢一些
+ InnoDB是为处理巨大数据量的最大性能设计的

**MyISAM存储引擎**

+ 主要的非事务处理存储引擎
+ MyISAM不支持事务，行级锁和外键，崩溃后无法安全恢复
+ 优势是访问比较快，5.5版本之前的默认存储引擎，对事务完整性没有要求或者以select和insert为主
+ MyISAM存储引擎使用的索引与数据分开，分别存储于两个不同的文件中，叶子结点存储的是数据的物理地址，而不是数据本身，相当于MyISAM中的索引全是二级索引
+ MyISAM的索引最后都有数据的物理地址，回表时仅需要通过物理地址直接定位到数据即可，一般都比较快
+ 主要应用于只读业务

<img src="MySQL部分.assets/image-20220720203141621.png" alt="image-20220720203141621" style="zoom:80%;" />

### 聚簇索引

聚簇索引是一种数据存储方式（所有的用户记录都存储在了叶子结点），也就是所谓的**索引即数据，数据即索引**

聚簇索引的特点：

+ 使用记录主键值的大小进行记录和页的排序：
  + 数据页内的记录按照主键的大小顺序排成一个`单向链表`
  + 各个存放`用户记录的页`也是根据页中用户记录的主键大小顺序拍成一个双向链表
  + 存放目录项记录的页分为不同的层次，在同一层次中的页也是根据页中目录项记录的主键大小顺序排成一个双向链表
+ 聚簇索引B+数的`叶子结点`存储的是一条完整的用户记录
  + 完整的用户记录指的是这个记录存储了所有列的值

InnoDB存储引擎会自动创建聚簇索引

优点：

+ `数据访问更快`，因为聚簇索引将索引和数据同时保存在B+树中，因此聚簇索引中获取数据比非聚簇索引更快
+ 聚簇索引对于主键的`排序查找`和`范围查找`速度非常快
+ 按照聚簇索引排序顺序，查询显示一定范围数据的时候，由于数据都是紧密相连的，数据库不用从多个数据块中提取数据，所以`节省了大量的IO操作`

缺点：

+ 向表中插入数据的速度严重依赖于插入顺序，如果乱序插入会导致页分裂，需要移动数据项，严重影响性能
+ 更新主键的代价很高，会导致被更新的行的数据项移动

限制：

+ MySQL数据库目前只有InnoDB支持聚簇索引，而MyISAM不支持聚簇索引
+ MySQL的表中只能有一个聚簇索引，一般情况下是该表的主键
+ 如果没有定义主键，InnoDB会选择一个`非空且唯一的索引`代替主键；如果没有这样的索引，InnoDB会隐式的定义一个主键

### 非聚簇索引

非聚簇索引B+树的叶子结点并不存储完整的用户记录，如果在表中以c2列的值创建了一个索引：

+ 数据页内的记录按照c2列的大小顺序排成一个单向链表
+ 各个存放用户记录的页也是根据页中c2的大小排成一个双向链表

**回表操作**：由于非聚簇索引叶子结点只存放了c2列和主键的值，所以类似于select * from table where c2 = xxx这样的查询语句不能只查询以c2为索引创建出来的，需要首先沿着非聚簇索引查询到c2=xxx这行的主键值，再根据得到的主键值去查询聚簇索引得到这行的完整数据，这就是回表操作

一张数据表可以有多个非聚簇索引

> **聚簇索引和非聚簇索引的区别**
>
> + 聚簇索引的叶子节点存储的是数据记录，非聚簇索引的叶子节点存储的是数据位置，非聚簇索引不会影响数据表的物理存储顺序
> + 一个表只能有一个聚簇索引，因为只能有一种物理存储方式，可以有多个非聚簇索引
> + 使用聚簇索引时，数据的查询效率比较高，但如果对数据进行插入、删除和更新等操作，效率比非聚簇索引低

### 其他索引结构

**Hash索引**

hash本身是一个散列函数，可以大幅度提高数据检索效率

hash算法是通过某种确定性算法将输入转换为输出，`相同的输入永远会得到相同的输出`

采用hash进行数据查询时的效率非常高，基本上一次检索就可以找到数据，复杂度为O(1)

hash索引的缺陷：

+ Hash索引只能满足 =、<>（不等于）和IN这样的等值查询，如果进行范围查询，哈希型的索引，时间复杂度救护退化为O(n)
+ Hash数据存储没有顺序，如果进行排序查询（order by）哈希还需要对数据进行重新排序
+ 如果索引列的重复值比较多，hash冲突的情况会变多，这时查找关键字就会非常耗时

目前仅Memory存储引擎支持hash索引，MyISAM和InnoDB都不支持hash索引

> InnoDB支持一个`自适应Hash索引`，比如如果某条数据经常被使用，当满足一定条件的时候，就会将这个数据页的地址存放到hash表中，这样，下次查询时会直接找到这个页的地址

**平衡二叉树索引**

平衡二叉树是一棵空树，或者它的左右子树的高度差的绝对值不超过1，并且左右两个子树都是平衡二叉树

<img src="MySQL部分.assets/image-20220721162952541.png" alt="image-20220721162952541" style="zoom:80%;" />

平衡二叉树每访问一次结点都需要进行一次磁盘IO，对于上面的树来说，就需要进行5次IO操作，随着记录的增多，树的深度慢慢增加，IO次数增多也会影响整体查询效率，所以可以考虑将二叉树改成M叉树，降低树的高度

**B树索引**

B树全称多路平衡查找树，它的高度远小于平衡二叉树的高度

<img src="MySQL部分.assets/image-20220721163702511.png" alt="image-20220721163702511" style="zoom:80%;" />

B树的每一个结点最多可以包括M个子节点，M就称为B树的阶，每个磁盘块中包括关键字和子节点指针

B树在插入和删除结点的时候如果导致树不平衡，就通过自动调整结点的位置来保持树的自平衡

关键字集合分布在整棵树中，即叶子结点和非叶子节点都存放数据，搜索有可能在非叶子结点结束

搜索的性能相当于在一个完整表数据中做一次二分查找

<img src="MySQL部分.assets/image-20220721163949592.png" alt="image-20220721163949592" style="zoom:80%;" />

### 索引设计原则

**1. 字段的数值有唯一性的限制**

索引本身可以起到约束作用，比如唯一索引、主键索引都可以起到唯一性约束，因此在数据表中，如果某个字段是唯一的，就可以创建唯一性索引，这样可以快速地通过该索引来确定某条记录

> 《阿里巴巴Java开发手册》中规定了：业务上具有唯一性索引的字段，即使是组合字段，也必须创建唯一索引

**2. 频繁作为Where查询条件的字段**

某个字段在SELECT语句的WHERE条件中经常被使用到，那么就需要给这个字段创建索引了，尤其是在数据量大的情况下，创建`普通索引`就可以大幅度提升数据查询效率

**3. 经常Group By和Order By的列**

索引就是让数据按照某种顺序进行存储检索，因此当我们使用Group By对数据进行分组查询，或者使用Order by对数据进行排序时，就需要对分组或者排序的字段进行索引，如果排序的列有多个，那么可以在这些列上建立索引

**4. DISTINCT字段需要创建索引**

如果要去去重，使用索引之后，值相同的数据经过B+树排序之后都在一起，去重效率自然就提高了

**5. 多表Join连接操作时，创建索引的注意事项**

首先，连接的表最好不要超过3张，因为每增加一张表就相当于增加了一次嵌套循环，数量级增长会非常快，严重影响查询效率

其次，对where条件创建索引，因为where才是对数据条件的过滤，如果在数量非常大的情况下，没有where条件过滤效率比较低

最后，对于连接的字段创建索引，并且该字段在多张表的类型必须一致，类型的转换会导致索引失效

**6. 使用列的类型小的创建索引**

类型大小指的是该类型表示的数据范围的大小，也就是占存储空间的大小

原因：

+ 数据类型越小，在查询时进行的比较操作就越快
+ 数据类型越小，索引占用的存储空间就越少，在一个数据页中就能放下更多记录，从而减少磁盘IO带来的性能损耗

**7. 使用字符串前缀创建索引**

如果字符串很长，那存储一个字符串就需要占用很大的存储空间，会引起几个问题：

+ B+树索引中的记录需要把该列的完整字符串存储起来，费时也浪费空间
+ 如果B+树中索引列存储的字符串很长，在做字符串比较时，只能一个一个比，性能也不好

所以，可以截取字段的前面一部分内容建立索引，这就叫字符串前缀索引

具体取前多少个字符，可以计算一下区分度：`count(distinct left(字符串字段, 截取长度)) / count(*)`比较各个前缀长度的区分度，取最接近1的那个

> 《阿里巴巴Java开发手册》中规定，在varchar字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度

**8. 区分度高的列适合创建索引**

区分度可以根据之前的公式计算，如果区分度高，在筛选的时候比较容易定位到数据

**9. 使用最频繁的列放到联合索引的左侧**

最左前缀原则

**10. 在多个字段都要创建索引的情况下，联合索引优于单值索引**



> **不适合创建索引的情况**
>
> + 在where、order by、group by中基本使用不到的字段不适合创建索引
> + 数据量小的表不适合创建索引（小于1000条）
> + 有大量重复的列上不适合创建索引
> + 经常更新的列不适合创建索引
> + 没有规律的列不适合创建索引
> + 删除不再使用或者很少使用的索引
> + 不要定义冗余的索引

### 索引失效的几种情况

通常对数据量比较大的表中的某些字段设置索引，但查询最后用不用索引，最终都是优化器说了算，优化器是基于cost开销（不是时间），怎么开销小怎么来

**1. 全值匹配**

```mysql
select * from table where col1 = xxx and col2 = xxx and col3 = xxx
使用的索引能够覆盖到的条件越多越好，即idx_col1_col2_col3
如果创建部分索引，即idx_col1_col2也能够使用到，但效果没有全值匹配的好
```

**2. 最左前缀原则**

```mysql
select * from table where col2 = xxx and col3 = xxx
创建索引使用的字段顺序很重要，像上面的查询语句，索引idx_col1_col2_col3(col1, col2, col3)不会起作用
因为整个联合索引排序的规则是先按照col1排序，在col1相等的情况下按照col2排序，最后再按照col3排序，如果查询中没有col1，那么这个联合查询也就不起作用了
select * from table where col1 = xxx and col3 = xxx
这个查询语句能够使用上idx_col1_col2_col3索引，但没有完全用上，只用上了col1的字段
```

MySQL可以为多个字段创建索引，一个索引可以包括16个字段，对于多列索引，**过滤条件要使用索引必须按照索引建立时的顺序，依次满足，一旦跳过某个字段，索引后面的字段都无法被使用**；如果查询条件中没有使用这些字段中第一个字段时，多列索引不会被使用



**3. 计算、函数、类型转换(自动或手动)导致索引失效**

**4. 范围条件右边的列索引失效**

```mysql
select * from table where col1 = 1 and  col2 > 2 and col3 = 3
索引idx_col1_col2_col3(col1, col2, col3)只会使用索引的前两个字段，col3字段并不会使用
所以设计索引的时候，尽量把确定的等值比较的字段放在前面
```

**5. 不等于(!= 或者<>)索引失效**

**6. is null可以使用索引，is not null无法使用索引**

**7.  like以通配符%开头索引失效**

> 索引是否会失效并不是绝对的，判断是否使用索引还是查询优化器决定的，不是说如果SQL语句满足上面的一些条件，索引就一定会失效，最终还是查询优化器计算成本决定的

### 查询优化

数据库调优的维度：

+ 索引失效，没有充分利用到索引  --- 索引优化
+ 关联查询太多JOIN，设计缺陷或不得已的需求  ---  SQL优化
+ 服务器调优及各个参数设置（缓冲，线程数等）  ---   调整my.cnf（配置文件）
+ 数据过多，IO瓶颈   ---   分库分表

数据库调优的目标：**响应时间更快，吞吐量更大**

一些常用的系统性能参数：

<img src="MySQL部分.assets/image-20220722153849999.png" alt="image-20220722153849999" style="zoom:80%;" />

**慢查询日志**

MySQL的慢查询日志，用来记录在MySQL中`响应时间超过阈值`的语句，具体指运行时间超过`long_query_time`值的SQL，则会被记录到慢查询日志中，`long_query_time`这个值默认是10，意思是运行10s以上的语句就认为是慢查询

慢查询日志的主要作用就是帮助我们发现那些执行时间特别长的SQL查询，并且有针对性地进行优化，从而提高系统的整体效率；MySQL默认是关闭慢查询日志的

```mysql
mysql> set global slow_query_log = on;   #开启慢查询日志功能
mysql> set global long_query_time = 1;   #修改慢查询阈值
```

除了`long_query_time`这个变量，MySQL中控制慢查询的还有一个系统变量: `min_examined_row_limit`，这个变量的意思是，查询扫描过的最少记录数，这个变量和查询执行时间，共同组成了判断一个查询是否是慢查询的条件

**慢查询分析工具**

`mysqldumpslow`

mysqldumpslow 命令的具体参数：

+ -a: 不将数字抽象成N，字符串抽象成S 
+ -s: 是表示按照何种方式排序： 
  + c: 访问次数 
  + l: 锁定时间 
  + r: 返回记录 
  + t: 查询时间 
  + al:平均锁定时间 
  + ar:平均返回记录数 
  + at:平均查询时间 （默认方式） 
  + ac:平均查询次数 
+ -t: 即为返回前面多少条的数据；
+ -g: 后边搭配一个正则匹配模式，大小写不敏感的

**查看SQL执行成本**

`show profile`命令

常用参数：① ALL：显示所有的开销信息。 ② BLOCK IO：显示块IO开销。 ③ CONTEXT SWITCHES：上下文切换开 销。 ④ CPU：显示CPU开销信息。 ⑤ IPC：显示发送和接收开销信息。 ⑥ MEMORY：显示内存开销信 息。 ⑦ PAGE FAULTS：显示页面错误开销信息。 ⑧ SOURCE：显示和Source_function，Source_file， Source_line相关的开销信息。 ⑨ SWAPS：显示交换次数开销信息。

**EXPLAIN查看执行计划**

```mysql
explain select * from table where xxx = xxx; #后面可以跟update, delete, insert等语句，不会真的执行，只是查看查询计划
或者
describe select * from table where xxx = xxx;  
```

explain执行之后结果字段的含义：

| 列名            | 描述                                                    |
| --------------- | ------------------------------------------------------- |
| `id`            | 在一个大的查询语句中每个SELECT关键字都对应一个 唯一的id |
| `select_type`   | SELECT关键字对应的那个查询的类型                        |
| `table`         | 表名                                                    |
| `partitions`    | 匹配的分区信息                                          |
| `type`          | 针对单表的访问方法                                      |
| `possible_keys` | 可能用到的索引                                          |
| `key`           | 实际上使用的索引                                        |
| `key_len`       | 实际使用到的索引长度                                    |
| `ref`           | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息  |
| `rows`          | 预估的需要读取的记录条数                                |
| `filtered`      | 某个表经过搜索条件过滤后剩余记录条数的百分比            |
| `Extra`         | 一些额外的信息                                          |

具体说明：

（1）`table`：查询的每一条记录都对应了一张单表，如果是连接查询，explain执行之后会有多条记录，一条记录对应一张表，比如：

```mysql
explain select * from s1 inner join s2;
```

放在前面的记录称为驱动表，后面的记录称为被驱动表

<img src="MySQL部分.assets/image-20220722161901656.png" alt="image-20220722161901656" style="zoom:80%;" />

这个表还包括临时表，由MySQL自动创建，用户感受不到

（2）`id`：在一个大查询中，每个select关键字都对应一个id

```mysql
explain select * from s1 inner join s2;   #虽然有两条记录，但id都为1
explain select * from s1 where col1 in (select * from s2);  #子查询就会有两条，因为有两个select关键字
```

查询优化器可能会对子查询进行重写，改写为连接查询，所以有时候子查询进行explain操作时，结果只有一个id值

+ id如果相同，可以认为是一组，从上往下顺序执行 
+ 在所有组中，id值越大，优先级越高，越先执行 
+ **关注点：id号每个号码，表示一趟独立的查询, 一个sql的查询趟数越少越好**

（3）`select_type`：一条大的查询语句里边可以包含若干个select关键字，每个select关键字代表着一个小的查询语句，而每个查询关键字的from子句中都可以包括若干张表（这些表用来做连接查询），每一张表都对应着执行计划输出中的一条记录，对于同一个select关键字中的表来说，它们的id值是相同的

MySQL为每一个select关键字代表的小查询都定义了一个称之为`select_type`的属性，意思是这个小查询在整个大查询中国扮演了什么样的角色

**（4）**`type`:执行计划的一条记录就代表着MySQL对某个表的执行查询时的访问方法，又称“访问类型”，其中的`type`列就表明了这个访问方法是什么，是较为重要的一个指标

完整的访问方法有：`system ， const ， eq_ref ， ref ， fulltext ， ref_or_null ， index_merge ， unique_subquery ， index_subquery ， range ， index ， ALL `

其中如果`type`是前面几个类型的，那么SQL执行的效率就比较高，越靠后，SQL执行效率就比较低

+ `system`：当表中只有一条记录，并且该表使用的存储引擎的统计数据是精确的，比如MyISAM，Memory，那么对该表的访问方法就是system

  举个例子：在MyISAM中，会使用一个变量直接记录表中的数据有多少条，这个system就代表着直接将变量拿来输出，这个效率是最高的

+ `const`：当使用主键或者唯一二级索引列与常数进行等值匹配时，对单表的访问就是`const`

  比如，id是主键`select * from table where id = 1`因为id是唯一的，在找到记录时就可以直接返回，不必要再去查找是否有其余相同的记录

+ `eq_ref`：在连接查询中，如果被驱动表是通过主键或者唯一索引等值匹配的方式进行访问的，则对被驱动表的访问方法就是`eq_ref`

+ `ref`：当通过普通的二级索引列与常量进行等值匹配时来查询某个表，那么对该表的访问方法就可能是`ref`

+ `ref_or_null`：就是在`ref`的基础上，匹配的列有可能是null值

> 结果值从最好到最坏依次是： `system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL` 其中比较重要的几个提取出来。SQL 性能优化的目标：至少要达到 range 级别，要求是 ref 级别，最好是 consts级别。（阿里巴巴 开发手册要求）

（5）` possible_keys`和`key`：` possible_keys`表示在某个查询语句中，对某个表执行单表查询时可能用到的索引有哪些，一般如果查询涉及到的字段上存在索引，则该索引将被列出，但不一定会用；` key`就表示实际用到的索引有哪些，如果为null，就没有使用索引



### 连接查询优化

**外连接**

```mysql
select * from table1 left join table2 on table.col1 = table2.col2;
table1称为驱动表，table2称为被驱动表
添加索引时，如果添加单个索引，需要添加给被驱动表，并且连接的字段类型要一样
```

外连接驱动表和被驱动表是确定的`left join / right join`之前的表是驱动表，后面的是被驱动表

**内连接**

```mysql
select * from table1 inner join table2 on tablw.col1 = table2.col1;
内连接中，两个表的地位是相同的，所以优化器可以决定那张表做驱动表，那张表作为被驱动表
```

如果只有一个表有索引，优化器就会将有索引的表作为被驱动表

对于内连接来说，在两个表的连接条件都存在索引的情况下，优化器会选择数据量较小的表作为驱动表，因为驱动表要全表扫描，数据量肯定是越小越好

**Join子句的原理**

（1）驱动表和被驱动表

驱动表就是主表，被驱动表就是从表

+ 对于内连接来说：`select * from A join B on ...`

  A表不一定是驱动表，查询优化器会根据查询语句做优化，决定先查哪张表，先查询的那张表就是驱动表，可以使用explain语句查看

+ 对于外连接来说：`select * from A left(right) / join B on ... `

  通常来说A是驱动表，B是被驱动表，但也会有例外，查询优化器可能会把外连接改造成内连接，这样谁是驱动表谁是被驱动表就是查询优化器说了算了

（2）简单嵌套循环连接

算法比较简单，从A表中取出一条数据a，遍历B表，将匹配到的数据放到一个集合中，以此类推；将A表中的每一条记录与被驱动表B的记录进行判断

<img src="MySQL部分.assets/image-20220724141618667.png" alt="image-20220724141618667" style="zoom:80%;" />

当然这种算法的效率是很低的：

| 开销统计         |           |
| ---------------- | --------- |
| 驱动表扫描次数   | 1         |
| 被驱动表扫描次数 | A         |
| 读取记录数       | A + B * A |
| join比较次数     | B * A     |
| 回表             | 0         |

（3）索引嵌套循环连接

这个算法的思路主要是为了减少内层表数据的匹配次数，要求**被驱动表**上必须有索引；通过外层表匹配条件直接与内层表索引进行匹配，避免和内层表的每条记录去进行比较，这样极大的减少了对内层表的匹配次数

<img src="MySQL部分.assets/image-20220724142401115.png" alt="image-20220724142401115" style="zoom:80%;" />

这个算法的效率会稍微高一些

| 开销统计         |                                                              |
| ---------------- | ------------------------------------------------------------ |
| 驱动表扫描次数   | 1                                                            |
| 被驱动表扫描次数 | 0（有索引，不必全表扫描）                                    |
| 读取记录数       | A + B（匹配记录数）                                          |
| join             | A * index(B+数的高度)（取一条A的记录在B表的B+树种比较，一条记录的比较次数就是B+树的高度） |
| 回表             | B（如果索引不是聚簇索引，就需要回表）                        |

（4）块嵌套循环连接

这是对之前简单嵌套的一个优化操作，不再是逐条获取驱动表的数据，而是一块一块的获取，引入了`join buffer缓冲区`，将驱动表相关的部分数据列缓存到`join buffer`中，然后全表扫描被驱动表，被驱动表的每一条记录一次性和`join buffer`中的所有驱动表记录进行匹配，将简单嵌套循环中的多次比较合并成一次，降低了被驱动表的访问频率

<img src="MySQL部分.assets/image-20220724143418529.png" alt="image-20220724143418529" style="zoom:80%;" />

相比于简单嵌套的全表扫描，块嵌套循环节省了IO次数：

| 开销统计         |                                                              |
| ---------------- | ------------------------------------------------------------ |
| 驱动表扫描次数   | 1                                                            |
| 被驱动表扫描次数 | A * used_col_size / join_buffer_size + 1（A表中待查询属性的大小 / buffer的大小） |
| 读取记录数       | A + B * 扫描次数                                             |
| join             | B * A                                                        |
| 回表             | 0                                                            |

块嵌套循环有几个参数：

+ `block_nested_loop`，表示是否开启块嵌套循环，默认是开启的
+ `join_buffer_size`，join缓存的大小，默认是256K，32位机器上限就是4G

**总结**：

+ 整体效果比较：索引嵌套优于块嵌套优于简单嵌套
+ 永远用**小结果集驱动大结果集**（本质就是减少外层循环的数据数量）
+ 为被驱动表匹配的匹配条件增加索引（减少内层表的循环匹配次数）
+ 增大join buffer的大小（一次缓存的数据越多，内层表的扫描次数就越少）
+ 减少驱动表不必要的字段查询（字段越少，join buffer所缓存的数据就会越多）

> MySQL8.0.20之后将抛弃块嵌套而采用hash join的方式

### 覆盖索引

索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据；当能通过读取索引就可以得到想要的数据，那就不需要读取行了。**一个索引包含了满足查询结果的数据就叫做覆盖索引。** 

非聚簇复合索引的一种形式，它包括在查询里的SELECT、JOIN和WHERE子句用到的**所有列** （即建索引的字段正好是覆盖查询条件中所涉及的字段）。

 简单说就是， 索引列+主键包含 SELECT 到 FROM之间查询的列 。即使用二级索引时，不需要回表操作，可以直接返回结果

**覆盖索引的利弊**

优点：

+ 避免Innodb表进行索引的二次查询（回表）
+ 可以把随机IO编程顺序IO，覆盖索引可以减少树的搜索次数，显著提升查询性能，所以覆盖索引是一个常用的性能优化手段

缺点：

+ 索引字段的维护是有代价的，因此建立冗余索引来支持覆盖索引时就需要权衡考虑了

### 索引下推

索引下推（Index Condition Pushdown）是MySQL5.6之后的新特性，是一种在存储引擎层使用索引过滤数据的一种优化方式，直接上实例：

```mysql
CREATE TABLE `people` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `zipcode` VARCHAR(20) COLLATE utf8_bin DEFAULT NULL,
  `firstname` VARCHAR(20) COLLATE utf8_bin DEFAULT NULL,
  `lastname` VARCHAR(20) COLLATE utf8_bin DEFAULT NULL,
  `address` VARCHAR(50) COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `zip_last_first` (`zipcode`,`lastname`,`firstname`)
) ENGINE=INNODB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb3 COLLATE=utf8_bin;


INSERT INTO `people` VALUES 
('1', '000001', '三', '张', '北京市'), 
('2', '000002', '四', '李', '南京市'), 
('3', '000003', '五', '王', '上海市'), 
('4', '000001', '六', '赵', '天津市');

EXPLAIN SELECT * FROM people WHERE zipcode='000001'
AND lastname LIKE '%张%'
AND address LIKE '%北京市%';
```

针对上面那个SQL语句，由于lastname字段使用了左模糊查询，所以，并不会使用这部分的索引，但zipcode却能够使用

索引下推的含义就是，由于lastname也属于索引的一部分，当查询开始时，首先根据二级索引（注意只会使用zipcode字段的索引）查找到相应的数据，由于二级索引中包含了lastname这个字段，所以，可以顺便去比较lastname字段是否符合 lastname like '%张%'这个条件，最后再去做回表操作

如果不使用索引下推，假设利用zipcode索引筛选出来了10000条数据，需要将这10000条数据全部回表进行剩余字段的比较，这样性能就不好；如果使用索引下推，利用联合索引中含有的lastname字段，在二级索引的时候就顺便去过滤掉lastname不包含'张'的数据，假设过滤之后只剩500条数据了，那么只需要将这500条数据回表即可，不需要回表10000条数据，提高了效率

默认情况下MySQL开启了索引下推，可以设置成关闭的



### 数据库事务的属性

**事务：**一组逻辑操作单元，使数据从一种状态变换到另一种状态。 

**事务处理的原则：**保证所有事务都作为一个工作单元来执行，即使出现了故障，都不能改变这种执行方式。当在一个事务中执行多个操作时，要么所有的事务都被`提交(commit)`，那么这些修改就永久地保存下来；要么数据库管理系统将放弃所作的所有修改 ，整个事务`回滚(rollback)`到最初状态。



事务的属性：ACID

+ `原子性(Automicity)`：原子性指的是事务是一个不可分割的工作单位，要么全部提交，要么全部失败回滚；即要么转账成功，要么转账失败，不存在中间状态

+ `一致性(Consistency)`：根据定义，一致性是指事务执行前后，数据从一个**合法性状态**变换到另外一个**合法性状态**。这种状态是语义上 的而不是语法上的，跟具体的业务有关。

  > 那什么是合法的数据状态呢？满足预定的约束的状态就叫做合法的状态。通俗一点，这状态是由你自己来定义的（比如满足现实世界中的约束）。满足这个状态，数据就是一致的，不满足这个状态，数据就是不一致的！如果事务中的某个操作失败了，系统就会自动撤销当前正在执行的事务，返回到事务操作之前的状态。

+ `隔离性(Isolation)`：事务的隔离性是指一个事务的执行不能被其他事务干扰 ，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，**并发执行的各个事务之间不能互相干扰。**

+ `持久性(Durability)`：持久性是指一个事务一旦被提交，**它对数据库中数据的改变就是永久性的** ，接下来的其他操作和数据库故障不应该对其有任何影响。

  > 持久性是通过`事务日志`来保证的。日志包括了`重做日志`和`回滚日志`。当我们通过事务对数据进行修改的时候，首先会将数据库的变化信息记录到重做日志中，然后再对数据库中对应的行进行修改。这样做的好处是，即使数据库系统崩溃，数据库重启后也能找到没有更新到数据库系统中的重做日志，重新执行，从而使事务具有持久性。

### MySQL事务的隔离级别

**数据并发问题**

（1）脏写：对于两个事务 Session A、Session B，如果事务Session A 修改了另一个`未提交事务`Session B修改过的数据，那就意味着发生了脏写

<img src="MySQL部分.assets/image-20220726150007575.png" alt="image-20220726150007575" style="zoom:80%;" />

（2）脏读：对于两个事务Session A、Session B，Session A 读取了`已经被 Session B 更新但还没有被提交的字段`。 之后若 Session B 回滚 ，Session A 读取的内容就是`临时且无效`的。 

Session A和Session B各开启了一个事务，Session B中的事务先将studentno列为1的记录的name列更新为'张三'，然后Session A中的事务再去查询这条studentno为1的记录，如果读到列name的值为'张三'，而 Session B中的事务稍后进行了回滚，那么Session A中的事务相当于读到了一个不存在的数据，这种现象就称之为脏读 。

<img src="MySQL部分.assets/image-20220726150248421.png" alt="image-20220726150248421" style="zoom:80%;" />

（3）不可重复读：对于两个事务Session A、Session B，Session A 读取了一个字段，然后 Session B 更新了该字段。 之后 Session A 再次读取同一个字段， 值就不同了。那就意味着发生了不可重复读。 

我们在Session B中提交了几个隐式事务 （注意是隐式事务，意味着语句结束事务就提交了），这些事务都修改了studentno列为1的记录的列name的值，每次事务提交之后，如果Session A中的事务都可以查看到最新的值，这种现象也被称之为不可重复读 。

<img src="MySQL部分.assets/image-20220726150426078.png" alt="image-20220726150426078" style="zoom:80%;" />

（4）幻读：对于两个事务Session A、Session B, Session A 从一个表中读取 了一个字段, 然后 Session B 在该表中插入了一些新的行。 之后, 如果 Session A 再次读取同一个表, 就会多出几行。那就意味着发生了幻读。 

Session A中的事务先根据条件 studentno > 0这个条件查询表student，得到了name列值为'张三'的记录； 之后Session B中提交了一个 隐式事务 ，该事务向表student中插入了一条新记录；之后Session A中的事务 再根据相同的条件 studentno > 0查询表student，得到的结果集中包含Session B中的事务新插入的那条记 录，这种现象也被称之为幻读 。我们把新插入的那些记录称之为幻影记录 。

<img src="MySQL部分.assets/image-20220726150535291.png" alt="image-20220726150535291" style="zoom:80%;" />

**数据库事务隔离级别**

| 隔离级别          | 概述                                                         | 解决问题               |
| ----------------- | ------------------------------------------------------------ | ---------------------- |
| `Read Uncommited` | 读未提交，在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。 | 脏写                   |
| `Read Commited`   | 读已提交，它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。 | 脏写、脏读             |
| `Repeatable Read` | 可重复读，事务A在读到一条数据之后，此时事务B对该数据进行了修改并提交，那么事务A再读该数据，读到的还是原来的内容。这是MySQL的默认隔离级别。 | 脏写、脏读、不可重复读 |
| `Serializable`    | 可串行化，确保事务可以从一个表中读取相同的行。在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作。性能十分低下。 | 所有问题               |

### Redo日志和Undo日志

**Redo日志**

+ Redo日志称为重做日志，提供再写入操作，恢复提交事务修改的页操作，**用来保证事务的持久性**

+ Redo日志是`存储引擎生成的日志`，记录的是"物理级别"上的页修改操作，比如`页号，偏移量，写入数据`等，主要为了保证数据的可靠性

InnoDB存储引擎以页为单位管理存储空间，在真正访问页面之前，需要把在磁盘上的页缓存到`内存中的缓冲池`中才可以访问，所有的变更都必须`先更新缓冲池`中的数据，然后缓冲池中的脏页会以一定的频率被刷入磁盘，通过缓冲池来优化CPU与磁盘之间的性能鸿沟，这样就可以保证整体的性能不会下降太快

事务具有持久性的特征，当事务提交之后，即使系统崩溃了，这个事务对数据库中所做的更改页不能丢失，一个最简单的方法是，每执行一次修改，就把这个修改直接刷新到磁盘中，但这样会带来很大的性能损耗：

+ `修改量与刷新磁盘的工作量严重不成比例`，如果修改的仅是某一行数据中的某一个字段的值，有可能仅仅只是几个字节，但从磁盘加载数据的时候必须以页为单位，也就是16KB都必须全部加载进来，但其他数据完全是不必要的，而且刷新到磁盘的时候也需要以页为单位刷新，有很多不必要的修改
+ `随机IO`，一个整体事务有可能包含多条修改操作，并且可能针对不同的数据行，这些数据行物理上存储的位置可能在磁盘的不同页，那么在加载和刷新磁盘的过程中需要进行很多次随机IO，随机IO比顺序IO要慢很多

所以，另一个解决思路就是记录一下这个事务修改了哪些东西，也就是这个Redo日志，先将事务的修改写到Redo日志中（日志也是磁盘上的一份文件，但由于是顺序读写以及本身大小也不大，所以远比之前的随机IO和固定大小加载好很多），比如某条修改语句将10号页中偏移量为100处的字节值从1修改为2，把这些关键位置记录到Redo日志即可

InnoDB采用了WAL技术，这种技术的思想就是先写日志，再写磁盘，只有日志写入成功（写入磁盘），才算事务提交成功，当发生宕机且数据未刷到磁盘时，可以通过Redo日志来恢复

> **Redo日志的特点**
>
> + 日志降低了刷盘的频率
> + Redo日志占用的空间比较小
> + Redo日志是顺序写入磁盘的
> + 事务执行过程中，Redo日志不断记录
>
> **Redo日志的基本过程**
>
> <img src="MySQL部分.assets/image-20220806144828117.png" alt="image-20220806144828117" style="zoom:80%;" />

**Undo日志**

+ Undo日志称为回滚日志，回滚记录到某个特定的版本，用来保证事务的`原子性，一致性`

+ Undo日志也是存储引擎生成的日志，记录的是"逻辑操作"，比如对某一行数据进行了Insert操作，那么undo日志就相应的记录一条与之相反的delete操作

主要用于事务的回滚和一致性非锁定读（MVCC），Undo日志是事务原子性的保证

每当对一条数据记录做改动时，都需要“留一手”，把修改之前的状态记录下来，比如：

+ 插入一条记录，至少要把插入的主键值记录下来，回滚的时候，再把对应主键值的记录删掉
+ 删除一条记录，至少要把记录的内容保存下来，回滚的时候，再把这条记录插回去
+ 修改一条记录，至少要把修改之前记录的值保存下来，这样回滚之后再把这条记录更新为旧值

Undo日志是`逻辑日志`，只是将数据库逻辑地恢复为原来的样子，比如：插入一条数据，新插入的数据需要开辟一个新的页，但之后事务执行的时候出现异常，新插入的数据需要被删除，但其开辟出来的新数据页却不会被删除，这是为了在并发场景下，多事务执行时，保证这个事务回滚，不会影响到其他事务

### Binlog

binlog即binary log，二进制日志文件，也叫做变更日志，它记录了数据库所有执行的`DDL和DML`等数据库更新事件的语句，但是不包含没有修改任何数据的语句（查询语句和show语句）

binlog以事件形式记录并保存在二进制文件中，通过这些信息，可以再现数据更新操作的全过程

binlog的主要应用场景：

+ 一是用于**数据恢复**，如果MySQL数据库意外停止可以通过二进制日志文件来查看用户执行了哪些操作，对数据库服务器文件做了哪些修改，然后根据二进制文件中的记录来恢复数据库服务器
+ 二是用于**数据复制**，由于日志的延续性和时效性，在主从复制结构中，主机把其二进制日志传递给从机来达到主从机数据一致的目的

相关参数：

```
log-bin=mysql-bin   ##存储binlog时使用的前缀，比如mysql-bing，之后的binlog都是mysql-bin00001
binlog_expire_logs_seconds=600    ##表示希望binlog文件保留的时长，单位是秒，默认是30天
max_binlog_size=100M  ##控制单个binlog的大小，当binlog文件大小超过这个值时会再新建一个binlog文件进行记录
##默认是1GB，该设置并不能严格控制binlog的大小，尤其是binlog接近最大值又遇到一个比较大的事务时，为保证事务的完整性，可能不会进行切换日志的动作
```

binlog日志可以通过mysqlbinlog命令工具进行查看

```
mysqlbinlog 具体binlog的文件路径   ##显示的SQL语句以二进制的形式显示
mysqlbinlog -v 具体binlog文件路径   ##会将二进制的SQL语句以事件的形式显示出来
```

**使用Binlog恢复数据**



### MySQL锁

MySQL事务的并发访问分为三种情况：

+ 读 - 读：即并发事务相继读取相同的记录，读操作本身不会对记录有任何影响，并不会引起什么问题
+ 写 - 写：即并发事务相继对相同的记录做出改动，在这种情况下就会出现脏写的问题
+ 读 - 写 / 写 - 读：即一个事务进行读取操作，另一个进行改动操作，这种情况下就会发生脏读，不可重复读，幻读的问题

锁类型的划分：

<img src="MySQL部分.assets/image-20220920104849433.png" alt="image-20220920104849433" style="zoom:80%;" />



 **读锁和写锁**

+ `共享锁`：也称为读锁，用S表示，针对同一份数据，多个事务的读操作可以同时进行而不会相互影响，不相互阻塞
+ `排他锁`：也称为写锁，用X表示，当前写操作没有完成前，会阻断其他写锁和读锁，这样就能确保在给定的时间里，只有一个事务能执行写入，并防止其他用户读取正在写入的同一资源

> 在innodb存储引擎下，读锁和写锁即可以加在表上，也可以加在数据行上
>
> 比如行级读写锁：如果一个事务T1获得了行A的读锁，那么此时另外的一个事务T2仍然可以获得行A的读锁；但是如果事务T3想获得行A的写锁，就会被阻塞，需要等到T1，T2都释放读锁之后才能获得行A的写锁，但T3仍然可以获得同一张表下其他记录的写锁

对读取的记录加S锁：

```mysql
select ... lock in share mode;
#或者
select ... for share   #mysql8.0之后的语法
```

读读取的记录加X锁：

```mysql
select ... for update;
```

**表级锁、页级锁和行锁**

为了尽可能提高数据库的并发度，每次锁定的数据范围越小越好，理论上每次只锁定当前操作的数据的方案会得到最大的并发度，但管理锁是比较消耗资源的，因此数据库在高并发和系统性能两方面进行平衡，就产生了锁粒度的概念

+ `表锁`：锁定整张表，是MySQL中最基本的锁策略，并不依赖于特定的存储引擎，并且锁的开销最小，表级锁一次锁定整张表，可以很好地避免死锁的问题，但并发量不高
  + 表级S锁、X锁：InnoDB存储引擎下很少会使用表锁，所以一般在MyISAM存储引擎下使用
  + 表级意向锁：意向锁是为了协调行锁和表锁的关系，支持多粒度的锁（主要是表锁和行锁）共存，意向锁的语义是某个事务正在某些行持有了该锁，主要分为两种，意向共享锁IS和意向排他锁IX；其中，如果我们给数据表的某一行加上了排他锁，数据库会自动给更大一级的空间加上意向锁，告诉其他事务这个数据页或者数据表已经有人上过排他锁了
  + 表级自增锁：就是字段的`AUTO_INCREMENT`属性
  + 表级元数据锁：元数据锁的作用是，保证读写的正确性，当对一个表做增删改查操作的时候，加上MDL读锁；当要对表做结构变更操作的时候，加MDL写锁；写锁之间是互斥的，用来保证变更表结构操作的安全性
+ `行锁`：也称为记录锁，顾名思义就是锁住某一行，行级锁只在存储引擎层实现（InnoDB存储引擎），锁的粒度最小，发生锁冲突的概率低，并发度较高
  + 记录锁：仅把一条记录锁上，也区分读锁或者写锁
  + 间隙锁：间隙锁的提出仅仅是为了防止插入幻影记录，如果给id为8的记录加上了间隙锁，意味着不允许别的事务在8这条记录前的间隙插入新记录
  + 临键锁：记录锁 + 间隙锁，既能保护当前记录，也能阻止别的事务将新记录插入到当前事务之前

### MVCC

MVCC即多版本并发控制，通过数据行的多个版本管理来实现数据库的并发控制，简单的说就是在查询一些正在被另一个事务更新的行时可以看到它们被更新之前的值，这样在做查询的时候就不用等待另一个事务释放锁

MVCC没有正式的标准，在不同的DBMS中MVCC的实现方式可能不同，并且也不是普遍使用的

MySQL在可重复读的隔离级别下，一定程度上解决了幻读的问题，就是靠的这个MVCC机制（快照读的情况下，即不加锁的读操作，select * from xxx）

MVCC依赖于三个比较重要的部分：**隐藏字段**，**Undo log**和**ReadView**







### MySQL日志

MySQL共支持六种类型的日志，分别为：

+ `二进制日志`：记录所有**更改数据**的语句，可以用于主从服务器之间的数据同步，以及服务器遇到故障时数据的无损失恢复。
+ `通用查询日志`：记录所有连接的起始时间和终止时间，以及连接发送给数据库服务器的所有指令，对我们复原操作的实际场景、发现问题，甚至是对数据库操作的审计都有很大的帮助。
+ `错误日志`：记录MySQL服务的启动、运行或停止MySQL服务时出现的问题，方便我们了解服务器的状态，从而对服务器进行维护。
+ `慢查询日志`：记录所有执行时间超过long_query_time的所有查询，方便我们对查询进行优化。
+ `中继日志`：用于主从服务器架构中，从服务器用来存放主服务器二进制日志内容的一个中间文件。 从服务器通过读取中继日志的内容，来同步主服务器上的操作
+ `数据定义语句日志`：记录数据定义语句执行的元数据操作。
