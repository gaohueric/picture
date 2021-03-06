#### 内容提要

内容提要：

- ​
- 为什么 try 要包括整段业务代码而不是最小的粒度？
- try 和 for 这俩代码块谁包住谁更好？
- 自定义异常的一些规范和建议是什么？
- 项目里异常一般如何设计？项目里整体的异常框架设计，比如 Dao 层，Service 层，Action 层，是不是层层向上抛出？
- 应当什么时候设计自定义异常？
- 比如我去打开一个配置文件，找不到就返回 false 那放在 catch 里返回可以吗？
- 项目中如何优雅的设计异常处理，有没有最佳实践讲解？
- 建议使用最大的 Exception 还是针对不同的情况抛各种异常？'我见过5年以上的开发工程师，对空指针的处理还是不到位' 这句能详细说明一下情况吗？如何针对性的对某些可预判的错误做处理？
- 全局异常检测是否只是针对于业务层面或者数据层为错校验没有的情况下的处理办法，除此之外还有什么别的优点吗？

------

**问：为什么try 要包括整段业务代码而不是最小的粒度？**

**答：**这个问题分2个层面来说明：

1. 有的人说try的粒度要小。
2. 有的人说try的粒度要大。

2种说法各有各的用处，我建议使用 try 包括一整段代码。为什么选择第2种，因为只实际项目中，吃了太多的亏，总结的经验和教训。对于初学开发者或者经验不足的开发者，他们对异常处理意识淡漠，很容易给项目带来灾难。

我给大家讲个案例说明为什么建议使用第2种，大家看下这个图，我们从这里引出问题。

![enter image description here](http://images.gitbook.cn/0f3f5210-1bc7-11e8-be31-55e3e5673003)

异常继承与 Throwable 类，又分为 Error 和 Exception。Error 一般不需要程序员处理，发生这种问题，jvm 就会出错，我们平常说的异常更多的指的是 Exception 类。Exception 类又分为2种，一种是校验异常，一种是运行时异常。其实我们经常处理的是 RuntimeException，因为检验异常在程序开发的过程中，不进行处理的话，编译器就不通过。编译器校验异常如下:

![enter image description here](http://images.gitbook.cn/08818d30-1bc7-11e8-be31-55e3e5673003)

checked exceptin 就是指编译器会检验，比如 IOException，你写程序不处理，不行。因此对于异常处理薄弱的程序员来说，会出现一个现象，就是他写的程序通篇只有校验异常，因为如果他不处理校验异常，编译器会报错。编译器会根据校验住的异常提醒用户处理，eclipse 中快捷键是 ctrl+1。我们一般选 try.catch，处理掉这些校验异常，而编译器只会处理产生异常的代码，这样就引出了我们异常处理的颗粒度问题。

毫无疑问，我们说的这种校验异常默认处理的方式属于颗粒度小的。为什么我不建议把粒度控制的小，而是建议包括一段代码，也就是通常所说的大 try。因为我们程序运行出错的很大部分是运行时异常，因为程序运行你是控制不住地。运行时异常有哪些，空指针，数组越界，类型不匹配，除数为0等等。这种运行时异常的发生，因为编译器不会检查出来，因此你不做处理的话，项目里是会有很多漏网之鱼的，会给项目带来极大的风险和灾难。

Java 程序员面临的最大异常就是空指针。我见过5年以上的开发工程师，对空指针的处理还是不到位，运行过程中，凡是出现空的问题，你没有用大 try，势必会产生问题。这就是为什么建议大家使用大 try 的原因，这么做会对项目的质量起到一定的保障，同时结合日志记录异常，会给后期系统的维护带来很大的便利，否则的话，产生问题，无从查起，真是令人抓狂。

当然我们也可以使用大 try 里包括小 try 的方式进行开发，这样能对异常进行具体的捕捉和说明以及响应。理解背后的原因后，大家开发的过程中可以组合使用。

用大 try 不是为了发现运行时异常，是为了补漏，避免出了问题没人知道，有异常同时要记录日志，这些 Exception 的日志结合监控服务，能正真发现业务问题，避免产生业务风险。

------

**问：try 和 for 这俩代码块谁包住谁更好？**

**答：**这个问题得看 for 循环的业务是什么，首先一点不管 try 写在哪里，出了异常，代码一定会走到try里，这样就看你的业务要求是什么。如果要求循环一出了问题，后续的循环继续执行，那么你就把 try 写到 for 里；如果要求后续的循环不执行，那么就写到 for 外。

另外：用多线程实现的定时任务在循环处理数据时出现异常必须及时处理，否则执行时会退出。

------

**问：自定义异常的一些规范和建议是什么？**

**答：**谈这个问题之前，我们要说说为什么需要自定义异常。其实在实际开发项目中，每个项目一般都有自定义异常。原因是虽然 Java 提供了大量的异常类，但是这些异常类有时很难满足开发者或者项目的实际需求，所以我们不得不根据自己的需要定义自己的异常类。当然这个需要有可能是我们的用户或者客服，也可能是程序员本身。

自定义异常的特点是什么，那就是它只需要继承 Exception 类即可。同时覆写它里面的一些方法。

![enter image description here](http://images.gitbook.cn/f5cc4680-1bc6-11e8-be31-55e3e5673003)

主要是这几个方法。自己定义自己的异常就好，根据项目或者开发者需求自己定义把控即可。自定义异常主要是为了把 Exception 进行封装，形成自己的东西，便于理解。比如对用户来说，界面的友好，发生异常后，我们给用户进行友好的提示信息。

------

**问：项目里异常一般如何设计？项目里整体的异常框架设计，比如 Dao 层，Service 层，Action 层，是不是层层向上抛出？**

**答：**异常设计的事情我理解更多的就是指自定义异常，每个项目的要求不一样，就像刚才提问的同学说的他们的项目就会划分的很细，有的项目就不是太细。这怎么选择，还是看系统设计的时候，需求怎么提的，架构怎么设计的。一般来说，自定义异常会有几种类型，比如业务异常，用户异常，系统异常，接口异常，网络异常，参数异常等。运行时异常也是集成自 Exception，我们的自定义异常也是集成自 Exception，这就说明我们的自定义异常和运行时异常属于平级，如果自定义异常继承了运行时异常，那么就是 Exception 的孙子级了。

你说的层层上抛是我们建议的一种写法。尤其是在项目开发中，应该形成一个习惯，大家的风格都是层层上抛，中央处理。除非你自己能决定的异常，对大局不产生影响，则可以内部消化，不需要上抛。这个和我们管理的模式是一样的，地方不能决定的问题需要中央处理，地方能解决的，地方解决，类似。

------

**问：应当什么时候设计自定义异常？**

**答：**根据项目需要，异常可以进行细分，比如业务级：你想控制到什么粒度，写代码的人最清楚自己在写什么，比如写的是订单保存，那么你针对订单保存的业务可以在不同的代码逻辑提示不同的异常信息。接口级异常也是如此。

不建议使用判断语句进行异常处理，这样后期维护特别费劲，而且程序也不够清晰明了，异常处理就该用异常的处理机制解决，这是正道。

------

**问：比如我去打开一个配置文件，找不到就返回 false 那放在 catch 里返回可以吗？**

**答：**其实配置文件找不到，这是很严重的问题，一般在程序启动前就要进行检查配置文件是否存在，如果配置文件存在，而是里面的一些配置项少了，那么在你配置项使用的地方，一定要告警，结合日志进行监控和分析，及时解决问题。

------

**问：项目中如何优雅的设计异常处理，有没有最佳实践讲解？**

**答：**我认为这个问题没有标准答案，他是一个开放性的问题。我们把握几个原则就好，掌握了心法，就可以随心所欲了。毕竟每个公司，每个项目的代码都不一样，各有各的写法，但是，万变不离其宗，掌握本质就好。

可以参考我 Chat 文章里写的一些对应异常的感悟，本质来说：

1. 用户层面：用户提示信息要优雅。
2. 系统层面：让自己维护起来更方便。
3. 接口层面：查询问题更直观，更轻松，为自己留证据，避免推诿扯皮。
4. 网络层面：及时发现问题，及时进行处理。

------

**问：建议使用最大的 Exception 还是针对不同的情况抛各种异常？'我见过5年以上的开发工程师，对空指针的处理还是不到位' 这句能详细说明一下情况吗？如何针对性的对某些可预判的错误做处理？**

**答：**大 try 建议使用 Exception，这样能包住所有的异常，除了 error 外，因为 error 本身我们也不用处理。大 try 里面的一些需要特别说明的或是特别处理的就要看情况抛出了，一般经过封装后，抛出我们自定义的异常，上层接收后，进行二次处理和转化，直到最外层的调用处，由最外层调用者决定最终如何处理。

![enter image description here](http://images.gitbook.cn/ff123ce0-1bc6-11e8-be31-55e3e5673003)

可以参考这种写法，或者在 Exception 的 try 里，根据不同的 message 进行处理。

对空指针的处理还是不到位，指的是缺乏空指针判断意识。只要在使用对象之前，进行对象的非空判定，基本就能杜绝空指针的问题。有的程序员不具有这种意识，往往就会栽更头。我见到最惨的就是因为空指针引咎辞职，说以说养成一个好的开发习惯，受益终身。用我们之前一个同事的话说：一般空指针处理不好的程序员不是一个合格的 Java 程序员。

参数的问题就更得判断了，业务逻辑不通处理方式不同，但是不处理是不行的，建议处理得当，一个有经验的程序员就是因为他考虑问题比较全面，遇到的坑多了，自己就不会入坑了。另外异常处理一定要和日志结合起来使用，日志记录的约优雅，维护起来越轻松。日志的问题一下也说不清楚，本质是为了发现问题，查询问题和解决问题。

------

**问：全局异常检测是否只是针对于业务层面或者数据层为错校验没有的情况下的处理办法，除此之外还有什么别的优点吗？**

**答：**全局异常是不是指的我们说的大 try，如果是大 try 的话，他的有点主要是为了兜底。为了避免一些程序运行过程中产生的未知异常，可以及时发现，并能友好的处理掉。aop 的那种项目里一般做的是统一的处理，比如404页面等等。异常处理也是一个需要经验积累的东西，就想老酒一样，时间越长越纯。

对于初级程序员，需要了解异常处理的重要性和对项目的影响，多看项目里的代码，多学，慢慢地积累经验就好了。主要是了解本质。



#### 文章实录

**异常**是指程序在运行过程中由于外部问题（如硬件错误、输入错误）等导致的程序异常事件。

**异常处理**是编程语言或计算机硬件里的一种机制，用于处理软件或信息系统中出现的异常状况（即超出程序正常执行流程的某些特殊的情况）。

**本场 Chat 将从以下几个方面介绍异常处理相关知识。**

1. 为什么需要异常处理？
2. 你了解异常处理吗？
3. 异常处理有哪些原则？
4. Java 异常处理机制介绍；
5. 异常处理对项目质量的影响；
6. 异常处理对工资待遇的影响。

### 为什么需要异常处理？

在项目开发的过程中，即使程序员把代码写得尽善尽美，在系统的运行过程中仍然会遇到一些问题。因为很多问题不是靠代码能够避免的，比如：客户输入数据的格式，读取文件是否存在，网络是否始终保持通畅等等。

程序运行时，发生的不被期望的事件，它阻止了程序按照程序员的预期正常执行，或者说程序未按照期望的流程执行，这就导致了异常的发生。

异常发生时，如果不进行任何处理，势必导致程序不能正常运行，不能达到我们所期望的结果。

因此在项目开发过程中，异常处理机制是非常重要而且也是非常必要的，它能使我们对程序运行的结果进行控制及异常发生时进行有效地处理，以达到程序的预期效果。

**下面通过“用户注册”的例子说明为什么需要异常处理。**

用户注册的用户名称要求长度不能超过32位。

我们考虑一种情况：用户张三，输入的用户名称为“zhonghuarenmingongheguo*zhangsan*666”，长度超过32位。

假设数据库中用户表的用户名称字段最大长度为32位，那么如果不考虑异常情况，势必在用户提交注册后，保存到数据库的时候，数据库一定会报错“value too large for column”。

这样的话，程序没有做异常处理的话，程序产生的异常会直接反馈到前端界面上，用户体验极差。因此我们需要对这类异常进行有效的处理，以保证良好的用户体验。

### 你了解异常处理吗？

异常处理是每个项目里非常重要的一部分，项目质量的高低好坏，与异常处理水平的高低好坏紧密相连。最近发现有很多开发者对 Java 异常处理没有自己的认识和标准，有的具备2-3年开发经验却对异常处理仍然没有深刻地认识。

**下面我列举几种程序的场景，供大家参考。**

1. 通篇的程序没有任何异常处理，任程序自生自灭，不做任何处理。
2. 程序使用 throw 输出错误给用户。
3. 程序使用 try...catch 进行一小段代码的包括，catch 里只打印了堆栈信息，或者 catch 里什么都没做，或者把异常 throws 出去。
4. 程序使用 try...catch 进行整段代码的包括，catch 里只打印了堆栈信息，或者 catch 里什么都没做，或者把异常 throws 出去。
5. 程序使用 try...catch 进行整段代码的包括，catch 里只打印了堆栈信息，并且输出了相应的错误日志。
6. 程序使用 try...catch 进行整段代码的包括，catch 里只打印了堆栈信息，输出了相应的错误日志，并且 throws 了自定义异常，选择友好的提示信息给用户展示。
7. 程序使用 try...catch 进行整段代码的包括，输出了相应的错误日志，如果不是最外层调用则 catch 里不打印堆栈信息，并且 throws 了自定义异常，同时把堆栈信息带出去，最外层调用方，根据自定义异常，选择友好的提示信息给用户展示，并且打印出异常的堆栈信息。即程序根据实际情况对异常进行了综合处理，并通过日志进行了异常的详细记录，以便核查问题。

毫无疑问，第7种是程序进行异常处理的最有效手段。

### 异常处理有哪些原则

既然我们知道异常处理对项目或者程序意义重大，那么在面对具体的项目情况，面对异常场景时，想要有效地进行异常处理，这里面有哪些原则呢？

我总结为以下**“三大异常处理原则”**。

**1.程序层面异常处理原则**

- 要避免使用异常处理代替错误处理；

有的人写代码会用异常处理来做判断逻辑或者做业务逻辑处理，这样会降低程序的清晰性，并且效率低下。

- 处理异常不可以代替简单测试，只能在异常情况下使用异常机制；
- 不要进行小粒度的异常处理，应该将整个业务代码包装在一个 try 语句块中；
- 异常往往在高层处理，且不能忽视每一个异常

**2.需求层面异常处理原则**

- 搞清楚业务边界，用代码防止异常；

在写代码之前，就要明确业务需求，同时了解该需求功能的边界，写代码的时候，用专门的代码来防止异常的发生。

- 给用户友好的提示；

在发现异常情况后，代码进行特殊处理后，给用户友好的提示，不能直接抛出异常到用户界面。

- 程序员不定义业务异常流程。

业务需求是通过代码来实现的，程序员一般不了解业务，因此在写某个业务需求过程中，发现处理不了的异常，一定要与客户或者产品负责人来确定异常流程，从业务逻辑层面，考虑异常处理的业务逻辑，程序员不去定义业务异常流程。

**3.接口层面异常处理原则**

- 输入、输出参数校验；

接口服务提供方一般要对输入参数做校验，校验通过后，才会提供服务。接口调用方一般要对输出参数做校验，校验通过后，才会使用服务。

- 请求和响应的日志记录；

接口要对请求和响应做好日志记录，这在后期遇到问题反查的时候特别重要。

- 接口超时等异常的业务处理。

接口开发时，经常会遇到接口超时等网络问题，因此在开发的过程中，尤其要处理好这些异常。比如：超时反查机制，超时重发机制等。

### Java 异常处理机制介绍

Java 异常类层次如下图（Java 是采用面向对象的方式来处理异常的）。

![Java异常类层次](http://images.gitbook.cn/7e0991c0-1865-11e8-8087-9b6b3b447adf)

从上图，可以看出所有的异常跟错误都继承于 Throwable 类，也就是说所有的异常都是一个对象。Exception 类是 Throwable 类的子类，除了 Exception 类外，Throwable 还有一个子类 Error 。

**1.Error（错误）**

Java 程序通常不捕获错误。错误一般发生在严重故障时，它们在 Java 程序处理的范畴之外。Error 用来指示运行时环境发生的错误。

例如：JVM 运行时出现的 OutOfMemoryError 以及 Socket 编程时出现的端口占用等程序无法处理的错误。程序一般不会从错误中恢复。

**2.Exception（异常）**

所有的异常类是从 java.lang.Exception 类继承的子类，异常可分为运行时异常跟编译异常。

- 运行时异常（即 RuntimeException 及其之类的异常）

这类异常在代码编写的时候不会被编译器所检测出来，是可以不需要被捕获即可以不处理，但是我们可以编写代码处理（使用 try…catch…finally）这样的异常。对于这些异常，我们应该修正代码，而不是去通过异常处理器处理，这样的异常发生的原因多半是代码写的有问题（一般的做法是进行判断后处理）。

常见的 RUNtimeException 有：NullpointException（空指针异常）、ClassCastException（类型转换异常）、IndexOutOfBoundsException（数组越界异常）等。

- 编译异常（RuntimeException 以外的异常）

这类异常在编译时编译器会提示需要捕获，如果不进行捕获则编译错误。在方法中要么用 try…catch 语句捕获它并处理，要么用 throws 子句声明抛出它，否则编译不会通过。这样的异常一般是由程序的运行环境导致的。因为程序可能被运行在各种未知的环境下，而程序员无法干预用户如何使用他编写的程序，于是程序员就应该为这样的异常时刻准备着。

常见编译异常有：IOException（流传输异常）、SQLException（数据库操作异常）等。

**3.Java处理异常的机制**

抛出异常以及捕获异常 ，一个方法所能捕捉的异常，一定是 Java 代码在某处所抛出的异常。简单地说，异常总是先被抛出，后被捕捉的。

- 抛出异常

在执行一个方法时，如果发生异常，则这个方法生成代表该异常的一个对象，停止当前执行路径，并把异常对象提交给 JRE。

- 捕获异常

JRE 得到异常后，寻找相应的代码来处理该异常。JRE 在方法的调用栈中查找，从生成异常的方法开始回溯，直到找到相应的异常处理代码为止。

- 五个关键字：try、catch、finally、throws、throw。

  ![异常处理5个关键字](http://images.gitbook.cn/92e79620-18b8-11e8-8087-9b6b3b447adf)

**异常处理是通过 try…catch…finally 语句实现的。代码结构如下。**

```
try{
    ......    //可能产生异常的代码
}
catch( ExceptionName1 e ){
    ......    //当产生ExceptionName1型异常时的处置措施
}
catch( ExceptionName2 e ){
    ......     //当产生ExceptionName2型异常时的处置措施
}  
[ finally{
   ......     //无论是否发生异常，都无条件执行的语句
 }  ]

```

### 异常处理对项目质量的影响

如果一个项目里的各个功能在异常处理方面下的功夫不够，势必会极大地影响项目的质量。

下面我列举几个例子，供大家参考。

**1.空指针异常。**

我见过5年开发经验的程序员对空指针居然不做任何处理，空指针的处理是 Java 程序员的基本功，如果连这个都处理不了，真的不能算是合格的 Java 程序员。

空指针的处理很简单，就是在使用一个对象的时候，记得加一层非空判断就ok了，代码如下所示。

```
if (list != null && list.size() > 0) {
    user = list.get(0);
}

```

**2.网络超时异常。**

在实际开发项目中，会使用到很多接口服务，比如 WebService 或者 HTTP 接口，这些服务的调用很有可能会导致网络超时等异常，只要发生超时异常，如果不进行处理的话，势必会影响所涉及的功能，进而影响项目质量。

网络超时异常，一般会设置接口超时时间，如果超时了，开发测试过程中去优化服务提供方的代码，看看是不是有不合理的查询或者低效率的查询。作为服务调用方来说，接口超时后，一般会进行接口重发调用，如果接口提供方提供了服务执行结果查询接口的话，那么服务调用方应该在超时处理时，先调用服务执行结果查询接口，根据查询的结果再来确定是否要重新发起服务调用。网络超时处理机制如下图所示。

![网络超时处理机制](http://images.gitbook.cn/782ae5d0-18b3-11e8-8087-9b6b3b447adf)

**3.友好的用户体验。**

如果后台程序出现异常后，不做任何处理，直接把异常堆栈显示到用户界面里，这种项目的质量是极其差的，尤其是刚做项目的新人要特别注意。

用户体验还包括用户的输入项，及时做校验，及时提醒给用户，除了在前端界面使用 JavaScript 校验外，提交到后端的信息也要再做一次校验，防止程序提交。

**4.异常处理与日志打印。**

异常处理的同时一定要配合日志使用，用日志记录下来4个 W：What、Where、When、Why，即什么出了错（或者错误是什么），哪里出了错，什么时间出的错，错误产生的原因。

如果有异常处理，但是没有日志记录，那么我们遇到问题，反查的时候就很棘手，无从下手。记录日志一般用 log4j，把日志记录到特定的文件里，而不建议使用 e.printStackTrace() 打印堆栈。

另外最不可取的就是程序代码 catch 了异常，catch 块里什么都没写。这样功能发生异常后，我们根本无法察觉。如下代码所示。

```
try{
    Do something
}catch(Exception e){
   //此处无任何代码（最不可取的代码）
}

```

下面我们举个小例子，来看下 Java 代码，如何合理地处理异常。一个推荐的异常处理实例，如下面代码所示：

![Java异常最佳实践](http://images.gitbook.cn/d70b1570-18c2-11e8-8087-9b6b3b447adf)

在最外层调用处，处理 ServiceException，打印堆栈信息，并转化成友好的提示信息 message 返回给用户界面，请见下面代码。

```
catch (ServiceException e) {
    LOGGER.error("根据姓名查询用户失败" + e.toString());
    message = "根据姓名查询用户失败，姓名为：" + userName;
}

```

### 异常处理对工资待遇的影响

一个程序员的异常处理能力的高低与项目质量的高低紧密相连。因此，一个优秀的程序员，不仅要能完成项目需求和功能，还需对业务逻辑的异常处理要专业。

有的程序员写的代码只能在正常环境下，正常流程下，程序可以正常执行。凡是稍微与预定的流程不吻合，程序就会出现各种问题，导致程序的功能无法使用。也就是说，程序员在完成业务功能的同时，没有考虑清楚边界。

在项目开发中，一个优秀的程序员和一个新手的区别就是对业务的理解，然后业务转化成代码，并且能考虑各种异常流程和优良的用户体验。

如果能做到在项目中对异常的合理和全面地处理，那么就有理由要求提高自己的工资待遇（可以提高20-30%），因为只有优秀的程序员才能做到面面俱到。



