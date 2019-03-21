## 性能优化之 JVM 高级特性

![enter image description here](http://images.gitbook.cn/21814bd0-7de3-11e8-b87a-bd0081979594)

### 1、JVM体系结构

![enter image description here](http://images.gitbook.cn/3369d140-739e-11e8-be2f-59116f069d39)

#### 线程共享内存

可以被所有线程共享的区域，包括堆区、方法区、运行时常量池。

##### **1.1 堆（Heap）**

大多数时候，Java 堆是 Java 虚拟机管理的内存里最大的一块，所有的对象实例和数组都要在堆上分配内存空间，Java 对象可以分为两类，一类是快速创建快速消亡的，另一类是长期使用的。所以针对这种情况大多收集器都是基于分代收集算法进行回收。

Java 的堆可以分为新生代（Young Generation）和老年代（Old Generation），而新生代（Young Generation）又可以分为 Eden Space 空间 (伊甸园区)、From Survivor 空间（From 生存区）、To Survivor 空间（To 生存区）。

Java 堆是一块共享的区域，会出现线程安全的问题，而操作共享区域就需要锁和同步，通过 `- Xms` 设置堆的最小值，堆内存越小越容易发生内存不够用的情况而触犯 Full GC（对新生代、老年代、永久代进行垃圾回收）。官方推荐新生代大小占整个堆大小的 3/8，通过 `- Xmx` 设置堆的最大值，堆内存超过此值会发抛出 OutOfMemoryError 异常：

![enter image description here](http://images.gitbook.cn/c42866d0-73bf-11e8-9e76-835574e7e257)

##### **1.2 方法区（Method Area）**

方法区（Method Area）在 HotSpot 虚拟机上可以看作是永久代（Permanent Generation），对于其他虚拟机（JRockit 、J9 等）来说是不存在永久代的。方法区也是所有线程共享的区域，主要存储被虚拟机加载的类信息、常量、静态变量，堆存储对象数据，方法区存储静态信息。

方法区不像 Java 堆区那样经常发生垃圾回收，但不表示不会发生。永久代的大小跟新生代、老年代比都是很小的，通过设置 `- XX:MaxPermSize` 来指定最大内存，方法区需要的内存超过此值会抛出 OutOfMemoryError 异常。

##### **1.3 运行时常量池（Runtime Constant Pool）**

Java 通过类加载机制可以把字节码文件中的常量池表加载进运行时常量池，而我们也可使用 String 类的 intern() 方法在运行期将新的常量放入池中，运行时常量池是方法区的一部分，在 JDK1.7 的 HotSpot 中，已经把原本放在方法区的常量池移出来了。

#### 线程私有内存

只允许被所属的线程私自访问的内存区，包括 PC 寄存器、Java 栈和本地方法栈。

##### **1.4 栈（Java Stack）**

Java Stack 描述的是 Java 方法执行时的内存模型，每个方法执行时都会创建一个栈帧（Stack Frame），栈帧包含局部变量表（存放编译期间的各种基本数据类型，对象引用等信息）、操作数栈、动态链接、方法出口等数据。一个线程运行时会分配栈空间，每个线程的栈空间数据是相互隔离的，所以栈是私有的，堆是共享的，一个线程执行多个方法，会入栈出栈多个栈帧（多个方法），栈是先进后出的数据结构，最先入栈的栈帧，最后出栈，可以通过 `-Xss` 设置每个线程栈的大小，越小，能创建的线程数就越多，但并不是可以无限的，在一个进程里（JVM 进程）能生成的线程数最多不超过五千

![enter image description here](http://images.gitbook.cn/d676b180-73c3-11e8-be2f-59116f069d39)

##### **1.5 本地方法栈（Native Stack）**

虚拟机栈（Java Stack）为执行 Java 方法（就是字节码）服务，而本地方法栈（Native Stack）则为 Native 方法（比如用 C/C++ 编写的代码）服务，其他方面都很类似。

##### **1.6 PC 寄存器（程序计数器）**

JVM 字节码解析器通过改变 PC 寄存器的值来明确下一条需要执行的字节码指令，每个线程都会分配一个独立的 PC 寄存器。

### 2、JVM 垃圾回收算法

JVM 垃圾收集算法不同虚拟机的具体实现会不一样，这里先讲解几种经典的垃圾收集算法的思想，后面再以使用得最广泛的 HotSpot 虚拟机为例讲解具体的垃圾收集器算法。

#### 2.1 引用计数法

给每个对象维护一个引用计数器，每当被引用一次就加 1，每当引用失效一次就减 1，引用计数器为 0，表明当前对象没有被任何对象引用，则可以当作垃圾回收。但是当 A 对象和 B 对象相互引用对方的时候，大家的计数器值都不为 0，而如果对象 A 和对象 B 都已经不被外部引用，就是说两个无用的对象因为相互引用而无法进行垃圾回收。这就是循环引用的缺陷，故现在 JVM 虚拟机大多不采用这种方式做垃圾回收。

#### 2.2 根搜索算法（Tracing）

1. 复制 （Coping）
2. 标记-清除 （Mark-Sweep）
3. 标记-压缩（Mark-Compact）
4. 分代收集算法（Generational Collection）

根搜索算法从那些被称为 GC Roots 的对象（虚拟机栈中引用的对象、方法区中静态属性引用的对象、方法区中常量引用的对象、本地方法栈 JNI 引用的对象）作为起始节点，从这些节点向下搜索，搜索所形成的路径叫引用链。当一个对象到 GC Roots 的所有对象都没有一条引用链，则说明该对象不可用，所以根搜索算法又叫可达性算法，GC Roots 到该对象不可达则表明该对象不可用，表明该对象可回收。

根搜索算法有四种，其中复制算法应用在新生代。

##### **2.2.1 复制算法**

![enter image description here](http://images.gitbook.cn/3f286180-748a-11e8-9ce0-8f87da4d301a)

复制算法将内存划分相等的两块，当一块内存用完了，将还存活的对象移动到另一块内存，将之前使用的内存一次清理，新生代的内存空间通常都是所有代里最大的，适用复制算法，实际上垃圾回收时是把 Eden Space 和 From Survivor 上还存活的对象复制到 To Survivor，而不需要按照 1:1 的比例来划分。

通常 Eden Space:From Survivor:To Survivor = 8:1:1，如果出现状况 To Survivor 空间不足以容纳复制过来的还存活的对象，那通过分配担保机制，这些对象可直接进入老年代，然后下一次垃圾回收发生时 From Survivor 和 To Survivor 交换身份，内存直接从 To Survivor 分配，回收到 From Survivor。

- 优点：没有标记和清除的过程，效率高，没有内存碎片，可以利用 Bump-the-pointer（指针碰撞）技术实现快速内存分配，因为已用和未用的内存各自一边，内存分布规整有序，当新对象分配时就可以通过修改指针偏移量将其分配在第一个空闲的内存位置上，从而快速分配内存，否则只能使用空闲列表（Free List）方式分配内存，如下面要讲的标记-清除（Mark-Sweep）垃圾回收算法。
- 缺点：开辟专门的空间存放存活对象，占用更多的内存。

##### **2.2.2 标记-清除（Mark-Sweep）**

上面的内存图示是为了理解，其实内存是线性的。

从根集合开始扫描，对存活动对象进行标记，然后重新扫描整个内存空间清除未被标记的对象，优点是不需要额外的空间，缺点是重复扫描，性能低，而且产生内存碎片。

![enter image description here](http://images.gitbook.cn/901f6930-7912-11e8-afa8-8db2b8bc59f2)

![enter image description here](http://images.gitbook.cn/0d2f8e00-7913-11e8-97d2-5b3665c292ea)

##### **2.2.3 标记-压缩（Mark-Compact）**

一样是从从根集合开始扫描，对存活动对象进行标记，然后重新扫描整个内存空间，并往一个方向移动存活对象，虽然移动对象的消耗时间，但不产生内存碎片，可以通过 Bump-the-pointer（指针碰撞）快速分配内存。

![enter image description here](http://images.gitbook.cn/95141160-7913-11e8-ae3a-c9b56e7fe402)

![enter image description here](http://images.gitbook.cn/df8c8d30-7913-11e8-afa8-8db2b8bc59f2)

##### **2.2.4 标记-清除-压缩**

是“标记-清除”和“标记-压缩”算法的结合，唯一的不同是要等“标记-清除”多次以后，也就是多次垃圾回收进行以后才进行移动对象（压缩），以避免每次 GC（垃圾回收）后都压缩一次，降低了移动对象的耗时。

JVM 几种经典的垃圾回收算法已经讲完，下面直接进入 HotSpot 虚拟机的讲解。

#### 2.3 HotSpot 内存分配

##### **2.3.1 内存分配实例**

```
public class Earth {
    String name;
    // 单位：亿年
    int age;
    String size;

    public Earth(String name, int age, String size) {
        super();
        this.name = name;
        this.age = age;
        this.size = size;
    }

    Earth e = new Earth("地球", 46, "1.0832×10^12立方千米");

}

```

当我们去 new 一个对象时，首先会在栈空间为对象的引用 e 分配内存，这是声明 Earth e，但由于 Earth 还是一个空对象，无法使用，不指向任何实体，接着 new Earth 会在堆开辟内存给成员变量 name、age、size，并初始化为各个数据类型的默认值，然后才是初始化为自定义的赋值，接着调用构造函数通过参数列表 (“地球”, 46, “1.0832×10^12 立方千米”) 为成员变量再赋值，最后返回对象的引用（首地址）给变量 e。

```
    Earth e1 = new Earth("地球", 46, "1.0832×10^12立方千米");
    Earth e2 = new Earth("地球", 46, "1.0832×10^12立方千米");
    Earth e3 = new Earth("地球", 46, "1.0832×10^12立方千米");

```

就算创建多个对象，在堆中还是一个实例，在栈中 e1、e2、e3 共同指向一个实例，而由于对象创建都是频繁而且短生命周期，故一般对象被分配在堆的新生代的 Eden 区。而堆空间是非线程安全的，是线程共享的，为对象分配内存时就要保证原子性，防止产生脏数据，这是会消耗性能的。为此 JVM 做了优化，优先为加载完成对类在 TLAB（Thread Local Allocation，本地线程分配缓冲区）中为对象实例分配内存空间。

TLAB 是在堆中 Eden 区里开辟的空间，但却是一块线程私有区域，并不是所有对象都能在这成功分配，但在 TLAB 分配是个优选项，为了优化内存分配，还可以使用 JVM 的堆外内存存放对象实例，堆空间不是唯一 一个可以存放对象实例的地方，当一个对象作用域在方法体内，但随时间推移，一旦其引用被方法体外的成员变量引用时，就发生了逃逸；反过来，如果作用域还是局限于方法体内，JVM 就可以为对象在栈帧里分配空间，对象分配在方法的栈帧上，随着方法创建而创建，随着方法退出而消亡，无需垃圾回收。

##### **2.3.2 运行时常量池**

我们再来看一下跟运行时常量池相关的内存分配的例子（面试常客）：

```
public class StringDemo {
    public static void stingDemo() {
        // 池化的思想，把共享的数据放入字符串池中，以减少频繁创建销毁对象的开销
        // 引用s1被放入栈中，字符串字面量"123"如果存在于字符串池中，返回其引用给s1，否则，在池中新建字符串
        String s1 = "123";
        // 池中已经存在"123"，不会再创建，直接拿到引用（地址值）
        String s2 = "123";
        // 就是new对象的创建过程，不会去池查找，直接在堆开辟空间存放实例对象，返回对象地址给s3
        String s3 = new String("123");
        // String类本身是被final修饰的，final修饰会保证s4的值不变，也就是s4=123，而这个字符串字面量直接可以从池中拿到，不需要创建对象
        String s4 = "1" + "23";

        String str = "1";
        // 由于被final修饰，str的值"1"不能改变，s5的值也不能变，其值是str+"23"创建对象所返回的地址（不是指对象内容
        // 123不许变），所以这里会新建一个对象
        String s5 = str + "23";
        // 两个基本类型用==比较，比较的是两个基本类型的值是否相等
        // 一个包装类型和一个基本类型用==比较，比较包装类型的值和基本类型的值是否相等
        // 两个包装类型用==比较，比较两个对象返回的地址是否相等
        // 两个包装类型用equals比较，比较两个对象是否相等,
        // 包括两个对象类的类型是否相同，对象里的值是否完全相同，对象的hashcode是否相同，不比较地址，所以hashcode相同，对象不一定相等，对象相等，hashcode一定相同
        // s1==s2:true
        System.out.println("s1==s2:" + (s1 == s2));
        // s1==s3:false
        System.out.println("s1==s3:" + (s1 == s3));
        // s1.equals(s3):true
        // Sting类已经帮我们覆写了Object的equals方法，使得equals的比较正常，
        // 如果没覆写，底层还是用==做的比较，我们自定义对象要用equals比较的前提是记得覆写equals方法
        System.out.println("s1.equals(s3):" + (s1.equals(s3)));
        // s1=s4:true
        System.out.println("s1=s4:" + (s1 == s4));
    }

    public static void main(String[] args) {
        StringDemo.stingDemo();
    }

}

public class TestInteger {

    public static void main(String[] args) {

        int i1 = 129;
        // java在编译的时候,会变成 Integer i2 = Integer.valueOf(129)
        Integer i2 = 129;
        // int和integer(无论是否new出来的)比，都为true，因为会把Integer自动拆箱为int再去比较
        System.out.println("int i1 = 129 == Integer i2= 129 :" + (i1 == i2));

        Integer i3 = new Integer(129);
        // Integer与new Integer不会相等。不会经历拆箱过程，i3与i2指向是两个不同的地址，为false
        System.out.println("Integer i2= 129 == Integer i3 = new Integer(129) :" + (i2 == i3));

        Integer i4 = new Integer(129);
        // 两个都是new出来的,开辟不同的内存空间，都为false
        System.out.println("Integer i3 = new Integer(129) == Integer i4 =new Integer(129) :" + (i3 == i4));

        Integer i5 = 129;

        /*
         * Integer i2 = 129 会被编译成Integer.valueOf(129) 而valueOf的源码如下 public static
         * Integer valueOf(int i) { 
         * assert IntegerCache.high >= 127;
         * 如果值在-128到127之间，直接从缓存取值，不需要重新创建 
         * if (i >= IntegerCache.low && i <=IntegerCache.high) 
         * return IntegerCache.cache[i + (-IntegerCache.low)]; 
         * return new Integer(i); 
         * }
         */

        // 两个都是非new出来的Integer，如果数在-128到127之间，则是true,否则为false，超过范围不会在常量池里取，会重新创建两个Integer，==比较Integer的值，即是地址，肯定false
        System.out.println("Integer i2= 129 == Integer i5 = 129 :" + (i2 == i5));
        i2 = 127;
        i5 = 127;
        // 在-128到127之间，从常量池取同一个引用给Integer，肯定是true
        System.out.println("Integer i2= 127 == Integer i5 = 127 :" + (i2 == i5));

    }

}

```

HotSpot 内存管理里，新生代 80% 的对象生命周期较短，GC 频率高，适合采用效率较高的复制算法，经历了多次 GC 仍然存活的对象或者一些超过设定值大小的对象会分配到老年代，老年代 GC 频率较低，适合使用“标记 - 清除 - 压缩”这种综合的算法。

回收算法还有回收方式的不同，串行回收（Serial），是指就算有多个 CPU 核，某一时刻也只能有一个 CPU 核可以执行垃圾回收线程，此时用户线程会被挂起处于暂停状态，只有等回收线程执行完毕，用户线程才能继续执行，也就是会产生所谓的 Stop-the-world，JVM 短时间内卡顿不会工作。

并行回收是指某一时刻可以由多个 CPU 核同时执行各自的垃圾回收线程，不过一样会出现 Stop-the-world，而并发回收是指用户线程和垃圾回收线程交替执行，大大缩短 Stop-the-world 的停顿时间。现在大型项目动辄使用上百 G 的内存，内存越大，回收时间越久，而 Stop-the-world 的卡顿时间也会越久，目前还没有算法可以做到零停顿的。

算法的思想讲完了，下面就讲垃圾收集算法的具体实现垃圾收集器。

![enter image description here](http://images.gitbook.cn/8b03dc70-792f-11e8-9353-3d7605954bd0)

上面红色横线的地方就是安全点，用户线程执行时，要到达了安全点，才能暂停，让回收线程执行；当触发回收线程执行时，不会直接中断用户线程，而是设置一个标志位，让用户线程轮询。发现为中断标志时就运行到最近的安全点再将自己挂起让 CPU 执行回收线程，但如果此时用户线程处于 Waiting 或者 Blocked 状态，无法轮询标志位，就会造成回收线程长时间无法运行的情况。

为此引入了安全区，安全区就是引用关系不会发生变化的代码，在这段代码的任何地方发起 GC 都是安全的。所以当用户线程进入到安全区，恰好这时回收线程要执行就会直接中断用户线程，用户线程离开安全区时，只需检查回收线程是否已经完成，如果完成则可以离开，否则等待直到 GC 完毕。

### 3、HotSpot 垃圾回收器

#### 3.1 新生代可用的垃圾回收器

Serial Coping（串行复制），Parallel Scavenge（并行复制），ParNew（并发复制）这三种回收器都是基于复制算法，复制 young eden 和 young from 中还存活的对象到 young to，或者根据设定对象的大小和 GC 次数直接晋升到 old，清空 young eden 和 young from 中的垃圾，下一次 GC 发生交换 young from 和 young to，只可使用于新生代，是在 young eden 内存空间不足以分配给对象时触发 Minor GC（新生代垃圾回收）。

#### 3.2 老年代可用垃圾回收器

- Serial Old （串行标记-清理-压缩）
- Parallel Old（并行标记-压缩）
- CMS Concurrent Mark-Sweep（并发标记清除）

#### 3.3 垃圾收集器的组合

- Serial Coping（串行复制）

适合客户端工作，不适合在服务器运行，针对单 CPU，小新生代，不太在乎暂停时间的应用，可通过 `- XX:+UseSerialGC` 手动指定新生代使用 Serial Coping（串行复制）收集器，老年代使用 Serial Old （串行标记 - 清理 - 压缩）收集器执行内存回收。

- ParNew（并发复制）

是 Serial Coping（串行复制）的多线程版本，在多 CPU 核情况下可以提高收集能力，但如果是单 CPU 条件下，还要来回切换任务，不一定比 Serial Coping（串行复制）收集能力强，通过 `- XX:+UseParNewGC` 手动指定新生代使用 ParNew（并发复制）收集器，老年代使用 Serial Old （串行标记 - 清理 - 压缩）收集器执行内存回收。

- Parallel Scavenge（并行复制）

跟 ParNew（并发复制）相比更注重于吞吐量而不是低延迟，如果吞吐量优先，必然会降低 GC 的频次，也就造成 GC 回收垃圾量更多、时间更长。如果低延迟优先，为了降低每次的暂停时间，就得高频的回收，这频繁的回收又会导致吞吐量的下降，所以吐吞量和低延迟是对矛盾体，适合多 CPU、高 IO 密集操作、高计算消耗的应用，通过 `XX:+UseParallelGC` 手动指定新生代使用 Parallel Scavenge（并行复制）收集器，老年代使用 Serial Old （串行标记 - 清理 - 压缩）收集器执行内存回收。

- Serial Old （串行标记 - 清理 - 压缩）

单线程串行回收，停顿时间长，可以使用 `- XX:+PrintGCApplicationStoppedTime` 查看暂停时间，适合客户端使用，不会产生内存碎片

- Parallel Old（并行标记 - 压缩）

根据 GC 线程数划分若干区域（Region），并行做标记，重新扫描，定位到需要压缩的 Region，统计 Region 里所有存活对象的下次要移动的目的地地址，然后并行的往一端压缩，不产生内存碎片，整理后的空闲区域是连续的，通过 `- XX:+UseParallelOldGC` 手动指定新生代使用 Parallel Scavenge（并行复制）收集器，老年代使用 Parallel Old（并行标记 - 压缩）收集器执行内存回收。

- CMS Concurrent Mark-Sweep（并发标记清除）

第一阶段是初始标记，需要 Stop-the-world，这阶段标记出那些与根对象集合所连接的不可达的对象，标记完就会被暂停的应用线程；

第二阶段是并发标记，这阶段是应用线程和回收线程交替执行，把第一步标记为不可达的对象标记为垃圾对象，由于是交替进行，一开始被标记为垃圾的对象，后面应用线程可能更改对象的引用关系导致标记错误；

所以第三阶段重新标记，需要 Stop-the-world，修正上个阶段由于对象引用或者新对象创建导致的标记错误，这阶段只有回收线程执行，确保修正的正确性。

经过三个阶段的标记，第四个阶段会并发的清除无有的对象释放内存，这阶段是应用线程和回收线程交替执行，如果用户应用线程产生了新的垃圾（浮动垃圾），只能留到下次 GC 进行回收，极端情况如果产生的新的垃圾，而老年代的预留空间又不够，就会产生 Concurrent Mode Failure，这个时候只能通过后备的 Serial Old （串行标记 - 清理 - 压缩）来进行垃圾回收。

又因为 CMS 并没有用到压缩算法，回收后会产生内存碎片，为新对象分配内存无法使用 Bump-the-pointer（指针碰撞）技术实现快速内存分配，只能使用空闲列表（Free List ：JVM 会维护一张可用内存地址的列表，当需要分配空间，就从列表搜索一段和对象大小一样的连续内存块用于存放要生成的对象实例）方式分配内存。

但也可以通过 `- XX:CMSFullGCsBeforeCompaction`，用于指定经过多少次 Full GC 后对内存碎片整理压缩，由于内存碎片不是并发执行，会带来更长的停顿时间，通过 `- XX:+UseConcMarkSweepGC` 设定新生代使用 ParNew（并发复制）收集器，老年代使用 CMS Concurrent Mark-Sweep（并发标记清除）收集器执行内存回收，当出现浮动垃圾导致 Concurrent Mode Failure 或者新对象分配内存失败时，通过备用组合新生代使用 ParNew（并发复制）收集器，老年代使用 Serial Old （串行标记 - 清理 - 压缩）收集器执行内存回收，适用于要求暂停时间短，追求快速响应的应用，如互联网应用。

**JVM回收需要注意的点：**

> - 在执行 Minor GC 的时候，JVM 会检查老年代中最大连续可用空间是否大于了当前新生代所有对象的总大小，如果大于，则直接执行 Minor GC；如果小于了，JVM 会检查是否开启了空间分配担保机制；如果开启了，则 JVM 会检查老年代中最大连续可用空间是否大于了历次晋升到老年代中的平均大小；如果大于则会执行 Minor GC，如果小于则执行改为执行 Full GC，如果没有开启则直接改为执行 Full GC。
> - 当老年代（Major GC）和永久代发生 GC 时，除了 CMS 外都会触发 Full GC，Full GC 就是先按新生代 GC 方式进行 Minor GC，再按照老年代的配置进行 Major GC，包含对老年代和永久代进行 GC，若 JVM 估计 Minor GC 会产生晋升失败，则会采用 Major GC 的配置进行 Full GC。
> - 如果 Minor GC 执行失败则会执行 Full GC。
> - 吞吐量：应用运行时间/总时间，暂停时间：每次 GC 造成的暂停。

- 分区分代增量式式收集器：G1（Garbage-First）收集器

传统的分代收集也提供了并发收集，但最致命的是分代收集把整个堆空间划分成固定间隔的内存块，每次收集都很容易触发 Full GC 从而扫描整个堆空间，这会拖慢应用，而且要对整个堆空间都做内存碎片整理会很麻烦。

而增量式的收集方式是一种新的收集思想，增量收集把堆空间划分成一系列的小内存块（内存块大小可配置），使用时只使用部分内存块，等这部分内存块空间不足时，再把存活对象移动到未被使用过的内存块，避免整个堆用完了再 Full GC，可以一边使用内存一边收集垃圾。

G1 收集器将整个 Java 堆区分成约 2048 大小相同的 Region 块（分新生 Region 块、幸存 Region 块、老年 Region 块），Region 块大小在 1MB 到 32MB 之间，每个对象会被分配到 Region 块里，既可以被块内对象引用也可以被块外对象引用，在判断对象是否存活时，为了避免全堆扫描或者遗漏，是通过 Remembered Set 来检查 Reference 引用的对象是否存在不同的 Region 块中的。G1 在收集垃圾时，会对各个 Region 块的回收价值和成本做排序，根据用户配置的期望停顿时间来进行回收。

![enter image description here](http://images.gitbook.cn/5bfc9510-794e-11e8-ae3a-c9b56e7fe402)

G1 收集器与 CMS 收集器执行过程类似。初始标记阶段，Stop-the-World，标记 GC Roots 可直接访问到的对象，为下一个阶段并发标记时，和应用线程交替执行时，有正确可有的 Region 来分配新建对象，并发标记阶段识别上个阶段标记对象的下层对象的活跃状态，找出存活的对象，也就是标记 GC Roots 可达对象；

最终标记阶段，Stop-the-World，修正上次线程交替执行产生的变动；

清除复制阶段，Stop-the-World，这阶段并不是最终标记执行完了就一定执行，毕竟是要 Stop-the-World，为了达到准实时（可配置在 M 毫秒内最多只占用 N 毫秒的时间进行垃圾回收）会根据用户配置的 GC 时间去决定是否做清除。

还有，因为清除复制阶段使用的是复制算法，每次清理都必须保证”to space” 空间是足够的（将存活的对象复制到未使用的 Region 块），所以只有已用空间达到了（1-h）*堆大小（h 是 G1 定义的一个堆空间的百分比阈值，是个固定值）才执行清除，把存活的对象往一个方向移动到”to space” 并整理内存，不会产生内存碎片。

接着把”Eden space” “from space” 的垃圾对象清理，根据维护的优先列表，优先回收价值最大的 Region，通过五个阶段完成垃圾收集，可以通过设定 - XX:UseG1GC 在整个 Java 堆使用 G1 进行垃圾回收，G1 适合高吞吐、低延时、大堆空间的应用。

### 4、垃圾回收器配置和 GC 日志分析

#### 4.1 堆典型配置：

32 位操作系统限制堆大小介于 1.5G 到 2G，64 位操作系统无限制，同时系统可用虚拟内存，可用物理内存都会限制最大堆的配置。

堆空间分配典型设置：

- `-Xms`：初始堆大小
- `-Xmx`：最大堆大小
- `-XX:NewSize=n`：设置年轻代大小
- `-XX:NewRatio=n`：设置年轻代和年老代的比值。如 n 为 2，表示年轻代与年老代比值为 1:2，年轻代占整个年轻代年老代和的 1/3
- `-XX:SurvivorRatio=n`：年轻代中 Eden 区与两个 Survivor 区的比值。注意 Survivor 区有两个。如 n 为 2，表示 Eden:Survivor=1:2，一个 Survivor 区占整个年轻代的 1/4
- `-XX:MaxPermSize=`：设置持久代大小

```
-Xmx5120m –Xms5120m -Xmn2g -Xss128k -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m -XX:MaxTenuringThreshold=0

```

- `-Xmn2g`：设置年轻代大小为 2G。整个 JVM 内存大小 = 年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为 64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun 官方推荐配置为整个堆的 3/8。
- `-Xss128k`：设置每个线程的栈大小。JDK5.0 以后每个线程堆栈大小为 1M，以前每个线程堆栈大小为 256K。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的在 3000~5000 间。
- `-XX:SurvivorRatio=4`：设置年轻代中 Eden 区与 Survivor 区的大小比值。
- `-XX:MaxPermSize=16m`：设置持久代大小为 16m。
- `-XX:MaxTenuringThreshold=0`：设置年轻代最大年龄。如果设置为 0 的话，则年轻代对象不经过 Survivor 区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在 Survivor 区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概率。
- `-XX:+DisableExplicitGC`：这个将会忽略手动调用 GC 的代码使得 System.gc() 的调用就会变成一个空调用，完全不会触发任何 GC

#### 4.2 垃圾回收器配置

- Serial 收集器：

> - `-XX:MaxTenuringThreshold` 默认值是 15，新生代对象晋升为老年代对象需要经过 15 次 GC。

- ParNew 收集器：

> - `-XX:MaxTenuringThreshold` 默认值是 15，新生代对象晋升为老年代对象需要经过 15 次 GC。
> - `-XX:UseAdaptiveSizePolicy` JVM 根据运行参数，动态调整堆空间大小及晋升年龄值。

- Parallel Scavenge/Parallel Old 收集器：

> - -XX:ParallelGCThreads 设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。
> - -XX:UseAdaptiveSizePolicy JVM根据运行参数，动态调整堆空间大小及晋升年龄值
> - -XX:MaxTenuringThreshold 默认值是15，新生代对象晋升为老年代对象需要经过15次GC
> - -XX:GCTimeRatio 设置垃圾回收时间占程序运行时间的百分比（默认99），公式为1/(1+n)
> - -XX:MaxGCPauseMillis 设置并行收集最大暂停时间

- CMS 收集器

> - -XX:ParallelCMSThreads 垃圾收集器线程数
> - -XX:CMSFullGCsBeforeCompaction CMS 采用标记-清理算法，会产生内存碎片，配置执行多少次 FullGC 后对内存进行整理
> - -XX:UseCMSCompactAtFullCollection 配置 FullGC 后是否立即整理内存碎片
> - -XX:CMSInitiatingOccupancyFraction 配置老年代内存使用率达到多少后进行内存回收（ JDK6 及以上版本默认值 92%）
> - -XX:CMSInitiatingOccupancyOnly 默认 false，不允许 HostSpot 根据成本自行进行决定何时进行垃圾回收
> - -XX:CMSClassUnloadingEnabled 配置方法区使用 CMS 进行垃圾回收
> - -XX:+CMSIncrementalMode 设置为增量模式，适用于单 CPU 情况
> - -XX:+PrintGCApplicationStoppedTime 打印程序 Stop-the-World 的暂停时间

- G1 收集器

> - -XX:G1ReservePercent 默认值 10%，预留的空闲空间的百分比
> - -XX:G1HeapRegionSize 配置 Region 块的大小，范围 1MB 到 32MB，设置后会根据最小堆 Java 堆内存划分出 2048 个 Region 块

#### 4.3 垃圾统计配置：

> - -XX:+PrintGC
> - -XX:+PrintGCDetails
> - -XX:+PrintGCTimeStamps：可与上面参数一起使用
> - -XX:+PrintGCApplicationConcurrentTime：打印每次垃圾回收前，程序未中断的执行时间，可与上面参数一起使用
> - -XX:+PrintGCApplicationStoppedTime：打印垃圾回收期间程序暂停的时间，可与上面参数一起使用
> - -XX:PrintHeapAtGC：打印 GC 前后的详细堆栈信息
> - -Xloggc:filename：与上面几个配合使用，把日志信息记录到文件来分析
> - 通过设定 -XX:+UseG1GC 在整个 Java 堆使用 G1 进行垃圾回收
> - 通过 -XX:+UseConcMarkSweepGC 设定新生代使用 ParNew（并发复制）收集器，老年代使用 CMS Concurrent Mark-Sweep（并发标记清除）收集器执行内存回收
> - 通过 -XX:+UseParallelOldGC 手动指定新生代使用 Parallel Scavenge（并行复制）收集器，老年代使用 Parallel Old（并行标记-压缩）收集器执行内存回收
> - 通过 -XX:+UseSerialGC 手动指定新生代使用 Serial Coping（串行复制）收集器，老年代使用 Serial Old （串行标记-清理-压缩）收集器执行内存回收
> - 通过 -XX:+UseParNewGC 手动指定新生代使用 ParNew（并发复制）收集器，老年代使用 Serial Old （串行标记-清理-压缩）收集器执行内存回收
> - 通过 -XX:+UseParallelGC 手动指定新生代使用 Parallel Scavenge（并行复制）收集器，老年代使用 Serial Old （串行标记-清理-压缩）收集器执行内存回收

#### 4.4 GC 日志：

```
/**
 * 堆溢出
 * 通过run configurations配置下列参数
 * VM Args：-Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails 
 * -XX:SurvivorRatio=8 -XX:+UseSerialGC    
 * -XX:+HeapDumpOnOutOfMemoryError  
 * 参数-XX：+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便事后进行分析,文件在项目中lib目录的外层目录下
 */
public class HeapOutOfMemory {
    static class OutOfMemoryObject {
    }

    public static void main(String[] args) {
        List<OutOfMemoryObject> list = new ArrayList<OutOfMemoryObject>();
        while (true) {
            list.add(new OutOfMemoryObject());
        }
    }
}

```

在mian函数右键：

![enter image description here](http://images.gitbook.cn/c443c6a0-7a18-11e8-842c-e13a980f1cb3)

![enter image description here](http://images.gitbook.cn/16ad0910-7a19-11e8-a0bd-d1cf011bb3b2)

然后点击 run，如果报错，那就是虚拟机参数里带了中文标点或者参数名字错了，使用 Serial Coping（串行复制）/Serial Old （串行标记-清理-压缩）组合打印的日志，日志跟下面的使用 -XX:+UseParallelOldGC 打印的日志类似，参考下面 Parallel 的日志分析。

```
[GC[DefNew: 7369K->1023K(9216K), 0.0107131 secs] 7369K->5193K(19456K), 0.0107576 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC[DefNew: 9215K->1024K(9216K), 0.0116251 secs] 13385K->11040K(19456K), 0.0116443 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[GC[DefNew: 8499K->8499K(9216K), 0.0000099 secs][Tenured: 10016K->8019K(10240K), 0.0333165 secs] 18515K->16331K(19456K), [Perm : 2629K->2629K(21248K)], 0.0333701 secs] [Times: user=0.05 sys=0.00, real=0.04 secs] 
[Full GC[Tenured: 8019K->8008K(10240K), 0.0300905 secs] 16331K->16320K(19456K), [Perm : 2629K->2628K(21248K)], 0.0301088 secs] [Times: user=0.01 sys=0.00, real=0.03 secs] 
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid16228.hprof ...
Heap dump file created [27926626 bytes in 0.100 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space

```

-XX:+UseParNewGC 使用 ParNew（并发复制）/Serial Old （串行标记-清理-压缩），日志跟下面的使用 -XX:+UseParallelOldGC 打印的日志类似，参考下面 Parallel 的日志分析。

```
[GC[ParNew: 7465K->1024K(9216K), 0.0113652 secs] 7465K->5222K(19456K), 0.0114082 secs] [Times: user=0.06 sys=0.00, real=0.01 secs] 
[GC[ParNew: 9216K->1024K(9216K), 0.0128881 secs] 13414K->11092K(19456K), 0.0129381 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC[ParNew: 8516K->8516K(9216K), 0.0000260 secs][Tenured: 10068K->8053K(10240K), 0.0322287 secs] 18585K->16360K(19456K), [Perm : 2629K->2629K(21248K)], 0.0323169 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
[Full GC[Tenured: 8053K->8013K(10240K), 0.0325796 secs] 16360K->16320K(19456K), [Perm : 2629K->2628K(21248K)], 0.0326293 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid24236.hprof ...
Heap dump file created [27926626 bytes in 0.097 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space

```

-XX:+UseParallelOldGC 使用 Parallel Scavenge（并行复制）/ Parallel Old（并行标记-压缩） 。

```
[GC [PSYoungGen: 7469K->1016K(9216K)] 7469K->5241K(19456K), 0.0110178 secs] [Times: user=0.05 sys=0.02, real=0.01 secs] 
/*
 * [PSYoungGen: 7469K->1016K(9216K)] PSYoungGen在新生代发生Minor
 * GC，回收前内存占用7469K，回收后内存占用1016K，新生代的总内存大小 7469K->5241K(19456K)
 * Java整个堆空间的内存占用变化，回收前7469K，回收 后5241K，整个堆19456K 0.0110178 secs，整个Minor
 * GC耗时多少秒，[Times: user=0.05 sys=0.02, real=0.01 secs] user程序耗时，sys系统耗时，real实际耗时
 * 多少秒
 */
[GC-- [PSYoungGen: 9208K->9208K(9216K)] 13433K->19440K(19456K), 0.0166318 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
[Full GC [PSYoungGen: 9208K->0K(9216K)] [ParOldGen: 10232K->10207K(10240K)] 19440K->10207K(19456K) [PSPermGen: 2632K->2631K(21504K)], 0.1698064 secs] [Times: user=0.34 sys=0.00, real=0.17 secs] 
[Full GC [PSYoungGen: 7735K->7728K(9216K)] [ParOldGen: 10207K->8097K(10240K)] 17942K->15826K(19456K) [PSPermGen: 2631K->2631K(21504K)], 0.1632131 secs] [Times: user=0.42 sys=0.00, real=0.15 secs] 
/*
 * Full GC代码整个堆的GC，PSYoungGen新生代内存使用变化，ParOldGen老年代内存使用变化，PSPermGen永久代内存使用变化，0.
 * 1632131 secs本次Full GC消耗多少秒，Times: user=0.42 sys=0.00, real=0.15 secs 本次Full
 * GC各种时间消耗，所有参数含义跟上面一样。
 */
[Full GC [PSYoungGen: 7728K->7728K(9216K)] [ParOldGen: 8097K->8089K(10240K)] 15826K->15818K(19456K) [PSPermGen: 2631K->2631K(21504K)], 0.0952587 secs] [Times: user=0.33 sys=0.02, real=0.10 secs] 
java.lang.OutOfMemoryError: Java heap space
// 生成的堆快照文件java_pid26276.hprof，在lib目录下
Dumping heap to java_pid26276.hprof ...

```

-XX:+UseConcMarkSweepGC 使用ParNew（并发复制）/ CMS Concurrent Mark-Sweep（并发标记清除）

```
[GC[ParNew: 7469K->1024K(9216K), 0.0339335 secs] 7469K->7079K(19456K), 0.0339810 secs] [Times: user=0.06 sys=0.00, real=0.03 secs] 
// 新生代使用ParNew做垃圾回收，参数含义跟上面分析的一样
Total time for which application threads were stopped: 0.0341067 seconds
/* 程序暂停时间0.0341067秒，-XX:+PrintGCApplicationStoppedTime打印程序Stop-the-World的暂停时间 */
[GC[ParNew: 9216K->9216K(9216K), 0.0000205 secs][CMS: 6055K->8941K(10240K), 0.0319501 secs] 15271K->13881K(19456K), [CMS Perm : 2630K->2629K(21248K)], 0.0320168 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
Total time for which application threads were stopped: 0.0321285 seconds
[GC [1 CMS-initial-mark: 8941K(10240K)] 13881K(19456K), 0.0035077 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
/*CMS初始标记，老年代内存占用大小8941K，总大小10240K，标记跟根对象集合直接相连接的对象的可达性*/
Total time for which application threads were stopped: 0.0035747 seconds
[Full GC[CMS[CMS-concurrent-mark: 0.011/0.011 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
//并发标记，从上次初始标记对象出发，标记垃圾对象和可回收对象
 (concurrent mode failure): 8941K->8940K(10240K), 0.0385219 secs] 14326K->13896K(19456K), [CMS Perm : 2632K->2632K(21248K)], 0.0385659 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
/*
 * 并发标记期间，新生成的对象在老年代无法有足够的内存容纳，产生concurrent mode
 * failure，老年代改用串行收集器，如果不产生concurrent mode
 * failure，后面还有CMS-remark，做最终标记，修正标记，会有暂停时间和内存占用和总内存，最后就是CMS-cocurrent-sweep，
 * 并发清除，会有清除耗时
 */
Total time for which application threads were stopped: 0.0387397 seconds

```

把代码修成：

```
public class HeapOutOfMemory {

    static class OutOfMemoryObject {
        public byte[] spaceSize = new byte[1024 * 1024];
        // 产生1024*1024个字节，也就是1024*1KB的内存 ，大小1MB
    }

    public static void creatHeap(int num) throws Exception {
        ArrayList<OutOfMemoryObject> list = new ArrayList<OutOfMemoryObject>();
        for (int i = 0; i < num; i++) {
            list.add(new OutOfMemoryObject());
        }
        System.gc();
    }

    public static void main(String[] args) throws Exception {
        while (true) {
            creatHeap(99);
        }

    }
}

```

```
[GC[ParNew: 8111K->8111K(9216K), 0.0000122 secs][CMS: 7176K->9216K(10240K), 0.0104259 secs] 15287K->14839K(19456K), [CMS Perm : 2632K->2632K(21248K)], 0.0104885 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
Total time for which application threads were stopped: 0.0106245 seconds
[GC [1 CMS-initial-mark: 9216K(10240K)] 15863K(19456K), 0.0004189 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Total time for which application threads were stopped: 0.0006342 seconds
[Full GC[CMS[CMS-concurrent-mark: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
 (concurrent mode failure): 9216K->9216K(10240K), 0.0043732 secs] 16947K->16887K(19456K), [CMS Perm : 2632K->2632K(21248K)], 0.0044245 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid1264.hprof ...
Total time for which application threads were stopped: 0.0171680 seconds
Heap dump file created [17857505 bytes in 0.018 secs]
[GC [1 CMS-initial-mark: 9216K(10240K)] 16876K(19456K), 0.0005822 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
Total time for which application threads were stopped: 0.0014073 seconds
[CMS-concurrent-mark: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-abortable-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC[YG occupancy: 7863 K (9216 K)][Rescan (parallel) , 0.0002252 secs][weak refs processing, 0.0000048 secs][scrub string table, 0.0001071 secs] [1 CMS-remark: 9216K(10240K)] 17079K(19456K), 0.0003718 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
/*
 * CMS-remark，做最终标记，修正标记，老年代已用内存9216K，总内存10240K，耗时 0.0003718 
 */
Total time for which application threads were stopped: 0.0005235 seconds
[CMS-concurrent-mark: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
/*
 * CMS-cocurrent-sweep，并发清除耗时0.002秒
 */ 

```

通过设定 -XX:+UseG1GC 在整个 Java 堆使用 G1 进行垃圾回收：

```
[GC pause (young) (initial-mark), 0.0028719 secs]
   [Parallel Time: 2.2 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 168.7, Avg: 168.8, Max: 168.9, Diff: 0.2]
      [Ext Root Scanning (ms): Min: 0.9, Avg: 1.1, Max: 1.5, Diff: 0.6, Sum: 4.6]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 2.0, Max: 8, Diff: 8, Sum: 8]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.1, Avg: 0.3, Max: 0.5, Diff: 0.4, Sum: 1.3]
      [Termination (ms): Min: 0.0, Avg: 0.4, Max: 0.5, Diff: 0.5, Sum: 1.4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
      [GC Worker Total (ms): Min: 1.6, Avg: 1.8, Max: 2.0, Diff: 0.4, Sum: 7.3]
      [GC Worker End (ms): Min: 170.3, Avg: 170.6, Max: 170.8, Diff: 0.5]
   [Code Root Fixup: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 0.6 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.5 ms]
      [Ref Enq: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 1024.0K(10.0M)->0.0B(9216.0K) Survivors: 0.0B->1024.0K Heap: 4857.6K(20.0M)->4720.2K(20.0M)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
Total time for which application threads were stopped: 0.0036639 seconds
// 初始标记以及暂停时间0.0036639 seconds
[GC concurrent-root-region-scan-start]
[GC concurrent-root-region-scan-end, 0.0005495 secs]
// 扫描根region,耗时0.0005495 secs
[GC concurrent-mark-start]
[GC concurrent-mark-end, 0.0004523 secs]
// 并发标记，耗时0.0004523 secs
[GC remark [GC ref-proc, 0.0000279 secs], 0.0006380 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
Total time for which application threads were stopped: 0.0008879 seconds
[GC cleanup 6809K->6809K(20M), 0.0006380 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
 // 清除阶段
Total time for which application threads were stopped: 0.0006810 seconds
[GC pause (young) (to-space exhausted), 0.0044659 secs]
   [Parallel Time: 4.0 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 176.6, Avg: 176.6, Max: 176.6, Diff: 0.0]
//Parallel Time  并行处理的部分占用时间，Worker Start – 工作线程启动的时刻
      [Ext Root Scanning (ms): Min: 0.3, Avg: 0.3, Max: 0.3, Diff: 0.1, Sum: 1.1]
//External root scanning  扫描外部根锁使用的时间      
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
//Update Remembered Set  开始前更新缓存列表，后续并发线程可以正确处理
         [Processed Buffers: Min: 0, Avg: 0.5, Max: 2, Diff: 2, Sum: 2]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
//Scanning Remembered Sets 查询指向收集区域的指针.
      [Object Copy (ms): Min: 2.6, Avg: 3.4, Max: 3.7, Diff: 1.1, Sum: 13.6]
//Object copy  每个独立线程复制和消亡对象锁花费的时间
      [Termination (ms): Min: 0.0, Avg: 0.3, Max: 1.1, Diff: 1.1, Sum: 1.2]
//Termination time 当一个工作线程结束了它对特定对象的复制和扫描
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 4.0, Avg: 4.0, Max: 4.0, Diff: 0.0, Sum: 15.9]
//GC Worker Total– 所有GC线程所使用的时间
      [GC Worker End (ms): Min: 180.6, Avg: 180.6, Max: 180.6, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.4 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.0 ms]
      [Ref Enq: 0.0 ms]
      [Free CSet: 0.0 ms]
//释放那些已经被收集过的区域，remembered sets所花费的时间
   [Eden: 1024.0K(9216.0K)->0.0B(10.0M) Survivors: 1024.0K->0.0B Heap: 9901.8K(20.0M)->9901.8K(20.0M)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
Total time for which application threads were stopped: 0.0046272 seconds
[Full GC 9901K->9719K(20M), 0.0030929 secs]
   [Eden: 0.0B(10.0M)->0.0B(10.0M) Survivors: 0.0B->0.0B Heap: 9901.8K(20.0M)->9719.5K(20.0M)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC 9719K->9708K(20M), 0.0028649 secs]
   [Eden: 0.0B(10.0M)->0.0B(10.0M) Survivors: 0.0B->0.0B Heap: 9719.5K(20.0M)->9708.5K(20.0M)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
Total time for which application threads were stopped: 0.0060723 seconds
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid11772.hprof ...
Total time for which application threads were stopped: 0.0118791 seconds
Heap dump file created [10622284 bytes in 0.012 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
    at com.jvm.outofmemory.HeapOutOfMemory$OutOfMemoryObject.<init>(HeapOutOfMemory.java:14)
    at com.jvm.outofmemory.HeapOutOfMemory.creatHeap(HeapOutOfMemory.java:21)
    at com.jvm.outofmemory.HeapOutOfMemory.main(HeapOutOfMemory.java:28)
Heap
 garbage-first heap   total 20480K, used 9708K [0x00000000f9a00000, 0x00000000fae00000, 0x00000000fae00000)
  region size 1024K, 1 young (1024K), 0 survivors (0K)
 compacting perm gen  total 20480K, used 2663K 
//region 大小1M，一个年轻代region，没有幸存代region
[0x00000000fae00000, 0x00000000fc200000, 0x0000000100000000)
   the space 20480K,  13% used [0x00000000fae00000, 0x00000000fb099cd0, 0x00000000fb099e00, 0x00000000fc200000)
No shared spaces configured.

```

#### 4.5 异常信息：

> java.lang.OutOfMemoryError: PermGen space：永久代被占满，无法为 class 字节码文件分配空间，现在反射，动态代理使用越来越多，字节码插桩技术可以在类加载的前后插入自定义的内容，这些都会增大永久代的占用。
>
> java.lang.OutOfMemoryError: Java heap space：内存泄漏

Java引用类型：

- 强引用：强引用指向的对象在任何时候都不会被系统回收。
- 软引用：一个持有软引用的对象，不会被 JVM 很快地回收，JVM 会根据当前堆的使用情况来判断何时回收。（通常用于缓存）
- 弱引用：在系统 GC 时，只要发现弱引用，不管系统队空间是否足够，都会将对象进行回收。
- 虚引用：和没有引用几乎一样。

```
public class WeakReference {
    // 弱引用
    public static Map map = new WeakHashMap();
    // 强引用
    public static List list = new ArrayList();

    public static void main(String[] args) {

        for (int i = 0; i < 999; i++) {
            Integer j = new Integer(i);
            // j强引用
            list.add(j);
            // 存放在weakHashMap中的key都存在强引用，那么weakHashMap就退化为HashMap，即强引用
            map.put(j, new byte[i]);
            // 当内存足不足时，会报OutOfMemoryError，因为存在j强引用，没有被使用的map存在的垃圾却无法被清理，造成内存泄漏
        }
    }

}
//把for循环替换成
        for(int i=0;i<999;i++){
        Integer k = new  Integer(i);
        map.put(k,new byte[i]); 
//当内存不足时，会被自动回收，weakHashMap 会在内存紧张时，自动释放持有弱引用的数据
    }

```

> java.lang.StackOverflowError：栈空间溢出，通常是栈帧深度过深，如递归调用没有出口，造成死循环。
>
> Fatal: Stack size too small：线程栈爆满，通常是方法体代码过多造成，可以通过拆分方法，让方法职责单一避免溢出，也可以通过 -Xss 增大线程栈空间。

### 5、JVM 问题排查

#### 5.1 CPU 使用率过高时：

1. 通过 top 命令查看服务器的负载情况
2. 通过 `top+H` 查看线程的使用情况
3. 通过 `jstat -gcutil pid` 查看具体线程的 GC 前后内存变化（pid的地方用进程号代替）
4. `ps -mp pid -o THREAD,tid,time` 查看线程列表
5. `printf "%x\n" tid` 将 tid 转换成 16 进制格式
6. `jstack pid |grep tid -A 60` 打印线程（转换后 16 进制格式的数字）tid 的堆栈信息
7. 通常计算密集型应用产生死锁或者死循环或超时重试导致的线程堆积都会占用 CPU 大量资源，CPU 使用率可能达到 200%

#### 5.2 线上频繁 Full GC：

1.通过 `top+H` 查看线程的使用情况

2.`jmap -histo:live pid`（进程号） 这个会立即触发 Full GC

3.线上开启了 `-XX:+HeapDumpBeforeFullGC` 也就是 FullGC 前保存内存快照，JVM 在执行 dump 操作的时是会发生 stop the word，此时所有的用户线程都会暂停运行。为了对外能正常提供服务，可用分布式部署，匹配负载均衡

4.通过 MAT（Memory Analyzer Tool）分析内存快照，把 “Keep unreachable objects” 勾上，否则 MAT 会把堆中不可达的对象去除掉，反而不利于分析， 通过 `-XX：+HeapDumpOnOutOfMemoryError` 可以让虚拟机在出现内存溢出异常时 Dump 出当前的内存堆转储快照，然后看到 GC 日志 `Dumping heap to java_pid11772.hprof ...`，我们可以在工程目录里打开 pid11772.hprof 日志文件（在 Eclipse 应用市场下载 Memory Analyzer Tool 插件并安装，就可以直接打开）

5.查看 `Dominator Tree` 选项，内存中所有对象都按照内存消耗排名从高到低进行排序

![enter image description here](http://images.gitbook.cn/f457a020-7ab6-11e8-8a2d-af6f3f96c3a3)

> - Class Name 是 Java 类的全限定名 Shallow Heap 是对象本身消耗内存大小
> - Retained Heap 是对象本身和它所引用的对象的内存大小总和
> - Percenttage 是对象消耗占整个堆快照的比率

6.查看是否使用了大对象，或者长期持有对象的引用，或者大量堆积了全局的本地缓存

电商平台，在做活动促销时，瞬间有大量的客户登录，但是登录页面假死，造成登录失败，在服务器的 log 日志发现大量的超时信息，报线程耗尽，通过 `jstack pid` 打印进程中线程堆栈信息发现有上百个 thread 不断的重试发送请求，基本定位是大量超时请求重试导致服务器高负荷运行而假死。

我们把服务端提供者登录接口的线程并发数从 10 调成 15，并把消费者重试次数改为 0，请求失败后直接返回，让用户的客户端 Http 进行重试，同时在登录时关闭 Druid 的 SQL 监控功能，避免增加额外资源消耗，最后重新上线，问题修复。

### 6、JVM 类加载器和 OSGI 实战

#### 6.1 JVM 类加载器：

一个类在使用前，如通过类调用静态字段、静态方法，或者 new 一个实例对象，第一步就是需要类加载，然后是连接和初始化，最后才能使用。

类加载的就是将 .java 代码文件编译成 .class 字节码文件后，Java 虚拟机的类加载器通过读取此类的二进制字节流，转换成目标类的实例。

除了 Java 会生成字节码外，运行在 JVM 上的 JRuby、Scala、Groovy 同样需要编译成对应的 .class 字节码文件，这里列举了四种不同的字节码，不单是 Java 才生成字节码文件。

常用的类加载器有四种：

- Bootstrap ClasssLoader：启动类加载器，加载 `JAVA_HOME/lib` 目录下的类
- ExtClassLoader：扩展类加载器，加载 `JAVA_HOME/lib/ext` 目录下的类

![enter image description here](http://images.gitbook.cn/8cdbbfe0-7ad3-11e8-be93-2154b2b481b1)

- AppClassLoader：应用程序类加载器，加载用户指定的 classpath（存放 src 目录 Java 文件编译之后的 class 文件和 xml、properties 等资源配置文件的 src/main/webapp/WEB-INF/classes 目录）下的类
- UserClassLoader：用户自定义的类加载器（只要继承 ClassLoader并实现 findClass(String name) 方法），自定义加载路径

类加载时并不需要等到某个类被首次主动使用时再加载它，JVM 类加载器会在预料某个类要被使用时预先加载。

![enter image description here](http://images.gitbook.cn/46ef0660-7ad6-11e8-9d5c-33b66bdd578b)

Java 类加载基于双亲委派模型——当有类加载请求时，从下往上检查类是否被加载，如果没被加载，UserClassLoader 就委托父类 AppClassLoader 加载，AppClassLoader 继续委托其父类 ExtClassLoader 加载，接着分派给 Bootstrap ClasssLoader 加载；

如果无法加载就返回到发起加载请求的类加载一直到由最开始发起加载请求的 UserClassLoader 加载，所有类最终都会去到顶层。Bootstrap ClasssLoader 开始加载，无法加载就返回子加载器处理，一直到最开始的加载器。

这样子，就算用户自定义了 java.lang.Object 类和系统的 java.lang.Object 类重复，也不会被加载，下面我们就来自定义自己的类加载器。

```
package com.jvm.outofmemory;

public class MyClassLoader extends ClassLoader {

    public MyClassLoader() {
        super();

    }

    public MyClassLoader(ClassLoader parent) {
        super(parent);

    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // do something
        // 自己先不加载，先让父类加载
        return super.findClass(name);
    }

    public static void main(String[] args) throws ClassNotFoundException {

        MyClassLoader myLoader = new MyClassLoader();
        // 打印当前类路径
        System.out.println(System.getProperty("java.class.path"));
        // C:\Program Files\Java\jdk1.7.0_45\jre\lib\resources.jar;
        // C:\ProgramFiles\Java\jdk1.7.0_45\jre\lib\charsets.jar;
        // C:\ProgramFiles\Java\jdk1.7.0_45\jre\lib\jfr.jar;
        // C:\ProgramFiles\Java\jdk1.7.0_45\jre\lib\ext\sunjce_provider.jar;
        // C:\ProgramFiles\Java\jdk1.7.0_45\jre\lib\ext\sunmscapi.jar;
        // C:\ProgramFiles\Java\jdk1.7.0_45\jre\lib\ext\zipfs.jar;J:\workspace\jvm\bin;J:\workspace\jvm\lib\activemq-all-5.9.0.jar;......

        // ClassPath路径下并不存在Demo.class类，故抛出异常
        System.out.println(myLoader.loadClass("Demo").getClassLoader().getClass().getName());

    }

}
//ClassNotFoundException异常被抛出则表明父类加载器加载失败，自定义类加载器的父类加载器是AppClassLoader  
Exception in thread "main" java.lang.ClassNotFoundException: Demo
    at java.lang.ClassLoader.findClass(ClassLoader.java:531)
    at com.jvm.outofmemory.MyClassLoader.findClass(MyClassLoader.java:19)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
    at com.jvm.outofmemory.MyClassLoader.main(MyClassLoader.java:43)

```

自定义类加载器后，我们看下源码里双亲委派模型是怎么加载类的。

```
public abstract class ClassLoader {
    private final ClassLoader parent;
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 先检查类是否已经被加载
            Class c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                //如果存在父类加载器，就委托给父类加载器加载
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                    //如果不存在父类加载器，就委托给顶层的启动类加载器加载
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException异常被抛出则表明父类加载器加载失败  
                }
                if (c == null) {
                    // 如果父类无法加载，就自己加载      
                    long t1 = System.nanoTime();
                    c = findClass(name);
sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0); 

sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);                 

sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
    }

```

我们看到上面 loadClass 类里有同步代码块 synchronized (getClassLoadingLock(name))，而在 JDK1.6 之前是方法 `protected synchronized Class<?> loadClass(String name, boolean resolve)` 上锁，锁住方法当前对象。

这就导致一个问题，当 A 包依赖 B 包，A 在自己的类加载器的 loadClass 方法中，最终调用到 B 的类加载器的 loadClass 方法。A 先锁住自己的类加载器，然后去申请 B 的类加载器的锁，当 B 也依赖 A 包时，B 加载 A 的包时，过程相反，在多线程下，就容易产生死锁。如果类加载器是单线程运行就会安全，但效率会很低 同步代码块 synchronized (getClassLoadingLock(name)) 锁住的是一个特定对象。

```
    private final ConcurrentHashMap<String, Object> parallelLockMap;

    protected Object getClassLoadingLock(String className) {
        Object lock = this;
        // parallelLockMap是一个ConcurrentHashMap
        if (parallelLockMap != null) {
            // 锁对象
            Object newLock = new Object();

            // putIfAbsent(K, V)方法查看K（className）和V（newLock）是否相互对应，
            // 是的就返回V（newLock），否则返回null
            // 每个className关联一个锁，并将这个锁返回，缩小了锁定粒度了,只要类名不同，就会匹配不同的锁，
            // 就是并行加载，类似ConcurrentHashMap里面的分段锁，
            // 不锁住整个Map,而是锁住一个Segment，每次只需要对Segment上锁或解锁，以空间换时间

            lock = parallelLockMap.putIfAbsent(className, newLock);
            if (lock == null) {
                // 创建一个新锁对象
                lock = newLock;
            }
        }
        return lock;

```

通过并行加载，可以提升加载效率，然后讲下类加载的面试题，在 Java 反射中 Class.forName() 加载类和使用 ClassLoader 加载类是不一样的。

```
public class MyCase {
    static {
        System.out.println("执行了静态代码块");
    }

    private static String field = methodCheck();

    public static String methodCheck() {

        System.out.println("执行了静态代方法");
        return "给静态变量赋值";
    }

}


public class DemoTest {

    public static void main(String[] args) {
        try {
            System.out.println("Class.forName开始执行：");
            Class.forName("com.jvm.outofmemory.MyCase");
            System.out.println("ClassLoader开始执行：");
            ClassLoader.getSystemClassLoader().loadClass("com.jvm.outofmemory.MyCase");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

    }

}
控制台输出
Class.forName开始执行：
执行了静态代码块
执行了静态代方法
ClassLoader开始执行：

```

Class.forName 是加载 MyCase 类并完成初始化，给静态代码块和静态变量赋值，而 ClassLoader 只是将类加载进 JVM 虚拟机，并没有初始化。

```
 @CallerSensitive
    public static Class<?> forName(String className)
                throws ClassNotFoundException {
        return forName0(className, true,
                        ClassLoader.getClassLoader(Reflection.getCallerClass()));
    }

```

Class.forName 底层也是调用了 ClassLoader，只是第二个参数为 true，即加载类并初始化，默认就会初始化类，JDBC 连接就是用 Class.forName 加载驱动。所以注册连接驱动会在静态代码块执行，Sprng 里的 IOC 是通过 ClassLoader 来产生，可以控制 Bean 的延迟加载（首次使用才创建）。

#### 6.2 OSGI 实战

为了实现代码热替换，模块化和动态化，就像鼠标一样即插即用，双亲委派这种树状的加载器就难以胜任，于是出现了 OSGI 加载模型，OSGI 里每个程序模块（Bundle，就是普通的 jar 包, 只是加入了特殊的头信息，是最小的部署模块）都会有自己的类加载器，当需要更换程序时，就连同 Bundle 和类加载器一起替换，是一种网状的加载模型，Bundle 间互相委托加载，并不是层次化的。

Java 类加载机制的隔离是通过不同类加载器加载指定目录来实现的，类加载的共享机制是通过双亲委派模型来实现，而 OSGI 实现隔离靠的是每个 Bundle 都自带一个独立的类加载器 ClassLoader。

![enter image description here](http://images.gitbook.cn/db7519e0-7b68-11e8-95d0-05d53c09e644)

OSGI 加载 Bundle 模块的顺序

1. 首先检查包名是否以 java.* 开头，或者是否在一个特定的配置文件（org.osgi.framework.bootdelegation）中定义。如果是，则 bundle 类加载器立即委托给父类加载器（通常是 Application 类加载器），如果不是则进入 2
2. 检查是否在 Import-Package、Require-Bundle 委派列表里，如果是委托给对应 Bundle 类加载器，如果不是，进入 3
3. 检查是否在当前 Bundle 的 Classpath 里，如果是使用自己的类加载器加载，如果不是，进入 4
4. 搜索可能附加在当前 bundle 上的 fragment 中的内部类，找到则委派给 Fragment bundle 类加载器加载，如果找不到，进入 5
5. 查找动态导入列表里的 Bundle，委派给对应的类加载器加载，否则类加载失败

如果用 Java 的结构的项目去部署，当项目复杂度提升时，每次上线，代码只是增加或者修改了部分功能，但都得关掉服务，重新部署所有的代码和配置，管理沟通成本都很高，很容产生线上事故，而 OSGI 的应用是一个模块化的系统，避免了部署时 jar 或 classpath 错综复杂依赖管理，发布应用和更新应用都很强大，可以热替换特定的 Bundle 模块，提高部署可靠性。

我们来创建一个 OSGI 应用。打开 Eclipse，File->New->Project：

![enter image description here](http://images.gitbook.cn/bbb02240-7b76-11e8-b075-95736529b697)

![enter image description here](http://images.gitbook.cn/feef67a0-7b76-11e8-95d0-05d53c09e644)

选择 OSGI 框架 Equniox（Eclipse 强大的插件机制就是构建于 OSGI Bundle 之上，Eclipse 本身就包含了 Equniox） ：

![enter image description here](http://images.gitbook.cn/5e2221e0-7b77-11e8-b36a-d9daff3efc07)

勾选创建 Activator 类，每个 Bundle 启动时都会调用 Bundle（模块）里 Activator（类）的 start 方法，停止时调用 stop 方法，点击 Finish，生成工程：

```
package com.osgi.bundle.demo;

import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;

public class Activator implements BundleActivator {

    private static BundleContext context;

    static BundleContext getContext() {
        return context;
    }

    public void start(BundleContext bundleContext) throws Exception {
        Activator.context = bundleContext;
        //添加输出This is OSGI Projcect
        System.out.println("This is OSGI Projcect");
    }

    public void stop(BundleContext bundleContext) throws Exception {
        Activator.context = null;
    }

}

```

Run->Run Configuration-> 双击 OSGI Framework 生成项目配置：

![enter image description here](http://images.gitbook.cn/8d2e16a0-7b78-11e8-95d0-05d53c09e644)

添加对应的 Bundle，最后点击 Validate，弹出下框证明正常，可点击 Run 运行：

![enter image description here](http://images.gitbook.cn/b53a7800-7b78-11e8-b36a-d9daff3efc07)

可以看到控制台输出 This is OSGI Projcect。在控制台我们输入 ss ( short status) 查看服务状态：

```
This is OSGI Projcect
osgi> ss
"Framework is launched."


id    State       Bundle
0    ACTIVE      org.eclipse.osgi_3.12.100.v20180210-1608
1    ACTIVE      org.apache.felix.gogo.runtime_0.10.0.v201209301036
2    ACTIVE      org.apache.felix.gogo.command_0.10.0.v201209301215

// ACTIVE表明 com.osgi.bundle.demo Bundle运行中 

3    ACTIVE      com.osgi.bundle.demo_1.0.0.qualifier
4    ACTIVE      org.apache.felix.gogo.shell_0.10.0.v201212101605
5    ACTIVE     org.eclipse.equinox.console_1.1.300.v20170512-2111
// 停止 com.osgi.bundle.demo Bundle  
osgi> stop com.osgi.bundle.demo
osgi> ss
"Framework is launched."


id    State       Bundle
0    ACTIVE      org.eclipse.osgi_3.12.100.v20180210-1608
1    ACTIVE      org.apache.felix.gogo.runtime_0.10.0.v201209301036
2    ACTIVE      org.apache.felix.gogo.command_0.10.0.v201209301215

// RESOLVED 表明 Bundle com.osgi.bundle.demo 停止了 

3    RESOLVED    com.osgi.bundle.demo_1.0.0.qualifier
4    ACTIVE      org.apache.felix.gogo.shell_0.10.0.v201212101605
5    ACTIVE      org.eclipse.equinox.console_1.1.300.v20170512-2111
// 通过close关闭整个应用框架
osgi> close
Really want to stop Equinox? (y/n; default=y)  y
osgi> 

```

一个 Bundle 包含 MANIFEST.MF，也就是 Bundle 的头信息，Java 代码以及配置文件（XML，Properties），其中 MANIFEST.MF 包含了下面的信息。

```
版本号
Manifest-Version: 1.0
Bundle-ManifestVersion: 2
名字
Bundle-Name: Demo
Bundle-SymbolicName: com.osgi.bundle.demo
Bundle-Version: 1.0.0.qualifier
Bundle类
Bundle-Activator: com.osgi.bundle.demo.Activator
Bundle-Vendor: OSGI
依赖环境
Bundle-RequiredExecutionEnvironment: JavaSE-1.7
导入的包
Import-Package: org.osgi.framework;version="1.3.0"
Bundle-ActivationPolicy: lazy

```

#### 6.3 Equinox OSGi 命令列表

- 控制框架

> - launch 启动框架
> - shutdown 停止框架
> - close 关闭、退出框架
> - exit 立即退出，相当于 System.exit
> - init 卸载所有 bundle（前提是已经 shutdown）
> - setprop 设置属性，在运行时进行

- 控制 Bundle

> - Install 安装 uninstall 卸载
> - Stop 停止
> - Refresh 刷新
> - Update 更新

- 展示状态

> - Status 展示安装的 bundle 和注册的服务
> - Ss 展示所有 bundle 的简单状态
> - Services 展示注册服务的详细信息
> - Packages 展示导入、导出包的状态
> - Bundles 展示所有已经安装的 bundles 的状态
> - Headers 展示 bundles 的头信息，即 MANIFEST.MF 中的内容
> - Log 展示 LOG 入口信息

- 其他

> - Exec 在另外一个进程中执行一个命令（阻塞状态）
> - Fork 和 EXEC 不同的是不会引起阻塞
> - Gc 促使垃圾回收
> - Getprop 得到属性，或者某个属性

- 控制启动级别

> - Sl 得到某个 bundle 或者整个框架的 start level 信息
> - Setfwsl 设置框架的 start level
> - Setbsl 设置 bundle 的 start level
> - setibsl 设置初始化 bundle 的 start level

### 7、JVM 内存模型

Java 内存模型规定所有的变量都是存在主存当中（物理内存），每个线程都有自己的工作内存（高速缓存）。线程对变量只能操作自己工作内存，也不能访问其他线程的内存，更不能直接对主存进行操作。

#### 7.1 原子性

Java 内存模型只保证对基本数据类型（int、long等）读取和赋值是原子性操作，也就是要么成功，要么失败，不会出现中断，通过 synchronized 和 Lock 来保证任意时间代码块都是单线程执行的，实现大范围的原子性。

```
a = 1;         // 把1赋值给a，具备原子性，线程直接把值放入工作内存
b = a;         // 先读取a，再赋给b，再放入工作内存，不具备原子性
a++;           // 读取a，加1，写入新的值到工作内存，不具备原子性

```

在 32 位系统对 64 位的 long 类型进行多线程读写，是不具备原子性的，同样的操作会有不同的值。

#### 7.2 可见性

```
//线程1执行 
int a = 0;
a = 1;

//线程2执行
b = a;

```

a 是普通变量，不具备可见性。当线程 1 给 a 赋值 1，但是操作工作内存，还没同步到主存，这时候线程 2 读到了主存 a 的旧值 0，那就产生了脏读，并不能拿到最新的值，如果写成 volatile int a 那就能保证 a 的可见性。

Java 的 volatile 关键字修饰的变量具备可见性，当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，确保正确的值，synchronized 和 Lock 是来同步线程的，也可保证可见性。

#### 7.3 有序性

Java 内存模型本身就具备一定的 “有序性”，这称为 happens-before 原则。如果两个操作的执行顺序无法从 happens-before 原则推导出来，那么虚拟机可以随意地对它们进行重排序而不具备有序性。

- happens-before 原则（先行发生原则）：
- 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
- 锁定规则：一个 unLock 操作先行发生于后面对同一个锁额 lock 操作
- volatile 变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
- 传递规则：如果操作 A 先行发生于操作 B，而操作 B 又先行发生于操作 C，则可以得出操作 A 先行发生于操作 C
- 线程启动规则：Thread 对象的 start() 方法先行发生于此线程的每个一个动作
- 线程中断规则：对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 Thread.join() 方法结束、Thread.isAlive() 的返回值手段检测到线程已经终止执行
- 对象终结规则：一个对象的初始化完成先行发生于他的 finalize() 方法的开始

```
//线程1:
data = loadData();    
flag= true;              

//线程2:
while(!flag){
  validate(data );
}

```

`data = loadData()` 和 `flag= true` 没有数据依赖，可能被重排序，`flag= true` 先执行，结果线程 2 又拿到了锁，判断 flag 是 true，直接执行 `validate(data )`，但 data 的数据还没有拿到，最后就会导致方法抛异常，单线程不受指令重排的影响，多线程会受到影响，必须要保证原子性、可见性以及有序性才能让并发程序正确地执行。

### 参考文献

[1] Tim Lindholm，Java虚拟机规范Java SE7版[M].机械工业出版社，2014年1月 [2] Sun，[HotSpot内存管理白皮书[EB/OL\]](https://wenku.baidu.com/view/9f8c977831b765ce0508149b.html) [3] 高翔龙，Java虚拟机精讲[M].电子工业出版社，2015年5月 [4] 文纳斯，深入Java虚拟机[M].机械工业出版社，2003年9月 [5] 周志明，Java虚拟机：JVM高级特性与最佳实践（第2版）.机械工业出版社，2013年6月 [6] Oracle，[Garbage-First Garbage Collection[EB/OL\]](http://www.oracle.com/technetwork/articles/java/g1gc-1984535.html)



（一）JVM的体系结构不是Java 8的（是经典的JVM的结构模型，JVM体系演进都以此为基础），在Java 8里面常量池已经被元数据区取代，JVM取消了PermGen（永久代），但类的元数据信息（metadata）保留，其位置不在连续的堆空间上，而是移动到本地内存（Native memory）中，也叫Metaspace （元数据区），因为PermGen很难调整，PermGen中类的元数据信息在每次Full GC的时候收集效果较差，很难确定为PermGen分配空间的大小，加载的class的大小，常量的大小，方法的大小都会影响永久代的大小，元数据区和Java Heap 具备相同的地址空间，可以简化垃圾收集的工作量，降低暂停时间。Java 8的函数式编程，也就是函数（方法）作为传递的参数是基于JVM底层的invoke dynamic指令实现的，Java 8默认的垃圾收集器是G1



（二）阿里巴巴JVM团队在OpenJDK8进行优化和定制，版本被命名为AJDK(Alibaba JDK)，引入三大特性，第一个特性：协程（轻量级的线程，类似Go语言的协程），一个JVM进程运行多个线程，每个线程运行多个协程，每个协程有独立的栈空间，基于HotSpot TM JVM 上的协程实现，使用协程作为异步 IO 的抽象机制，使得现有基于独占线程模型的代码透明的跑在事件驱动模型上，获得性能提升；第二个特性：多租户共享内存（资源隔离，类似Docker），多个应用可以共享一个JVM实例，在JVM的内部，为每个应用单元创建一个虚拟的Container（容器），单独管理Heap和CPU等重要资源，实现资源隔离，也就是结束一组线程不影响其他线程的运行，但垃圾回收不是隔离的。第三个特性：即时编译预热（适合大规模Java应用部署），避免CPU 被即时编译（JIT）拖累，JVM在应用启动时没有JIT的充分参与，性能没有达到最优状态，所以这个过程中要不断重新编译，优化，会占用大量线程资源，影响工作线程，采用分层编译可以降低运行负载，但会占用大量的空间，容易引发JVM崩溃，Alibaba JDK提供了辅助工具JIT Warmup，它可以记录上一次运行时（预发布）被编译的方法名字，类初始化顺序等信息，在下次启动（正式发布）时直接读取这些信息编译对应的方法。文章下次更新时会加入这些JAVA8的内容。





