查看本场Chat

#### Redis 基础

Redis是基于内存的，存储形式为key-value的非关系型数据库，它的value不仅包括基本的string类型，还有list、set、sorted set和hash类型。

Redis其实也是一种缓存机制，缓存一般是针对查询方法的，但是也有写操作。当Redis启动时候，可以去Mysql中读数据，然后根据键把数据存放到Redis中。

当应用程序查找数据的时候，会先在Redis中查找，若找到了，则返回，若找不到，则去Mysql中找，找到了则把数据返回，并把该数据放到Redis中。

当应用程序写数据的时候，会先在Redis中写数据，然后Redis主键自增，然后队列处理器会定时去将二者进行同步，若同步失败，则可以直接插入到数据库中，然后清除缓存。

#### 集群与分布式的区别

分布式是实现不同业务，而集群是实现同一功能。

分布式的每个节点都可以是集群。

通过Redis可以实现分布式业务的调度，也可以实现集群服务器的数据缓存。

#### 轻松搭建Redis服务器

（1）下载Redis 的linux版本

（2）解压Redis：tar zxvf redis.tar.gz

（3）进入Redis的解压目录：cd redis.tar.gz

（4）Make && make install：Make

（5）启动服务端：./redis-server redis.conf

（6）启动客户端：./redis-cli

（7）测试

![enter image description here](http://images.gitbook.cn/0c03d230-f901-11e7-a523-99ceeced3b99)

（8）容易出现的问题

A、系统是mini，所以导致很多命令，例如make、vim等常用命令，一般会报-bash:make:command not found。

解决办法是：yum -y install gcc automake autoconf libtool make。

B、如果cc 问题，一般是没有gcc环境：

解决办法是：yum -y install gcc

#### Redis分布式Demo

Redis分布式问题，主要是将大任务化为小任务，多个计算机分布式来完成自己的任务，从而实现高效率的工作方式。

该Demo是以爬虫（爬取淘宝网的商品信息为例）为例，而Redis在此过程中起的是调度作用。

该Demo主要分为5个模块（获取商品列表、获取商品链接、获取商品图片、获取商品标题和分类、获取商品详情等），每个模块部署在不同的服务器上，同时完成不同的爬取任务，以达到分布式运行，从而提高效率。

其架构图简易如下：

![enter image description here](http://images.gitbook.cn/8ff0dca0-f901-11e7-a523-99ceeced3b99)

**解决方案（伪代码）：**

说明：面向微服务开发，使用disconf统一配置管理，用Maven做项目管理工具。

**（1）生产任务（produce-assign-server）：**

![enter image description here](http://images.gitbook.cn/0145ec60-f902-11e7-a523-99ceeced3b99)

![enter image description here](http://images.gitbook.cn/63ca42a0-f902-11e7-a523-99ceeced3b99)

图3

![enter image description here](http://images.gitbook.cn/6d0abac0-f902-11e7-a523-99ceeced3b99)

图4

**（2）云存储（spider-img-server）:**

![enter image description here](http://images.gitbook.cn/839e4d10-f902-11e7-a523-99ceeced3b99)

**（3）商品价格（spider-price-server）:**

![enter image description here](http://images.gitbook.cn/940cff20-f902-11e7-a523-99ceeced3b99)

**（4）商品详情（spider-detail-server）:**

![enter image description here](http://images.gitbook.cn/a2b8a470-f902-11e7-a523-99ceeced3b99)

**（5）商品标题和类型（spider-content-server）:**

![enter image description here](http://images.gitbook.cn/b13634e0-f902-11e7-a523-99ceeced3b99)

**（6）注意事项：**

- 去重（精准去重），若链接或标题匹配度达到90%以上的可以视为相同；
- 抓取图片数量受限；
- 图片栏若有视频，则可同类处理；
- 若任务已被领取，或已被执行，则不能再执行该任务了。

#### Redis缓存封装类

**（1）将Redis交给Spring容器来管理。**

![enter image description here](http://images.gitbook.cn/e20a5bf0-f902-11e7-a523-99ceeced3b99)

**（2）spring-redis.xml的详细配置**

![enter image description here](http://images.gitbook.cn/f4390d80-f902-11e7-a523-99ceeced3b99)

**（3）封装类**

![enter image description here](http://images.gitbook.cn/08a07100-f903-11e7-a523-99ceeced3b99)

![enter image description here](http://images.gitbook.cn/153e44f0-f903-11e7-a523-99ceeced3b99)

#### Redis与MySQL的交互

Redis中的数据和MySQL中的数据进行同步或异步更新，其原理就是定时或不定时的将数据在二者之间进行转化，当然这其中涉及到Redis的持久化操作，在网上有很多这样的原理论述，在此就不再赘述。

以下实例是以手动写定时器来定时将二者数据进行同步。

定时器，在这里用的是Spring整合Quartz，定时执行某个任务。

![enter image description here](http://images.gitbook.cn/386068f0-f903-11e7-a523-99ceeced3b99)

**（1）定时将MySQL中的变动数据存入Redis（大并发下的查询操作）**

A、定义表

B、查询所有字段is_new为1的数据

![enter image description here](http://images.gitbook.cn/676415c0-f903-11e7-a523-99ceeced3b99)

C、替换Redis中的相关数据



![enter image description here](http://images.gitbook.cn/80ffb7f0-f903-11e7-a523-99ceeced3b99)

**（2）定时读取Redis中的变动数据，并将其存入到MySQL中**

A、用户改变自己的数据（例如修改密码）

B、将变动数据存入到Redis中

![enter image description here](http://images.gitbook.cn/ab1b4a40-f903-11e7-a523-99ceeced3b99)

C、将Redis中的变动数据解析成相应对象

![enter image description here](http://images.gitbook.cn/beb46140-f903-11e7-a523-99ceeced3b99)

D、定时将数据从Redis中同步到MySQL

![enter image description here](http://images.gitbook.cn/d34daa30-f903-11e7-a523-99ceeced3b99)

