## 内容提要

内容提要：

- 为什么阿里的 Java 开发手册提到“线程池不允许使用 Executors 创建”？
- 为什么很多公司在实际工作中必须通过线程池创建 Tread，不允许手动创建？
- 如何理解 volatile 关键字？
- 能大概说一下线程池的工作原理吗？
- 我面试的岗位是阿里拍卖平台测试开发，岗位要求是熟悉 Linux系统，请问针对这条一般会考察哪些方面的知识？能分析一下前端 Javascript/html/CSS 的常考点吗？
- 平时常用 Xshell 等工具查看后台报文，反倒没有关注 Linux 系统的相关知识，怎样做到以点到面地描述？
- 现在的工作中没有用到线程，对线程的知识停留在会用的基础上，请问该怎样提高对线程的了解及实战运用能力？
- 请问你有没有阅读过框架源码，如何通过阅读框架源码来提高自己写代码、写框架的能力？

------

**问：为什么阿里的 Java 开发手册提到“线程池不允许使用 Executors 创建”？**

**答：**这个问题很好，想要明白这个问题，首先要知道 Excutor 和 Excutors 的区别，以及 Excutors 的用处。Excutor 是一个接口，是很多线程执行的接口父类，大家看源码就能看出来。而 Excutors 是个工具类，正如我们文章里面描述的一样，它帮我们提供了四种便捷地创建线程池的方法。

我们都只知道线程池是要解决两个问题的，一个是高并发的时候创建线程过多会引起资源耗尽。而我们去看 FixedThreadPool 和 SingleThreadPool：

```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

```

工作线程的大小虽然帮我做了最大的限制，但我们去看 LinkedBlockingQueue 的源码会发现，其长度为 Integer.MAX_VALUE。

Queue 的大小经验证的值，一般是 100-500 之间。具体可以根据单个线程的执行时间，所占用的内存 ，然后根据服务器的配置，再合理计算一下区间即可。

------

**问：为什么很多公司在实际工作中必须通过线程池创建 Tread，不允许手动创建？**

**答：**其实可以换个角度去想这个问题，线程池为什么存在？如果离开线程池去手动在 Service 的方法中去创建线程会有什么后果？

答案和上面是一样的，线程资源必须通过线程池供，不允许在应用中自行显式创建线程。使用线程池的好处是减少在创建和销毁线程上所花的时间以及系统资源的开销，可解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。

至于创建和销毁线程上系统资源的开销主要在哪里，其实这个系统资源，咱们在平时是感受不出来的，CPU 创建线程对象的时候，需要用到 Java 的内存模型里的线程栈，这时候内存里面是要进行内存空间的开辟与分配。操作系统里面的资源，狭义上说就是 CPU 的执行碎片时间和内存。

------

**问：如何理解 volatile 关键字？**

**答：** Volatile 的意思是易变的，这是 Java jvm 里面保留的关键字，即被这个关键字修饰的变量的变化都是对其它线程可见的。Volatile 主要是解决多线程内存不可见的问题。对于一写多读，是可以解决变量同步问题。

但是 Volatile 不是线程安全的，大家要铭记于心。如果是写的很频繁，就要小心了，它是会相互覆盖的。可以用 atomic 代替。另外，和 Volatile 同级的另一个东西叫原子操作。如 AtomicInteger，它们都是基本类型，这个是线程安全的。

------

**问：能大概说一下线程池的工作原理吗？**

**答：**这个问题的水平有点高，不过这也是大家常讨论的问题。线程池源码里有两个重要的东西。

1. worker 线程队列，我们称为工作队列；
2. 等待线程队列，这个就是 Int 最大值，我们称为任务等待队列。

课后看一下源码就能发现。真正起作用的是 worker 线程，也就是我们初始的线程池的大小、最大值和最小值及其超时时间。这里会用到线程的状态机制。线程有 wait 和 notify 两个状态。因为 wait 和 notify 是在同一个锁下面才会有用，所以源码中还会用到同步锁。当我们去等待队列任务时，可以想一下线程池的结构图。

![enter image description here](http://images.gitbook.cn/52fc7840-cf8f-11e7-8b2d-0bc5503ba130)

箭头 1 指向的地方是工作线程，箭头 2 指向的地方是队列。

------

**问：我面试的岗位是阿里拍卖平台测试开发，岗位要求是熟悉 Linux系统，请问针对这条一般会考察哪些方面的知识？能分析一下前端 Javascript/html/CSS 的常考点吗？**

**答：**其实任何知识点，无外乎几个方面，第一层是使用程度，第二层是优化和晋级部分，第三层是原理，所以可以想象一下 linux 会考哪些方面，作为基础会用的程度的话，要掌握工作中的基本命令。下面这张图为我们提供了一种思路：

![enter image description here](http://images.gitbook.cn/5bb41150-cf8f-11e7-8b2d-0bc5503ba130)

前端 Javascript/html/CSS 的考点，就是常见的基本使用，及其浏览器的兼容问题，还有就是屏幕适配问题。

------

**问：平时常用 Xshell 等工具查看后台报文，反倒没有关注 Linux 系统的相关知识，怎样做到以点到面地描述？**

**答：**其实这还是属于那些基本命令，具体我们该怎么学习呢？不管学习任何东西，我们首先得学会查看帮助文档。

1. 从工作实际出发，这样最容易记得牢靠；
2. 就是工作中用到哪个命令要看它的 help，帮助手册，再结合自己平时的时候，逐步看懂帮助手册的玩法；
3. 学会举一反三，看看有没有类似的命令；
4. 多看，多向身边的大牛学习。

这样就会以一个点，逐渐覆盖到面上。另外我给大家推荐个东西：[Archlinux 的安装指南](https://wiki.archlinux.org/index.php/Installation_guide)，大家可以从头到尾实践一下，保证你能熟练体会 linux 是什么，知道是怎么回事。

------

**问：“有可能两个线程同时取到最后修改的值” 的风险是什么，不是有缓存一致性协议保证只能写一个吗？**

**答：**我想提问者其实是看到了 Volatile 的介绍，针对它的不安全的角度考虑的。其实回忆一下我们前面讲的 Volatile 就明白了。单个写是没问题，问题是会覆盖。

------

**问：现在的工作中没有用到线程，对线程的知识停留在会用的基础上，请问该怎样提高对线程的了解及实战运用能力？**

**答：**这个问题有点偏实战了，其实大家稍微想想就会有动力去看这个东西了，因为线程是无处不在，CPU 执行的最小单位。哪个程序能离开线程存在？我们的哪些代码里有用到？

肯定也是无处不在，我们只要把我们代码里面用到线程和线程池的地方看明白，基本上大家也能达到第二层次。如线程池、tomcat 里面有设置？为什么一个请求就是一个线程？这些问题想一个就会跟着一个个的接踵而至。分享一句行话：”当你发现自己知道地越多，你就发现你不会的越多“。

另外，多看 Java 的 bin 目录下的工具，只要多看监控平台，很多问题就会想明白。

------

**问：请问您有没有阅读过框架源码，如何通过阅读框架源码来提高自己写代码、写框架的能力？**

**答：**设一个断点，看源码，看调用栈。一次看不明白，就看两次。再不行和你公司大牛交流，多交流才会有火花。分析源码用 idea 这个工具，真的比我们很早之前用的 eclipse 方便太多了。再看不明白，那就得自己勤快一些了。另外，写代码要注重提炼自己的思维。



## 文章实录



### 一、Java-Thread 概念

> 我们想搞懂多线程必须先明白以下几个重要概念。

1. 什么是进程

   是资源分配的最小单位;（资源，包括各种表格、内存空间、磁盘空间） 同一进程中的多条线程将共享该进程中的全部系统资源。

2. 什么是线程

   线程是CPU调度的最小单位。线程只由相关堆栈（系统栈或用户栈）寄存器和线程控制表组成。 而寄存器可被用来存储线程内的局部变量。

3. 什么是并行和并发

   - 并行运行：总线程数<=CPU数量*核心数。
   - 并发运行：总线程数>CPU数量*核心数。 (如：有的操作系统CPU线程切换之间用的时间片轮转进程调度算法)

4. 线程创建的4个方法大家想一下。

### 二、安全和锁

> Java里面如果谈到线程，最核心要搞明白的就是线程安全和线程锁的问题。

#### 1. 何为安全

我先问一下各位小伙伴什么叫线程安全或者是不安全的？思考一下：何为安全？？？思考2分钟。

我总结出来的一个定理啊，一定要铭记：

> Jack定理1：
>
> 离开单例、全局共享变量来谈线程安全问题都是耍流氓。

那么问题来了？单例的一定是线程不安全的吗？答案是否定的，只要你单例的类里面没有全局变量那一定是线程安全的。

所以只有单例模式下共享全局变量的时候才会有线程不安全的问题，这个时候我们就要引入锁的概念了。

#### 2. 锁

经常在工作中听到我们的小伙伴们谈论什么乐观锁，悲观锁，排它锁，共享锁等等，但记住这些只是结果。在我们Java中我认为只有两种锁：隐式锁和显示锁两种实现手段。

**隐式锁: synchronized**

1. 同一个对象锁下面的， synchronized 区域是互斥的
2. 方法锁(默认是当前对象的锁)
3. 代码快锁(性能高于方法锁，可以指定哪个对象的锁)

> Jack定理2：
>
> 离开对象来谈synchronized，也是耍流氓。synchronized一定是加在对象上的切记。

使用方法案例：

```
public synchronized void updateUser(UserInfo userInfo){
    。。。。//共享数据操作
}
public  void updateUser(UserInfo userInfo){
    synchronized(this) {
    。。。。//共享数据操作
    }
}

```

**显示锁：java.util.concurrent.lock**

1. 需要手动关/开
2. 注意自己的代码逻辑不要产生死锁了

使用案例：

```
public void updateUser(UserInfo userInfo){
    Lock lock = new ReentrantLock();
    lock.lock();//加锁
    try {
        。。。。//共享数据操作
    } finally {
        lock.unlock();//释放锁，一定要释放
    }
}

```

#### 3. synchronized与lock区别

- lock更灵活，方法更多，能实现各种锁的场景。
- 性能上如果都指定锁都是一个对象，那基本上没什么差别。
- 默认情况下synchronized锁是当前对象，而lock是不一样的。

### 三、Concurrent包

> java.util.concurrent包是必须要了解的，如果你不知道有这个包的存在就别谈多线程。

我们可以把这个包下面的内容分成四部分

#### 1. 原子性操作类

原子操作(atomic operation)是不需要synchronized，也可以实现多线程的安全，效率要比lock高很多。底层是通过一定的算法将内存中分割了一个独立排它的内存空间，来做单线程操作。目前只有一些AtomicBoolean、AtomicInteger、AtomicLong等一些基本类型。

#### 2. 线程队列

我们学习数据结构的时候都知道有栈和队列两种结构，而Java给我提供了一些线程安全的队列操作的类。

![Java线程安全Queue类结构图](http://images.gitbook.cn/52fa1b70-c785-11e7-ad0d-af5dea044937)

而其中关键的几个类，我们大概介绍一下：

![enter image description here](http://images.gitbook.cn/5e255720-c786-11e7-ad0d-af5dea044937)

- BlockingQueue很好的解决了多线程中高效安全“传输”数据的问题；基于java.util.Queue的基础上做了一些线程安全的封装；
- ArrayBlockingQueue 基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定长数组，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue内部还保存着两个整形变量，分别标识着队列的头部和尾部在数组中的位置。
- LinkedBlockingQueue基于链表的阻塞队列，同ArrayListBlockingQueue类似，其内部也维持着一个数据缓冲队列。
- DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。使用场景：DelayQueue使用场景较少，但都相当巧妙，常见的例子比如使用一个DelayQueue来管理一个超时未响应的连接队列。
- PriorityBlockingQueue基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），但需要注意的是PriorityBlockingQueue并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。
- SynchronousQueue一种无缓冲的等待队列，同步队列没有任何内部容量，甚至连一个队列的容量都没有；其中每个 put 必须等待一个 take，反之亦然。无锁的机制实现（可想而知高并发的时候性能肯定是最高的）。

> 关于队列只介绍个大概，大家知道有这么回事，具体使用可以查询相关的API文档。为什么要提一下呢，因为我们在说明线程池的时候有用到安全队列。

#### 3. 线程阀

线程阀：控制线程的开(开始)与关(结束)。如果用Queue来管理线程的队列即开始，那么用线程阀管理整体线程的调配工作，即线程结束之后的开与关。我们这里大概介绍4个类：

1. CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。
2. CyclicBarrier是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。
3. Semaphore：一个计数信号量。从概念上讲，信号量维护了一个许可集合。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。就像一个排队进入上海博物馆一样，放几个人等一下，有几个人走了然后再放几个人进入，像是一种排队机制。
4. Future->FutureTask：一般FutureTask多用与耗时的计算，主线程再完成自己的任务后，再去获取结果。只有在计算完成时获取，否则会一直阻塞直到任务完成状态。

> 具体语法和使用可以查询相关文档。

#### 4. Java提供的线程安排工具类

```
java.util.concurrent.ConcurrentHashMap   
java.util.concurrent.ConcurrentLinkedQueue   
java.util.concurrent.ConcurrentMap   
java.util.concurrent.ConcurrentNavigableMap   
java.util.concurrent.ConcurrentSkipListMap   
java.util.concurrent.ConcurrentSkipListSet   

```

........等等基于lock的算法实现

#### 5. volatile关键字

我们通过查看源码，会发现java的另外一个关键字volatile，线程在每次使用变量的时候，都会读取变量修改后的最的值。（其实是有风险的，并行情况下不一定正确，有可能两个线程同时取到最后修改的值）

### 四、线程池

#### 1. 线程池要解决的问题：

我们掌握线程池必须要明白线程池要接解决的两个问题：

- 解决频繁创建线程所产生的开销。减少在创建和销毁线程上所花的时间以及系统资源的开销。
- 解决无限制的创建线程引起的系统崩溃。如不使用线程池，有可能造成系统创建大量线程而导致消耗完系统内存以及”过度切换” 。

#### 2. Executors给我们提供的四种创建线程池的方法

**创建一个可重用固定线程数的线程池**

```
ExecutorService pool = Executors.newFixedThreadPool(5); 

```

newFixedThreadPool的参数指定了可以运行的线程的最大数目，超过这个数目的线程加进去以后，不会立马运行，会做队列等待。其次，加入线程池的线程属于托管状态，线程的运行不受加入顺序的影响

**单任务线程池**

```
ExecutorService pool = Executors.newSingleThreadExecutor();

```

一个一个执行，这种基本上很少用到。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

**可变尺寸的线程池**

```
ExecutorService pool = Executors.newCachedThreadPool();

```

如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

**定时以及周期性执行任务的线程池**

```
ScheduledThreadPoolExecutor exec = Executors.ScheduledThreadPoolExecutor(1);
        exec.scheduleAtFixedRate(new Runnable() {
                      publicvoid run() {
.....////每隔一段时间就触发的线程内容
                      }
                  }, 1000, 5000,TimeUnit.MILLISECONDS);

```

#### 3. 自定义线程池

我们查看Executors的源码发现底层都是调用ThreadPoolExecutor来实现的，里面有几个重要参数我们一定要记一下：

```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }   
    //ThreadPoolExecutor构造器
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

```

- corePoolSize：池中所保存的线程数，包括空闲线程。
- maximumPoolSize：池中允许的最大线程数。
- keepAliveTime: 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间。
- unit: keepAliveTime参数的时间单位。
- BlockingQueue: 执行前用于保持任务的队列。此队列仅保持由execute方法提交的Runnable任务。常见的三种队列：
  - 直接提交。SynchronousQueue。
  - 无界队列。使用无界队列（例如，不具有预定义容量的LinkedBlockingQueue）
  - 有界队列。当使用有限maximumPoolSizes时，有界队列（如ArrayBlockingQueue）有助于防止资源耗尽。
- ThreadFactory： 执行程序创建新线程时使用的工厂。默认情况下为Executors.defaultThreadFactory()：我们可以采用自定义的ThreadFactory工厂，增加对线程创建与销毁等更多的控制。
- RejectedExecutionHandler： (拒绝策略)由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。
  - AbortPolicy(默认)：这种策略直接抛出异常，丢弃任务。
  - DiscardPolicy：不能执行的任务将被删除；这种策略和AbortPolicy几乎一样，也是丢弃任务，只不过他不抛出异常。
  - DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部的任务被删除，然后重试执行程序（如果再次失败，则重复此过程）。
  - CallerRunsPolicy： 使用此策略，如果添加到线程池失败，那么主线程会自己去执行该任务，不会等待线程池中的线程去执行。就像是个急脾气的人，我等不到别人来做这件事就干脆自己干。
  - 当然也可以自定义。

> Jack定理3：
>
> 离开全局和单例谈论线程池那也是耍流氓。工作中看到有人把线程池写在方法里面的局部变量，那有用吗？

#### 4. 线程的监控和分析方法

**VisualVM的使用**

VisualVM是JDK的一个集成的分析工具，自从JDK 6 Update 7以后已经作为Sun的JDK的一部分。VisualVM可以做的：监控应用程序的性能和内存占用情况、监控应用程序的线程、进行线程转储(Thread Dump)或堆转储(Heap Dump)、跟踪内存泄漏、监控垃圾回收器、执行内存和CPU分析，保存快照以便脱机分析应用程序。

![VisualVM的使用](http://images.gitbook.cn/8e2826a0-c795-11e7-ad0d-af5dea044937)

**Jconsole的使用**

JConsole 是一个内置 Java 性能分析器，可以从命令行或在 GUI shell 中运行。

![Jconsole的使用](http://images.gitbook.cn/a84295c0-c795-11e7-ad0d-af5dea044937)

**在Java Visualvm工具里面安装JTA插件**

![JTA插件的使用](http://images.gitbook.cn/c6035090-c795-11e7-ad0d-af5dea044937)

**利用linux的top&jstack命令**

例如：top先找到Java进程，top -p 8442 -H 找到哪个线程，jstack 8442> ./8442_dump.txt输出thread的demp文件。

在实际生产环境，一般我们都是自己公司的监控平台的，只需要到各大监控平台开thread即可，内容基本上一样。

> Jack定理4：
>
> 任何Java执行的类和相关信息都在堆栈里面，就是我们如何想办法看到他们的问题，万变不离其宗。

### 五、线程和线程池工作中的应用场景：

1. ervlet 我们java开发最基本的东西，其启动的时候其实是开辟了一个main线程的。而其中servlet类是单例的所以它是线程不安全的，但是在没有共享全局变量的情况，而reqest和response是一个请求是一个实例，而其本身的数据设计又是线程安全的。
2. Tomcat Servlet的容器tomcat其实是对线程的线程池做了控制的。提高请求处理效率和避免请求太多把容器弄挂。
3. Spring 默认加载bean的方式是单例的，所以其是线程不安全的。
4. 数据库连接池，其实也是多线程。
5. nginx 前端网关请求，也是利用了线程池的原理。
6. 而我们的客户端ios，android其实也都是有主线程和子线程的说法，如果你能很好的将器里面的线程掌握基本上此种客户端开发就能掌握一半。

> Jack定理5：
>
> 线程无处不在，线程池也无处不在，只不过是换不同的马甲，不通形式存在就看你知道不知道。
>
> Jack一句话总结：
>
> Java线程是围绕着java的进程的共享内存的管理和数据访问，而围绕线程本身的管理产生了通讯，争抢和队列管理的线程池。

### 六、开放性问题?

1. 线程安全和数据库数据线程安全是一回事吗？
2. 我们实际工作中服务的最大并发量是多少？为什么？
3. 除了数据库连接池，我们还有在哪些地方用过线程池？
4. 你面试的时候多线程你都被问到了哪些问题？





