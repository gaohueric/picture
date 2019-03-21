## 内容提要



内容提要：

- 多表查询的时候，会出现一个映射文件一堆association 或者collection 或者 left/inner join，这在后期维护时很麻烦。请问如何去解决这种问题？如何高效的利用resultMap？
- 每一次mapper里查询调用都会创建一个SqlSession，所以对应的如文中示例StudentMapper永远不会有一级缓存命中这是怎么办？是否只有在手动通过sqlsession多次查询同一Sql才会命中？
- Mybatis中#和$符号的主要区别在哪，除了说 #{} 使用了JDBC的preparedStatement进行预编译， 防止Sql 注入，还有其他的吗？
- 咨询Mybatis 针对mysql 和oracle 分页的解决方案有什么？
- Mybatis查询例如30万、100万的数据，如何优化？
- mysql过千万数据删除需要注意什么问题？
- 现在我们是使用spring Mybatis的搭配下使用，使用过程中对Mybatis的某个查询在一个方法中进行了多次调用，由于方法查询出来后就会解密，然后Mybatis缓存了解密后的数据，导致程序出现了解密报错的信息，Mybatis是怎么样进行隔离查询是不是要去读取缓存的？在我看来两次查询应该不算是同一个sqlsession对吗？
- Mybatis主要缺点是什么？何时应该选用Hibernate而非Mybatis？
- Mybatis二级缓存：假设在Mapper1中做了查询的缓存， 在 Mapper2下进行修改， 那么缓存读出来的数据会正确吗？
- 在业务中使用缓存，需要考虑缓存失效清空等操作的地方很多，使用mapper二级缓存会方便很多，但是如作者所说，有很多问题，不建议使用，能够方便的使用缓存，尤其是与redis结合。有没有推荐的其他工具或者库吗？
- 二级缓存有局限性，细粒度的二级缓存有什么好的实现方式就是修改一个基本用户信息（即执行commit（）方法），只修改缓存中这一个用户信息，其余的信息不清空。具体生产环境是如何实现？Mybatis的延迟加载和缓存技术有什么更好方式，有例子举例吗？
- mysql使用中间件做读写分离和程序代码层面做读写分离，哪个更好，原因是什么？
- 请问如何着手分析Mybatis源码？
- 请问有哪些常用的对性能有改善的配置建议？
- Mybatis能实现物理分页吗？最好是一次返回`PagingResult<XXXDto>`，包含分页信息和当前的数据。目的是想实现分页查询的逻辑部分复用，这样可以吗？

------

**问：多表查询的时候，会出现一个映射文件一堆association 或者collection 或者 left/inner join，这在后期维护时很麻烦。请问如何去解决这种问题？如何高效的利用resultMap？**

**答：**一般是不建议去做很多表的查询的，这样做的后果是业务逻辑都放在数据库去做，和数据库表强耦合，如果你后续对业务想去拆分，或者是做分库分表，其实很困难。而且数据库中放入过多的逻辑的话，也容易拖慢数据库的性能，在业务量上去后，瓶颈很容易出现在数据库这一侧，我平时的建议就是通过简单的Sql，在上层进行业务逻辑的拼装，而不是把业务逻辑都下沉下去，也是互联网开发中比较常见的做法。而Mybatis中的resultmap其实只是用来做映射的，关键还是如何去做好数据库表的设计。在我之前实习过的阿里和现在在的点评，都是希望Sql尽可能的简单，逻辑放到业务层上去做。

------

**问：每一次mapper里查询调用都会创建一个SqlSession，所以对应的如文中示例StudentMapper永远不会有一级缓存命中怎么办？是否只有在手动通过sqlsession多次查询同一Sql才会命中？**

**答：**只要是一个sqlsession中拿到的mapper，都会共享一个一级缓存。至于是否命中取决于查询语句是否相同，有无清空缓存的操作，前几个实验都有同一个mapper多次查询，验证缓存效果。

![enter image description here](http://images.gitbook.cn/99d0a3e0-6586-11e7-b46b-072d2365dc14)

整个cache是存在于你创建sqlsession时生成的executor的局部变量，在一个sqlsession中多次查询，且查询条件相同，满足MappedStatement的Id、Sql的offset、Sql的limit、Sql本身以及Sql中的参数相同，那么localcache就会生效，如果你和spring结合使用Mybatis的话，spring会托管sqlsession 的创建过程。

------

**问：Mybatis中#和$符号的主要区别在哪，除了说 #{} 使用了JDBC的preparedStatement进行预编译， 防止Sql 注入，还有其他的吗？**

**答：**这个我写的了一篇文章[《关于Mybatis的$和#,你真的知道他们的细节吗?》](http://mp.weixin.qq.com/s/x0bla9vTCU1v0h4rJ5zx_A)，可以看一下，大致说来，Mybatis默认都是使用PrepareStatement来作为底层的语句的。

使用`$`的话，初始化时，会交给DynamicSource处理。

使用`#`的话，初始化时，会交给StaticSqlSource处理。

在执行阶段，会统一获取BoundSql，不同的符号获取BoundSql的时候操作有所不同，使用$的话会直接替换字符串，拼装Sql。#的话，会被替换成问号，同时生成参数映射对，在交给StatementHandler参数化的时候进行字符的转义操作。

关于后续的预编译，其实和转义没有强关联。当没有开启预编译的时候，PrepareStatement会在本地进行转义话。当开启预编译后，会在数据库服务端进行这个操作，但这是分两步的：

1. 是预编译你带?号的Sql。
2. 是接收你的参数并进行转义。可以看我的这篇文章[《JDBC的小小研究》](http://mp.weixin.qq.com/s/rNJt2QMDzoiX0TSIzwJ3Yw) ，其中分析了这个问题。

通常说来，我们的运维同志们都没有在服务器端开启预编译，因此转义的工作是交给prepareStatement的实现类来做的，具体看每个厂家的实现，mysql的prepareStatement中做了转义操作，因此有预防Sql注入的效果。

------

**问：咨询Mybatis 针对mysql 和oracle 分页的解决方案有什么？**

**答：**一般来说，每个数据库的厂商分页用的语句都可能不同，mysql 一般是limit，然后加上偏移量，oracle我不熟，但我查阅了一下资料是使用rownum。通常来说，有这么几种做法：

第一种：最以前的内存分页，把数据全部捞到内存，其实就是假分页，通过游标的方式，截取我们需要的记录，Mybatis原生的rowBounds就支持这种方式，一种就是最以前的内存分页，把数据全部捞到内存，其实就是假分页，通过游标的方式，截取我们需要的记录，Mybatis原生的rowBounds就支持这种方式。

第二种：在我们的映射文件中，如果知道所服务的数据库，可以映射文件里，写下他对应的需要分页用的语句，比如mysql，SELECT XXXX limit #{start},#{limit}，然后在业务层计算出start的位置，带进去进行查询。

第三种：使用拦截器，应该也是大多数分页插件的做法。拦截StatementHandler，在运行的时候对Sql进行改写，你只需要传入你需要的页数，和每页的一个大小。根据不同的厂商，对你的Sql进行改写。不过在使用此类分页插件的时候需要注意一个问题，如果分页插件是在statement拦截的话，在进入executor时的Sql都是一样的，这个时候如果你开启了缓存，可能会使用你的分页插件失效，因为statmenthandler已经是过了executor了，在Mybatis看来，你传进来的Sql可能是一样的，需要额外注意，一个小tip。

------

**问：Mybatis查询例如30万、100万的数据，如何优化？**

**答：** PageHelper 以及我司开源的zebra都是对分页支持的比较好的插件之一。Mybatis只是一个封装了JDBC的框架，这个问题，如果是想问，查询大数据量如何优化查询的话，就是建立合适的索引，优化Sql。如果是问，一次性查这么多数据的话，不是Mybatis关心的范畴，可能给的优化建议就是分批查询，同时选择一个底层合适的存储，具体可以结合网上的一些专业的评测，这里我就具体不展开了。

------

**问：mysql过千万数据删除需要注意什么问题？**

**答：**这个问题就很偏DBA这一块了，在我们的日常场景中，像后端开发，我们一般是读写数据会比较多一些，不过在之前的工作中，也遇到过删除大数据时遇到过问题。

之前我有一次跑定时任务，删了大量数据，进入了一个很大的事务，然后引起数据库IO出现问题，影响了其他应用的访问。如果一次删除大量数据的话，也很容易引起主从延时。

我遇到问题之后得到的一些经验就是批量删除，每次删除前先sleep一会，可以减小对数据库IO的一个突发压力，也防止从库和主库差距过大。另外就是，如果要删除大量数据的话，可以看一下你是否有需要保留的数据，这个量级怎么样，可以创建一个新表，把你需要的数据放过去，然后切表。同时推荐的就是分批删，因为一个很大的删除操作，事务会很长，如果发生问题，回滚就很慢。分批的话，也利于容错。

------

**问：现在我们是使用spring Mybatis的搭配下使用，使用过程中对Mybatis的某个查询在一个方法中进行了多次调用，由于方法查询出来后就会解密，然后Mybatis缓存了解密后的数据，导致程序出现了解密报错的信息，Mybatis是怎么样进行隔离查询是不是要去读取缓存的？在我看来两次查询应该不算是同一个sqlsession对吗？**

**答：**这个问题其实我不是很看的明白，我理解的是Mybatis查出数据后你们进行解密，加密的数据是存储在数据库中的，那么Mybatis只会缓存没有解密的数据，关于为何引起解密报错就不太明白了。

第二个就是两次查询，如果是和spring结合的话，取决于你是否开启了事务，当开启事务的时候，多次查询都会共享一个localcache，如果没有开启事务的话，一个操作就是一个sqlsession。另外我建议还是把一级缓存关了，容易引起脏数据，如果在确定是读多远远大于写的时候才建议使用。

------

**问：Mybatis主要缺点是什么？何时应该选用Hibernate而非Mybatis？**

**答：**先来说说Mybatis的优点，Mybatis框架相对简单很容易上手，使用XML把Sql和代码解耦开来，然后解决了。JDBC冗余的开发过程以及数据映射问题，Mybatis是一个很轻量级的框架，关注的就是POJO与Sql之间的映射关系。然后通过映射配置文件，将Sql所需的参数，以及返回的结果字段映射到指定POJO做的是一个mapping的操作，那么相对来说，Mybatis对于开发人员的要求就比较高。而Hibernate对数据库结构提供了较为完整的封装，Hibernate的O/R Mapping实现了POJO 和数据库表之间的映射，以及Sql的自动生成和执行。而且提供了一些简单的操作方法，解放了很多的开发时间。我个人觉得两者都可以，看你团队和公司的一个使用习惯，我个人倾向于对开发人员可控性更强的Mybatis，hibernate相对来说，它控制的部分更多一些。

------

**问：Mybatis 二级缓存：假设 在 Mapper1中做了查询的缓存， 在 Mapper2下进行修改， 那么缓存读出来的数据会正确吗？**

**答：**首先在文中解释过了，二级缓存是基于namespace的，你在其他mapper下数据的更改，另一个mapper无法感应到，当你有连接操作的时候，你放在任何一个映射文件中，当其他namespace影响了你的表的数据时候。你没法感知到，因为用的都不是一个cache，除非你使用cacheref，让多个mapper用一个cache，不过那样的话，cache的影响因素就太多了，存在的意义也就不大了。

![enter image description here](http://images.gitbook.cn/c9b82100-6586-11e7-b46b-072d2365dc14)

------

**问：在业务中使用缓存，需要考虑缓存失效清空等操作的地方很多，使用mapper二级缓存会方便很多，但是如作者所说，有很多问题，不建议使用，能够方便的使用缓存，尤其是与redis结合。有没有推荐的其他工具或者库吗？**

**答：**其实一般来说，我们平时的业务场景中，都是直接使用redis来做业务缓存的，如果有额外的定制需求的话，Mybatis也提供接入自定义cache的设置，可以实现cache这个接口，使用redis，来达到利用二级缓存的目的。常见的还有edcache，可以私下了解一下。

------

**问：二级缓存有局限性，细粒度的二级缓存有什么好的实现方式 就是修改一个基本用户信息（即执行commit（）方法），只修改缓存中这一个用户信息，其余的信息不清空。具体生产环境是如何实现？Mybatis的延迟加载和缓存技术有什么更好方式，有例子举例吗？**

**答：**这个和刚刚的问题有些类似，Mybatis的二级缓存用起来要考虑的点很多，其实可以直接就使用分布式缓存来做你的业务缓存，比如你说的这个用户信息，你完全可以自己构造合适的key，在更新用户相关的操作时，清楚这个对应的缓存，其实不一定要使用Mybatis的二级缓存。Mybatis的延迟加载，当需要用到某一部分数据的时候再去加载那一部分的数据，在实际业务场景中，并没有使用这个特性，所以无法深入解答。

------

**问：mysql使用中间件做读写分离和程序代码层面做读写分离，哪个更好，原因是什么？**

**答：**没有哪个更好。如果是在程序代码做的话，你可以自己控制你走路由的规则，进行合理的主从读写路由设计。当这个程序代码做的很好的时候，可以抽象出来，作为一个中间件，把路由的规则做成可配置话，比如哪个库可读，哪个库做读写，在开发的层面，也可以设计一些规则，由你告诉中间件，我这个Sql要强制走主库，或者走从库等，中间件最开始肯定是在程序代码中慢慢演进出来的，成为一个可以被多个业务接入使用的组件。

------

**问：请问如何着手分析Mybatis源码？**

**答：**我理解如何分析一个组件的源码，就是你要知道为什么会有它。比如在Mybatis之前，我们都使用JDBC，那么JDBC在使用的时候有一些问题，数据库列名和java对象列名的映射麻烦；类型转换工作有大量的代码要写；开发的过程中Sql和java代码结合过于紧密 有很多模板代码要写，这些就是Mybatis出生的原因，那么你可以从一些点进入分析，比如 Mybatis 是如何解决列名到java对象字段名的映射的。

![enter image description here](http://images.gitbook.cn/ea4409c0-6586-11e7-b46b-072d2365dc14)

![enter image description here](http://images.gitbook.cn/062bbed0-6587-11e7-b46b-072d2365dc14)

你可以了解resultmap的特性，那么到后来你可以切入了解，他是怎么实现数据转换的，在根据你使用的时候的细节，了解其中的特性和运作流程，那么你对这个组件就掌握了。

------

**问：请问有哪些常用的对性能有改善的配置建议？**

**答：**可以了解下Mybatis动态语句的一些特性，比如批量操作，可以去使用Mybatis的foreach标签。通过一些别名的方式，优化可读性，其实myabtis本质在性能上可调的空间不大，应为他本质上是做一个映射的框架。可以在Sql层面做尽可能多的优化。

------

**问：Mybatis能实现物理分页吗？最好是一次返回PagingResult<XXXDto>，包含分页信息和当前的数据。目的是想实现分页查询的逻辑部分复用。这样可以吗？**

**答：**刚才在说分页的时候提到了，Mybatis可以通过插件的方式实现分页，具体可以参考一下之前网友推荐的pagehelper或者自己去写拦截器实现，原生的话，我了解是支持游标的方式。



## 文章实录

### 前言

基于个人的兴趣，开了这场chat，主题是Mybatis一级和二级缓存的应用及源码分析。希望在本场chat结束后，能够帮助读者朋友明白以下三点。

1. Mybatis是什么。
2. Mybatis一级和二级缓存如何配置使用。
3. Mybatis一级和二级缓存的工作流程及源码分析。

本次分析中涉及到的代码和数据库表均放在Github上，地址: [mybatis-cache-demo](https://github.com/kailuncen/mybatis-cache-demo)。

### 目录

为达到以上三个目的，本文按照以下顺序展开。

1. Mybatis的基础概念。
2. 一级缓存介绍及相关配置。
3. 一级缓存工作流程及源码分析。
4. 一级缓存总结。
5. 二级缓存介绍及相关配置。
6. 二级缓存源码分析。
7. 二级缓存总结。
8. 全文总结。

### Mybatis的基础概念

本章节会对Mybatis进行大体的介绍，分为官方定义和核心组件介绍。

首先是Mybatis官方定义，如下所示。

> MyBatis是支持定制化SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。MyBatis可以对配置和原生Map使用简单的XML或注解，将接口和Java 的POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

其次是Mybatis的几个核心概念。

1. SqlSession : 代表和数据库的一次会话，向用户提供了操作数据库的方法。
2. MappedStatement: 代表要发往数据库执行的指令，可以理解为是Sql的抽象表示。
3. Executor: 具体用来和数据库交互的执行器，接受MappedStatement作为参数。
4. 映射接口: 在接口中会要执行的Sql用一个方法来表示，具体的Sql写在映射文件中。
5. 映射文件: 可以理解为是Mybatis编写Sql的地方，通常来说每一张单表都会对应着一个映射文件，在该文件中会定义Sql语句入参和出参的形式。

下图就是一个针对Student表操作的接口文件StudentMapper，在StudentMapper中，我们可以若干方法，这个方法背后就是代表着要执行的Sql的意义。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-20-54-43.jpg)

通常也可以把涉及多表查询的方法定义在StudentMapper中，如果查询的主体仍然是Student表的信息。也可以将涉及多表查询的语句单独抽出一个独立的接口文件。

在定义完接口文件后，我们会开发一个Sql映射文件，主要由mapper元素和select|insert|update|delete元素构成，如下图所示。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-30-16-28-23.jpg)

mapper元素代表这个文件是一个映射文件，使用namespace和具体的映射接口绑定起来，namespace的值就是这个接口的全限定类名。select|insert|update|delete代表的是Sql语句，映射接口中定义的每一个方法也会和映射文件中的语句通过id的方式绑定起来，方法名就是语句的id，同时会定义语句的入参和出参，用于完成和Java对象之间的转换。

在Mybatis初始化的时候，每一个语句都会使用对应的MappedStatement代表，使用namespace+语句本身的id来代表这个语句。如下代码所示，使用mapper.StudentMapper.getStudentById代表其对应的Sql。

```
SELECT id,name,age FROM student WHERE id = #{id}

```

在Mybatis执行时，会进入对应接口的方法，通过类名加上方法名的组合生成id，找到需要的MappedStatement，交给执行器使用。

至此，Mybatis的基础概念介绍完毕。

### 一级缓存

#### 一级缓存介绍

在系统代码的运行中，我们可能会在一个数据库会话中，执行多次查询条件完全相同的Sql，鉴于日常应用的大部分场景都是读多写少，这重复的查询会带来一定的网络开销，同时select查询的量比较大的话，对数据库的性能是有比较大的影响的。

如果是Mysql数据库的话，在服务端和Jdbc端都开启预编译支持的话，可以在本地JVM端缓存Statement,可以在Mysql服务端直接执行Sql，省去编译Sql的步骤，但也无法避免和数据库之间的重复交互。关于Jdbc和Mysql预编译缓存的事情，可以看我的这篇博客[JDBC和Mysql那些事](https://my.oschina.net/kailuncen/blog/905395)。

Mybatis提供了一级缓存的方案来优化在数据库会话间重复查询的问题。实现的方式是每一个SqlSession中都持有了自己的缓存，一种是SESSION级别，即在一个Mybatis会话中执行的所有语句，都会共享这一个缓存。一种是STATEMENT级别，可以理解为缓存只对当前执行的这一个statement有效。如果用一张图来代表一级查询的查询过程的话，可以用下图表示。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-28-20-50-32.jpg)

每一个SqlSession中持有了自己的Executor，每一个Executor中有一个Local Cache。当用户发起查询时，Mybatis会根据当前执行的MappedStatement生成一个key，去Local Cache中查询，如果缓存命中的话，返回。如果缓存没有命中的话，则写入Local Cache，最后返回结果给用户。

#### 一级缓存配置

上文介绍了一级缓存的实现方式，解决了什么问题。在这个章节，我们学习如何使用Mybatis的一级缓存。只需要在Mybatis的配置文件中，添加如下语句，就可以使用一级缓存。共有两个选项，SESSION或者STATEMENT，默认是SESSION级别。

```
<setting name="localCacheScope" value="SESSION"/>

```

#### 一级缓存实验

配置完毕后，通过实验的方式了解Mybatis一级缓存的效果。每一个单元测试后都请恢复被修改的数据。 首先是创建了一个示例表student,为其创建了对应的POJO类和增改的方法，具体可以在entity包和Mapper包中查看。

```
CREATE TABLE `student` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(200) COLLATE utf8_bin DEFAULT NULL,
  `age` tinyint(3) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

```

在以下实验中，id为1的学生名称是凯伦。

**实验1**

开启一级缓存，范围为会话级别，调用三次getStudentById，代码如下所示:

```
public void getStudentById() throws Exception {
        SqlSession sqlSession = factory.openSession(true); // 自动提交事务
        StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
        System.out.println(studentMapper.getStudentById(1));
        System.out.println(studentMapper.getStudentById(1));
        System.out.println(studentMapper.getStudentById(1));
    }

```

执行结果:

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-26-21-42-07.jpg)

我们可以看到，只有第一次真正查询了数据库,后续的查询使用了一级缓存。

**实验2**

在这次的试验中，我们增加了对数据库的修改操作，验证在一次数据库会话中，对数据库发生了修改操作，一级缓存是否会失效。

```
@Test
public void addStudent() throws Exception {
        SqlSession sqlSession = factory.openSession(true); // 自动提交事务
        StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
        System.out.println(studentMapper.getStudentById(1));
        System.out.println("增加了" + studentMapper.addStudent(buildStudent()) + "个学生");
        System.out.println(studentMapper.getStudentById(1));
        sqlSession.close();
}

```

执行结果:

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-26-21-47-28.jpg)

我们可以看到，在修改操作后执行的相同查询，查询了数据库，**一级缓存失效**。

**实验3**

开启两个SqlSession，在sqlSession1中查询数据，使一级缓存生效，在sqlSession2中更新数据库，验证一级缓存只在数据库会话内部共享。

```
@Test
public void testLocalCacheScope() throws Exception {
        SqlSession sqlSession1 = factory.openSession(true); 
        SqlSession sqlSession2 = factory.openSession(true); 

       StudentMapper studentMapper = sqlSession1.getMapper(StudentMapper.class);
       StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);

        System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
        System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
        System.out.println("studentMapper2更新了" + studentMapper2.updateStudentName("小岑",1) + "个学生的数据");
        System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
        System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentById(1));
}

```

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-26-22-07-30.jpg)

我们可以看到，sqlSession2更新了id为1的学生的姓名，从凯伦改为了小岑，但session1之后的查询中，id为1的学生的名字还是凯伦，出现了脏数据，也证明了我们之前就得到的结论，一级缓存只存在于只在数据库会话内部共享。

#### 一级缓存工作流程&源码分析

这一章节主要从一级缓存的工作流程和源码层面对一级缓存进行学习。

**工作流程**

根据一级缓存的工作流程，我们绘制出一级缓存执行的时序图，如下图所示。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-28-21-57-17.png)

主要步骤如下:

- 对于某个Select Statement，根据该Statement生成key。
- 判断在Local Cache中,该key是否用对应的数据存在。
- 如果命中，则跳过查询数据库，继续往下走。
- 如果没命中：
- 去数据库中查询数据，得到查询结果。
- 将key和查询到的结果作为key和value，放入Local Cache中。
- 将查询结果返回。
- 判断缓存级别是否为STATEMENT级别，如果是的话，清空本地缓存。

**源码分析**

了解具体的工作流程后，我们队Mybatis查询相关的核心类和一级缓存的源码进行走读。这对于之后学习二级缓存时也有帮助。

**SqlSession**: 对外提供了用户和数据库之间交互需要的所有方法，隐藏了底层的细节。它的一个默认实现类是DefaultSqlSession。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-28-22-14-32.jpg)

**Executor**: SqlSession向用户提供操作数据库的方法，但和数据库操作有关的职责都会委托给Executor。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-28-22-17-44.jpg)

如下图所示，Executor有若干个实现类，为Executor赋予了不同的能力，大家可以根据类名，自行私下学习每个类的基本作用。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-28-22-20-30.jpg)

在一级缓存章节，我们主要学习BaseExecutor。

**BaseExecutor**: BaseExecutor是一个实现了Executor接口的抽象类，定义若干抽象方法，在执行的时候，把具体的操作委托给子类进行执行。

```
protected abstract int doUpdate(MappedStatement ms, Object parameter) throws SQLException;
protected abstract List<BatchResult> doFlushStatements(boolean isRollback) throws SQLException;
protected abstract <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException;
protected abstract <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql) throws SQLException;

```

在一级缓存的介绍中，我们提到对Local Cache的查询和写入是在Executor内部完成的。在阅读BaseExecutor的代码后，我们也发现Local Cache就是它内部的一个成员变量，如下代码所示。

```
public abstract class BaseExecutor implements Executor {
protected ConcurrentLinkedQueue<DeferredLoad> deferredLoads;
protected PerpetualCache localCache;

```

**Cache**: Mybatis中的Cache接口，提供了和缓存相关的最基本的操作，有若干个实现类，使用装饰器模式互相组装，提供丰富的操控缓存的能力。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-28-22-23-50.jpg)

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-28-22-26-04.jpg)

BaseExecutor成员变量之一的PerpetualCache，就是对Cache接口最基本的实现，其实现非常的简内部持有了hashmap，对一级缓存的操作其实就是对这个hashmap的操作。如下代码所示。

```
public class PerpetualCache implements Cache {
  private String id;
  private Map<Object, Object> cache = new HashMap<Object, Object>();

```

在阅读相关核心类代码后，从源代码层面对一级缓存工作中涉及到的相关代码，出于篇幅的考虑，对源码做适当删减，读者朋友可以结合本文，后续进行更详细的学习。

为了执行和数据库的交互，首先会通过DefaultSqlSessionFactory开启一个SqlSession，在创建SqlSession的过程中，会通过Configuration类创建一个全新的Executor，作为DefaultSqlSession构造函数的参数，代码如下所示。

```
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
      ............
    final Executor executor = configuration.newExecutor(tx, execType);     
    return new DefaultSqlSession(configuration, executor, autoCommit);
}

```

如果用户不进行制定的话，Configuration在创建Executor时，默认创建的类型就是SimpleExecutor,它是一个简单的执行类，只是单纯执行Sql。以下是具体用来创建的代码。

```
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    // 尤其可以注意这里，如果二级缓存开关开启的话，是使用CahingExecutor装饰BaseExecutor的子类
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);                      
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}

```

在SqlSession创建完毕后，根据Statment的不同类型，会进入SqlSession的不同方法中，如果是Select语句的话，最后会执行到SqlSession的selectList，代码如下所示。

```
@Override
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
      MappedStatement ms = configuration.getMappedStatement(statement);
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
}

```

在上文的代码中，SqlSession把具体的查询职责委托给了Executor。如果只开启了一级缓存的话，首先会进入BaseExecutor的query方法。代码如下所示。

```
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    BoundSql boundSql = ms.getBoundSql(parameter);
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}

```

在上述代码中，会先根据传入的参数生成CacheKey，进入该方法查看CacheKey是如何生成的，代码如下所示。

```
CacheKey cacheKey = new CacheKey();
cacheKey.update(ms.getId());
cacheKey.update(rowBounds.getOffset());
cacheKey.update(rowBounds.getLimit());
cacheKey.update(boundSql.getSql());
//后面是update了sql中带的参数
cacheKey.update(value);

```

在上述的代码中，我们可以看到它将MappedStatement的Id、sql的offset、Sql的limit、Sql本身以及Sql中的参数传入了CacheKey这个类，最终生成了CacheKey。我们看一下这个类的结构。

```
private static final int DEFAULT_MULTIPLYER = 37;
private static final int DEFAULT_HASHCODE = 17;

private int multiplier;
private int hashcode;
private long checksum;
private int count;
private List<Object> updateList;

public CacheKey() {
    this.hashcode = DEFAULT_HASHCODE;
    this.multiplier = DEFAULT_MULTIPLYER;
    this.count = 0;
    this.updateList = new ArrayList<Object>();
}

```

首先是它的成员变量和构造函数，有一个初始的hachcode和乘数，同时维护了一个内部的updatelist。在CacheKey的update方法中，会进行一个hashcode和checksum的计算，同时把传入的参数添加进updatelist中。如下代码所示。

```
public void update(Object object) {
    int baseHashCode = object == null ? 1 : ArrayUtil.hashCode(object); 
    count++;
    checksum += baseHashCode;
    baseHashCode *= count;
    hashcode = multiplier * hashcode + baseHashCode;

    updateList.add(object);
}

```

我们是如何判断CacheKey相等的呢，在CacheKey的equals方法中给了我们答案，代码如下所示。

```
@Override
public boolean equals(Object object) {
    .............
    for (int i = 0; i < updateList.size(); i++) {
      Object thisObject = updateList.get(i);
      Object thatObject = cacheKey.updateList.get(i);
      if (!ArrayUtil.equals(thisObject, thatObject)) {
        return false;
      }
    }
    return true;
}

```

除去hashcode，checksum和count的比较外，只要updatelist中的元素一一对应相等，那么就可以认为是CacheKey相等。只要两条Sql的下列五个值相同，即可以认为是相同的Sql。

> Statement Id + Offset + Limmit + Sql + Params

BaseExecutor的query方法继续往下走，代码如下所示。

```
list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
if (list != null) {
    // 这个主要是处理存储过程用的。
    handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
    } else {
    list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
}

```

如果查不到的话，就从数据库查，在queryFromDatabase中，会对localcache进行写入。

在query方法执行的最后，会判断一级缓存级别是否是STATEMENT级别，如果是的话，就清空缓存，这也就是STATEMENT级别的一级缓存无法共享localCache的原因。代码如下所示。

```
if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
        clearLocalCache();
}

```

在源码分析的最后，我们确认一下，如果是insert/delete/update方法，缓存就会刷新的原因。

SqlSession的insert方法和delete方法，都会统一走update的流程，代码如下所示。

```
@Override
public int insert(String statement, Object parameter) {
    return update(statement, parameter);
  }
   @Override
  public int delete(String statement) {
    return update(statement, null);
}

```

update方法也是委托给了Executor执行。BaseExecutor的执行方法如下所示。

```
@Override
public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    clearLocalCache();
    return doUpdate(ms, parameter);
}

```

每次执行update前都会清空localCache。

至此，一级缓存的工作流程讲解以及源码分析完毕。

#### 总结

1. Mybatis一级缓存的生命周期和SqlSession一致。
2. Mybatis的缓存是一个粗粒度的缓存，没有更新缓存和缓存过期的概念，同时只是使用了默认的hashmap，也没有做容量上的限定。
3. Mybatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，有操作数据库写的话，会引起脏数据，建议是把一级缓存的默认级别设定为Statement，即不使用一级缓存。

### 二级缓存

#### 二级缓存介绍

在上文中提到的一级缓存中，其最大的共享范围就是一个SqlSession内部，那么如何让多个SqlSession之间也可以共享缓存呢，答案是二级缓存。

当开启二级缓存后，会使用CachingExecutor装饰Executor，在进入后续执行前，先在CachingExecutor进行二级缓存的查询，具体的工作流程如下所示。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-08-18-27.png)

在二级缓存的使用中，一个namespace下的所有操作语句，都影响着同一个Cache，即二级缓存是被多个SqlSession共享着的，是一个全局的变量。

当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

#### 二级缓存配置

要正确的使用二级缓存，需完成如下配置的。

在Mybatis的配置文件中开启二级缓存。

```
<setting name="cacheEnabled" value="true"/>

```

在Mybatis的映射XML中配置cache或者 cache-ref 。

```
<cache/>   

```

cache标签用于声明这个namespace使用二级缓存，并且可以自定义配置。

- type: cache使用的类型，默认是PerpetualCache，这在一级缓存中提到过。
- eviction: 定义回收的策略，常见的有FIFO，LRU。
- flushInterval: 配置一定时间自动刷新缓存，单位是毫秒。
- size: 最多缓存对象的个数。
- readOnly: 是否只读，若配置可读写，则需要对应的实体类能够序列化。
- blocking: 若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存。

```
<cache-ref namespace="mapper.StudentMapper"/>

```

cache-ref代表引用别的命名空间的Cache配置，两个命名空间的操作使用的是同一个Cache。

#### 二级缓存实验

在本章节，通过实验，了解Mybatis二级缓存在使用上的一些特点。

在以下实验中，id为1的学生名称是点点。

**实验1**

测试二级缓存效果，不提交事务，sqlSession1查询完数据后，sqlSession2相同的查询是否会从缓存中获取数据。

```
@Test
public void testCacheWithoutCommitOrClose() throws Exception {
        SqlSession sqlSession1 = factory.openSession(true); 
        SqlSession sqlSession2 = factory.openSession(true); 

        StudentMapper studentMapper = sqlSession1.getMapper(StudentMapper.class);
        StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);

        System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
        System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentById(1));
}

```

执行结果:

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-16-15-24.jpg)







我们可以看到，当sqlsession没有调用commit()方法时，二级缓存并没有起到作用。

**实验2**

测试二级缓存效果，当提交事务时，sqlSession1查询完数据后，sqlSession2相同的查询是否会从缓存中获取数据。

```
@Test
public void testCacheWithCommitOrClose() throws Exception {
        SqlSession sqlSession1 = factory.openSession(true); 
        SqlSession sqlSession2 = factory.openSession(true); 

        StudentMapper studentMapper = sqlSession1.getMapper(StudentMapper.class);
        StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);

        System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
        sqlSession1.commit();
        System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentById(1));
}

```

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-16-20-38.jpg)

从图上可知，sqlsession2的查询，使用了缓存，缓存的命中率是0.5。

**实验3**

测试update操作是否会刷新该namespace下的二级缓存。

```
@Test
public void testCacheWithUpdate() throws Exception {
        SqlSession sqlSession1 = factory.openSession(true); 
        SqlSession sqlSession2 = factory.openSession(true); 
        SqlSession sqlSession3 = factory.openSession(true); 

        StudentMapper studentMapper = sqlSession1.getMapper(StudentMapper.class);
        StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);
        StudentMapper studentMapper3 = sqlSession3.getMapper(StudentMapper.class);

        System.out.println("studentMapper读取数据: " + studentMapper.getStudentById(1));
        sqlSession1.commit();
        System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentById(1));

        studentMapper3.updateStudentName("方方",1);
        sqlSession3.commit();
        System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentById(1));
}

```

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-16-38-30.jpg)

我们可以看到，在sqlSession3更新数据库，并提交事务后，sqlsession2的StudentMapper namespace下的查询走了数据库，没有走Cache。

**实验4**

验证Mybatis的二级缓存不适应用于映射文件中存在多表查询的情况。一般来说，我们会为每一个单表创建一个单独的映射文件，如果存在涉及多个表的查询的话，由于Mybatis的二级缓存是基于namespace的，多表查询语句所在的namspace无法感应到其他namespace中的语句对多表查询中涉及的表进行了修改，引发脏数据问题。

```
@Test
public void testCacheWithDiffererntNamespace() throws Exception {
        SqlSession sqlSession1 = factory.openSession(true); 
        SqlSession sqlSession2 = factory.openSession(true); 
        SqlSession sqlSession3 = factory.openSession(true); 

        StudentMapper studentMapper = sqlSession1.getMapper(StudentMapper.class);
        StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);
        ClassMapper classMapper = sqlSession3.getMapper(ClassMapper.class);

        System.out.println("studentMapper读取数据: " + studentMapper.getStudentByIdWithClassInfo(1));
        sqlSession1.close();
        System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentByIdWithClassInfo(1));

        classMapper.updateClassName("特色一班",1);
        sqlSession3.commit();
        System.out.println("studentMapper2读取数据: " + studentMapper2.getStudentByIdWithClassInfo(1));
}

```

执行结果:

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-16-52-33.jpg)

在这个实验中，我们引入了两张新的表，一张class，一张classroom。class中保存了班级的id和班级名，classroom中保存了班级id和学生id。我们在StudentMapper中增加了一个查询方法getStudentByIdWithClassInfo，用于查询学生所在的班级，涉及到多表查询。在ClassMapper中添加了updateClassName，根据班级id更新班级名的操作。

当sqlsession1的studentmapper查询数据后，二级缓存生效。保存在StudentMapper的namespace下的cache中。当sqlSession3的classMapper的updateClassName方法对class表进行更新时，updateClassName不属于StudentMapper的namespace，所以StudentMapper下的cache没有感应到变化，没有刷新缓存。当StudentMapper中同样的查询再次发起时，从缓存中读取了脏数据。

**实验5**

为了解决实验4的问题呢，可以使用Cache ref，让ClassMapper引用StudenMapper命名空间，这样两个映射文件对应的Sql操作都使用的是同一块缓存了。

执行结果:

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-16-50-50.jpg)

不过这样做的后果是，缓存的粒度变粗了，多个Mapper namespace下的所有操作都会对缓存使用造成影响，其实这个缓存存在的意义已经不大了。

#### 二级缓存源码分析

Mybatis二级缓存的工作流程和前文提到的一级缓存类似，只是在一级缓存处理前，用CachingExecutor装饰了BaseExecutor的子类，实现了缓存的查询和写入功能，所以二级缓存直接从源码开始分析。

**源码分析**

源码分析从CachingExecutor的query方法展开，源代码走读过程中涉及到的知识点较多，不能一一详细讲解，可以在文后留言，我会在交流环节更详细的表示出来。

CachingExecutor的query方法，首先会从MappedStatement中获得在配置初始化时赋予的cache。

```
Cache cache = ms.getCache();

```

本质上是装饰器模式的使用，具体的执行链是

SynchronizedCache -> LoggingCache -> SerializedCache -> LruCache -> PerpetualCache。

![img](http://or7kd5uf1.bkt.clouddn.com/gitchat/_image/2017-06-29-17-22-06.jpg)

以下是具体这些Cache实现类的介绍，他们的组合为Cache赋予了不同的能力。

- SynchronizedCache: 同步Cache，实现比较简单，直接使用synchronized修饰方法。
- LoggingCache: 日志功能，装饰类，用于记录缓存的命中率，如果开启了DEBUG模式，则会输出命中率日志。
- SerializedCache: 序列化功能，将值序列化后存到缓存中。该功能用于缓存返回一份实例的Copy，用于保存线程安全。
- LruCache: 采用了Lru算法的Cache实现，移除最近最少使用的key/value。
- PerpetualCache: 作为为最基础的缓存类，底层实现比较简单，直接使用了HashMap。

然后是判断是否需要刷新缓存，代码如下所示。

```
flushCacheIfRequired(ms);

```

在默认的设置中SELECT语句不会刷新缓存，insert/update/delte会刷新缓存。进入该方法。代码如下所示。

```
private void flushCacheIfRequired(MappedStatement ms) {
    Cache cache = ms.getCache();
    if (cache != null && ms.isFlushCacheRequired()) {      
      tcm.clear(cache);
    }
}

```

Mybatis的CachingExecutor持有了TransactionalCacheManager，即上述代码中的tcm。

TransactionalCacheManager中持有了一个Map，代码如下所示。

```
private Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();

```

这个Map保存了Cache和用TransactionalCache包装后的Cache的映射关系。

TransactionalCache实现了Cache接口，CachingExecutor会默认使用他包装初始生成的Cache，作用是如果事务提交，对缓存的操作才会生效，如果事务回滚或者不提交事务，则不对缓存产生影响。

在TransactionalCache的clear，有以下两句。清空了需要在提交时加入缓存的列表，同时设定提交时清空缓存，代码如下所示。

```
@Override
public void clear() {
    clearOnCommit = true;
    entriesToAddOnCommit.clear();
}

```

CachingExecutor继续往下走，ensureNoOutParams主要是用来处理存储过程的，暂时不用考虑。

```
if (ms.isUseCache() && resultHandler == null) {
    ensureNoOutParams(ms, parameterObject, boundSql);

```

之后会尝试从tcm中获取缓存的列表。

```
List<E> list = (List<E>) tcm.getObject(cache, key);

```

在getObject方法中，会把获取值的职责一路向后传，最终到PerpetualCache。如果没有查到，会把key加入Miss集合，这个主要是为了统计命中率。

```
Object object = delegate.getObject(key);
if (object == null) {
    entriesMissedInCache.add(key);
}

```

CachingExecutor继续往下走，如果查询到数据，则调用tcm.putObject方法，往缓存中放入值。

```
if (list == null) {
    list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    tcm.putObject(cache, key, list); // issue #578 and #116
}

```

tcm的put方法也不是直接操作缓存，只是在把这次的数据和key放入待提交的Map中。

```
@Override
public void putObject(Object key, Object object) {
    entriesToAddOnCommit.put(key, object);
}

```

从以上的代码分析中，我们可以明白，如果不调用commit方法的话，由于TranscationalCache的作用，并不会对二级缓存造成直接的影响。因此我们看看Sqlsession的commit方法中做了什么。代码如下所示。

```
@Override
public void commit(boolean force) {
    try {
      executor.commit(isCommitOrRollbackRequired(force));

```

因为我们使用了CachingExecutor，首先会进入CachingExecutor实现的commit方法。

```
@Override
public void commit(boolean required) throws SQLException {
    delegate.commit(required);
    tcm.commit();
}

```

会把具体commit的职责委托给包装的Executor。主要是看下tcm.commit()，tcm最终又会调用到TrancationalCache。

```
public void commit() {
    if (clearOnCommit) {
      delegate.clear();
    }
    flushPendingEntries();
    reset();
}

```

看到这里的clearOnCommit就想起刚才TrancationalCache的clear方法设置的标志位，真正的清理Cache是放到这里来进行的。具体清理的职责委托给了包装的Cache类。之后进入flushPendingEntries方法。代码如下所示。

```
private void flushPendingEntries() {
    for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
      delegate.putObject(entry.getKey(), entry.getValue());
    }
    ................
}

```

在flushPendingEntries中，就把待提交的Map循环后，委托给包装的Cache类，进行putObject的操作。

后续的查询操作会重复执行这套流程。如果是insert|update|delete的话，会统一进入CachingExecutor的update方法，其中调用了这个函数，代码如下所示，因此不再赘述。

```
private void flushCacheIfRequired(MappedStatement ms) 

```

#### 总结

1. Mybatis的二级缓存相对于一级缓存来说，实现了SqlSession之间缓存数据的共享，同时粒度更加的细，能够到Mapper级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。
2. Mybatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用的条件比较苛刻。
3. 在分布式环境下，由于默认的Mybatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将Mybatis的Cache接口实现，有一定的开发成本，不如直接用Redis，Memcache实现业务上的缓存就好了。

### 全文总结

本文介绍了Mybatis的基础概念，Mybatis一二级缓存的使用及源码分析，并对于一二级缓存进行了一定程度上的总结。

最终的结论是Mybatis的缓存机制设计的不是很完善，在使用上容易引起脏数据问题，个人建议不要使用Mybatis缓存，在业务层面上使用其他机制实现需要的缓存功能，让Mybatis老老实实做它的ORM框架就好了哈哈。