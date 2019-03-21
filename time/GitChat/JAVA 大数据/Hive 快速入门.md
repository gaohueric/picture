## 内容提要

内容提要：

- 请问和Hive类似的工具还有哪些，能否做一个简单的比较和使用场景分析？
- Hive有什么好的活跃社区推荐？
- 请问Hive与SparkSQL的目标与实现机制有河主要差异？适用场景有分别吗？
- 看你写的介绍，Hive适合利用SQL工具处理离线大数据的计算，想咨询一下，离线数据是按照什么规则和要求写进HDFS，以便可以让Hive能够计算的？
- 想知道如何进行Hive SQL的查询优化，有没有相关书籍推荐？
- 关于更新目标表，有几种比较好处理方法？
- 推荐下关于Hive转Mapreduce原理或者优化Hive SQL的书籍或者博客之类的资料？
- 我们现在正在迁移数据库（从MySQL到Hive上），MySQL有大量的存储过程，请问有什么比较好的处理方式吗？
- Hive的运维难度相比同类工具，难易程度？
- 你经历过的使用Hive的项目，数据量大概多大，对应的Hive Query的性能如何？

------

**问：请问和Hive类似的工具还有哪些，能否做一个简单的比较和使用场景分析？**

**答：**和Hive类似的有MapReduce、Pig、Impala。

用MapReduce处理HDFS上的数据更加的灵活，Hive是对MapReduce的封装，最终也是转换为MapReduce程序来执行的，当用Hive灵活性和功能性满足不了需求的时候，可以直接开发MapReduce程序处理数据。比如数据非常不规范、程序员更加精细地控制数据、数据不能映射为二维表。MapReduce实际中用的比较少，因为Hive太好用了。

Pig同样也是对MapReduce程序的封装，最终也是转换为MapReduce程序去执行，但是Pig对数据的规范性要求很低，可以同时处理结构化、半结构化、非结构化的数据，而Hive对数据规范性要求比较高。Pig好像市面上用的人更少了，因为它用起来确实复杂了一些。封装为SQL来实现复杂的功能，确实方便，是一种趋势，不止HiveSQL，SparkSQL等等都向这方面发展。

在对比Hive和Pig的时候，我有一个很有意思的想法，就是猜测Hive为什么命名为Hive（小蜜蜂），Pig为什么命名为Pig（猪），以下纯粹属于我瞎猜，没有任何依据：

<瞎猜开始>

Hive是小蜜蜂，小蜜蜂对食物（数据格式）要求比较高，只采颜色鲜艳的花朵，庞大的蜜蜂群（集群）能采很多花朵（大数据）；Pig是猪，猪的特点对食物（数据格式）没有要求，几乎可以吃任何东西，什么都能消化（处理），但是猪确实很聪明（可以优化执行），针对不同的食物（不同格式的数据）采取最优的消化方式（处理方式）。

<瞎猜结束>

Impala也是通过SQL查询大数据的工具，Impala查询非常的快速，可以看做实时交互，这是因为它底层没有采用MapReduce，而是使用分布式查询引擎。Impala适合于查询结果比较小的场景下使用，而Hive适合于输出数据量非常大的场景，实际使用时，Impala通常做为Hive的补充，比如先用Impala做分析结果的少数据量的验证，然后再用Hive进行真正的执行。

------

**问：Hive有什么好的活跃社区推荐？**

**答：**要说社区，还得算强大的Stack Overflow。你可能问的是怎么学习Hive，学习开源技术最好的参考是官方文档，这个是不变的真理。另一个真理是看源代码。可是一个客观的事实对很多人来说英语是个障碍，国外的社区也有这个问题。国内大牛写技术文章的也有很多，看起来容易多了。另外就是看书，我比较喜欢看书，因为书里的知识更系统。再在实际工作中进行印证，确实是个好玩的过程。

------

**问：请问Hive与SparkSQL的目标与实现机制有河主要差异？适用场景有分别吗？**

**答：** Hive SQL是大数据量的离线计算，SparkSQL是实时迭代计算。Hive实现机制实际是MapReduce，执行过程中将中间结果都存入硬盘，还有Shuffle阶段，所以速度很慢。SparkSQL被设计为通过RDD尽量在内存中计算，这是导致它计算快的很重要的一个原因。另外Spark SQL抛弃MapReduce，少了Shuffle，是计算快的主要原因。

------

**问：看你写的介绍，Hive适合利用SQL工具处理离线大数据的计算，想咨询一下，离线数据是按照什么规则和要求写进HDFS，以便可以让Hive能够计算的？**

**答：**线索数据来源，Hive表里的数据来源一般有三类：业务数据的定时接入、日志类数据、离线计算的结果数据。要先定义好了Hive表结构，这个表结构就是写入数据必须遵守的格式规则，在写入数据之前要经过数据清洗，数据清洗是一个数据规范化的过程，去除异常数据、错误数据、重复数据，然后再写入Hive表。这个过程叫做ETL。ETL是抽取（extract）、转换（transform）、加载（load）的过程，从数据源抽取出所需要的数据、经过数据清洗，最终按照预先定义好的格式加载到Hive表里去。总而言之，对于写入Hive表的数据的要求是：符合Hive表定义的格式和没有异常、重复数据的。这里其实体现了一个Hive的局限性，就是hive要求数据是有一定格式的，是有一定格式规范的。这里我想再说明一点，Hive的表结构看上去跟MySQL的表结构很像，是一张二维表，但是不同点是Hive中的数据不用遵守关系型数据库的范式，第一范式都不用遵守，更没有主键、外键约束，也没有索引，它只是一个HDFS上文件内容的一个二维化的映射而已。

------

**问：想知道如何进行Hive SQL的查询优化，有没有相关书籍推荐？**

**答：**我的文章里也介绍了很多查询优化的方式，网上也有很多，不过这些都是告诉你其然，没有告诉你所以然。能够对Hive SQL进行很好地优化，最根本方法就是研究明白MapReduce的执行细节，并弄清楚HiveSQL怎么转换为MapReduce程序的，这很关键的。有一本是《MapReduce设计模式》挺好的，讲到了基本上常见的MapReduce，读完那本书会对Hive的执行会有深刻了解，便于理解Hive的优化。我挺看好这本书的，不过Hive方面的书中文版确实少。还有另外一点很重要，就是在实践中不断摸索总结，技巧和原理相互印证，这个确实是个对Hive和MapReduce学习的过程。

------

**问：关于更新目标表，有几种比较好处理方法？**

**答：**目标表不知道说的是什么意思，什么叫目标表，就当是要更新的表。

Hive数据是存在HDFS上的，HDFS上的数据是不能更新的，只能追加、删除。不能单条记录更新的，HDFS被设计的就不支持。在Hive里有分区的概念，一个分区的数据存储在HDFS上的一个目录中，可以对某一个分区的数据采用删除后重新写入的方式实现更新操作。就一个分区里数据，全部删除，再写入。可以理解为，HDFS就是一个只能insert，不能update和delete操作的数据库。比如一个分区里存储一天数据， 这能这一天的数据全部删除后，再插入，就达到更新的效果了。只能一个分区里的全删，不能单条记录更新，HDFS就是这个特点。这个特点是为了达到高的吞吐量而牺牲的。HBase可以做到单条更新，HBase的数据也是存储在HDFS上，但是HBase采用了很巧妙的方式达到了单条更新的目的。

------

**问：推荐下关于Hive转Mapreduce原理或者优化Hive SQL的书籍或者博客之类的资料？**

**答：** Hive 转 Mapreduce原理，有个美团的文章写的非常好：

<http://tech.meituan.com/hive-sql-to-mapreduce.html>。

Hive转Mapreduce的原理还有就是《MapReduce设计模式》这本书，讲的其实挺好的。比如最复杂的join，讲了各种join的过程实现，Hive就是那样实现的，比较经典就是 map join 优化是怎么做到的。

------

**问：我们现在正在迁移数据库（从MySQL到Hive上），MySQL有大量的存储过程，请问有什么比较好的处理方式吗？**

**答：**存储过程是迁移不了的，迁移的只是数据，不能迁移MySQL的计算，而且把数据迁移到Hive上，如果MySQL上没有了备份就做不到了实时查询，Hive不是实时查询的数据库。我的关系型数据库的每个表都有一个lasttime字段，是记录最新的更改时间，每天凌晨会根据这个字段将截止到0点的数据全量传输到Hive，MySQL里还留着，这些Hive数据用于离线计算，归档数据可以不保留，签入Hive。

------

**问：Hive的运维难度相比同类工具，难易程度？**

**答：**Hive是非常简单的，运维难度不大，说到底，Hive只不过是一个HDFS文件的映射关系而已，数据不存在Hive中，而存在HDFS，计算也不用Hive，而是转为Mapreduce运行在Hadoop 的YARN上，至于运维，其实就是Hadoop的运维，Hive的运维难度很大程度上就是hadoop的运维难度。

------

**问：你经历过的使用Hive的项目，数据量大概多大，对应的Hive Query的性能如何？**

**答：**实际做项目的时候，其实最怕的就是数据量大，首先想到的就是怎么先减少数据量再计算，并不是数据量越大感觉自己越厉害。Hive就像一把大刀，确实可以砍大块的肉，但是却不能蛮砍，得尽量躲着点骨头，这就是我的感觉。我处理过1P数据，没有用，你真的需要这1P全部参与处理吗？大多数是不需要的。大部分的Hive SQL脚本都在半个小时内要处理完。关于怎么减少处理的数据？举个例子，Hive里的数据在创建表的时候就规划好，分好区，查询的时候尽量带着分区查询条件，这样就能减少很多数据量。再就是，分步进行，可以将符合条件的数据存入一个临时表，再用这个筛选后的参与关联。

------

本文首发于GitChat，未经授权不得转载，转载需与GitChat联系。

------

在此感谢[异步社区](http://www.epubit.com.cn/)为本次活动提供的赠书《Hive编程指南》。

[异步社区](http://www.epubit.com.cn/)是人民邮电出版社旗下IT专业图书旗舰社区，也是国内领先的IT专业图书社区，致力于优质学习内容的出版和分享，实现了纸书电子书的同步上架。

![enter image description here](http://images.gitbook.cn/bdc19040-50a3-11e7-9c1f-95dd0ff2f11d)

## 文章实录



### 前言

我写这篇文章的目的是尽可能全面地对Hive进行入门介绍，这篇文章是基于hive-1.0.0版本介绍的，这个版本的Hive是运行在MapReduce上的，新的版本可以运行在Tez上，会有一些不同。

Hive是对数据仓库进行管理和分析数据的工具。但是大家不要被“数据仓库”这个词所吓倒，数据仓库是很复杂的东西，但是如果你会MYSQL或者MSSQL，就会发现Hive是那么的简单，简单到甚至不用学就可以使用Hive做出业务所需要的东西。

但是Hive和MYSQL毕竟不同，执行原理、优化方法，底层架构都完全不相同。 大数据离线分析使用Hive已经成为主流，基于工作中Hive使用的经验，我整理了这个入门级别的文章，希望能给想入门的同学提供一些帮助。

### 一、Hive简介

Facebook为了解决海量日志数据的分析而开发了Hive，后来开源给了Apache软件基金会。

> 官网定义：
>
> The Apache Hive ™ data warehouse software facilitates reading, writing, and managing large datasets residing in distributed storage using SQL.

Hive是一种用类SQL语句来协助读写、管理那些存储在分布式存储系统上大数据集的数据仓库软件。

#### Hive的几个特点

- Hive最大的特点是通过类SQL来分析大数据，而避免了写MapReduce程序来分析数据，这样使得分析数据更容易。
- 数据是存储在HDFS上的，Hive本身并不提供数据的存储功能
- Hive是将数据映射成数据库和一张张的表，库和表的元数据信息一般存在关系型数据库上（比如MySQL）。
- 数据存储方面：它能够存储很大的数据集，并且对数据完整性、格式要求并不严格。
- 数据处理方面：因为Hive语句最终会生成MapReduce任务去计算，所以不适用于实时计算的场景，它适用于离线分析。

### 二、Hive架构

![img](http://images.gitbook.cn/7721d2a0-400c-11e7-919d-27e09bb923c4)

#### Hive的核心

Hive的核心是驱动引擎，驱动引擎由四部分组成：

- 解释器：解释器的作用是将HiveSQL语句转换为语法树（AST）。
- 编译器：编译器是将语法树编译为逻辑执行计划。
- 优化器：优化器是对逻辑执行计划进行优化。
- 执行器：执行器是调用底层的运行框架执行逻辑执行计划。

#### Hive的底层存储

Hive的数据是存储在HDFS上的。Hive中的库和表可以看作是对HDFS上数据做的一个映射。所以Hive必须是运行在一个Hadoop集群上的。

#### Hive语句的执行过程

Hive中的执行器，是将最终要执行的MapReduce程序放到YARN上以一系列Job的方式去执行。

#### Hive的元数据存储

Hive的元数据是一般是存储在MySQL这种关系型数据库上的，Hive和MySQL之间通过MetaStore服务交互。

| 元数据项           | 说明           |
| -------------- | ------------ |
| Owner          | 库、表的所属者      |
| LastAccessTime | 最后修改时间       |
| Table Type     | 表类型（内部表、外部表） |
| CreateTime     | 创建时间         |
| Location       | 存储位置         |
|                | 表的字段信息       |

#### Hive客户端

Hive有很多种客户端。

- cli命令行客户端：采用交互窗口，用hive命令行和Hive进行通信。
- HiveServer2客户端：用Thrift协议进行通信，Thrift是不同语言之间的转换器，是连接不同语言程序间的协议，通过JDBC或者ODBC去访问Hive。
- HWI客户端：hive自带的一个客户端，但是比较粗糙，一般不用。
- HUE客户端：通过Web页面来和Hive进行交互，使用的比较多。

### 三、基本数据类型

Hive支持关系型数据中大多数基本数据类型，同时Hive中也有特有的三种复杂类型。

下面的表列出了Hive中的常用基本数据类型：

| **数据类型**  | **长度**           | **备注**                             |
| --------- | ---------------- | ---------------------------------- |
| Tinyint   | 1字节的有符号整数        | -128~127                           |
| SmallInt  | 1个字节的有符号整数       | -32768~32767                       |
| Int       | 4个字节的有符号整数       | -2147483648 ~ 2147483647           |
| BigInt    | 8个字节的有符号整数       |                                    |
| Boolean   | 布尔类型，true或者false | true、false                         |
| Float     | 单精度浮点数           |                                    |
| Double    | 双精度浮点数           |                                    |
| String    | 字符串              |                                    |
| TimeStamp | 整数               | 支持Unix timestamp，可以达到纳秒精度          |
| Binary    | 字节数组             |                                    |
| Date      | 日期               | 0000-01-01 ~ 9999-12-31，常用String代替 |
| ---       | ---              | ---                                |

### 四、DDL语法

#### 创建数据库

创建一个数据库会在HDFS上创建一个目录，Hive里数据库的概念类似于程序中的命名空间，用数据库来组织表，在大量Hive的情况下，用数据库来分开可以避免表名冲突。Hive默认的数据库是default。

创建数据库例子：

```
hive> create database if not exists user_db;

```

#### 查看数据库定义

Describe 命令来查看数据库定义，包括：数据库名称、数据库在HDFS目录、HDFS用户名称。

```
hive> describe database user_db;
OK
user_db        hdfs://bigdata-51cdh.chybinmy.com:8020/user/hive/warehouse/user_db.db   hadoop  USER

```

user_db是数据库名称。

hdfs://bigdata-51cdh.chybinmy.com:8020/user/hive/warehouse/user*db.db 是user*db库对应的存储数据的HDFS上的根目录。

#### 查看数据库列表

```
hive> show databases;   
OK      
user_db 
default

```

```

```

#### 删除数据库

删除数据库时，如果库中存在数据表，是不能删除的，要先删除所有表，再删除数据库。添加上cascade后，就可以先自动删除所有表后，再删除数据库。（友情提示：慎用啊！）删除数据库后，HDFS上数据库对应的目录就被删除掉了。

```
hive> drop database if exists testdb cascade;

```

#### 切换当前数据库

```
hive> use user_db;

```

#### 创建普通表

```
hive> create table if not exists userinfo  
    > (
    >   userid int,
    >   username string,
    >   cityid int,
    >   createtime date    
    > )
    > row format delimited fields terminated by '\t'
    > stored as textfile;
    OK
    Time taken: 2.133 seconds

```

以上例子是创建表的一种方式，如果表不存在，就创建表userinfo。row format delimited fields terminated by '\t' 是指定列之间的分隔符；stored as textfile是指定文件存储格式为textfile。

创建表一般有几种方式：

- create table 方式：以上例子中的方式。
- create table as select 方式：根据查询的结果自动创建表，并将查询结果数据插入新建的表中。
- create table like tablename1 方式：是克隆表，只复制tablename1表的结构。复制表和克隆表会在下面的Hive数据管理部分详细讲解。

#### 创建外部表

外部表是没有被hive完全控制的表，当表删除后，数据不会被删除。

```
hive> create external table iislog_ext (
    >  ip string,
    >  logtime string    
    > )
    > ;

```

#### 创建分区表

Hive查询一般是扫描整个目录，但是有时候我们关心的数据只是集中在某一部分数据上，比如我们一个Hive查询，往往是只是查询某一天的数据，这样的情况下，可以使用分区表来优化，一天是一个分区，查询时候，Hive只扫描指定天分区的数据。

普通表和分区表的区别在于：一个Hive表在HDFS上是有一个对应的目录来存储数据，普通表的数据直接存储在这个目录下，而分区表数据存储时，是再划分子目录来存储的。一个分区一个子目录。主要作用是来优化查询性能。

```
--创建经销商操作日志表
create table user_action_log
(
companyId INT comment   '公司ID',
userid INT comment   '销售ID',
originalstring STRING comment   'url', 
host STRING comment   'host',
absolutepath STRING comment   '绝对路径',
query STRING comment   '参数串',
refurl STRING comment   '来源url',
clientip STRING comment   '客户端Ip',
cookiemd5 STRING comment   'cookiemd5',
timestamp STRING comment   '访问时间戳'
)
partitioned by (dt string)
row format delimited fields terminated by ','
stored as textfile;

```

这个例子中，这个日志表以dt字段分区，dt是个虚拟的字段，dt下并不存储数据，而是用来分区的，实际数据存储时，dt字段值相同的数据存入同一个子目录中，插入数据或者导入数据时，同一天的数据dt字段赋值一样，这样就实现了数据按dt日期分区存储。

当Hive查询数据时，如果指定了dt筛选条件，那么只需要到对应的分区下去检索数据即可，大大提高了效率。所以对于分区表查询时，尽量添加上分区字段的筛选条件。

#### 创建桶表

桶表也是一种用于优化查询而设计的表类型。创建通表时，指定桶的个数、分桶的依据字段，hive就可以自动将数据分桶存储。查询时只需要遍历一个桶里的数据，或者遍历部分桶，这样就提高了查询效率。举例：

```
------创建订单表
create table user_leads
(
leads_id string,
user_id string,
user_id string,
user_phone string,
user_name string,
create_time string
)
clustered by (user_id) sorted by(leads_id) into 10 buckets 
row format delimited fields terminated by '\t' 
stored as textfile;

```

对这个例子的说明：

- clustered by是指根据user*id的值进行哈希后模除分桶个数，根据得到的结果，确定这行数据分入哪个桶中，这样的分法，可以确保相同user*id的数据放入同一个桶中。而经销商的订单数据，大部分是根据user_id进行查询的。这样大部分情况下是只需要查询一个桶中的数据就可以了。
- sorted by 是指定桶中的数据以哪个字段进行排序，排序的好处是，在join操作时能获得很高的效率。
- into 10 buckets是指定一共分10个桶。
- 在HDFS上存储时，一个桶存入一个文件中，这样根据user_id进行查询时，可以快速确定数据存在于哪个桶中，而只遍历一个桶可以提供查询效率。

**分桶表读写过程**

![enter image description here](http://images.gitbook.cn/3fb89800-4046-11e7-8a79-013e9f3a234c)

#### 查看有哪些表

```
--查询库中表
show tables;
Show TABLES '*info';  --可以用正则表达式筛选要列出的表

```

#### 查看表定义

查看简单定义：

```
describe userinfo;

```

查看表详细信息：

```
describe formatted userinfo;

```

执行结果如下所示：

| 备注              | **col_name**                 | **data_type**                            | **comment** |
| --------------- | ---------------------------- | ---------------------------------------- | ----------- |
| 列信息             | # col_name                   | data_type                                | comment     |
|                 | NULL                         | NULL                                     |             |
| userid          | int                          |                                          |             |
| username        | string                       |                                          |             |
| cityid          | int                          |                                          |             |
| createtime      | date                         |                                          |             |
|                 |                              | NULL                                     | NULL        |
|                 | # Detailed Table Information | NULL                                     | NULL        |
| 所在库             | Database:                    | user_db                                  | NULL        |
| 所属HUE用户         | Owner:                       | admin                                    | NULL        |
| 表创建时间           | CreateTime:                  | Tue Aug 16 06:05:14 PDT 2016             | NULL        |
| 最后访问时间          | LastAccessTime:              | UNKNOWN                                  | NULL        |
|                 | Protect Mode:                | None                                     | NULL        |
|                 | Retention:                   | 0                                        | NULL        |
| 表数据文件在HDFS上路径   | Location:                    | hdfs://bigdata-51cdh.chybinmy.com:8020/user/hive/warehouse/user_db.db/userinfo | NULL        |
| 表类型(内部表或者外部表)   | Table Type:                  | MANAGED_TABLE                            | NULL        |
| 表分区信息           | Table Parameters:            | NULL                                     | NULL        |
|                 |                              | transient_lastDdlTime                    | 1471352714  |
|                 |                              | NULL                                     | NULL        |
|                 | # Storage Information        | NULL                                     | NULL        |
| 序列化反序列化类        | SerDe Library:               | org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe | NULL        |
| mapreduce中的输入格式 | InputFormat:                 | org.apache.hadoop.mapred.TextInputFormat | NULL        |
| mapreduce中的输出格式 | OutputFormat:                | org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat | NULL        |
| 压缩              | Compressed:                  | No                                       | NULL        |
| 总占用数据块个数        | Num Buckets:                 | -1                                       | NULL        |
|                 | Bucket Columns:              | []                                       | NULL        |
|                 | Sort Columns:                | []                                       | NULL        |
|                 | Storage Desc Params:         | NULL                                     | NULL        |
|                 |                              | field.delim                              | \t          |
|                 |                              | serialization.format                     | \t          |
| ---             | ---                          | ---                                      | ---         |

#### 修改表

对表的修改操作有：修改表名、添加字段、修改字段。

**修改表名：**

```
--将表名从userinfo改为user_info
alter table userinfo rename to user_info;

```

**添加字段：**

```
---在user_info表添加一个字段provinceid，int 类型
alter table user_info add columns (provinceid int );

```

**修改字段：**

```
alter table user_info replace columns (userid int,username string,cityid int,joindate date,provinceid int);

```

修改字段，只是修改了Hive表的元数据信息（元数据信息一般是存储在MySql中），并不对存在于HDFS中的表数据做修改。

并不是所有的Hive表都可以修改字段，只有使用了native SerDe (序列化反序列化类型)的表才能修改字段

#### 删除表

```
--如果表存在，就删除。
drop table if exists user_info;

```

### 五、DML语法

#### 向Hive中加载数据

**加载到普通表**

可以将本地文本文件内容批量加载到Hive表中，要求文本文件中的格式和Hive表的定义一致，包括：字段个数、字段顺序、列分隔符都要一致。

这里的user_info表的表定义是以\t作为列分隔符，所以准备好数据后，将文本文件拷贝到hive客户端机器上后，执行加载命令。

```
load data local inpath '/home/hadoop/userinfodata.txt' overwrite into table user_info;

```

- local关键字表示源数据文件在本地，源文件可以在HDFS上，如果在HDFS上，则去掉local，inpath后面的路径是类似”hdfs://namenode:9000/user/datapath”这样的HDFS上文件的路径。
- overwrite关键字表示如果hive表中存在数据，就会覆盖掉原有的数据。如果省略overwrite，则默认是追加数据。

加载完成数据后，在HDFS上就会看到加载的数据文件。

**加载到分区表**

```
load data local inpath '/home/hadoop/actionlog.txt' overwrite into table user_action_log 
PARTITION (dt='2017-05-26');

```

partition 是指定这批数据放入分区2017-05-26中。

**加载到分桶表**

```
------先创建普通临时表
create table user_leads_tmp
(
leads_id string,
user_id string,
user_id string,
user_phone string,
user_name string,
create_time string
)
row format delimited fields terminated by ',' 
stored as textfile;
------数据载入临时表
load data local inpath '/home/hadoop/lead.txt' overwrite into table user_leads_tmp;
------导入分桶表
set hive.enforce.bucketing = true;
insert overwrite table user_leads select * from  user_leads_tmp;

```

set hive.enforce.bucketing = true; 这个配置非常关键，为true就是设置为启用分桶。

#### 导出数据

```
 --导出数据，是将hive表中的数据导出到本地文件中。
insert overwrite local directory '/home/hadoop/user_info.bak2016-08-22 '
select * from user_info;

```

去掉local关键字，也可以导出到HDFS上。

#### 插入数据

**insert select 语句**

上一节分桶表数据导入，用到从user*leads*tmp表向user_leads表中导入数据，用到了insert数据。

```
insert overwrite table user_leads select * from  user_leads_tmp;

```

这里是将查询结果导入到表中，overwrite关键字是覆盖目标表中的原来数据。如果缺省，就是追加数据。

如果是插入数据的表是分区表，那么就如下所示：

```
insert overwrite table user_leads PARTITION (dt='2017-05-26') 
select * from  user_leads_tmp;

```

**一次遍历多次插入**

```
from user_action_log
insert overwrite table log1 select companyid,originalstring  where companyid='100006'
insert overwrite table log2 select companyid,originalstring  where companyid='10002'

```

每次hive查询，都会将数据集整个遍历一遍。当查询结果会插入多个表中时，可以采用以上语法，将一次遍历写入多个表，以达到提高效率的目的。

#### 复制表

复制表是将源表的结构和数据复制并创建为一个新表，复制过程中，可以对数据进行筛选，列可以进行删减。

```
create table user_leads_bak
row format delimited fields terminated by '\t'
stored as textfile
as
select leads_id,user_id,'2016-08-22' as bakdate
from user_leads
where create_time<'2016-08-22';

```

上面这个例子是对user_leads表进行复制备份，复制时筛选了2016-08-22以前的数据，减少几个列，并添加了一个bakdate列。

#### 克隆表

克隆表时会克隆源表的所有元数据信息，但是不会复制源表的数据。

```
--克隆表user_leads，创建新表user_leads_like
create table user_leads_like like  user_leads;

```

#### 备份表

备份是将表的元数据和数据都导出到HDFS上。

```
export table user_action_log partition (dt='2016-08-19')
to '/user/hive/action_log.export'

```

这个例子是将user*action*log表中的一个分区，备份到HDFS上，to后面的路径是HDFS上的路径。

#### 还原表

将备份在HDFS上的文件，还原到user*action*log_like表中。

```
import table user_action_log_like from '/user/hive/action_log.export';

```

### 六、HQL语法

#### Select 查询

**指定列表**

```
select * from user_leads;

select leads_id,user_id,create_time from user_leads;
select e.leads_id from user_leads e;

```

**函数列**

```
select companyid,upper(host),UUID(32) from user_action_log;

```

可以使用hive自带的函数，也可以是使用用户自定义函数。

上面这个例子upper()就是hive自带函数，UUID()就是用户自定义函数。

关于函数详细介绍，可以参考后面的章节。

**算数运算列**

> select companyid,userid, (companyid + userid) as sumint from user*action*log;

可以进行各种算数运算，运算结果做为结果列。

| **运算符** | **描述** | **运算符** | **描述** |
| ------- | ------ | ------- | ------ |
| A+B     | 数字相加   | A-B     | 数字相减   |
| A*B     | 相乘     | A/B     | 相除     |
| A%B     | 模除     |         |        |

**限制返回条数**

类似于sql server里的top N，或者mysql里的limit。

```
select * from user_action_log limit 100;

```

**Case When Then语句**

```
---case when 两种写法
select case companyid when 0 then '未登录' else companyid end from user_action_log;

select case  when companyid=0 then '未登录' else companyid end from user_action_log;

```

这个例子，判断companyid值，如果为0则显示为未登录，如果不为0，则返回companyid的值。

#### Where筛选

| **操作符**           | **说明**                | **操作符**               | **说明**               |
| ----------------- | --------------------- | --------------------- | -------------------- |
| A=B               | A等于B就返回true，适用于各种基本类型 | A<=>B                 | 都为Null则返回True，其他和=一样 |
| A<>B              | 不等于                   | A!=B                  | 不等于                  |
| A<B               | 小于                    | A<=B                  | 小于等于                 |
| A>B               | 大于                    | A>=B                  | 大于等于                 |
| A Between B And C | 筛选A的值处于B和C之间          | A Not Between B And C | 筛选A的值不处于B和C之间        |
| A Is NULL         | 筛选A是NULL的             | A Is Not NULL         | 筛选A值不是NULL的          |
| A Link B          | %一个或者多个字符_一个字符        | A Not Like B          | %一个或者多个字符_一个字符       |
| A RLike B         | 正则匹配                  |                       |                      |
| ---               | ---                   | ---                   | ---                  |

#### Group By 分组

Hive不支持having语句，有对group by 后的结果进行筛选的需求，可以先将筛选条件放入group by的结果中，然后在包一层，在外边对条件进行筛选。

如果需要进行如下查询：

```
SELECT col1 FROM t1 GROUP BY col1 HAVING SUM(col2) > 10
可以用下面这种方式实现：
SELECT col1 FROM 
(SELECT col1, SUM(col2) AS col2sum
       FROM t1 GROUP BY col1
) t2
WHERE t2.col2sum > 10

```

#### 子查询

Hive对子查询的支持有限，只允许在 select from 后面出现。比如：

```
---只支持如下形式的子查询
select * from (
  select userid,username from user_info i where i.userid='10595'
  ) a;

---不支持如下的子查询
select 
(select username from user_info i where i.userid=d.user_id) 
from user_leads d where d.user_id='10595';

```

### 七、JOIN

#### Hive Join的限制

**只支持等值连接**

Hive支持类似SQL Server的大部分Join操作，但是注意只支持等值连接，并不支持不等连接。原因是Hive语句最终是要转换为MapReduce程序来执行的，但是MapReduce程序很难实现这种不等判断的连接方式。

```
----等值连接
select lead.* from user_leads lead
left join user_info info 
on lead.user_id=info.userid;
---不等连接（不支持）
select lead.* from user_leads lead
left join user_info info 
on lead.user_id!=info.userid;

```

**连接谓词中不支持or**

```
---on 后面的表达式不支持or
select lead.* from user_leads lead
left join user_info info 
on lead.user_id=info.userid or lead.leads_id=0;

```

#### Inner join

内连接同SQL Sever中的一样，连接的两个表中，只有同时满足连接条件的记录才会放入结果表中。

#### Left join

同SQL Server中一样，两个表左连接时，符合Where条件的左侧表的记录都会被保留下来，而符合On条件的右侧的表的记录才会被保留下来。

#### Right join

同Left Join相反，两个表左连接时，符合Where条件的右侧表的记录都会被保留下来，而符合On条件的左侧的表的记录才会被保留下来。

#### Full join

Full Join会将连接的两个表中的记录都保留下来。

#### Left Semi-Join ( exists 语句)

SQL Server中有exists语句，类似下面的语句，但是Hive中不支持 Exists语句。

```
--SQL Sever中的exists语句，但是hive中不支持
SELECT i.* FROM  userInfo i
WHERE EXISTS (SELECT 1 FROM userScopeRelation s WHERE s.userInfoId=i.userInfoID AND s.CompanyID=i.CompanyID
)

```

对于这种需求，Hive使用Left Semi-Join（左半开连接）来解决。

```
SELECT i.* from userInfo i left semi-join userScopeRelation s 
on i.userInfoId=s.userInfoId and i.CompanyID=s.CompanyID

```

但是这里注意，select 后面的列，不能有left semi-join右边表的字段，只能是左边表的字段。

### 八、排序

#### Order By

```
select * from user_leads order by user_id

```

Hive中的Order By达到的效果和SQL Server中是一样的，会对查询结果进行全局排序，但是Hive语句最终要转换为MapReduce程序放到Hadoop分布式集群上去执行，Order By这样的操作，肯定要在Map后汇集到一个Reduce上执行，如果结果数据量大，那就会造成Reduce执行相当漫长。

**所以，Hive中尽量不要用Order By，除非非常确定结果集很小。**

但是排序的需求总是有的，Hive中使用下面的几种排序来满足需求。

#### Sort By

```
select * from user_leads sort by user_id

```

这个例子中，Sort By是在每个reduce中进行排序，是一个局部排序，可以保证每个Reduce中是按照user*id进行排好序的，但是全局上来说，相同的user*id可以被分配到不同的Reduce上，虽然在各个Reduce上是排好序的，但是全局上不一定是排好序的。

#### Distribute By 和 Sort By

```
--Distribute By 和Sort By实例
select * from user_leads where user_id!='0' 
Distribute By cast(user_id as int) Sort by cast(user_id as int);

```

Distribute By 指定map输出结果怎么样划分后分配到各个Reduce上去，比如Distribute By user*id，就可以保证user*id字段相同的结果被分配到同一个reduce上去执行。然后再指定Sort By user*id，则在Reduce上进行按照user*id进行排序。

但是这种还是不能做到全局排序，只能保证排序字段值相同的放在一起，并且在reduce上局部是排好序的。

需要注意的是Distribute By 必须写在Sort By前面。

#### Cluster By

如果Distribute By和Sort By的字段是同一个，可以简写为 Cluster By.

```
select * from user_leads where user_id!='0' 
Cluster By cast(user_id as int) ;

```

#### 常见全局排序需求

常见的排序需求有两种：要求最终结果是有序的、按某个字段排序后取出前N条数据。

**最终结果是有序的**

最终分析结果往往是比较小的，因为客户不太可能最终要的是一个超级大数据集。所以实现方式是先得到一个小结果集，然后在得到最终的小结果集上使用order by 进行排序。

```
select * from (
select user_id,count(leads_id) cnt from user_leads  
where user_id!='0' 
group by user_id
) a order by a.cnt;

```

这个语句让程序首先执行group by语句获取到一个小结果集，group by 过程中是不指定排序的，然后再对小结果集进行排序，这样得到的最终结果是全局排序的。

**取前N条**

```
select a.leads_id,a.user_name from (
  select leads_id,user_name  from user_leads  
distribute by length(user_name)  sort by length(user_name) desc limit 10
 ) a order by length(a.user_name) desc limit 10;

```

这个语句是查询user*name最长的10条记录，实现是先根据user*name的长度在各个Reduce上进行排序后取各自的前10个，然后再从10*N条的结果集里用order by取前10个。

这个例子一定要结合MapReduce的执行原理和执行过程才能很好的理解，所以这个最能体现：看Hive语句要以MapReduce的角度看。

### 九、自定义函数

#### Hive内置函数

Hive中自带了大量的内置函数，详细可参看如下资源：

官方文档：

<https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-Built-inFunctions>.

网友整理的中文文档：<http://blog.csdn.net/wisgood/article/details/17376393>.

开发过程中应该尽量使用Hive内置函数，毕竟Hive内置函数经过了大量的测试，性能普遍较好，任何一点性能上的问题在大数据量上跑时候都会被放大。

#### 自定义函数

同SQL Server一样，Hive也允许用户自定义函数，这大大扩展了Hive的功能，Hive是用Java语言写的，所以自定义函数也需要用Java来写。

编写一个Hive的自定义函数，需要新建一个Java类来继承UDF类并实现evaluate()函数，evaluate()函数中编写自定义函数的实现逻辑，返回值给Hive使用，需要注意的是，evaluate()函数的输入输出都必须是Hadoop的数据类型，以便可以被MapReduce程序来进行序列化反序列化。编写完成后将Java程序打成Jar包，在Hive会话中载入Jar包来使用自定义函数。

在执行Hive语句时，遇到一个自定义函数就会实例化一个类，并执行对应的evaluate()函数，每行输入都会调用一次evaluate()函数，所以在编写自定义函数时，一定要注意大数据量时的资源占用问题。

Hive中的自定义函数依据输入输出数据的个数，分为以下几类：

**UDF用户自定义函数（一进一出）**

这种是最普通最常见的自定义函数，类似内置函数length()、yaer()等函数，输入为一个值，输出也为一个值。下面是一个获取唯一ID的自定义函数例子：

```
package com.autohome.ics.bigdata.common.Date;
import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.hive.ql.udf.UDFType;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import java.util.Date;

/*
 生成一个指定长度的随机字符串(最长为36位)
 */
@org.apache.hadoop.hive.ql.exec.Description(name = "UUID",
        extended = "示例：select UUID(32) from src;",
        value = "_FUNC_(leng)-生成一个指定长度的随机字符串(最长为36位)")
@UDFType(deterministic = false)
public class UUID extends UDF {
    public Text evaluate(IntWritable leng) {
        String uuid = java.util.UUID.randomUUID().toString();
        int le = leng.get();
        le = le > uuid.length() ? uuid.length() : le;
        return new Text(uuid.substring(0, le));
    }

/*
 生成一个随机字符串
*/
public Text evaluate() {
    String uuid = java.util.UUID.randomUUID().toString();
    return new Text(uuid);
}
}

```

- 这个实例是获取一个指定长度的随机字符串自定义函数，这个自定义函数创建了一个类UUID，继承于UDF父类。
- UUID类要实现evaluate函数，获取一个指定长度的随机字符串。
- evaluate函数是可以有多个重载的。
- Description是自定义函数的描述信息。
- 这里有一个参数deterministic，是标识这个自定义函数是否是那种输入确定时输出就确定的函数，默认是true，比如length函数就是如果输入同一个值，那么输出肯定是一致的，但是我们这里的UUID就算输入确定，但是输出也是不确定的，所以要将deterministic设置为false。

**UDAF用户自定义聚合函数（多进一出）**

UDAF是自定义聚合函数，类似于sum()、avg()，这一类函数输入是多个值，输出是一个值。

UDAF是需要hive sql语句和group by联合使用的。

聚合函数常常需要对大量数组进行操作，所以在编写程序时，一定要注意内存溢出问题。

UDAF分为两种：简单UDAF和通用UDAF。简单UDAF写起来比较简单，但是因为使用了JAVA的反射机制导致性能有所损失，另外有些特性不能使用，如可变参数列表，通用UDAF可以使用所有功能，但是写起来比较复杂。

**(1) 简单UDAF实例**

```
package com.autohome.ics.bigdata.common.number;
import org.apache.hadoop.hive.ql.exec.UDAF;
import org.apache.hadoop.hive.ql.exec.UDAFEvaluator;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import java.util.regex.Pattern;
/**
 * Created by 鸣宇淳 on 2016/8/30.
*对传入的字符串列表，按照数字大小进行排序后，找出最大的值
*/
public class MaxIntWithString extends UDAF {
    public static  class MaxIntWithStringUDAFEvaluator implements  UDAFEvaluator{
        //最终结果最大的值
        private IntWritable MaxResult;
        @Override
        public void init() {
            MaxResult=null;
        }
        //每次对一个新值进行聚集计算都会调用iterate方法
        //这里将String转换为int后，进行比较大小
        public boolean iterate(Text value)
        {
            if(value==null)
                return false;
            if(!isInteger(value.toString()))
            {
                return false;
            }
            int number=Integer.parseInt(value.toString());
            if(MaxResult==null)
                MaxResult=new IntWritable(number);
            else
                MaxResult.set(Math.max(MaxResult.get(), number));
            return true;
        }
        //Hive需要部分聚集结果的时候会调用该方法
        //会返回一个封装了聚集计算当前状态的对象
        public IntWritable terminatePartial()
        {
            return MaxResult;
        }
        //合并两个部分聚集值会调用这个方法
        public boolean merge(Text other)
        {
            return iterate(other);
        }
        //Hive需要最终聚集结果时候会调用该方法
        public IntWritable terminate()
        {
            return MaxResult;
        }
        private static boolean isInteger(String str) {
            Pattern pattern = Pattern.compile("^[-\\+]?[\\d]*$");
            return pattern.matcher(str).matches();
        }
    }
}

```

- UDAF要继承于UDAF父类org.apache.hadoop.hive.ql.exec.UDAF。
- 内部类要实现org.apache.hadoop.hive.ql.exec.UDAFEvaluator接口。
- MaxIntWithStringUDAFEvaluator类里需要实现init、iterate、terminatePartial、merge、terminate这几个函数，是必不可少的.
- init()方法用来进行全局初始化的。
- iterate()中实现比较逻辑。
- terminatePartial是Hive部分聚集时调用的，类似于MapReduce里的Combiner，这里能保证能得到各个部分的最大值。
- merge是多个部分合并时调用的，得到了参与合并的最大值。
- terminate是最终Reduce合并时调用的，得到最大值。

> 这里参考了：
>
> <http://computerdragon.blog.51cto.com/6235984/1288567>
>
> <http://blog.csdn.net/xch_w/article/details/16886179>

**(2) 通用UDAF实例**

开发通用UDAF有两个步骤，第一个是编写resolver类，第二个是编写evaluator类。resolver负责类型检查，操作符重载。evaluator真正实现UDAF的逻辑。通常来说，顶层UDAF类继承org.apache.hadoop.hive.ql.udf.GenericUDAFResolver2，里面编写嵌套类evaluator 实现UDAF的逻辑。 通用UDAF使用场景较少，详情可以参看内置函数的源码，或者官方文档。

#### UDTF自定义表生成函数（一进多出）

UDTF是将一个输入值转变为一个数组。

下面这个例子是从nginx日志中的agent信息中提取浏览器名称和版本号的自定义函数，输入参数类似于：

”Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML like Gecko) Chrome/31.0.1650.63 Safari/537.36”，输出为：Chrome 31.0.1650.63。

```
package com.autohome.ics.bigdata.common.String;
import cz.mallat.uasparser.OnlineUpdater;
import cz.mallat.uasparser.UASparser;
import cz.mallat.uasparser.UserAgentInfo;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.*;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
/**
 * 解析浏览器信息UDTF
 *
 * 将日志中的http_user_agent，得到 browser_name,browser_version两个字段
 *  Created by ad on 2016/7/29.
 */
public class ParseUserAgentUDTF  extends GenericUDTF{
    private static UASparser uaSparser;
    static{
        try {
            uaSparser = new UASparser(OnlineUpdater.getVendoredInputStream());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    private final ListObjectInspector listIO = null;
    private final Object[] forwardObj = new Object[2];
    /**
     * 声明解析出来的字段名称和类型
     * @param argOIs
     * @return
     * @throws UDFArgumentException
     */
    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {
        if(argOIs.getAllStructFieldRefs().size() != 1){
            throw new UDFArgumentException("args error!");
        }
        ArrayList<String> fieldNames = new ArrayList<String>();
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<>();
        fieldNames.add("browser_name");
        fieldNames.add("browser_version");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,fieldOIs);
    }
    @Override
    public void process(Object[] args) throws HiveException {
        // 真正解析的地方
      /*
      输入字符串："Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.63 Safari/537.36"
      解析为：
       */
        String userAgent = args[0].toString();
        try {
            UserAgentInfo userAgentInfo = uaSparser.parse(userAgent);
            List<String> bws = new ArrayList<>();
            bws.add(userAgentInfo.getUaFamily());
            bws.add(userAgentInfo.getBrowserVersionInfo());
            super.forward(bws.toArray(new String[0]));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    @Override
    public void close() throws HiveException {
    }
}

```

自定义函数使用方式：

1. 打包为jar包（例如包名为：common.jar），因为这个程序引用了依赖uasparser，所以打包时注意应该将依赖uasparser打进去。

2. hive中添加jar包

   add jar lib/common.jar;

3. 声明函数

   create temporary function ParseUserAgent as 'com.autohome.ics.bigdata.common.String. ParseUserAgentUDTF';

4. 查询 select ParseUserAgent(agent) from user*action*log_like;

5. 结果 Chrome 31.0.1650.63 Chrome 31.0.1650.63 Chrome 31.0.1650.63 Chrome 31.0.1650.63 Chrome 31.0.1650.63

> 参考：[cwiki.apache.org](https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide+UDTF)

### 十、MapReduce执行过程

Hive语句最终是要转换为MapReduce程序放到Hadoop上去执行的，如果想深入了解Hive，并能够很好地优化Hive语句，了解MapReduce的执行过程至关重要，因为只有知道了MapReduce程序是怎么执行的，才能了解Hive语句是怎么执行的，才能有针对性地优化。

#### 执行过程简介

MapReduce过程大体分为两个阶段：map函数阶段和reduce函数阶段，两个阶段之间有有个shuffle。

- Hadoop将MapReduce输入的数据划分为等长的小分片，一般每个分片是128M，因为HDFS的每个块是128M。Hadoop1.X中这个数是64M。
- map函数是数据准备阶段，读取分片内容，并筛选掉不需要的数据，将数据解析为键值对的形式输出，map函数核心目的是形成对数据的索引，以供reduce函数方便对数据进行分析。
- 在map函数执行完后，进行map端的shuffle过程，map端的shuffle是将map函数的输出进行分区，不同分区的数据要传入不同的Reduce里去。
- 各个分区里的数据传入Reduce后，会先进行Reduce端的Shuffle过程，这里会将各个Map传递过来的相同分区的进行排序，然后进行分组，一个分组的数据执行一次reduce函数。
- reduce函数以分组的数据为数据源，对数据进行相应的分析，输出结果为最终的目标数据。
- 由于map任务的输出结果传递给reduce任务过程中，是在节点间的传输，是占用带宽的，这样带宽就制约了程序执行过程的最大吞吐量，为了减少map和reduce间的数据传输，在map后面添加了combiner函数来就map结果进行预处理，combiner函数是运行在map所在节点的。

下面的示例图描述了整个MapReduce的执行过程：

![enter image description here](http://images.gitbook.cn/757f2810-406d-11e7-8a79-013e9f3a234c)

#### 工作机制

![enter image description here](http://images.gitbook.cn/c38ed460-406d-11e7-ae67-138e994b76e1)

#### 分片

HDFS上的文件要用很多mapper进程处理，而map函数接收的输入是键值对的形式，所以要先将文件进行切分并组织成键值对的形式，这个切分和转换的过程就是数据分片。

在编写MapReduce程序时，可以通过job.setInputFormatClass()方法设置分片规则，如果没有指定默认是用TextInputFormat类。分片规则类都必须继承于FileInputFormat。

#### Map过程

每个数据分片将启动一个Map进程来处理，分片里的每个键值对运行一次map函数，根据map函数里定义的业务逻辑处理后，得到指定类型的键值对。

#### Map Shuffle过程

Map过程后要进行Map端的Shuffle阶段，Map端的Shuffle数据处理过程如下图所示：

![enter image description here](http://images.gitbook.cn/ee870b10-406d-11e7-8a79-013e9f3a234c)

**环形缓冲区**

Map输出结果是先放入内存中的一个环形缓冲区，这个环形缓冲区默认大小为100M(这个大小可以在io.sort.mb属性中设置)，当环形缓冲区里的数据量达到阀值时（这个值可以在io.sort.spill.percent属性中设置）就会溢出写入到磁盘，环形缓冲区是遵循先进先出原则，Map输出一直不停地写入，一个后台进程不时地读取后写入磁盘，如果写入速度快于读取速度导致环形缓冲区里满了时，map输出会被阻塞直到写磁盘过程结束。

**分区**

从环形缓冲区溢出到磁盘过程，是将数据写入mapred.local.dir属性指定目录下的特定子目录的过程。 但是在真正写入磁盘之前，要进行一系列的操作，首先就是对于每个键，根据规则计算出来将来要输出到哪个reduce，根据reduce不同分不同的区，分区是在内存里分的，分区的个数和将来的reduce个数是一致的。

**排序**

在每个分区上，会根据键进行排序。

**Combiner**

combiner方法是对于map输出的结果按照业务逻辑预先进行处理，目的是对数据进行合并，减少map输出的数据量。

排序后，如果指定了conmbiner方法，就运行combiner方法使得map的结果更紧凑，从而减少写入磁盘和将来网络传输的数据量。

**合并溢出文件**

环形缓冲区每次溢出，都会生成一个文件，所以在map任务全部完成之前，会进行合并成为一个溢出文件，每次溢出的各个文件都是按照分区进行排好序的，所以在合并文件过程中，也要进行分区和排序，最终形成一个已经分区和排好序的map输出文件。

在合并文件时，如果文件个数大于某个指定的数量（可以在min.num.spills.for.combine属性设置），就会进再次combiner操作，如果文件太少，效果和效率上，就不值得花时间再去执行combiner来减少数据量了。

**压缩**

Map输出结果在进行了一系列的分区、排序、combiner合并、合并溢出文件后，得到一个map最终的结果后，就应该真正存储这个结果了，在存储之前，可以对最终结果数据进行压缩，一是可以节约磁盘空间，而是可以减少传递给reduce时的网络传输数据量。

默认是不进行压缩的，可以在mapred.compress.map.output属性设置为true就启用了压缩，而压缩的算法有很多，可以在mapred.map.output.compression.codec属性中指定采用的压缩算法，具体压缩详情，可以看本文的后面部分的介绍。

#### Reduce Shuffle过程

Map端Shuffle完成后，将处理结果存入磁盘，然后通过网络传输到Reduce节点上，Reduce端首先对各个Map传递过来的数据进行Reduce 端的Shuffle操作，Reduce端的Shuffle过程如下所示：

![enter image description here](http://images.gitbook.cn/3a2912c0-406e-11e7-8a79-013e9f3a234c)

**复制数据**

各个map完成时间肯定是不同的，只要有一个map执行完成，reduce就开始去从已完成的map节点上复制输出文件中属于它的分区中的数据，reduce端是多线程并行来复制各个map节点的输出文件的，线程数可以在mapred.reduce.parallel.copies属性中设置。

reduce将复制来的数据放入内存缓冲区（缓冲区大小可以在mapred.job.shuffle.input.buffer.percent属性中设置）。当内存缓冲区中数据达到阀值大小或者达到map输出阀值，就会溢写到磁盘。

写入磁盘之前，会对各个map节点来的数据进行合并排序，合并时如果指定了combiner，则会再次执行combiner以尽量减少写入磁盘的数据量。为了合并，如果map输出是压缩过的，要在内存中先解压缩后合并。

**合并排序**

合并排序其实是和复制文件同时并行执行的，最终目的是将来自各个map节点的数据合并并排序后，形成一个文件。

**分组**

分组是将相同key的键值对分为一组，一组是一个列表，列表中每一组在一次reduce方法中处理。

**执行Reduce方法**

Reduce端的Shuffle完成后，就交由reduce方法来进行处理了。

#### Reduce过程

Reduce端的Shuffle过程后，最终形成了分好组的键值对列表，相同键的数据分为一组，分组的键是分组的键，值是原来值得列表，然后每一个分组执行一次reduce函数，根据reduce函数里的业务逻辑处理后，生成指定格式的键值对。

### 十一、性能优化

Hadoop启动开销大，如果每次只做小数量的输入输出，利用率将会很低。所以用好Hadoop的首要任务是增大每次任务所搭载的数据量。Hadoop的核心能力是parition和sort，因而这也是优化的根本。 Hive优化时，把hive Sql当做mapreduce程序来读，而不是当做SQL来读。

#### HiveQL层面优化

**利用分区表优化**

分区表是在某一个或者某几个维度上对数据进行分类存储，一个分区对应于一个目录。在这中的存储方式，当查询时，如果筛选条件里有分区字段，那么Hive只需要遍历对应分区目录下的文件即可，不用全局遍历数据，使得处理的数据量大大减少，提高查询效率。

当一个Hive表的查询大多数情况下，会根据某一个字段进行筛选时，那么非常适合创建为分区表。

**利用桶表优化**

桶表的概念在前面有详细介绍，就是指定桶的个数后，存储数据时，根据某一个字段进行哈希后，确定存储在哪个桶里，这样做的目的和分区表类似，也是使得筛选时不用全局遍历所有的数据，只需要遍历所在桶就可以了。

- hive.optimize.bucketmapJOIN 为true
- sort-merge JOIN hive.input.format=org.apache.hadoop.hive.ql.io.bucketizedHiveInputFormat; hive.optimize.bucketmapjoin=true; hive.optimize.bucketmapjoin.sortedmerge=true;

**join优化**

- 优先过滤后再join,最大限度地减少参与Join的数据量。
- 小表join大表原则。 应该遵守小表join大表原则，原因是Join操作在reduce阶段，位于join左边的表内容会被加载进内存，将条目少的表放在左边，可以有效减少发生内存溢出的几率。join中执行顺序是从左到右生成Job，应该保证连续查询中的表的大小从左到右是依次增加的。
- join on 条件相同的放入一个job. hive中，当多个表进行join时，如果join on的条件相同，那么他们会合并为一个MapReduce Job，所以利用这个特性，可以将相同的join on的放入一个job来节省执行时间。

```
select pt.page_id,count(t.url) PV
from rpt_page_type pt
join
(
    select url_page_id,url from trackinfo where ds='2016-10-11'
) t    on pt.page_id=t.url_page_id
join
(
    select page_id from rpt_page_kpi_new where ds='2016-10-11'
) r    on t.url_page_id=r.page_id
group by pt.page_id;

```

**启用mapjoin**

mapjoin是将join双方比较小的表直接分发到各个map进程的内存中，在map进程中进行join操作，这样就省掉了reduce步骤，提高了速度。

mapjoin相关参数如下：

```
<property>
<name>hive.auto.convert.join</name>
    <value>true</value>
</property>

<property>
<name>hive.auto.convert.join</name>
    <value>true</value>
</property>  

<property>
 <name>hive.auto.convert.join.noconditionaltask.size</name>
    <value>10000000</value>
</property>  

<property>
<name>hive.auto.convert.join.use.nonstaged</name>
    <value>false</value>   
</property>

```

- hive.auto.convert.join

  为true时，join方数据量小的表会整体分发到各个map进程的内存中，在map进程本地进行join操作，这样能大大提高运算效率，牺牲的是内存容量，所以数据量小于某一个值的才允许用mapjoin分发到各个map节点里，而这个值用以下参数来配置。

- hive.auto.convert.join.noconditionaltask

  设置为true，hive才基于输入文件大小进行自动转换为mapjoin.

- hive.auto.convert.join.noconditionaltask.size

  指定小于多少的表数据放入map内存，使用mapjoin,默认是10M.

- 这个优化只对join有效，对left join、right join 无效。

**桶表mapjoin**

当两个分桶表join时，如果join on的是分桶字段，小表的分桶数时大表的倍数时，可以启用map join来提高效率。启用桶表mapjoin要启用hive.optimize.bucketmapjoin参数。

```
<property>
   <name>hive.optimize.bucketmapjoin</name>
   <value>true</value>
   <description>Whether to try bucket mapjoin</description>
</property>

```

**Group By数据倾斜优化**

Group By很容易导致数据倾斜问题，因为实际业务中，通常是数据集中在某些点上，这也符合常见的2/8原则，这样会造成对数据分组后，某一些分组上数据量非常大，而其他的分组上数据量很小，而在mapreduce程序中，同一个分组的数据会分配到同一个reduce操作上去，导致某一些reduce压力很大，其他的reduce压力很小，这就是数据倾斜，整个job执行时间取决于那个执行最慢的那个reduce。 解决这个问题的方法是配置一个参数：set hive.groupby.skewindata=true。

当选项设定为 true，生成的查询计划会有两个 MR Job。第一个 MR Job 中， Map的输出结果会随机分布到 Reduce 中，每个 Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的，在第一个Job中通过聚合操作减少了数据量；第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 GroupBy Key 被分布到同一个 Reduce 中），最后完成最终的聚合操作。

**Order By 优化**

因为order by只能是在一个reduce进程中进行的，所以如果对一个大数据集进行order by,会导致一个reduce进程中处理的数据相当大，造成查询执行超级缓慢。在要有进行order by 全局排序的需求时，用以下几个措施优化：

- 在最终结果上进行order by，不要在中间的大数据集上进行排序。如果最终结果较少，可以在一个reduce上进行排序时，那么就在最后的结果集上进行order by。
- 如果需求是取排序后前N条数据，那么可以使用distribute by和sort by在各个reduce上进行排序后取前N条，然后再对各个reduce的结果集合并后在一个reduce中全局排序，再取前N条，因为参与全局排序的Order By的数据量最多有reduce个数*N，所以速度很快。 例子：

```
select a.leads_id,a.user_name from 
(
  select leads_id,user_name  from user_leads  
distribute by length(user_name)  sort by length(user_name) desc limit 10
 ) a order by length(a.user_name) desc limit 10;

```

**Group By Map端聚合**

并不是所有的聚合操作都需要在 Reduce 端完成，很多聚合操作都可以先在 Map端进行部分聚合，最后在 Reduce 端得出最终结果。

hive.map.aggr = true 是否在 Map 端进行聚合，默认为 True。

hive.groupby.mapaggr.checkinterval = 100000 在 Map 端进行聚合操作的条目数目

**一次读取多次插入**

有些场景是从一个表读取数据后，要多次利用，这时候就可以使用multi insert语法：

```
from user_action_log
 insert overwrite table log1 select companyid,originalstring  where companyid='100006'
 insert overwrite table log2 select companyid,originalstring  where companyid='10002'

```

每次hive查询，都会将数据集整个遍历一遍。当查询结果会插入多个表中时，可以采用以上语法，将一次遍历写入多个表，以达到提高效率的目的。

**Join字段显示类型转换**

当参与join的字段类型不一致时，Hive会自动进行类型转换，但是自动转换有时候效率并不高，可以根据实际情况通过显示类型转换来避免HIVE的自动转换。

**使用orc、parquet等列式存储格式**

创建表时，尽量使用orc、parquet这些列式存储格式，因为列式存储的表，每一列的数据在物理上是存储在一起的，Hive查询时会只遍历需要列数据，大大减少处理的数据量。

#### Hive架构层面优化

**不执行MapReduce**

hive中有个参数：hive.fetch.task.conversion，定义如下：

```
<property>
  <name>hive.fetch.task.conversion</name>
  <value>minimal</value>
  <description>
    Some select queries can be converted to single FETCH task minimizing latency.
    Currently the query should be single sourced not having any subquery and should not have
    any aggregations or distincts (which incurs RS), lateral views and joins.
    1. minimal : SELECT STAR, FILTER on partition columns, LIMIT only
    2. more    : SELECT, FILTER, LIMIT only (TABLESAMPLE, virtual columns)
  </description>
</property>

```

Hive从HDFS读取数据，有两种方式：启用MapReduce读取、直接抓取。

很显然直接抓取数据比MapReduce读取数据要快的多，但是只有少数操作可以直接抓取数据，hive.fetch.task.conversion参数就是设置什么情况下采用直接抓取方法，它的值有两个：

- minimal:只有 select * 、在分区字段上where过滤、有limit这三种场景下才启用直接抓取方式。
- more:在select、where筛选、limit时，都启用直接抓取方式。

启用fetch more模式: set hive.fetch.task.conversion=more;

实例：

```
set hive.fetch.task.conversion=more;
select userid,username from user_info where cityid is not null;

```

这个例子中，如果set hive.fetch.task.conversion=minimal，那么下面的查询语句会以MapReduce方法执行，运行时间比较长，但是改为more后，发现查询速度非常快。

**本地模式执行MapReduce**

Hive在集群上查询时，默认是在集群上N台机器上运行，需要多个机器进行协调运行，这个方式很好地解决了大数据量的查询问题。但是当Hive查询处理的数据量比较小时，其实没有必要启动分布式模式去执行，因为以分布式方式执行就涉及到跨网络传输、多节点协调等，并且消耗资源。这个时间可以只使用本地模式来执行mapreduce job，只在一台机器上执行，速度会很快。

启动本地模式涉及到三个参数：

| **参数名**                                  | **默认值** | **备注**             |
| ---------------------------------------- | ------- | ------------------ |
| hive.exec.mode.local.auto                | false   | 让hive决定是否在本地模式自动运行 |
| hive.exec.mode.local.auto.input.files.max | 4       | 不启用本地模式的task最大个数   |
| hive.exec.mode.local.auto.inputbytes.max | 128M    | 不启动本地模式的最大输入文件大小   |

各个参数定义如下：

```
<property>
  <name>hive.exec.mode.local.auto</name>
  <value>false</value>
  <description> Let Hive determine whether to run in local mode automatically </description>
</property>

<property>
    <name>hive.exec.mode.local.auto.input.files.max</name>
    <value>4</value>
    <description>When hive.exec.mode.local.auto is true, the number of tasks should less than this for local mode.</description>
</property>

<property>
    <name>hive.exec.mode.local.auto.inputbytes.max</name>
    <value>134217728</value>
    <description>When hive.exec.mode.local.auto is true, input bytes should less than this for local mode.</description>
</property>

```

set hive.exec.mode.local.auto=true是打开hive自动判断是否启动本地模式的开关，但是只是打开这个参数并不能保证启动本地模式，要当map任务数不超过hive.exec.mode.local.auto.input.files.max的个数并且map输入文件大小不超过hive.exec.mode.local.auto.inputbytes.max所指定的大小时，才能启动本地模式。

**JVM重用**

因为Hive语句最终要转换为一系列的MapReduce Job的，而每一个MapReduce Job是由一系列的Map Task和Reduce Task组成的，默认情况下，MapReduce中一个Map Task或者一个Reduce Task就会启动一个JVM进程，一个Task执行完毕后，JVM进程就退出。这样如果任务花费时间很短，又要多次启动JVM的情况下，JVM的启动时间会变成一个比较大的消耗，这个时候，就可以通过重用JVM来解决。

set mapred.job.reuse.jvm.num.tasks=5

这个设置就是制定一个jvm进程在运行多次任务之后再退出，这样一来，节约了很多的JVM的启动时间。

**并行化**

一个hive sql语句可能会转为多个mapreduce Job，每一个job就是一个stage，这些job顺序执行，这个在hue的运行日志中也可以看到。但是有时候这些任务之间并不是是相互依赖的，如果集群资源允许的话，可以让多个并不相互依赖stage并发执行，这样就节约了时间，提高了执行速度，但是如果集群资源匮乏时，启用并行化反倒是会导致各个job相互抢占资源而导致整体执行性能的下降。

启用并行化：

set hive.exec.parallel=true;

### 十二、后记

还是那句话，Hive入门使用很容易，这得益于它采用了类似SQL语句的方式与用户交互，这也是Hive被大量使用的原因，但是最好还是要理解Hive背后的执行原理，这样才能开发出高效的程序。 以上就是对Hive入门者的建议。

我第一次参与GitChat的分享，不知效果如何，如果大家对本次Hive还算满意的话，我接下来进行一次Hive进阶和MapReduce的分享，感谢大家的参与。

### 参考资料

1. 《Hive编程指南》 Eduard Capriolo、Dean Wampler、Jason Rutberglen （著），曹坤（译）。
2. Hive官方文档：<https://cwiki.apache.org/confluence/display/Hive/GettingStarted>.
3. 互联网上其他资源。