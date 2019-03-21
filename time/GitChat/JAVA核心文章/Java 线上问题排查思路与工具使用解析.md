#### 内容提要

内容提要：

- ​
- ​
- 一般线上系统怎么做监控？
- 一次发布，生产环境里某个节点里的容器里面服务挂了，但是在预生产环境无法重现，而生产环境监控里面显示 cpu、内存、堆栈都正常。而那个节点重启之后就正常，但是过段时间又出现问题。这种情况有什么好的方法排查问题？
- 如何生成 dump 文件，如何分析？jvm crash 日志如何生成？
- 业务日志和异常输出格式一般怎么设计会比较容易统计分析？
- 能深入讲解一下线上监控吗？
- 线上排查问题 dump 或 jstack 导出文件分析经常无响应倒置无法导出查看内存问题和线程问题，有什么经验吗？
- 如果判断 gc 是正常的 gc，还是严重影响正常应用性能的 gc？
- 日志都会通过 flume 采集器传输至数据中心，有什么作用，数据中心在做什么？

------

**问：一般线上系统怎么做监控？**

**答：** 线上监控基本上从两个方面着手：应用监控和系统监控。

1. 线上接口调用情况可视化（调用次数、平均耗时、失败率等）；
2. 数据库调用可视化（调用次数、平均耗时、失败率等）；
3. 缓存调用情况日志分析；
4. 消息中间件调用情况日志采集分析；
5. 线上系统异常日志采集分析。

业务监控一般需要一套分布式日志调用跟踪系统，如淘宝鹰眼。

系统监控可以采用 zabbix + grafana 等(可视化视图) 实时统计分析 CPU、内存、磁盘、网络情况。

------

**问：一次发布，生产环境里某个节点里的容器里面服务挂了，但是在预生产环境无法重现，而生产环境监控里面显示 cpu、内存、堆栈都正常。而那个节点重启之后就正常，但是过段时间又出现问题。这种情况有什么好的方法排查问题？**

**答：** UAT 环境启动正常没办法重现，而 ONLINE 环境启动时而正常时而异常。第一种可能性就是 UAT 与 ONLINE 环境的底层物理硬件可能不一致， 你可以分析 UAT 与 ONLINE 的系统内核版本是否一致，tomcat 服务版本等等是否一致，甚至你可以你分析磁盘型号是否为一致的。如果是物理原因，可以找运维配合一起排查问题。

第二种可能性就是自身应用存在 bug，真的没办法可以考虑降低日志等级，打印出更为详细有价值的启动/业务日志信息，业务日志到底怎么打印才有价值，后面我会深入跟大家说明的。

------

**问：如何生成 dump 文件，如何分析？jvm crash 日志如何生成？**

**答：** 如何生成 dump 文件：. `/jmap -dump:format=b,file=/usr/local/logs/gc/dump.hprof 3831`

可以压缩 dump 文件下载到本地导入 mat 等内存分析工具进行分析，mat 用法同学们可以找相关资料。crash 日志通过设置 JVM 参数-XX:ErrorFile， 但是 linux 系统发生 oom killer 时候不会产生可以查看系统日志 /var/logs/messages。

------

**问：业务日志和异常输出格式一般怎么设计会比较容易统计分析？**

**答：** 以我司的分布式日志链路跟踪调用系统为例，我们会打印出，

- 服务引用摘要 : common-sal-digest.log，
- 服务发布摘要日志 common-service-digest.log，
- web 摘要日志: common-page_digest.log，
- web 详情摘要日志: comon-web-detail.log，
- 数据层摘要： common-dal-digest.log，
- 通用日志: common-default-digest.log，
- 方法调用具体信息日志: common-method-debug.log，
- 错误日志: common-error.log，
- 业务异常日志: common-biz.log。
- 所有的 日志 都会带一个全局唯一的traceId。

![enter image description here](http://images.gitbook.cn/d17ac4b0-f47e-11e7-a568-2188478cb7d7)

以common-page-digest.log为例:

```
2017-12-28 15:26:45,011 - [(/xxx-analytic/,GET,200,Y,2ms)](traceId=e1e99ab562e54e15b16efb3d14baa26f)

```

------

**问：能深入讲解一下线上监控吗？**

**答：** 文章里面所有的工具/命令都挺是实操知识，目前我们业务日志平台，采用了 flume、kafka、es、hbase 等。系统监控采用 zabbix + grafa 等工具，不同规模的集群系统，所需监控能力是不一样的。你要仅仅是个单体应用，这些监控可能就显得笨重了。

------

**问：线上排查问题 dump 或 jstack 导出文件分析经常无响应倒置无法导出查看内存问题和线程问题，有什么经验吗？**

**答：** 增加 -F 强制，在 jmap-dump 使用，如果 pid 没有相应的回复。当然 -F 也可能没办法 dump 下来，那就只能去分析日志了。当然如果你决定手动 dump 内存时，dump 操作占据一定 CPU 时间片、内存资源、磁盘资源等，对系统的正常的运行可能会有一些影响。

------

**问：如何判断 gc 是正常的 gc 还是严重影响正常应用性能的 gc？**

**答：** gc 分为 Minor gc 与 full gc。Minor gc非常频繁，一般回收速度也比较快。 正常的 full gc 频次比较低，我们线上一天可能也就那么几次有时候甚至只有一次，还有 CMS 进行垃圾回收时也有可能会发生失败的情况。

prommotion failed：存活区内存不足，对象进入老年代。老年代仍然没有内存容纳对象将触发 Full GC。

concurrent mode failed：CMS 回收速度慢，CMS 完成前老年代已被占满，将导致一次 Full GC。

------

**问：日志都会通过 flume 采集器传输至数据中心，有什么作用，数据中心在做什么？**

**答：** flume 作为一个 agent 部署至云服务器上，用于采集上面我提到所有业务日志，将这些数据传送至 kafka sink。我们有一个类似 logstash 的中间件接收、处理、转发日志信息至数据中心。数据中心可以统计分析这些业务日志信息，比如系统的 pv、调用次数、调用成功率、失败率、平均耗时、调用链率查询。数据中心还有一个大数据平台，通过机器学习/数据挖掘建模，也用来做风控的数据来源等等，里面内容比较多，每个点都可以分析很长时间。



#### 文章实录

### 0. 前言

Java 语言是当前互联网应用最为广泛的语言，作为一名 Java 程序猿，当业务相对比较稳定之后平常工作除了 coding 之外，大部分时间（70%~80%）是会用来排查突发或者周期性的线上问题。由于业务应用 bug（本身或引入第三方库）、内外部环境、底层硬件问题等原因，Java线上服务出现故障/问题几乎不可避免。例如，常见的现象包括部分请求超时、用户明显感受到系统发生卡顿等等。

尽管线上问题从系统表象来看非常明显，但排查深究其发生的原因还是比较困难的，因此对开发测试或者是运维的同学产生了许多的困扰。排查定位线上问题是具有一定技巧或者说是经验规律的，排查者如果对业务系统了解得越深入，那么相对来说定位也会容易一些。

不管怎么说，掌握 Java 服务线上问题排查思路并能够熟练排查问题常用工具/命令/平台是每一个 Java 程序猿进阶必须掌握的实战技能。笔者依据自己的 工作经验总结出一套基本的线上问题排查流程，同学们可以根据自己的实际工作情况进行归纳总结。

### 1. Java 服务常见线上问题

所有 Java 服务的线上问题从系统表象来看归结起来总共有四方面：CPU、内存、磁盘、网络。例如 CPU 使用率峰值突然飚高、内存溢出(泄露)、磁盘满了、网络流量异常、FullGC 等等问题。基于这些现象我们可以将线上问题分成两大类: 系统异常、业务服务异常。

#### 1.1 系统异常

常见的系统异常现象包括: CPU 占用率过高、CPU上下文切换频率次数较高、磁盘满了、磁盘 I/O 过于频繁、网络流量异常（连接数过多）、系统可用内存长期处于较低值（导致 oom killer）等等。这些问题可以通过 top（cpu）、free（内存）、df（磁盘）、dstat（网络流量）、pstack、vmstat、strace（底层系统调用）等工具获取系统异常现象数据。

此外，如果对系统以及应用进行排查后，均未发现异常现象的根本原因，那么也有可能是因为外部基础设施 IAAS 平台所引发的问题。例如运营商网络或者云服务提供商偶尔可能也会发生一些故障问题，你的引用只有某个区域如广东用户访问系统时发生服务不可用现象，那么极有可能是这些原因导致的。今天，我司部署在阿里云华东地域的业务系统中午时分突然不能为广东地区用户提供正常服务，对系统进行各种排查均为发现任何问题。最后，通过查询阿里云公告得知原因是"广东地区电信线路访问华东地区互联网资源（包含阿里云华东1地域）出现网络丢包或者延迟增大的异常情况"。

<https://help.aliyun.com/noticelist/articleid/20724342.html?spm=5176.789004748.n2.6.LeTsMp>

![img](http://images.gitbook.cn/ad615ff0-ec9d-11e7-a3aa-b3fbc77f24b9)

#### 1.2 业务服务异常

常见的业务服务异常现象包括: PV 量过高、服务调用耗时异常、线程死锁、多线程并发问题、频繁进行 Full GC、异常安全攻击扫描等。

### 2. 问题定位

我们一般会采用排除法，从外部排查到内部排查的方式来定位线上服务问题。

- 首先我们要排除其他进程（除主进程之外）可能引起的故障问题
- 其次排除业务应用可能引起的故障问题
- 最后可以考虑是否为运营商或者云服务提供商所引起的故障

#### 2.1 定位流程

**2.1.1 系统异常排查流程**

![enter image description here](http://images.gitbook.cn/c26fd8e0-ec9d-11e7-a902-eb5a7c4242cd)

**2.1.2 业务应用排查流程**

![enter image description here](http://images.gitbook.cn/cc8c3210-ec9d-11e7-b506-175d00ac319c)

#### 2.2 Linux 常用的性能分析工具

Linux 常用的性能分析工具使用包括 : top（cpu）、free（内存）、df（磁盘）、dstat（网络流量）、pstack、vmstat、strace（底层系统调用）等。

**2.2.1 CPU**

CPU 是系统重要的监控指标，能够分析系统的整体运行状况。监控指标一般包括运行队列、*CPU*使用率和上下文切换等。

**top**命令是*Linux*下常用的 CPU 性能分析工具,能够实时显示系统中各个进程的资源占用状况,常用于服务端性能分析。

![img](http://images.gitbook.cn/d589d4d0-ec9d-11e7-8bb8-73e9f039edac)

> top 命令显示了各个进程 CPU 使用情况，一般 CPU 使用率从高到低排序展示输出。其中 Load Average 显示最近1分钟、5分钟和15分钟的系统平均负载，上图各值为2.46，1.96，1.99。
>
> 我们一般会关注 CPU 使用率最高的进程，正常情况下就是我们的应用主进程。第七行以下：各进程的状态监控。

```
    PID : 进程id
    USER : 进程所有者
    PR : 进程优先级
    NI : nice值。负值表示高优先级，正值表示低优先级
    VIRT : 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
    RES : 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
    SHR : 共享内存大小，单位kb
    S : 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
    %CPU : 上次更新到现在的CPU时间占用百分比
    %MEM : 进程使用的物理内存百分比
    TIME+ : 进程使用的CPU时间总计，单位1/100秒
    COMMAND : 进程名称

```

**2.2.2 内存**

内存是排查线上问题的重要参考依据，内存问题很多时候是引起 CPU 使用率较高的见解因素。

系统内存：free 是显示的当前内存的使用，-m 的意思是M字节来显示内容。

```
free -m

```

![enter image description here](http://images.gitbook.cn/e69f0010-ec9d-11e7-a902-eb5a7c4242cd)

> 部分参数说明：
>
> total 内存总数: 3790M used 已经使用的内存数: 1880M free 空闲的内存数: 118M shared 当前已经废弃不用,总是0 buffers Buffer 缓存内存数: 1792M

**2.2.3 磁盘**

磁盘满了很多时候会连带引起系统服务不可用等问题

![enter image description here](http://images.gitbook.cn/ece567e0-f439-11e7-8fbf-c59fff073cfb)

```
df -h

```

![enter image description here](http://images.gitbook.cn/ee4c06f0-ec9d-11e7-8bb8-73e9f039edac)

```
du -m /path

```

![enter image description here](http://images.gitbook.cn/f4b1e7d0-ec9d-11e7-a902-eb5a7c4242cd)

**2.2.4 网络**

dstat 命令可以集成了 vmstat、iostat、netstat 等等工具能完成的任务。

```
    dstat -c  cpu情况
          -d 磁盘读写
          -n 网络状况
          -l 显示系统负载
          -m 显示形同内存状况
          -p 显示系统进程信息
          -r 显示系统IO情况

```

![enter image description here](http://images.gitbook.cn/fcb96ac0-ec9d-11e7-8bb8-73e9f039edac)

**2.2.5 其它**

vmstat：

```
vmstat 2 10 -t

```

vmstat 是 Virtual Meomory Statistics（虚拟内存统计）的缩写, 是实时系统监控工具。该命令通过使用 knlist 子程序和 /dev/kmen 伪设备驱动器访问这些数据，输出信息直接打印在屏幕。

使用 vmstat 2 10 -t命令，查看 io 的情况 （第一个参数是采样的时间间隔数单位是秒，第二个参数是采样的次数）。

![enter image description here](http://images.gitbook.cn/0a7fe030-ec9e-11e7-a902-eb5a7c4242cd)

```
r 表示运行队列(就是说多少个进程真的分配到CPU),b 表示阻塞的进程。    
swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器。
free   空闲的物理内存的大小，我的机器内存总共4G，剩余120M左右。
buff   Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存，我本机大概占用40多M
cache 文件缓存
si列表示由磁盘调入内存，也就是内存进入内存交换区的数量；
so列表示由内存调入磁盘，也就是内存交换区进入内存的数量
一般情况下，si、so的值都为0，如果si、so的值长期不为0，则表示系统内存不足，需要考虑是否增加系统内存。    
bi 从块设备读入数据的总量（读磁盘）（每秒kb）
bo 块设备写入数据的总量（写磁盘）（每秒kb）
随机磁盘读写的时候，这两个值越大((超出1024k)，能看到cpu在IO等待的值也会越大
这里设置的bi+bo参考值为1000，如果超过1000，而且wa值比较大，则表示系统磁盘IO性能瓶颈。
in 每秒CPU的中断次数，包括时间中断
cs(上下文切换Context Switch)

```

strace：strace常用来跟踪进程执行时的系统调用和所接收的信号。

```
strace -cp tid
strace -T -p tid  
       -T 显示每一调用所耗的时间.
       -p pid  跟踪指定的进程pid. 
       -v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出. 
       -V 输出strace的版本信息. 

```

![enter image description here](http://images.gitbook.cn/1dd92f10-ec9e-11e7-8bb8-73e9f039edac)

#### 2.3 JVM 定位问题工具

在 JDK 安装目录的 bin 目录下默认提供了很多有价值的命令行工具。每个小工具体积基本都比较小，因为这些工具只是 jdk\lib\tools.jar 的简单封装。

![enter image description here](http://images.gitbook.cn/28627ae0-ec9e-11e7-a3aa-b3fbc77f24b9)

其中，定位排查问题时最为常用命令包括：jps（进程）、jmap（内存）、jstack（线程）、jinfo（参数）等。

- jps：查询当前机器所有JAVA进程信息
- jmap：输出某个 Java 进程内存情况（如产生那些对象及数量等）
- jstack：打印某个 Java 线程的线程栈信息
- jinfo：用于查看 jvm 的配置参数

**2.3.1 jps 命令**

jps 用于输出当前用户启动的所有进程 ID，当线上发现故障或者问题时，能够利用 jps 快速定位对应的 Java 进程 ID。

```
jps -l -m
-m -l -l参数用于输出主启动类的完整路径

```

![enter image description here](http://images.gitbook.cn/37f63640-ec9e-11e7-a3aa-b3fbc77f24b9)

当然，我们也可以使用 Linux 提供的查询进程状态命令，例如：

```
ps -ef | grep tomcat

```

我们也能快速获取 Tomcat 服务的进程 id。

**2.3.2 jmap 命令**

```
jmap -heap pid   输出当前进程JVM堆新生代、老年代、持久代等请情况，GC使用的算法等信息
jmap -histo:live {pid} | head -n 10  输出当前进程内存中所有对象包含的大小
jmap -dump:format=b,file=/usr/local/logs/gc/dump.hprof {pid} 以二进制输出档当前内存的堆情况，然后可以导入MAT等工具进行

```

JMap（Java Memory Map）可以输出所有内存中对象的工具，甚至可以将 VM 中的 heap，以二进制输出成文本。

jmap -heap pid：

```
jmap -heap pid   输出当前进程JVM堆新生代、老年代、持久代等请情况，GC使用的算法等信息

```

jmap 可以查看 JVM 进程的内存分配与使用情况，使用 的 GC 算法等信息。

![enter image description here](http://images.gitbook.cn/492922b0-ec9e-11e7-8bb8-73e9f039edac)

![enter image description here](http://images.gitbook.cn/506fc6a0-ec9e-11e7-8bb8-73e9f039edac)

jmap -histo:live {pid} | head -n 10：

```
jmap -histo:live {pid} | head -n 10  输出当前进程内存中所有对象包含的大小

```

![enter image description here](http://images.gitbook.cn/58a362a0-ec9e-11e7-b506-175d00ac319c)

输出当前进程内存中所有对象实例数（instances）和大小（bytes），如果某个业务对象实例数和大小存在异常情况，可能存在内存泄露或者业务设计方面存在不合理之处。

jmap -dump：

```
jmap -dump:format=b,file=/usr/local/logs/gc/dump.hprof {pid} 

```

-dump:formate=b,file= 以二进制输出当前内存的堆情况至相应的文件，然后可以结合 MAT 等内存分析工具深入分析当前内存情况。

一般我们要求给 JVM 添加参数 **-XX:+Heap Dump On Out Of Memory Error OOM**确保应用发生 OOM 时 JVM 能够保存并 dump 出当前的内存镜像。当然如果你决定手动 dump 内存时，dump 操作占据一定 CPU 时间片、内存资源、磁盘资源等，因此会带来一定的负面影响。

此外，dump 的文件可能比较大,一般我们可以考虑使用**zip**命令对文件进行压缩处理，这样在下载文件时能减少带宽的开销。在下载 dump 文件完成之后，由于 dump 文件较大可将 dump 文件备份至制定位置或者直接删除，以释放磁盘在这块的空间占用。

**2.3.3 jstack 命令**

```
printf '%x\n' tid   -->  10进制至16进制线程ID(navtive线程) %d 10进制
jstack pid | grep tid -C 30 --color   
ps -mp 8278 -o THREAD,tid,time | head -n 40

```

某 Java 进程 CPU 占用率高，我们想要定位到其中 CPU 占用率最高的线程。

（1） 利用 top 命令可以查出占 CPU 最高的线程 pid

```
top -Hp {pid}

```

![enter image description here](http://images.gitbook.cn/6c256f30-ec9e-11e7-8bb8-73e9f039edac)

（2） 占用率最高的线程 ID 为 6900，将其转换为16进制形式（因为 java native 线程以16进制形式输出）

```
printf '%x\n' 6900

```

![enter image description here](http://images.gitbook.cn/7429e4e0-ec9e-11e7-b506-175d00ac319c)

（3） 利用 jstack 打印出 Java 线程调用栈信息

```
jstack 6418 | grep '0x1af4' -A 50 --color

```

![enter image description here](http://images.gitbook.cn/8cb66100-ec9e-11e7-a3aa-b3fbc77f24b9)

**2.3.4 jinfo 命令**

```
查看某个JVM参数值
jinfo -flag ReservedCodeCacheSize 28461
jinfo -flag MaxPermSize 28461

```

**2.3.5 jstat 命令**

```
jstat -gc pid
jstat -gcutil `pgrep -u admin java`

```

#### 2.4 内存分析工具 MAT

**2.4.1 什么是 MAT?**

MAT（Memory Analyzer Tool），一个基于 Eclipse 的内存分析工具，是一个快速、功能丰富的 JAVA heap 分析工具，它可以帮助我们查找内存泄漏和减少内存消耗。使用内存分析工具从众多的对象中进行分析，快速的计算出在内存中对象的占用大小，看看是谁阻止了垃圾收集器的回收工作，并可以通过报表直观的查看到可能造成这种结果的对象。

![enter image description here](http://images.gitbook.cn/9749bef0-ec9e-11e7-8bb8-73e9f039edac)

右侧的饼图显示当前快照中最大的对象。单击工具栏上的柱状图，可以查看当前堆的类信息，包括类的对象数量、浅堆（Shallow heap）、深堆（Retained Heap）。

**浅堆**表示一个对象结构所占用内存的大小。**深堆**表示一个对象被回收后，可以真实释放的内存大小。

- 支配树（The Dominator Tree）：

  列出了堆中最大的对象，第二层级的节点表示当被第一层级的节点所引用到的对象，当第一层级对象被回收时，这些对象也将被回收。这个工具可以帮助我们定位对象间的引用情况，垃圾回收时候的引用依赖关系

- Path to GC Roots

  被JVM持有的对象，如当前运行的线程对象，被systemclass loader加载的对象被称为GC Roots， 从一个对象到GC Roots的引用链被称为Path to GC Roots， 通过分析Path to GC Roots可以找出JAVA的内存泄露问题，当程序不在访问该对象时仍存在到该对象的引用路径。

### 3. 日志分析

#### 3.1 GC 日志分析

**3.1.1 GC 日志详细分析**

Java 虚拟机 GC 日志是用于定位问题重要的日志信息，频繁的 GC 将导致应用吞吐量下降、响应时间增加，甚至导致服务不可用。

```
-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/usr/local/gc/gc.log -XX:+UseConcMarkSweepGC

```

我们可以在 Java 应用的启动参数中增加**-XX:+PrintGCDetails **可以输出 GC 的详细日志，例外还可以增加其他的辅助参数，如 -Xloggc 制定 GC 日志文件地址。如果你的应用还没有开启该参数,下次重启时请加入该参数。

![enter image description here](http://images.gitbook.cn/a15157a0-ec9e-11e7-a3aa-b3fbc77f24b9)

上图为线上某应用在平稳运行状态下的GC日志截图。

```
2017-12-29T18:25:22.753+0800: 73143.256: [GC2017-12-29T18:25:22.753+0800: 73143.257: [ParNew: 559782K->1000K(629120K), 0.0135760 secs] 825452K->266673K(2027264K), 0.0140300 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 

```

```
[2017-12-29T18:25:22.753+0800: 73143.256] ： 自JVM启动73143.256秒时发生本次GC.
[ParNew: 559782K->1000K(629120K), 0.0135760 secs] : 对新生代进行的GC，使用ParNew收集器，559782K是新生代回收前的大小,1000K是新生代回收后大小,629120K是当前新生代分配的内存总大小, 0.0135760 secs表示本次新生代回收耗时 0.0135760秒
[825452K->266673K(2027264K), 0.0140300 secs]:825452K是回收堆内存大小,266673K是回收堆之后内存大小，2027264K是当前堆内存总大小,0.0140300 secs表示本次回收共耗时0.0140300秒
[Times: user=0.02 sys=0.00, real=0.02 secs] : 用户态耗时0.02秒,系统态耗时0.00,实际耗时0.02秒

```

无论是 minor GC 或者是 Full GC，我们主要关注 GC 回收实时耗时， 如 real=0.02secs，即 stop the world 时间，如果该时间过长，则严重影响应用性能。

**3.1.2 CMS GC 日志分析**

Concurrent Mark Sweep（CMS）是老年代垃圾收集器，从名字（Mark Sweep）可以看出，CMS 收集器就是“标记-清除”算法实现的，分为六个步骤：

- 初始标记（STW initial mark）
- 并发标记（Concurrent marking）
- 并发预清理（Concurrent precleaning）
- 重新标记（STW remark）
- 并发清理（Concurrent sweeping）
- 并发重置（Concurrent reset）

其中初始标记（STW initial mark） 和 重新标记（STW remark）需要“Stop the World”。

**初始标记** ：在这个阶段，需要虚拟机停顿正在执行的任务，官方的叫法 STW（Stop The Word）。这个过程从垃圾回收的"根对象"开始，只扫描到能够和"根对象"直接关联的对象，并作标记。所以这个过程虽然暂停了整个 JVM，但是很快就完成了。

**并发标记** ：这个阶段紧随初始标记阶段，在初始标记的基础上继续向下追溯标记。并发标记阶段，应用程序的线程和并发标记的线程并发执行，所以用户不会感受到停顿。

**并发预清理** ：并发预清理阶段仍然是并发的。在这个阶段，虚拟机查找在执行并发标记阶段新进入老年代的对象（可能会有一些对象从新生代晋升到老年代， 或者有一些对象被分配到老年代）。通过重新扫描，减少下一个阶段"重新标记"的工作，因为下一个阶段会 Stop The World。

**重新标记** ：这个阶段会暂停虚拟机，收集器线程扫描在 CMS 堆中剩余的对象。扫描从"跟对象"开始向下追溯，并处理对象关联。

**并发清理** ：清理垃圾对象，这个阶段收集器线程和应用程序线程并发执行。

**并发重置** ：这个阶段，重置 CMS 收集器的数据结构，等待下一次垃圾回收。

CMS 使得在整个收集的过程中只是很短的暂停应用的执行，可通过在 JVM 参数中设置 -XX:UseConcMarkSweepGC 来使用此收集器，不过此收集器仅用于 old 和 Perm（永生）的对象收集。CMS 减少了 stop the world 的次数，不可避免地让整体 GC 的时间拉长了。

Full GC 的次数说的是 stop the world 的次数，所以一次 CMS 至少会让 Full GC 的次数+2，因为 CMS Initial mark 和 remark 都会 stop the world，记做2次。而 CMS 可能失败再引发一次 Full GC。

![enter image description here](http://images.gitbook.cn/af04e150-ec9e-11e7-a3aa-b3fbc77f24b9)

上图为线上某应用在进行 CMS GC 状态下的 GC 日志截图。

```
2017-11-02T09:27:03.989+0800: 558115.552: [GC [1 CMS-initial-mark: 1774783K(1926784K)] 1799438K(2068800K), 0.0123430 secs] [Times: user=0.01 sys=0.01, real=0.02 secs] 
2017-11-02T09:27:04.001+0800: 558115.565: [CMS-concurrent-mark-start]
2017-11-02T09:27:04.714+0800: 558116.277: [CMS-concurrent-mark: 0.713/0.713 secs] [Times: user=1.02 sys=0.03, real=0.71 secs] 
2017-11-02T09:27:04.714+0800: 558116.277: [CMS-concurrent-preclean-start]
2017-11-02T09:27:04.722+0800: 558116.285: [CMS-concurrent-preclean: 0.008/0.008 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
2017-11-02T09:27:04.722+0800: 558116.286: [CMS-concurrent-abortable-preclean-start]
2017-11-02T09:27:04.836+0800: 558116.399: [GC2017-11-02T09:27:04.836+0800: 558116.400: [ParNew: 138301K->6543K(142016K), 0.0155540 secs] 1913085K->1781327K(2068800K), 0.0160610 secs] [Times: user=0.03 sys=0.01, real=0.02 secs] 
2017-11-02T09:27:05.005+0800: 558116.569: [CMS-concurrent-abortable-preclean: 0.164/0.283 secs] [Times: user=0.46 sys=0.02, real=0.28 secs] 
2017-11-02T09:27:05.006+0800: 558116.570: [GC[YG occupancy: 72266 K (142016 K)]2017-11-02T09:27:05.006+0800: 558116.570: [Rescan (parallel) , 0.2523940 secs]2017-11-02T09:27:05.259+0800: 558116.822: [weak refs processing, 0.0011240 secs]2017-11-02T09:27:05.260+0800: 558116.823: [scrub string table, 0.0028570 secs] [1 CMS-remark: 1774783K(1926784K)] 1847049K(2068800K), 0.2566410 secs] [Times: user=0.14 sys=0.00, real=0.26 secs] 
2017-11-02T09:27:05.265+0800: 558116.829: [CMS-concurrent-sweep-start]
2017-11-02T09:27:05.422+0800: 558116.986: [GC2017-11-02T09:27:05.423+0800: 558116.986: [ParNew: 120207K->2740K(142016K), 0.0179330 secs] 1885446K->1767979K(2068800K), 0.0183340 secs] [Times: user=0.03 sys=0.01, real=0.02 secs] 
2017-11-02T09:27:06.240+0800: 558117.804: [GC2017-11-02T09:27:06.240+0800: 558117.804: [ParNew: 116404K->3657K(142016K), 0.0134680 secs] 1286444K->1173697K(2068800K), 0.0138460 secs] [Times: user=0.03 sys=0.00, real=0.01 secs] 
2017-11-02T09:27:06.966+0800: 558118.530: [GC2017-11-02T09:27:06.966+0800: 558118.530: [ParNew: 117321K->2242K(142016K), 0.0135210 secs] 738838K->623759K(2068800K), 0.0140130 secs] [Times: user=0.03 sys=0.00, real=0.02 secs] 
2017-11-02T09:27:07.144+0800: 558118.708: [CMS-concurrent-sweep: 1.820/1.879 secs] [Times: user=2.88 sys=0.14, real=1.88 secs] 
2017-11-02T09:27:07.144+0800: 558118.708: [CMS-concurrent-reset-start]
2017-11-02T09:27:07.149+0800: 558118.713: [CMS-concurrent-reset: 0.005/0.005 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 

```

如果你已掌握 CMS 的垃圾收集过程，那么上面的 GC 日志你应该很容易就能看的懂，这里我就不详细展开解释说明了。

此外 CMS 进行垃圾回收时也有可能会发生失败的情况。

异常情况有：

伴随 prommotion failed，然后 Full GC：

[prommotion failed：存活区内存不足，对象进入老年代，而此时老年代也仍然没有内存容纳对象，将导致一次Full GC]

伴随 concurrent mode failed，然后 Full GC：

[concurrent mode failed：CMS回收速度慢，CMS完成前，老年代已被占满，将导致一次Full GC]

频繁 CMS GC：

[内存吃紧，老年代长时间处于较满的状态]

#### 3.2 业务日志

业务日志除了关注系统异常与业务异常之外，还要关注服务执行耗时情况，耗时过长的服务调用如果没有熔断等机制，很容易导致应用性能下降或服务不可用，服务不可用很容易导致雪崩。

![enter image description here](http://images.gitbook.cn/bc5e4e00-eca7-11e7-8bb8-73e9f039edac)

上面是某一接口的调用情况，虽然大部分调用没有发生异常，但是执行耗时相对比较长。

> grep '[0-9]{3,}ms' *.log
>
> 找出调用耗时大于3位数的dao方法，把3改成4就是大于4位数

互联网应用目前几乎采用分布式架构，但不限于服务框架、消息中间件、分布式缓存、分布式存储等等。那么这些应用日志如何聚合起来进行分析呢? 首先，你需要一套分布式链路调用跟踪系统，通过在系统线程上线文间透传 traceId 和 rpcId，将所有日志进行聚合，例如淘宝的鹰眼，spring cloud zipkin等等。

### 4. 案列分析

#### 4.1 CPU 使用率高问题定位

![enter image description here](http://images.gitbook.cn/cc075530-ec9e-11e7-b506-175d00ac319c)

按照2.1定位流程首先排除了系统层面的问题。

利用 top -Hp 6814 输出进程 ID 为 6814 的所有线程 CPU 使用率情况，发现某个线程使用率比较高，有些异常。

```
printf '%x\n' 2304     #输出线程ID的16进制
jstack pid | grep '0x900' -C 30 --color   

```

输出的日志表明该线程一直处于与 mysql I/O 状态：

```
"Thread-6563" daemon prio=10 tid=0x00007fda419a9000 nid=0x900 runnable [0x00007fda2b2b1000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.read(SocketInputStream.java:152)
        at java.net.SocketInputStream.read(SocketInputStream.java:122)
        at com.mysql.jdbc.util.ReadAheadInputStream.fill(ReadAheadInputStream.java:114)
        at com.mysql.jdbc.util.ReadAheadInputStream.readFromUnderlyingStreamIfNecessary(ReadAheadInputStream.java:161)
        at com.mysql.jdbc.util.ReadAheadInputStream.read(ReadAheadInputStream.java:189)
        - locked <0x00000007d03e81d0> (a com.mysql.jdbc.util.ReadAheadInputStream)
        at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:3116)
        at com.mysql.jdbc.MysqlIO.nextRowFast(MysqlIO.java:2224)
        at com.mysql.jdbc.MysqlIO.nextRow(MysqlIO.java:1999)
        at com.mysql.jdbc.MysqlIO.readSingleRowSet(MysqlIO.java:3507)
        at com.mysql.jdbc.MysqlIO.getResultSet(MysqlIO.java:490)
        at com.mysql.jdbc.MysqlIO.readResultsForQueryOrUpdate(MysqlIO.java:3198)
        at com.mysql.jdbc.MysqlIO.readAllResults(MysqlIO.java:2366)
        at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2789)
        at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2818)
        - locked <0x00000007ddcc84e8> (a com.mysql.jdbc.JDBC4Connection)
        at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:2157)
        - locked <0x00000007ddcc84e8> (a com.mysql.jdbc.JDBC4Connection)
        at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:1379)
        - locked <0x00000007ddcc84e8> (a com.mysql.jdbc.JDBC4Connection)
        at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_execute(FilterChainImpl.java:2931)
        at com.alibaba.druid.filter.FilterEventAdapter.preparedStatement_execute(FilterEventAdapter.java:440)
        at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_execute(FilterChainImpl.java:2929)
        at com.alibaba.druid.filter.FilterEventAdapter.preparedStatement_execute(FilterEventAdapter.java:440)
        at com.alibaba.druid.filter.FilterChainImpl.preparedStatement_execute(FilterChainImpl.java:2929)
        at com.alibaba.druid.proxy.jdbc.PreparedStatementProxyImpl.execute(PreparedStatementProxyImpl.java:131)
        at com.alibaba.druid.pool.DruidPooledPreparedStatement.execute(DruidPooledPreparedStatement.java:493)
        at org.apache.ibatis.executor.statement.PreparedStatementHandler.query(PreparedStatementHandler.java:56)
        at org.apache.ibatis.executor.statement.RoutingStatementHandler.query(RoutingStatementHandler.java:70)
        at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:57)
        at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:259)
        at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:132)
        at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:105)
        at cn.com.company.xqy.framework.daoframework.interceptor.PanDaoInterceptor.interceptCachePage(PanDaoInterceptor.java:172)
        at cn.com.company.xqy.framework.daoframework.interceptor.PanDaoInterceptor.intercept(PanDaoInterceptor.java:45)
        at org.apache.ibatis.plugin.Plugin.invoke(Plugin.java:57)
        at com.sun.proxy.$Proxy121.query(Unknown Source)
        at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:104)
        at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:98)
        at cn.com.company.xqy.framework.daoframework.PanSqlSessionTemplate$1.doInSqlSession(PanSqlSessionTemplate.java:111)
        at cn.com.company.xqy.framework.daoframework.PanSqlSessionTemplate.executeWith(PanSqlSessionTemplate.java:454)
        at cn.com.company.xqy.framework.daoframework.PanSqlSessionTemplate.selectList(PanSqlSessionTemplate.java:150)
        at cn.com.company.xqy.framework.daoframework.PanSqlSessionTemplate.selectList(PanSqlSessionTemplate.java:93)
        at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:114)
        at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:58)
        at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:43)
        at com.sun.proxy.$Proxy45.select(Unknown Source)
        at sun.reflect.GeneratedMethodAccessor4958.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
        at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:302)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
        at cn.com.company.xqy.framework.log.interceptor.DalDigestLogInterceptor.invoke(DalDigestLogInterceptor.java:28)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
        at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:208)
        at com.sun.proxy.$Proxy46.select(Unknown Source)
        at cn.com.company.xqy.finance.service.CustomerAccountTitleService.selectAllEnablesNoCache(CustomerAccountTitleService.java:1124)
        at cn.com.company.xqy.finance.service.CustomerAccountTitleService$$FastClassBySpringCGLIB$$363dc6fc.invoke(<generated>)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:204)
        at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:651)
        at cn.com.company
   .xqy.finance.service.CustomerAccountTitleService$$EnhancerBySpringCGLIB$$613b9107.selectAllEnablesNoCache(<generated>)
        at cn.com.company.xqy.finance.service.datafix.BalanceFixService.amountStatistics(BalanceFixService.java:1121)
        at cn.com.company.xqy.finance.service.datafix.BalanceFixService.scan(BalanceFixService.java:346)
        at cn.com.company.xqy.finance.service.datafix.BalanceFixService.access$100(BalanceFixService.java:38)
        at cn.com.company.xqy.finance.service.datafix.BalanceFixService$1.run(BalanceFixService.java:107)
        at java.lang.Thread.run(Thread.java:745)

```

利用 jmap -dump:format=b,file=/usr/local/logs/gc/dump.hprof {pid} 以二进制输出档当前内存的堆情况，然后可以导入 MAT 等工具进行分析。如下图所示，点击 MAT 的支配树可以发现存在某个超大对象数组，实例对象数目多大30多万个。

![enter image description here](http://images.gitbook.cn/d53074c0-ec9e-11e7-a3aa-b3fbc77f24b9)

经过分析发现数组中每一个对象都是核心业务对象，我们的业务系统有一个定时任务线程会访问数据库某张业务表中的所有记录，然后加载至内存然后进行处理因此内存吃紧，导致 CPU 突然飙升。发现该问题后，已对该方案进行重新设计。

#### 