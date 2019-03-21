### 开篇

人工智能、大数据快速发展的今天，对于 TB 甚至 PB 级大数据的快速检索已然成为刚需。Elasticsearch 作为开源领域的后起之秀，从2010年至今得到飞跃式的发展。 Elasticsearch 以其开源、分布式、RESTFul API 三大优势，已经成为当下风口中“会飞的猪”。

阿里云2018年2月5日已开价50-200W年薪招聘技术人员参与 Elasticsearch、Lucene 内核优化、改进。如果说，你错过了 Hadoop，错过了 Spark 的红利，难道 Elasticsearch 的机会你还要错过吗？

在学习 Elasticsearch 的过程中，你是不是多多少少有以下困惑：

- 面对 Elasticsearch1.X，2.X，5.X，6.X 的不同版本，你无从下手？
- 面对 ElasticStack（Elasticsearch、Logstash、Kibana、Beat），你不知道从何学起？
- 面对各种技术文档（官方的及非官方的），你是否感到非常困惑？
- 面对 Elasticsearch 出现的各种异常、Bug，好不容易找到一个技术群或提问，又没人解答？
- 市面上的书籍分两种：老外的原理透彻，但版本过时；国内的版本稍新、但不透彻，你是不是很迷茫……

本文：

- 不谈搜索引擎的原理；
- 不谈倒排索引的原理；
- 不谈乐观锁、悲观锁的机制；
- ……

只谈：

- 从产品开发、项目实战的角度，如何让一个 Java 程序员甚至 C/C++ 的程序员快速上手。
- 海量的版本中，告诉你明确的选择。
- ELKStack 技术体系，你的学习优先级。
- ELK 技术体系架构。
- ELK 技术栈的应用场景。
- 有了传统关系型数据库（MySQL、Oracle）、非关系型数据库（Mongo)，如何快速的导入 Elasticsearch，实现全文检索。
- Elasticsearch 实战中遇到问题，如何高效解决。
- Elasticsearch 集群部署。
- Elasticsearch 集群调优清单。
- Elasticsearch 高效进阶方法论。
- ……

横扫你学习 Elasticsearch 的诸多疑惑，让你少走半年弯路！

### ELK Stack 认知

考虑到有初学的朋友，先对 ELK Stack 的认知咱们达成共识。

ELK Stack 由最早期的最核心的 Elasticsearch（以下部分简称ES），集合 Logstash、Kibana、beats 等发展而来，形成 ELK Stack 体系。如下图所示。

![enter image description here](http://images.gitbook.cn/637075c0-18b2-11e8-8087-9b6b3b447adf)

#### Elasticsearch 认知

Elasticsearch 为开源的、分布式、基于 Restful API、支持 PB 甚至更高数量级的搜索引擎工具。

相对于 MySQL，给出如下的对应关系表会更好理解。

![enter image description here](http://images.gitbook.cn/56834950-18b2-11e8-8087-9b6b3b447adf)

从上表中可以看出：

1. MySQL 中的数据库（DataBase），等价于 ES 中的索引（Index）。
2. MySQL 中一个数据库下面有 N 张表（Table），等价于1个索引 Index 下面有 N 多类型（Type）。
3. MySQL 中一个数据库表（Table）下的数据由多行（Row）多列（column，属性）组成，等价于1个 Type 由多个文档（Document）和多 Field 组成。
4. MySQL 中定义表结构、设定字段类型等价于 ES 中的 Mapping。举例说明，在一个关系型数据库里面，Schema 定义了表、每个表的字段，还有表和字段之间的关系。与之对应的，在 ES 中，Mapping 定义索引下的 Type 的字段处理规则，即索引如何建立、索引类型、是否保存原始索引 JSON 文档、是否压缩原始 JSON 文档、是否需要分词处理、如何进行分词处理等。
5. MySQL 中的增 insert、删 delete、改 update、查 search 操作等价于 ES 中的增 PUT/POST、删 Delete、改 _update、查 `GET`。其中的修改指定条件的更新 update 等价于 ES 中的 `update_by_query`，指定条件的删除等价于 ES 中的 `delete_by_query`。
6. MySQL 中的 group by、avg、sum 等函数类似于 ES 中的 Aggregations 的部分特性。
7. MySQL 中的去重 distinct 类似 ES 中的 cardinality 操作。
8. MySQL 中的数据迁移等价于 ES 中的 reindex 操作。

以上，通过类比，能加快理解 Elasticsearch 的速度。

如下是传统的关系型数据库（如Oracle、MySQL）、非关系型的数据库（如 Mongo）所做不到的：

1.传统的关系型数据库虽然能支持类型“like *待检索词*”模糊语句匹配，但无法进行全文检索（分词检索）。

这里的全文检索，举例如下。

"text"："公路局正在治理解放大道路面积水问题"，对于这段待检索的文字，经过细粒度分词后能得出如下的分词结果：

> 公路局、公路、路局、路、局正、正在、正、治理、治、理解、理、解放、解、放大、大道、大、道路、道、路面、路、面积、面、积水、积、水、问题

如果进行全文检索，是针对以上分词后的结果逐个进行匹配，并由得分的高低快速的返回匹配结果。

这点，传统数据库几乎不可能做到。

2.非关系型数据库 Mongo 虽能进行简单的全文检索，但对中文支持的不好、数据量大性能会有问题，这点是在实际应用中总结出的。

#### Logstash 认知

可以把 Logstash 理解成流入、流出 Elasticsearch 的传送带。

支持：不同类型的数据或实施数据流经过 Logstash 写入 ES 或者从 ES 中读出写入文件或对应的实施数据流。

包括但不限于：

- 本地或远程文件；
- Kafka 实时数据流——核心插件有 logstash*input*kafka/logstash*output*kafka；
- MySQL、Oracle 等关系型数据库——核心插件有 logstash*input*jdbc/logstash*ouput*jdbc；
- Mongo 非关系型数据库——核心插件有 logstash*input*mongo/logstash*output*mongo；
- Redis 数据流；
- ……

#### Kibana 认知

Kibana 是 ES 大数据的图形化展示工具。集成了 DSL 命令行查看、数据处理插件、继承了 x-pack（收费）安全管理插件等。

#### Beats 认知

Beats 是一个开源的用来构建轻量级数据汇集的平台，可用于将各种类型的数据发送至 Elasticsearch 与 Logstash。

Beats目前有官方支持的多个子产品，如下：

- Packetbeat：用于监控局域网内服务器之间的网络流量信息；
- Filebeat：收集服务器上的日志信息——它是用来替代 Logstash Forwarder 的下一代 Logstash 收集器，是为了更快速稳定轻量低耗地进行收集工作，它可以很方便地与 Logstash 还有直接与 Elasticsearch 进行对接。
- 新推出的 Metricbeat，可以定期获取外部系统的监控指标信息。 除了以上三个核心产品外，还有：Winlogbeat(Windows事件日志轻量级工具)、Auditbeat(审计数据的轻量级工具)、Heartbeat(用于时间监控的轻量级工具)。 除此以外，你还可以非常方便的基于 libbeat 框架来构建你属于自己的专属 Beat。

#### 小结

通过以上的介绍，我们对 ELK Stack 中的核心成员：Elasticsearch、Logstash、Kibana、Beats 是什么以及能干什么有了相对一致的认知。

### 海量的版本中，告诉你明确的选择

#### ELK 历史版本更迭

Elasticsearch 于2010年提交到 GitHub。

- 2010年2月8日推出了 V0.4.0 的发行版本，2010年2月12日推出 V1.0.0 版本，2016年2月2日推出 V1.7.5 版本， 此为 1.X 最终版本，不再更新。
- 2015年10月28日推出 V2.0.0 版本，2017年7月25日推出 V2.4.6 版本，此为 2.X 最终版本，不再更新。
- 2016年10月26日推出 V5.0.0 版本，2018年2月20日推出 V5.6.8 版本，此并不是 5.X 的最终版本，还在更新中……
- 2017年11月14日推出 V6.0.0 版本，2018年2月20日推出 V6.2.2 版本。
- ……

Elasticsearch 版本更新还在持续迭代进行中。

从以上更新我们也能得出，ES5.X 的末期版本和 ES6.X 的初期版本时间存在重叠。

在 Elasticsearch5.X 之前的版本中，Kibana 和 Logstash 各有自己的一套版本管理体系。如 Kibana4.X 对应 Elasticsearch2.3.X。

为统一规范化版本管理，Elasticsearch 跃过 3.X 大版本、4.X 大版本，直接和 Kibana、Logstash、Feat 升级为相同的 5.X、6.X 乃至以后的 7.X 版本。

ELK Stack的版本历史查看参见[这里](https://www.elastic.co/downloads/past-releases)。

#### ELK Stack 版本的选择

##### **新手直接选择最新版本**

如果你是初次接触 Elasticsearch，建议从最新版本学起，当前最新的版本为 V6.2.2。

新版本的优点有：

- 由于 ELK 都是开源的，历史版本发现的问题，GitHub 上的 issue 都已经得到解决；
- 新的大版本往往都做过比较大的改动。

比如 5.X 版本较之前的 2.X、1.X 等历史版本，做过很大的改动——5.X 的字符串类型区改为分词相关的 text 和不区分分词的 keyword，不再使用 string 类型。

比如 Elasticsearch6.X 较 5.X，不再支持1个 index 下有多个 type，而是变成严格意义的一对一的关系。

新版本的缺点有：

- 最新版本 Elasticsearch 插件的支持可能没有那么好；
- 新特性未被实际的生产环境做过最充分验证。

权衡以上优缺点，如果能接受新版本的缺点，那么使用最新的 Elasticsearch 版本是最好的选择。

再明确点说：

- 如果你是第一次接触 ELK Stack，建议你直接使用最新版本的。截止2018年2月22日，最新版本为 V6.2.2。
- 如果你之前的项目/产品或自学的过程中，接触过早期的版本 1.X、2.X，一方面为了提升性能，建议升级为最新的版本，另一方面，由于各种外部原因（如代码升级成本高、业务系统已经稳定等），建议也要抽时间了解 Elasticsearch 的新版本的新特性。因为，这些新特性都是前人遇到坑的相对最优解决方案的优化后的结果。

##### **5.X 哪个版本相对稳定？**

根据一位携程架构师 wood 于2017年11月29日表示的，生产环境 5.3.2 有大规模部署，稳定性还不错。测试环境也有部署 5.6.4，目前也没发现什么不稳定的问题。

##### **不建议再以2.X、1.X或更早的版本进行学习。**

主要基于以下三点原因：

1. 从版本历史可以看出，近7年多的 ELK Stack 得到长足的发展。
2. 早期版本的一些设计缺陷历史问题、一些开源社区 Bug，在新版本都已经纠正。
3. 新版本在性能方面也得到较大幅度的提升。

#### 小结

本小节主要探讨了 ELK Stack 版本更迭历史，扫除你版本选型的困惑。

### ELK Stack 的应用场景

#### ELK Stack基础应用场景

**场景一：使用 ES 作为业务系统的后端。**

此时，ES 的作用类似传统业务系统中的 MySQL、PostgreSQL、Oracle 或者 Mongo 等的基础关系型数据库或非关系型数据库的作用。

我们举例说明。使用 ES 对基础文档进行检索操作，如将传统的 word 文档、PDF 文档、PPT 文档等通过 Openoffice 或者 pdf2htmlEX 工具转换为 HTML，再将 HTML 以JSON 串的形式录入到 ES，以对外提供检索服务。

**场景二：在原有系统中增加 ES、Logstash、Kibana等。**

原有的业务系统中存在 MySQL、Oracle、Mongo 等基础数据，但想实现全文检索服务，就在原有业务系统基础的加上一层 ELK。

举例一，将原有系统中 MySQL 中的数据通过 logstash*input*jdbc 插件导入到 ES 中，并通过 Kibana 进行图形化展示。

举例二，将原有存储在 Hadoop HDFS 中的数据导入到 ES 中，对外提供检索服务。

**场景三：使用 ELK Stack 结合现有工具对外提供服务。**

举例一，日志检索系统。将各种类型的日志通过 Logstash 导入 ES 中，通过 Kibana 或者 Grafana 对外提供可视化展示。

举例二，通过 Flume 等将数据导入 ES 中，通过 ES 对外提供全文检索服务。

**场景四：其他综合业务场景**

主要借助 ES 强大的全文检索功能实现，如分页查询、各类数据结果的聚合分析、图形化展示（饼图、线框图、曲线图等）。

举例说明，像那些结合实际业务的场景，如安防领域、金融领域、监控领域等的综合应用。

#### 小结

本小节主要探讨了 ELK Stack 三种基础应用场景和一种扩展综合应用场景，让你对 ES 的应用有个全局的认知。

### ELK Stack 你的学习优先级

#### ELK Stack 学习优先级

**我建议 Elasticsearch 为第一优先级。**需要掌握的内容如下。

（1）掌握 Elasticsearch 的基本概念，主要包括：

- 索引（index）
- 类型（type）
- 映射（mapping）
- 文档（document）
- 倒排索引原理
- 文档打分机制
- 集群（cluster）——单节点、集群安装与部署
- 健康状态（red/yellow/green）
- 数据存储
- 数据类型（long/date/text、keyword/nested等）
- 数据展示（结合Head插件的基础可视化）
- ……

（2）掌握 Elasitcsearch 的基本操作，主要包括：

- 新增（insert）
- 删除（delete/delete*by*query）
- 修改（update/update*by*query）
- 查找（search）
- 精确匹配检索（term、terms、range、exists）
- 模糊匹配检索（wildcard、prefix、negix正则）
- 分词全文检索（match/match_phrase等）
- 多条件 bool 检索（must/must_not/should多重组合）
- 分词（英文分词、拼音分词、中文分词）
- 高亮
- 分页查询
- 指定关键词返回
- 批量操作 bulk
- scroll 查询
- reindex 操作
- ……

（3）掌握 Elasticsearch 高级操作，主要包括：

- 聚合统计（数量聚合、最大值、最小值、平均值、求和等聚合操作）
- 图像化展示（hisgram 按照日期等聚合）
- 聚合后分页
- 父子文档
- 数组类型
- nested 嵌套类型
- ES 插件错误排查（集群问题、检索问题、性能问题）
- ES 性能调优（配置调优、集群调优等）
- ……

（4）掌握 Elasticsearch Java/Python 等API，主要包括：

- Elasticsearch 原生自带 API、JEST、Springboot 等 API 选型
- Elasticsearch 多条件 bool 复杂检索 API
- Elasticsearch 分页 API
- Elasticsearch 高亮 API
- Elasticsearch 聚合 API
- Elasticsearch 相关 JSON 数据解析
- ……

（5）Elasticsearch 结合场景开发实战，主要包括：

- 数据可视化（Kibana、Grafana 等 其中 Grafana 比较适合监控类场景）
- 通过 logstash/beats 等导入数据
- Elasticsearch 和 Kafka 结合的应用场景
- Elasticsearch 和 Mongo 结合的应用场景
- Elasticsearch 和 Hadoop 结合的应用场景
- 结合业务需求的定制化应用场景（日志分析、文档检索、全文检索、金融等各行业检索）
- ……

**建议的第二学习优先级为 Kibana。**需要掌握的内容如下。

- Kibana 安装与部署
- ES 节点数据同步到 Kibana
- Kibana Dev Tools 开发工具熟练使用
- Kibana 图像化组合展示
- 将 Kibana 图像化展示效果图应用到自己的开发环境中
- ……

**第三学习优先级为 Logstash。**需要掌握的内容如下。

- Logstash 的安装与部署
- Logstash 将本地文件导入 ES
- logstash*input*jdbc 插件（5.X后无需安装）将 MySQL/Oracle 等关系型数据库数据导入 ES，全量导入和增量导入实现。
- logstash*input*mongo插件将 Mongo 数据导入 ES
- logstash*input*kafaka 插件将 Kafak 数据导入 ES
- logstash*output** 插件将 ES 数据导入不同的数据库和实时数据流中
- ……

**第四学习优先级为 Beats。**需要掌握的内容如下。

- 不同类型的 Beats 安装与部署
- 将业务数据通过 Beats 导入 ES
- ……

#### 小结

本小节详细讲述了 Elasticsearch 由初级到高级逐步深入的学习优先级，以及 Kibana、Logstash、Beats 的实践优先级，使得你的基础和进阶学习不再迷茫。

### Elasticsearch 高效进阶方法论

#### 掌握最高效工具

推荐以下几种。

**1.Kibana 工具**

除了支持各种数据的可视化之外，最重要的是支持 Dev Tool 进行 RESTFUL API 增删改查操作。比 Postman 工具和 cURL 都要方便。如下面图所示。

![enter image description here](http://images.gitbook.cn/82eac770-18b2-11e8-8087-9b6b3b447adf)

![enter image description here](http://images.gitbook.cn/8c542450-18b2-11e8-8087-9b6b3b447adf)

**2.head 插件**

可实现 ES 集群状态查看、索引数据查看、ES DSL 实现（增、删、改、查操作），比较实用的地方是 JSON 串的格式化。如下图所示。

![enter image description here](http://images.gitbook.cn/96be4f60-18b2-11e8-8087-9b6b3b447adf)

**3.Cerebro 工具**

用于实现 ES 集群状态查看（堆内存使用率、CPU使用率、内存使用率、磁盘使用率）。如下图所示。

![enter image description here](http://images.gitbook.cn/9e952560-18b2-11e8-8087-9b6b3b447adf)

**4.ElasticHD工具**

其强势功能包括支持 SQL 转 DSL，不要完全依赖，可以借鉴用。如下图所示。

![enter image description here](http://images.gitbook.cn/a87f9920-18b2-11e8-8087-9b6b3b447adf)

![enter image description here](http://images.gitbook.cn/b6dad480-18b2-11e8-8087-9b6b3b447adf)

**5.中文分词工具**

比如有 [IK分词](https://github.com/medcl/elasticsearch-analysis-ik)、[ANSJ分词](https://github.com/NLPchina/elasticsearch-analysis-ansj)、[结巴分词](https://github.com/huaban/elasticsearch-analysis-jieba)。网上还有结巴分词的其他最新版本。

在这里建议选用 IK 分词，原因有以下几点：

- IK 分细粒度 ik*max*word 和粗粒度 ik_smart 两种分词方式。
- IK 更新字典只需要在词典末尾添加关键词即可，支持本地和远程词典两种方式。
- IK 分词插件的更新速度更快，和最新版本保持高度一致。

**6.类 SQL 查询工具**

在此，推荐 [elasticsearch-SQL](https://github.com/NLPchina/elasticsearch-sql)，其支持的 SQL，极大缩小了复杂 DSL 的实现成本。

通过 elasticsearch-SQL 工具可以基于以下 SQL 语句方式请求 ES 集群。

```
select COUNT(*),SUM(age),MIN(age) as m, MAX(age),AVG(age)
FROM bank GROUP BY gender ORDER BY SUM(age), m DESC

```

**7.测试工具**

在原来执行的 DSL 的基础上新增 profile 参数，我把它称作“**测试工具**”。

profile API的目的是，将 ES 高层的 ES 请求拉平展开，直观的让你看到请求做了什么，每个细分点花了多少时间。

profile API给你**改善性能**提供相关支撑工作。

使用举例如下：

```
GET /_search
{
  "profile": true,
  "query" : {
    "match" : { "message" : "message number" }
  }
}

```

**7.ES 性能分析工具**

推荐 [rally](https://github.com/elastic/rally)。相比传统的发包请求测试工具，rally 更加直观和准确、且指标很丰富。

#### 升级认知，不要惧怕新知识

我整理了以下几个常见的问题，并做出了回复。

Q：没有 Lucene 基础，能不能学习 Elasticsearch？

A：这个完全不需要Lucence基础的，遇到了相关底层问题，再回头查Lucene基础完全可以。

Q：C/C++ 程序员，能不能进行 Elasticsearch 开发？

A：这个问题就是 C/C++ 转 Java 的问题，几乎没有难度。

Q：Elasticsearch 如何部署（Linux、Windows等）？

A：如果没有 Linux 环境，Windows 搭建 Demo 无可厚非；如果有 Linux 环境，请不要在 Windows 上浪费时间，没有必要。

如果作为业务系统对外提供服务，建议至少搭建到配置相对高 Linux 服务器（CPU24核心以上、内存64GB以上、磁盘1TB左右以上）上。

现在各种新技术（VR、AR、深度学习、区块链技术等）层出不穷，但大神刘未鹏告诉我们“底层的技术永远不过时”， 对于 Elasticsearch 而言，倒排索引、打分机制、全文检索原理、分词原理等底层技术属于“永远不过时”的技术，要深究。

相信这点，由浅入深夯实基础，各种看似复杂的问题回头再去看都是“小Kiss”。

#### 找最快的方法

现身说法，我曾经对 Jest 使用摸索了很久。久久不能知道正确使用的方式。

最终发现，GitHub 官网的 readme.md 上面提供了详尽的说明，其他地方找的资料都是徒劳。

我的反思中，不要读别人的二手、三手的翻译资料，直接参考官网来的更快。

不要惧怕英文，看似最难的，往往是最快的。

在这点，ES 相关 API 的使用更是如此。比如，ES 的聚合后分页等操作实现，在官网 API 的介绍中都有详尽的描述。

#### 相信社区的力量

首先推荐几个英文社区。

**优先级1：Google**

英文能解决的，效率往往会很高。某度差的非常远。



例如，我在2018年2月23日遇到的集群状态为红色的解决方案。在 Google 输入关键词：elasticsearch ALLOCATION_FAILED，Google 搜索就能很快定位到集群状态为红色的解决方案。

**优先级2：Elasticsearch 英文官方论坛**

问题回答得都很深入，需要翻到底，结合 Google 翻译 translate.google.cn（无需翻墙）查看。

**优先级3： Stack Overflow**

一些问题的版本比较老，1.X 或者 2.X，不过问题的解决思路可以参考。

**优先级4：GitHub**

注意，GitHub的 issue 上有很多问题的解决方案，不要忽略了。

接着看一下中文社区，这里主要推荐 www.elasticsearch.cn 的 Elastic 中文社区。

相信你也会说有 CSDN 中文问答社区、Segmentfault 社区、相关 QQ 群、微信群等。

但是，我要说的是，毕竟 Elastic 中文社区是目前国内最专业的 ELK Stack 技术交流平台，这里的问题回复率非常快、有多位大牛常驻、质量非常好。

更为重要的是，每天都会有 ES 顶级大牛为你遴选最新的、最专业的 Elasticsearch 日报。

再强调一下，Elasticsearch 日报是最好的学习 ELK Stack 技术的方式，没有之一。

#### 站在巨人的肩上

这里推荐两位 ELK Stack 领域大牛 Medcl 和 携程 Wood 大神。

[Medcl](https://elasticsearch.cn/people/medcl)，Elasticsearch 布道者、ES 员工、中国最早接触 ES 的人、ES-IK 开源分词插件作者、gopa 开源爬虫作者。

[携程 Wood 大神](https://elasticsearch.cn/people/wood)，他的文章质量都是源码级实战剖析的结果，很深入、非常实用。

如何向两位大神学习呢？

方法一：精准方法。关注他们在 Elasticsearch 中文社区发表过的文章和回复过的问题，一个个的过一遍。

方法二：没有办法的办法——技术难题向他俩进行提问。

#### ELK Stack学习指南清单

主要包含：

- [《Elasticsearch 全文指南》英文官网文档](https://www.elastic.co/guide/en/elasticsearch/guide/current/_preface.html)，待更新。
- [《Elasticsearch 全文指南》中文官网文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)，请注意版本基于2.X，相关技术可以参考。
- [Elasticsearch 6.2 最新版本 JavaAPI 文档集合](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/index.html)，请注意，各种 API 的使用很详尽，最上方有搜索按钮，可以输入关键词搜索。
- [Elasticsearch5.X 必知必会清单](http://elasticsearch-cheatsheet.jolicode.com/#es5)，还有[中文版](http://blog.csdn.net/laoyang360/article/details/77412668)。

此清单言简意赅，非常实用。

#### 形成问题清单

学习和实践 ELK Stack 的过程中，势必会遇到很多问题，这些问题当时解决了，回头还可能会遇到。

建议形成自己专属的问题清单，记录各种遇到的问题。

比如，ELK 集群搭建部署清单、ELK 上线清单、ELK 集群优化清单、ES 聚合操作清单、ELK 常见问题排查清单。

这种思维方式来源于《清单革命》这本医学领域的巨著，对于我们学习 ELK Stack 也非常有帮助。

#### 形成属于自己的技术积累

问题清单是技术积累的一小部分内容，这里的技术积累还包括：

- 原理的透彻理解和积累——可以画画图、画成脑图、形成文字或博客加深印象。
- bool 组合查询语句、聚合语句 DSL 的积累——积小成多，慢慢的效率就提升了。
- 相关问题的排查思路、解决方案积累——形成问题排查集合。

大牛就是菜鸟解决了无数个问题逐步积累的结果。

#### 推荐书籍

主要有以下两本。

- 《这就是搜索引擎》——博士著作，对基础搜索引擎的原理解析的非常透彻，原理都有一定难度。
- 《深入理解 Elasticsearch》第二版，国外著作，原理解释的很透彻，就是版本有点旧。

#### 推荐视频教程

推荐观看《Elasticsearch 顶尖高手系列-高手进阶篇》视频教程，版权原因，不提供下载链接，可以通过百度网盘搜索工具免费下载。

原理解释的相对详细，可以加快速度过一过。

#### 博客或公众号推荐

主要推荐以下公众号和博客。

- 公众号1：西加加语言——自己写了一套搜索引擎，原理很深入，ES 大神给我推荐的。
- 公众号2：GinoBeFunny——前阿里巴巴员工的技术公众号，里面一些 ES 文字都很深入。
- 公众号3：铭毅天下——死磕 Elasticsearch（分为：基础、进阶、实战等几个部分），更新周期相对快。
- 博客1：[姚攀的博客](http://blog.csdn.net/napoay)，博文系列已经出书。
- 博客2：[铭毅天下](http://blog.csdn.net/laoyang360)，博客中的《深入详解 Elasticsearch》专栏已经有出版社联系出书。

### 小结

以上，是我近3年 ELK Stack 学习和实践经验的总结，历时大于10个小时。

ELK 的两个近200万的中大型项目经历使得我明白：“**必须要实践、实践出真知**”，你的想法再多、思路再清晰都要转换为 ES 的 DSL、Kibana 的可视化、Logstash 的配置文件进行反复实战来验证和调优。

经过以上招数，相信你实战中遇到问题，都能快速找到方法、高效解决。

智人千虑必有一失，如经过以上剖析和干货分享后仍有问题，可在微信公众号“铭毅天下”、CSDN 博客[《铭毅天下》](http://blog.csdn.net/laoyang360)下留言，我看到后都会第一时间回复。