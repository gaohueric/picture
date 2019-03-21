### 1. 前言

消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题。实现高性能，高可用，可伸缩和最终一致性架构，是大型分布式系统不可缺少的中间件。

消息队列在电商系统、消息通讯、日志收集等应用中扮演着关键作用，以阿里为例，其研发的消息队列 (MQ) 服务于阿里集团超过 11 年，在历次天猫双十一活动中支撑了万亿级的数据洪峰，为大规模交易提供了有力保障。

目前在生产环境，使用较多的消息队列有 ActiveMQ、RabbitMQ、ZeroMQ、Kafka、MetaMQ、RocketMQ 等。本场 Chat 将介绍基于 Kafka 和 ZooKeeper 的分布式消息队列 (Distributed Message Queue，下文简称 DMQ)。

如果你对消息队列的基本概念还不清楚的话，在正式开始之前，建议你看一组实例来直观的感受一下消息队列：[点击查看](https://blog.csdn.net/cws1214/article/details/52922267%E3%80%82)

### 2. 基于 Kafka 的 DMQ 总体架构

#### 2.1 什么是 Kafka？

> Kafka 是最初由 Linkedin 公司开发，是一个分布式、支持分区的（partition）、多副本的（replica），基于 zookeeper 协调的分布式消息系统，它的最大的特性就是可以实时的处理大量数据以满足各种需求场景：比如基于 hadoop 的批处理系统、低延迟的实时系统、storm/Spark 流式处理引擎，web/nginx 日志、访问日志，消息服务等等，用 scala 语言编写，Linkedin 于 2010 年贡献给了 Apache 基金会并成为顶级开源项目。 目前，Kafka 官网已经将自己修正为一个分布式的流式处理平台。本场 Chat 介绍的 DMQ 中，kafka 作为高吞吐量的分布式发布订阅消息系统。

Kafka 的特性:

- 高吞吐量、低延迟：kafka 每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个 topic 可以分多个 partition, consumer group 对 partition 进行 consume 操作；
- 可扩展性：kafka 集群支持热扩展；
- 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失；
- 容错性：允许集群中节点失败（若副本数量为 n, 则允许 n-1 个节点失败）；
- 高并发：支持数千个客户端同时读写；

kafka 以集群方式运行，producers 通过网络将消息发送到 kafka 集群，集群向消费者提供消息（需要说明的是，消息是由消费者主动消费的）。如下图：

![enter image description here](http://images.gitbook.cn/f5be3dc0-4a1e-11e8-b415-150fb777d214)

上文已经提到了分区 (partition)，创建一个 topic 时，同时可以指定分区的数目，分区数越多，其吞吐量也越大，但是需要的资源也越多，同时也会导致更高的不可用性，kafka 在接收到生产者发送的消息之后，会根据均衡策略将消息存储到不同的分区中，如下图：

![enter image description here](http://images.gitbook.cn/0ac0f6e0-4a1f-11e8-b790-016dbce67732)

#### 2.2 DMQ 总体架构是怎样的？

基于 kafka 的 DMQ 总体架构如下

![enter image description here](http://images.gitbook.cn/535a0030-4962-11e8-9e33-bf33f6000d05)

DMQ 是基于 Kafka 架构建立的，在 Kafka 架构中，有几个术语：

1. Producer：生产者，即消息发送者，push 消息到 Kafka 集群中的 broker（就是 server）中。
2. Broker：Kafka 集群由多个 Kafka 实例 (Server) 组成，每个实例构成一个 broker，说白了就是服务器。
3. Topic：producer 向 kafka 集群 push 的消息会被归于某一类别，即 Topic，这本质上只是一个逻辑概念，面向的对象是 producer 和 consumer，producer 只需要关系将消息 push 到哪一个 Topic 中，而 consumer 只需要关心自己订阅了哪个 Topic。
4. Partition：每一个 Topic 又被分为多个 Partitions，即物理分区；出于负载均衡的考虑，同一个 Topic 的 Partitions 分别存储于 Kafka 集群的多个 broker 上；而为了提高可靠性，这些 Partitions 可以由 Kafka 机制中的 replicas 来设置备份的数量；如上面的框架图所示，存在两个备份。
5. Consumer：消费者，从 Kafka 集群的 broker 中 pull 消息、消费消息。
6. Consumer group：high-level consumer API 中，每个 consumer 都属于一个 consumer group，每条消息只能被 consumer group 中的一个 Consumer 消费，但可以被多个 consumer group 消费。
7. replicas: partition 的副本，保障 partition 的高可用。
8. leader：replicas 中的一个角色， producer 和 consumer 只跟 leader 交互。
9. follower：replicas 中的一个角色，从 leader 中复制数据。
10. controller：kafka 集群中的其中一个服务器，用来进行 leader election 以及 各种 failover。
11. Zookeeper：kafka 通过 zookeeper 来存储集群的 meta 信息。

> 注：kafka 集群依赖于 Zookeeper 架构，借助其优越的一致性、可靠性、实时性、原子性以及顺序性来保障集群系统的可用性。早期版本的 kafka 用 Zookeeper 做 meta 信息存储：包括 consumerGroup/consumer、broker、Topic 等。鉴于 Zookeeper 本身的一些固有缺陷，新版本 Kafka 中弱化了对 zookeeper 的依赖，consumer 使用了 kafka 内部的 group coordination 协议。关于 Zookeeper，下文将详细介绍。

### 3. 关于上述概念的理解

#### 3.1 Kafka 为什么要将 Topic 进行分区？

> 简而言之：负载均衡 + 水平扩展。

前已述及，Topic 只是逻辑概念，面向的是 producer 和 consumer；而 Partition 则是物理概念。可以想象，如果 Topic 不进行分区，而将 Topic 内的消息存储于一个 broker，那么，这个 broker 将会压力过大而使得吞吐量陷入瓶颈，这显然是不符合高吞吐量应用场景的。有了 Partition 概念以后，假设一个 Topic 被分为 10 个 Partitions，Kafka 会根据一定的算法将 10 个 Partition 尽可能均匀的分布到不同的 broker（服务器）上，当 producer 发布消息时，producer 客户端可以采用 “random”、“key-hash” 及 “轮询” 等算法选定目标 partition，若不指定，Kafka 也将根据一定算法将其置于某一分区上。Partiton 机制可以极大的提高吞吐量，并且使得系统具备良好的水平扩展能力。 补充一点：Kafka 机制中，producer push 来的消息是追加（append）到 partition 中的，这是一种顺序写磁盘的机制，效率远高于随机写内存，况且，还有多个分区的存在，这种机制保障了 Kafka 的高吞吐率。

#### 3.2 什么是 Zookeeper？

> ZooKeeper，字面意为 “动物园管理员”。动物园里有各种动物，为了让不同种类的动物 “和谐” 相处，为游客提供良好的观赏服务，就需要动物园管理员按照动物的习性加以分类和管理。 在企业级应用中，通常各子系统不是孤立存在的，它们彼此之间需要协作和交互，即所谓的分布式系统。各个子系统就好比动物园里的动物，为了使各个子系统能正常为用户提供统一的服务，必需一种机制来进行协调——ZooKeeper。

如下图所示为 Zookeeper 架构：

![enter image description here](http://images.gitbook.cn/a2d6e630-4964-11e8-9e33-bf33f6000d05)

Zookeeper 集群主要角色有 Leader，Learner（Follower，Observer(当服务器增加到一定程度，由于投票的压力增大从而使得吞吐量降低，所以增加了 Observer。) 以及 client：

![enter image description here](http://images.gitbook.cn/9399d980-4968-11e8-9e33-bf33f6000d05)

> **官方说明：** ZooKeeper 是一个分布式的，开放源码的分布式应用程序协调服务，是 Google 的 Chubby 一个开源的实现。分布式应用程序可以基于它实现统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等工作。

Zookeeper 的核心是原子广播，这个机制保证了各个 Server 之间的同步。实现这个机制的协议叫做 Zab 协议。Zab 协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）。当服务启动或者在领导者崩溃后，Zab 就进入了恢复模式，当领导者被选举出来，且大多数 Server 完成了和 leader 的状态同步以后，恢复模式就结束了。状态同步保证了 leader 和 Server 具有相同的系统状态。

#### 3.3 Zookeeper 能做什么？

> 简而言之：文件系统 + 通知机制

**1. 文件系统** Zookeeper 维护一个类似文件系统的数据结构：

![enter image description here](http://images.gitbook.cn/4fe79050-4a13-11e8-b790-016dbce67732)

每个子目录项如 NameService 都被称作为 znode，和文件系统一样，我们能够自由的增加、删除 znode，在一个 znode 下增加、删除子 znode，唯一的不同在于 znode 是可以存储数据的。

| znode 类型              | 描述                                       |
| --------------------- | ---------------------------------------- |
| PERSISTENT            | 持久化目录节点 ，客户端与 zookeeper 断开连接后，该节点依旧存在    |
| PERSISTENT_SEQUENTIAL | 持久化顺序编号目录节点，客户端与 zookeeper 断开连接后，该节点依旧存在，只是 Zookeeper 给该节点名称进行顺序编号 |
| EPHEMERAL             | 临时目录节点，客户端与 zookeeper 断开连接后，该节点被删除       |
| EPHEMERAL_SEQUENTIAL  | 临时顺序编号目录节点，客户端与 zookeeper 断开连接后，该节点被删除，只是 Zookeeper 给该节点名称进行顺序编号 |

有四种类型的 znode：

| znode 类型              | 描述                                       |
| --------------------- | ---------------------------------------- |
| PERSISTENT            | 持久化目录节点 ，客户端与 zookeeper 断开连接后，该节点依旧存在    |
| PERSISTENT_SEQUENTIAL | 持久化顺序编号目录节点，客户端与 zookeeper 断开连接后，该节点依旧存在，只是 Zookeeper 给该节点名称进行顺序编号 |
| EPHEMERAL             | 临时目录节点，客户端与 zookeeper 断开连接后，该节点被删除       |
| EPHEMERAL_SEQUENTIAL  | 临时顺序编号目录节点，客户端与 zookeeper 断开连接后，该节点被删除，只是 Zookeeper 给该节点名称进行顺序编号 |

**2. 通知机制** 客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、被删除、子目录节点增加删除）时，zookeeper 会通知客户端。

**3. 具体用途**

- 命名服务

在分布式系统中，经常需要给一个资源生成一个唯一的 ID，在没有中心管理结点的情况下生成这个 ID 并不是一件很容易的事儿。zookeeper 就提供了这样一个命名服务。

- 配置管理

主要用于多个结点共享配置，并且在配置发生更新时，利用 zookeeper 可以让这些使用了这些配置的结点获得通知，进行重新加载等操作。

- 集群管理

所谓集群管理主要有两点：节点退出和加入、选举 master。

- 分布式锁

有了 zookeeper 的一致性文件系统，锁的问题变得容易。锁服务可以分为两类，一个是保持独占，另一个是控制时序。

- 其它 略

#### 3.4 Kafka 架构中 Zookeeper 以怎样的形式存在？

**1. broker 在 zookeeper 中的注册**

- 为了记录 broker 的注册信息，在 zookeeper 上，专门创建了属于 kafka 的一个节点，其路径为 / brokers。
- Kafka 的每个 broker 启动时，都会到 zookeeper 中进行注册，告诉 zookeeper 其 broker.id， 在整个集群中，broker.id 应该全局唯一，并在 zookeeper 上创建其属于自己的节点，其节点路径为 / brokers/ids/{broker.id}.
- 创建完节点后，kafka 会将该 broker 的 broker.name 及端口号记录到该节点。
- 另外，该 broker 节点属性为临时节点，当 broker 会话失效时，zookeeper 会删除该节点，这样，我们就可以很方便的监控到 broker 节点的变化，及时调整负载均衡等。

**2. Topic 在 zookeeper 中的注册** 在 kafka 中，所有 topic 与 broker 的对应关系都由 zookeeper 进行维护，在 zookeeper 中，建立专门的节点来记录这些信息，其节点路径为 / brokers/topics/{topic_name}。 前面说过，为了保障数据的可靠性，每个 Topic 的 Partitions 实际上是存在备份的，并且备份的数量由 Kafka 机制中的 replicas 来控制。那么问题来了：如下图所示，假设某个 TopicA 被分为 2 个 Partitions，并且存在两个备份，由于这 2 个 Partitions（1-2）被分布在不同的 broker 上，同一个 partiton 与其备份不能（也不应该）存储于同一个 broker 上。以 Partition1 为例，假设它被存储于 broker2，其对应的备份分别存储于 broker1 和 broker4，有了备份，可靠性得到保障，但数据一致性却是个问题。

![enter image description here](http://images.gitbook.cn/bba49490-4965-11e8-b415-150fb777d214)

为了保障数据的一致性，Zookeeper 机制得以引入。基于 Zookeeper，Kafka 为每一个 partition 找一个节点作为 Leader，其余备份作为 Follower；接续上图的例子，就 TopicA 的 partition1 而言，如果位于 broker2（Kafka 节点）上的 partition1 为 Leader，那么位于 broker1 和 broker4 上面的 partition1 就充当 Follower，则有下图：

![enter image description here](http://images.gitbook.cn/c7d5eb10-4965-11e8-867d-db68bbcced44)

基于上图的架构，当 producer push 的消息写入 partition(分区) 时，作为 Leader 的 broker(Kafka 节点) 会将消息写入自己的分区，同时还会将此消息复制到各个 Follower，实现同步。如果，某个 Follower 挂掉，Leader 会再找一个替代并同步消息；如果 Leader 挂了，Follower 们会选举出一个新的 Leader 替代，继续业务，这些都是由 zookeeper 完成的。

**3. consumer 在 zookeeper 中的注册**

- 注册新的消费者分组

当新的消费者组注册到 zookeeper 中时，zookeeper 会创建专用的节点来保存相关信息，其节点路径为 ls /consumers/{group_id}，其节点下有三个子节点，分别为 [ids, owners, offsets]。

1. ids 节点：记录该消费组中当前正在消费的消费者；
2. owners 节点：记录该消费组消费的 topic 信息；
3. offsets 节点：记录每个 topic 的每个分区的 offset，

- 注册新的消费者

当新的消费者注册到 kafka 中时，会在 / consumers/{group_id}/ids 节点下创建临时子节点，并记录相关信息

- 监听消费者分组中消费者的变化

每个消费者都要关注其所属消费者组中消费者数目的变化，即监听 / consumers/{group_id}/ids 下子节点的变化。一单发现消费者新增或减少，就会触发消费者的负载均衡。

#### 3.5 谁来充当 Zookeeper 的 Client？

很明显 Zookeeper 机制在 Kafka 架构中是用来管理 Topic 的 Partition 的，而 Topic 直接关联的是 producer 和 consumer：producer 将消息 push 到对应 Topic 的 partition，consumer 从订阅的 Topic 中 pull 消息。

producer 充当了 Client：push 消息，实质上是 “写” 操作，Zookeeper 机制中，只有 Leader 才能执行 “写” 操作，Leader 将消息写入本节点 (broker) 的对应 partition 后，再将消息同步到各个 Follower；Consumer 的 pull 消息，实质上就是 “读” 操作，不过，这个 “读” 操作并不简单。

#### 3.6 Kafka 的消息 - 订阅模式是同步 OR 异步？

显然是异步的：producer 向 Kafka 集群 push 的消息并不是实时传送到 consumer 的，producer push 到 Kafka 集群 partition 中的消息是以 “追加”(Append) 的形式顺序写入磁盘；消息到达 consumer 端却并不是 “推送” 的，而是 consumer 主动 pull（fetch 请求）的，consumer 可以根据自己的消费能力去 fetch 消息并执行处理，而且可以根据 offset（偏移量）来控制消费的进度。

此外，在 Kafka 中，一个 Consumer-Group 中只有一个消费者能够消费某个 broker partition 中的消息，

### 4. 全程解析（Producer-kafka-consumer）

#### 4.1 producer 发布消息

producer 采用 push 模式将消息发布到 broker，每条消息都被 append 到 patition 中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障 kafka 吞吐率）。producer 发送消息到 broker 时，会根据分区算法选择将其存储到哪一个 partition。

**其路由机制为：**

1. 指定了 patition，则直接使用；
2. 未指定 patition 但指定 key，通过对 key 的 value 进行 hash 选出一个 patition
3. patition 和 key 都未指定，使用轮询选出一个 patition。

**写入流程：**

1. producer 先从 zookeeper 的 "/brokers/.../state" 节点找到该 partition 的 leader
2. producer 将消息发送给该 leader
3. leader 将消息写入本地 log
4. followers 从 leader pull 消息，写入本地 log 后 leader 发送 ACK
5. leader 收到所有 ISR 中的 replica 的 ACK 后，增加 HW（high watermark，最后 commit 的 offset） 并向 producer 发送 ACK

#### 4.2 Broker 存储消息

物理上把 topic 分成一个或多个 patition，每个 patition 物理上对应一个文件夹（该文件夹存储该 patition 的所有消息和索引文件）

#### 4.3 Consumer 消费消息

high-level consumer API 提供了 consumer group 的语义，一个消息只能被 group 内的一个 consumer 所消费，且 consumer 消费消息时不关注 offset，最后一个 offset 由 zookeeper 保存。

**注意：**

1. 如果消费线程大于 patition 数量，则有些线程将收不到消息；
2. 如果 patition 数量大于线程数，则有些线程多收到多个 patition 的消息；
3. 如果一个线程消费多个 patition，则无法保证你收到的消息的顺序，而一个 patition 内的消息是有序的。

consumer 采用 pull 模式从 broker 中读取数据。

push 模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息。

对于 Kafka 而言，pull 模式更合适，它可简化 broker 的设计，consumer 可自主控制消费消息的速率，同时 consumer 可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。

### 5. 补充

#### 5.1 consumer-Group

kafka 的分配单位是 patition。每个 consumer 都属于一个 group，一个 partition 只能被同一个 group 内的一个 consumer 所消费（也就保障了一个消息只能被 group 内的一个 consuemr 所消费），但是多个 group 可以同时消费这个 partition。

#### 5.2 consumer rebalance

当有 consumer 加入或退出、以及 partition 的改变（如 broker 加入或退出）时会触发 rebalance。具体算法略。



#### 5.3 controller failover

当 controller 宕机时会触发 controller failover。每个 broker 都会在 zookeeper 的 "/controller" 节点注册 watcher，当 controller 宕机时 zookeeper 中的临时节点消失，所有存活的 broker 收到 fire 的通知，每个 broker 都尝试创建新的 controller path，只有一个竞选成功并当选为 controller。

#### 5.4 broker failover

1. controller 在 zookeeper 的 /brokers/ids/[brokerId] 节点注册 Watcher，当 broker 宕机时 zookeeper 会 fire watch。
2. controller 从 /brokers/ids 节点读取可用 broker
3. controller 决定 set_p，该集合包含宕机 broker 上的所有 partition
4. 对 set_p 中的每一个 partition：

```
从 /brokers/topics/[topic]/partitions/[partition]/state 节点读取 ISR
决定新 leader（如 4.3 节所描述）
将新 leader、ISR、controller_epoch 和 leader_epoch 等信息写入 state 节点
通过 RPC 向相关 broker 发送 leaderAndISRRequest 命令

```

#### 5.5 leader failover

当 partition 对应的 leader 宕机时，需要从 follower 中选举出新 leader。在选举新 leader 时，一个基本的原则是，新的 leader 必须拥有旧 leader commit 过的所有消息。

kafka 在 zookeeper 中（/brokers/.../state）动态维护了一个 ISR（in-sync replicas），ISR 里面的所有 replica 都跟上 leader，只有 ISR 里面的成员才能选为 leader。

参考文献

1. <https://www.zhihu.com/question/35139415>
2. <https://blog.csdn.net/cws1214/article/details/52922267>