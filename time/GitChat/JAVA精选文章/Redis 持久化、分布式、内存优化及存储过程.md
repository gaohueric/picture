本文基于 Redis 2.8 版本（即[腾讯云](https://cloud.tencent.com/document/product/239/3205)和[阿里云](https://help.aliyun.com/document_detail/55684.html)的 Redis 版本），使用 Python 2.7 作为编程语言。

开始本文之前，读者应当具有 Redis 基础经验，或完成 [《如何基于 Redis 构建应用程序组件》](http://gitbook.cn/books/5a1c5e8ed554272f0e84d34d/index.html) 阅读。

### Redis 数据持久化

Redis 的数据持久化，即：将内存中的数据存储到硬盘（本文中亦称之为 “落地”）。Redis 提供了 RDB 和 AOF 两种持久化的方法：

- RDB：基于特定的时间间隔将数据 “全量快照”，生成 RDB 文件并落地
- AOF (Append Only File)：将 Redis 接收到命令以 “增量追加” 的方式，写入 AOF 文件

Redis 允许使用任意一种持久化方法，亦允许同时使用或同时不使用。以下将阐述两者涉及的配置选项、命令以及优缺点。

#### RDB

##### **配置选项**

```
save 900 1                       # RDB 落地选项，900 秒内有 1 次写入，即落地新的 RDB 文件。
                                 # 允许多个 save 配置，满足任意配置即开始新的 RDB 文件落地；若无 save 配置，即表示关闭 RDB 数据持久化
                                 # 
stop-writes-on-bgsave-error yes  # RDB 文件写入失败时，Redis 是否停止接收写命令，默认 yes，即停止
rdbcompression yes               # 是否压缩 RDB 文件，默认 yes，即压缩
rdbchecksum yes                  # 是否启用 RDB 文件校验，默认 yes，即生成 CRC64，写入 RDB 文件结尾
dbfilename dump.rdb              # RDB 文件名称
dir ./                           # RDB 文件路径（说明：配置与 AOF 共享）


```

RDB 相关的配置，应当结合业务的实际，例如：

- 当 RDB 文件写入失败时，若能够通过其他的运维手段进行及时处理，则无需开启 `stop-writes-on-bgsave-error`，以避免线上服务的中断
- 若开启 `rdbcompression` 与 `rdbchecksum` 选项，RDB 文件的落地、Redis 启动时的 RDB 文件加载，将产生额外的性能损耗

##### **RDB 文件相关的命令**

- `SAVE`

“同步” 创建 RDB 文件，`SAVE` 将阻塞 Redis，Redis 将不能响应其他任何命令，直到 RDB 文件完成创建与落地。

- `BGSAVE`

“异步” 创建 RDB 文件，Redis 创建子进程：父进程继续提供服务，由子进程生成并落地 RDB 文件。

当配置选项中任意的 save 配置条件满足时，Redis 将自动地 “触发” `BGSAVE` 命令。

- `LASTSAVE`

获取最后一次成功落地 RDB 文件的 Unix 时间。

#### AOF

##### **配置选项**

```
appendonly no                    # 是否启用 AOF 数据持久化，默认 no，即关闭
appendfilename "appendonly.aof"  # AOF 文件名称


AOF 文件同步选项

appendfsync everysec             
no-appendfsync-on-rewrite no     


AOF 文件 rewrite 选项


auto-aof-rewrite-percentage 100  
auto-aof-rewrite-min-size 64mb   

aof-load-truncated yes           # 若 AOF 文件的结尾处损坏（由操作系统故障引起），Redis 启动时加载 AOF 文件，根据 aof-load-truncated 配置：
                                 #   默认 yes：忽略 AOF 文件结尾处的损坏
                                 #   no：Redis 进程退出
                                 #
dir ./                           # AOF 文件路径（说明：配置与 RDB 共享）


```

##### **AOF 文件同步**

AOF 文件同步，即通过系统调用 [fsync()](https://gitbook.cn/books/5a6ceb220b172b546f930839/index.html)将 AOF 文件位于操作系统缓冲区的部分（取决于 [write()](http://www.man7.org/linux/man-pages/man2/write.2.html)的工作机制）写入硬盘。

特别说明：fsync()并不确保缓冲区的内容一定能够写入硬盘，其工作机制取决于操作系统。

**appendfsync 配置选项：**

- always：每次写入 AOF 文件，都进行一次 `fsync()` 系统调用
- everysec：默认，每秒进行一次 `fsync()` 系统调用
- no：不进行 `fsync()` 系统调用，完全由操作系统控制

**no-appendfsync-on-rewrite 配置选项：**

- yes：即 Redis 进行 AOF 文件 rewrite 时（或落地 RDB 文件时），`fsync()` 系统调用暂停，以避免可能产生的阻塞
- no：默认，即 Redis 进行 AOF 文件 rewrite 时，继续进行 `fsync()` 系统调用

##### **AOF 文件 rewrite**

作为 Redis 命令的 “log”，AOF 文件的大小必须持续增长。Redis 提供 AOF 文件 rewrite 特性，能够移除 AOF 文件的冗余命令以减少 AOF 文件大小。

例如：Redis 针对一个 key 执行了 100 次的 SET，AOF 文件仅保留最后一次 `SET` 命令即可。

**BGREWRITEAOF 命令：**

“异步” 进行 Redis AOF 文件 rewrite

- 若 Redis 正在进行创建 RDB 文件，AOF 文件 rewrite 将等待 RDB 文件创建完成后开始
- 若 AOF 文件 rewrite 正在进行 ，`BGREWRITEAOF` 命令将不会开始新的 AOF 文件 rewrite

**auto-aof-rewrite-percentage & auto-aof-rewrite-min-size 配置选项：**

AOF 文件自动 rewrite 机制，当 AOF 文件大小达到以下阈值，Redis 即自动开始 AOF 文件 rewrite：

- 增长百分比超过 `auto-aof-rewrite-percentage`（相对于上一次 rewrite 完成时的 AOF 文件）
- 超过 `auto-aof-rewrite-min-size`

通过设置 `auto-aof-rewrite-percentage` 为 0，即关闭 AOF 文件的自动 rewrite。

### RDB 与 AOF 优缺点和选择

RDB

- 非常适合于备份以及灾难恢复的场景
- 能够最大化 Redis 性能
- 相对于 AOF，RDB 文件在 Redis 启动时能够更快加载
- 若期望将数据丢失的可能性最小化，RDB 并不适用

AOF

- 基于 “追加” 和 “文件同步” 的特性，AOF 具有更佳的 “持久化” 表现
- 对于相同的数据，AOF 文件大小通常将超过 RDB

综合而言，如果能够承担一定程度的数据丢失风险，仅启用 RDB 持久化即可。但并不建议只启用 AOF 持久化，毕竟 RDB 文件更适合于数据备份。

若 RDB 持久化和 AOF 持久化同时启用，Redis 启动时，将加载 AOF 文件，毕竟 AOF 具有更佳的 “持久化” 表现。

### Redis “主 - 从” 机制

Redis 提供 “主 - 从” 的数据复制：“从” Redis 即作为 “主” Redis 的数据副本。“从” Redis，既能够用于读性能的扩展，亦能够作为数据备份的一种手段。

同时，Redis 支持 [Redis Sentinel](https://redis.io/topics/sentinel)，实现 “主 - 从” 监控、故障迁移，限于篇幅，本文不予以展开。

#### 工作机制

##### **“主 - 从” 数据复制的基本工作机制**

- 已建立的 “主” - “从” 连接，“主” Redis 不断地将命令发送到 “从” Redis
- 若连接中断（例如：网络问题），“从” Redis 将尝试重新建立连接，并尝试 “半 - 重新同步”
- 若无法进行 “半 - 重新同步”，“从” Redis 将尝试进行 “重新同步”（“主 - 从” 连接首次建立，亦执行 “重新同步”）

##### **关于 “半 - 重新同步” & “重新同步”**

- “半 - 重新同步”：“从” Redis 将尝试获取连接中断期间于 “主” Redis 执行的命令（存储于 backlog）
- “重新同步”
  - “主” Redis 创建数据快照（RDB 文件）、同步到 “从” Redis，开始将 “主” Redis 执行的命令发送到 “从” Redis
  - “从” Redis 丢弃当前数据，加载 “主” Redis 的 RDB 文件，开始执行 “主” Redis 发送的命令

“数据复制” 对于 “主” Redis 全部是异步的；对于 “从” Redis，大部分是异步的，但 “重新同步” 涉及 “丢弃当前数据，加载 RDB 文件”，将引起 “短暂中断”。

#### “主 - 从” 配置

```
slaveof master_ip master_port  # “从” Redis 配置：“主” Redis - IP &  port
masterauth master_password     # “从” Redis 配置：“主” Redis - 密码

slave-serve-stale-data yes     # “从” Redis 配置：当 “主 - 从” 连接中断或 “从” Redis 正在进行初始化同步，“从” Redis 是否提供服务：
                               #   yes: 默认，以 “从” Redis 当前数据提供服务
                               #   no: 对于接收到的命令，“从” Redis 返回 “SYNC in progress”（<code>INFO</code>、<code>SLAVEOF</code> 命令除外）
                               #
slave-read-only yes            # “从” Redis 配置：是否 “只读”，默认 yes


“主” Redis 配置：根据已连接的 “从” Redis 情况，“主” Redis 是否接收 “写命令”
min-slaves-to-write 3
min-slaves-max-lag 10
表示：最少有 3 个已连接的 “从” Redis，且延迟小于等于 10 秒

min-slaves-to-write 3          # 默认 0，即无论 “从” Redis 的连接情况，始终接收 “写命令”
min-slaves-max-lag 10


```

以上的代码，仅列出了部分关键的配置。其他类似于：diskless 复制、backlog 配置，限于篇幅，未能列出，详情内容请参考 [redis.conf for Redis 2.8](https://raw.githubusercontent.com/antirez/redis/2.8/redis.conf)。

#### “主 - 从” 命令

**1. SLAVEOF host port**

将 Redis 配置作为 “从” Redis，其 “主” Redis 位置即为 host:port。

**2. SLAVEOF NO ONE**

终止 “从” Redis 自 “主” Redis 的数据同步。

特别说明：`SLAVEOF NO ONE` 包含了 Redis 设计之初，关于 “自由” 的思想：“*If slavery is not wrong, nothing is wrong.* -- Abraham Lincoln”。

#### “主 - 从” 链

“从” Redis 能够作为其他 Redis 的 “主” Redis，由此构建级联结构的 “主 - 从” 链。并且，“主” Redis 能够与多个 “从” Redis 建立连接，建立 “树状” 结构。

![img](http://images.gitbook.cn/bd16e990-03a6-11e8-a32f-dd8e34cf4c1e)

图中所示，对于扩展读性能，非常有益。

### 优化 Redis 内存使用

合理的 Redis 实例，内存的占有量不应当超过 60%，当内存使用率过高时，应该予以清理及优化。

#### 使用 ziplist & intset

##### **ziplist 优化机制**

ziplist 实现了 “紧凑” 的数据结构，通过尽可能减少非数据节点的占用，以提供内存密度。

![img](http://images.gitbook.cn/b24cb500-0426-11e8-ada6-2fd4765a7873)

图中所示，ziplist 整体结构：

- zl-bytes：整个 ziplist 占用内存的字节数
- zl-tail：ziplist 尾节点距离起始地址的字节数
- zl-len：ziplist 包含的节点数量
- entry：节点
- zl-end：ziplist 末端标记，固定 0xFF

ziplist 节点结构：

- previous-entry-length：前一个节点占用内存的字节数
- encoding：节点编码，明确节点存储内容属于 “字节数组” 或整数，并明确长度（即占用的字节数）
- content：节点存储内容

“散列表”、“链表”、“有序集合”，使用 ziplist，受益于其 “紧凑” 的数据结构，相较于 hashtable、linkedlist、skiplist，能够有效减少内存占用。

然而，受限于 “紧凑” 的数据结构，随着节点数量增长和节点大小膨胀，基于 ziplist 实现的 “散列表”、“链表”、“有序集合”，性能将显著下降。

##### **intset 优化机制**

intset 使用整型数组作为存储的数据结构。通常，hashtable 实现的 Redis 集合，其成员以 “字符串” 结构进行存储，intset 由此能够显著降低内存使用。

类似于 ziplist，同样受限于其整型数组，“集合” 成员数量的增长将引起 “集合” 性能的下降。

##### **涉及配置**

```
 “散列表” 使用 ziplist 的限制条件：
  - 成员数量不超过 hash-max-ziplist-entries
  - 最大内存占用的成员，内存占用不超过 hash-max-ziplist-value (字节)
两者必须同时具备，任意条件不满足，即无法使用 ziplist

hash-max-ziplist-entries 512
hash-max-ziplist-value 64


 “链表” 使用 ziplist 的限制条件

list-max-ziplist-entries 512
list-max-ziplist-value 64


“有序集合” 使用 ziplist 的限制条件

zset-max-ziplist-entries 128
zset-max-ziplist-value 64


“集合” 使用 intset 的限制条件:
- 成员全部为 64 位有符号整数
- 成员数量不超过 set-max-intset-entries

set-max-intset-entries 512


```

附加说明：ziplist & intset 的限制条件，是基于内存占用和性能的综合考虑。

#### 数据分片

##### **分布式 “数据分片”**

分布式 “数据分片”：选取合适的方式将 Redis 数据分布于不同的实例，由此降低单实例的内存使用，实现优化。请参考 “扩展 Redis 的容量和性能” 章节。

##### **单实例 “数据分片”**

通常而言，单实例 “数据分片”，并不能直接降低 Redis 内存使用，需要结合 ziplist 等内存优化方式，以 “散列表” 为例：

- 以散列表的键作为 “数据分片” 的 “路由”，将单个内存占用量大的 “散列表” 分片到多个内存占有量小的 “散列表”
- 内存占有量小的 “散列表”（例如：“散列表” 成员数量少） 能够以 ziplist 方式减少内存占用

由此，有效地实现内存使用的优化。

#### 基于业务进行优化

基于业务，通常能够取得良好的 Redis 内存优化效果，例如：

- 尽可能短的 Redis 键，例如：以 “u_178” 替代 “user_id_178”
- 选择合适的 Redis 数据结构，例如：合理地选择 “散列表” 替代 “字符串”，若 “字符串” 数量较少，使用一个 “散列表” 替代，通常能够减少内存使用
- 减少存储于 Redis 的业务数据量

### 扩展 Redis 的容量和性能

Redis 作为任何单实例的数据服务，最终会遇到容量和性能瓶颈。前文阐述的 Redis “主 - 从”，即为常见且有效的扩展 Redis 读性能的方案。

本章节，主要关注基于 “数据分片” 构建 Redis 集群。常用的方案包括：Redis Cluster、[twemproxy](https://github.com/twitter/twemproxy)、[Codis](https://github.com/CodisLabs)。

#### “数据分片” - 使用 [Codis](https://github.com/CodisLabs)

##### **选择 Codis 的原因**

Codis 来自于 “豌豆荚”，相对于 [twemproxy](https://github.com/twitter/twemproxy)，选择 Codis 的原因：

- twemproxy 无法实现动态水平扩展
- Codis 运行于多核机器能够获得更好的应用

相对于 Redis Cluster，选择 Codis 的原因：

- Redis Cluster 必须使用 Redis 3.0 以上版本的客户端
- Redis Cluster 无法支持 pipeline

##### **Codis 架构**

![img](http://images.gitbook.cn/1db29e20-03b1-11e8-a32f-dd8e34cf4c1e)

图中所示，Codis 架构中引入了 codis-proxy，由 codis-proxy 基于 Redis key 计算分片，将命令转发到 codis-group，因此：对于绝大多数的命令，客户端对于 Codis 的接入是透明的。

Codis 针对 Redis key 计算 CRC32，默认分为 1024 个 Slot，进而路由到特定的 codis-group，实现分片。

除了 “数据分片”，Codis 的特性还包括：

- 提供了 codis-fe & codis-dashborad 作为集群管理工具
- 允许多个 codis-proxy，实现 proxy 层的高可用
- codis-group 支持 “主 - 从”，引入 [redis-sentinel](https://redis.io/topics/sentinel) 实现 “主 - 从” 故障迁移

必须说明的是：“数据分片” 扩展容量和性能的同时，亦限制了 Redis 若干方便的能力，例如：Codis 不支持事务、[部分命令不支持](https://github.com/CodisLabs/codis/blob/release3.2/doc/unsupported_cmds.md)。

### 基于 Lua 的 Redis “存储过程”

Redis 内置了 Lua 解释器（Redis 2.6.0 起），允许使用 Lua（版本 5.1）进行服务器端脚本编程，即类似于 [“存储过程”](https://en.wikipedia.org/wiki/Stored_procedure) 相似。

#### Lua 服务器脚本编程涉及的 Redis 命令

**1. EVAL script numkeys key [key ...] arg [arg ...]**

于 Redis 服务端上下文：

- 执行 script 脚本
- 命令参数 numkeys key [key ...] 即为 key 集合（numkeys 即为集合大小）
- 命令参数 arg [arg ...] 即为传递到脚本的附加参数集合。

特别说明：建议将脚本可能会读取或写入的 key 全部通过 `EVAL` 命令的 numkeys key [key ...] 参数传递（于 Redis 集群非常有益）。

**2. SCRIPT LOAD script**

将 script 脚本载入到 Redis 服务端，并返回脚本的 SHA1 摘要。

**3. EVALSHA sha1 numkeys key [key ...] arg [arg ...]**

与 `EVAL` 命令相似，通过 `SCRIPT LOAD` 命令获得的 sha1 明确需要执行的脚本。

#### Lua 与 Redis 交互

##### **Lua 脚本获取 EVAL & EVALSHA 命令的参数**

通过 Lua 脚本的全局变量 `KEYS` 和 `ARGV`，能够访问 `EVAL` 和 `EVALSHA` 命令的 key [key ...] 参数和 arg [arg ...] 参数。

作为 Lua Table，能够将 `KEYS` 和 `ARGV` 作为一维数组使用，其下标从 1 开始。

##### **Lua 脚本内部执行 Redis 命令**

Lua 脚本内部允许通过内置函数执行 Redis 命令：

- `redis.call()`
- `redis.pcall()`

函数的第 1 个参数即为 Redis 命令，命令的参数即为函数的其他参数。

两者非常相似，区别在于：若 Redis 命令执行错误，`redis.call()` 将错误抛出（即 `EVAL`& `EVALSHA` 执行出错）；`redis.pcall()` 将错误内容返回。

##### **Lua 与 Redis 数据类型**

Redis 与 Lua 脚本的交互，必然涉及 Lua 和 Redis 的数据类型转换。限于篇幅，本文仅列出若干关键点。完整的数据类型转换请参阅 [redis.io](https://redis.io/commands/eval#conversion-between-lua-and-redis-data-types)。

**1. Lua 提供 boolean 数据类型，Redis 无 boolean 数据类型**

- “true” (Lua) 转换为 “整数 1” (Redis)
- “false” (Lua) 与 “nil” (Redis) 互相转换

**2. Lua 仅提供单一的数值类型 number，不区分浮点数和整数**

- number (Lua) 转换为 “整数”（Redis），舍弃 number 的小数部分
- 若需要传递 “浮点数”，必须使用字符串

**3. Lua Table 与 Redis “multi bulk reply” 相互转换**

若 Table (Lua) 的某一成员被转换为 nil (Redis)，Table (Lua) 后续成员的转换即终止

#### 示例： “接口调用延时” 统计分析的组件

使用 Lua，除了能够简化代码、减少与 Redis 的交互，一个重要的收益：Lua 脚本、单个 Redis 命令、“`MULTI` / `EXEC`” 事务，都是原子操作。

[《如何基于 Redis 构建应用程序组件》](http://gitbook.cn/books/5a1c5e8ed554272f0e84d34d/index.html) 阐述了 “接口调用延时” 统计分析的组件。本文章使用 Lua 脚本编程，重新实现 “接口调用延时” “上报” 接口。

```
 获取 Redis 连接

def get_connection():
return redis.Redis.from_url('redis://127.0.0.1:6379/')


“接口调用延时” 数据上报 - Lua 脚本初始化

def init_report(conn = None):
  if conn is None:
    conn = get_connection()

  return conn.script_load('''
  local time_delay = ARGV[1]

  local temp_key_for_max = KEYS[1]
  local temp_key_for_min = KEYS[2]
  local statistics_key_for_max = KEYS[3]
  local statistics_key_for_min = KEYS[4]
  local statistics_key_for_sum_count = KEYS[5]

  redis.call('zadd', temp_key_for_max, time_delay, 'max')
  redis.call('zunionstore', statistics_key_for_max, 2, statistics_key_for_max, temp_key_for_max, 'AGGREGATE', 'MAX')

  redis.call('zadd', temp_key_for_min, time_delay, 'min')
  redis.call('zunionstore', statistics_key_for_min, 2, statistics_key_for_min, temp_key_for_min, 'AGGREGATE', 'MIN')

  redis.call('zincrby', statistics_key_for_sum_count, 1, 'count')
  redis.call('zincrby', statistics_key_for_sum_count, time_delay, 'sum')

  redis.call('del', temp_key_for_max, temp_key_for_min)
  ''')


“接口调用延时” 数据上报

def report(interface, time_delay, conn = None, function = None):
  if conn is None:
    conn = get_connection()

  if function is None:
    function = init_report(conn)

  temp_key_for_max = str(uuid.uuid4())
  temp_key_for_min = str(uuid.uuid4())

  raw_key = 'statistics_%s_%d' % (interface, (int(time.time()) / 60))
  statistics_key_for_max = '%s_for_max' % (raw_key, )
  statistics_key_for_min = '%s_for_min' % (raw_key, )
  statistics_key_for_sum_count = '%s_for_sum_count' % (raw_key, )

  conn.evalsha(function, 5, temp_key_for_max, temp_key_for_min, statistics_key_for_max, statistics_key_for_min, statistics_key_for_sum_count, time_delay)


```

基于 Redis 特性，Lua 脚本执行过程中，Redis 不能接收其他命令，假如执行时间过长（超过 Redis 配置项 `lua-time-limit`），此时：

- 若 Lua 脚本没有进行 “写” 操作，能够通过 `SCRIPT KILL` 安全地终止 Lua 脚本；
- 否则，为了确保 Lua 脚本的 “原子性” ，仅允许通过 `SHUTDOWN NOSAVE` 强制终止 Redis 进程

### 写在最后

期望通过本文，读者能够进一步了解产生环境中应用 Redis 的各种事宜。

