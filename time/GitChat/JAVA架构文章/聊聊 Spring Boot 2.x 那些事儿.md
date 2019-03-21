## 内容提要

内容提要：

- 关于使用Spring Data Reactive时，提到的“用了 Reactivey 原来的 spring 事务管理就不好用了”，能否再详细介绍一下？另外，应用有强依赖事务，有没有对应的解决方案？
- Spring Data Reactive Repositories 不支持 MySQL，进一步也不支持 MySQL 事务，怎么办？
- 2.0与1.0相比，增加了哪些功能特性？生产应用中你推荐使用spring boot 的哪个版本，为什么？
- Spring Boot 新手的话，建议如何入门？
- 之前用Spring Boot做项目的时候发现Spring Boot+maven创建多模块然后打包过程很麻烦，所以想问一下，Spring Boot是不是不建议多模块构建项目？
- Spring Boot 1.x 如何平滑迁移到Spring Boot 2.x？
- 响应式能改我们带来哪些明显的好处？
- 我公司现在用的Spring Boot1.5.4，暂时也稳定，有必要为了版本升级而升级吗？因为每个版本都提到修正某些bug？
- 如何从Spring Boot过渡到Spring Cloud？两者有何区别？
- Spring Boot + themyleaf怎么做到前后台分离？
- jar包部署的情况下，用内置tomcat好吗，如果需要修改tomcat配置一般怎么做？
- 能否讲讲你是怎样理解Spring Boot的理念？Spring Boot跟groovy怎样结合？
- Spring Boot 前后端分离，页面的提示信息怎么返回，怎么验证用户身份？
- Spring Boot2.0相比1.0最大优化点是什么？
- 关于 Spring flux的应用你有什么计划？
- 通过命令行可以让Spring Boot加载不同的配置文件启动，在IDE里面可以实现吗？
- 多个模块分给多个人开发的时候，怎么处理？要一个人把所有模块都启动起来吗，例如Spring Cloud这样多模块的？

------

**问：关于使用Spring Data Reactive时，提到的“用了 Reactivey 原来的 spring 事务管理就不好用了”，能否再详细介绍一下？另外，应用有强依赖事务，有没有对应的解决方案？**

**答：**这个问题提的很好，我们先看看这张图。Spring Boot 2.0 这里有两条不同的线分别是： Spring Web MVC -> Spring Data，Spring WebFlux -> Spring Data Reactive。

![enter image description here](http://images.gitbook.cn/e2896b20-bfd7-11e7-81d4-83c6d35804de)

所以这个问题的答案是，如果使用 Spring Data Reactive ，原来的 Spring 针对 Spring Data （JDBC等）的事务管理肯定不起作用了。因为原来的 Spring 事务管理（Spring Data JPA）都是基于 ThreadLocal 传递事务的，其本质是基于阻塞 IO 模型，不是异步的。但 Reactive 是要求异步的，不同线程里面 ThreadLocal 肯定取不到值了。自然，我们得想想如何在使用 Reactive 编程是做到事务，有一种方式是回调方式，一直传递 conn ：newTransaction(conn ->{})

需要传递的原因是，每次操作数据库也是异步的，所以 connection 在 Reactive 编程中无法靠 ThreadLocal 传递了，只能放在参数上面传递。虽然会有一定的代码侵入行。进一步，也可以 kotlin 协程，去做到透明的事务管理，即把 conn 放到 协程的局部变量中去。

------

**问：Spring Data Reactive Repositories 不支持 MySQL，进一步也不支持 MySQL 事务，怎么办？**

**答：**这个问题其实和第一个问题也相关。为什么不支持 MySQL，即 JDBC 不支持。大家可以看到 JDBC 是所属 Spring Data 的。所以可以等待 Spring Data Reactive Repositories 升级 IO 模型，去支持 MySQL。也可以和上面也讲到了，如何使用 Reactive 编程支持事务。

------

**问：2.0与1.0相比，增加了哪些功能特性？生产应用中你推荐使用spring boot 的哪个版本，为什么？** 、 **答：**目前马上出来 M6 版本。新增的功能特性大致如下：

1. 嵌入式容器支持 Reactive。
2. Java 9 & Spring Framework 5 支持。
3. 新增了 Reactive Spring Web 框架，即 WebFlux 支持（文章最后一节都在讲这个）。
4. 新增 Spring Data Reactive Repositories。
5. ES 支持 5.0（终于等到了，基于 ES 2.0 我写了很多文章 https://www.bysocket.com/?tag=elasticsearch）。
6. 还有些小的 bug 处理和优化就不一一列举了。

具体大家可以看看 Spring Boot 的 milestones： <https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes>。

------

**问：Spring Boot 新手的话，建议如何入门？**

**答：**这个问题，考虑入门的话。跑个 Hello World 算入门吗？肯定不算，入门需要对下面知识点去了解：

1. Spring Boot 是什么（包括 特性、Starter 组件、应用场景）？
2. 快速入门 Hello World。
3. Spring Boot 工程构建（包括 工程创建、结构目录、运行方式）

- 安装使用 Spring Boot （Maven 方式、Gradle 方式、Spring Boot CLI）

------

**问：之前用Spring Boot做项目的时候发现Spring Boot+maven创建多模块然后打包过程很麻烦，所以想问一下，Spring Boot是不是不建议多模块构建项目？**

**答：** Spring Boot 本身和 Maven 模块化工程没多大关系。我重新理解下，你的问题应该是比如一个大工程为 xx-web、xx-service、xx-dao 等时，xx-web 依赖其他模块，想要运行这个工程时，打包过程很麻烦？如果是的话，这多半是 Maven 的知识点，不是 Spring Boot ，那看看是否这样处理。 比如需要建一个 xx-parent 工程：

```
xx-parent
             |-- pom.xml (pom)
             |
             |-- xx-dao
             |        |-- pom.xml (jar)
             |
             |-- xx-service
             |        |-- pom.xml (jar)
             |
             |-- xx-web
                      |-- pom.xml (war)    

```

这样依赖关系变成了，xx-service --> xx-dao 、xx-web --> xx-service 。编译项目直接在 parent 底层目录，mvn clen install 即可。这样有什么好处？

1. 方便重用，比如另一个 web 需要引用 xx-service 即可。
2. 模块化，使得项目结构更清晰、当项目越来越复杂，这样的结构有助于开发节省时间。

------

**问：Spring Boot 1.x 如何平滑迁移到Spring Boot 2.x？**

**答：**任何版本在生产应用都需要升级。一般的步骤大同小异，大版本升级要考虑它的变化性大小。常见考虑的点：

1. 配置兼容：比如 application properties 配置的属性名会更改。
2. API 兼容：比如有些类会包路径更改。
3. 依赖 jar 兼容。

在微服务模块化的情况下，先升级非核心模块，然后进行平滑灰度升级。比如这个模块有 10 个生产服务，先升级一个，然后监控这一个。一批一批升级。

------

**问：响应式能改我们带来哪些明显的好处？**

**答：** Spring 有 WebFlux ，说明作为 Java 框架支持响应式编程，肯定有它强烈的需求场景。这样的带来的好处，Javaer 或 Java 应用一方面：

1. 代码抽象层次低。
2. 代码编写更灵活。

另一方面，满足一些对动态性要求更高场景。比如海量的实时事件处理需要，高互动的用户体验的场景。比如股票图功能，所以综上，适合技术场景：

1. 请求海量实时调用。
2. 高并发消息处理。
3. 弹性网络。

------

**问：我公司现在用的Spring Boot1.5.4，暂时也稳定，有必要为了版本升级而升级吗？因为每个版本都提到修正某些bug？**

**答：**和上面讲的一样，每次小版本升级都是 fix bugs。大版本升级，比如 2.0 新增了很多：

1. 新增 Spring Data Reactive Repositories。
2. Java 9 & Spring Framework 5 支持。

3.新增了 Reactive Spring Web 框架，即 WebFlux 支持。

所以升级需谨慎，建议不马上将生产的版本升级到 2.0，等小版本升级到差不多再升级的 2.x 版本即可，别做小白鼠。

------

**问：如何从Spring Boot过渡到Spring Cloud？两者有何区别？**

**答：** Spring Cloud 是可构建微服务，是微服务开发和治理解决方案（一套组件）。Spring Boot 是 Spring 快速开发框架。

Spring Cloud 包括了 Spring Boot。Boot 可以开发某个微服务，Cloud 关注全局服务治理。离开 Cloud，Boot 可以开发单应用。Cloud 离不开 Boot ，是基于 Boot 实现的。

Spring < Spring Boot < Spring Cloud。

所以学习 Spring Boot 需要兼顾 Spring Framework 知识。学好前面两个，再学 Cloud。

------

**问：Spring Boot + themyleaf怎么做到前后台分离？**

**答：** Ajax 形式算前后端分离。前后端分离，是从 MVC 演进过来的，普通的 MVC ：Model-View-Controller 组成。Ajax 模式算改进版的 MVC，也称前后端分离。即输入的是 AJAX 请求，输出的是 JSON 数据。

我司是这样的，前端 React 自己一个 web 服务，通过 HTTP OVER JSON 访问后端 REST 服务，获取数据然后渲染到页面。

------

**问：jar包部署的情况下，用内置tomcat好吗，如果需要修改tomcat配置一般怎么做？**

**答：**企业实践中，内置TOMCAT够用。tomcat 配置：在下面链接 ctrl + f (command + f)，搜 tomcat： <https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html>。

![enter image description here](http://images.gitbook.cn/525060d0-bfd8-11e7-81d4-83c6d35804de)

关于部署，我简单说下：Win 服务器后台运行：推荐使用 AlwaysUp。Linux 服务器后台运行： nohup 命令。用途：不挂断地运行命令。

------

**问：能否讲讲你是怎样理解Spring Boot的理念？Spring Boot跟groovy怎样结合？**

**答：**具体 groovy 整合，我没写过教程。请关注： <https://www.bysocket.com/?page_id=1639>，我到时候写写。

Spring Boot 的理念：

Spring Boot 是 Spring 的快速脚手架 0 配置，Spring Boot 提供了新的编程模式，让开发 Spring 应用变得更加简单方便。Spring Boot （Boot 顾名思义，是引导的意思）框架是用于简化 Spring 应用从搭建到开发的过程。应用开箱即用，只要通过一个指令，包括命令行 java -jar 、SpringApplication 应用启动类 、 Spring Boot Maven 插件等，就可以启动应用了。

Spring Boot 常用特性如下：

1. SpringApplication 应用类。
2. 自动配置。
3. 外化配置。
4. 内嵌容器。
5. Starter 组件。

还有对日志、Web、消息、测试及扩展等支持。

------

**问：Spring Boot 前后端分离，页面的提示信息怎么返回，怎么验证用户身份？**

**答：**前后端分离，是从 MVC 演进过来的，普通的 MVC ：Model-View-Controller 组成。Ajax 模式算 改进版的 MVC，也称 前后端分离。即 输入的是 AJAX 请求，输出的是 JSON 数据。我司是这样的，前端 React 自己一个 web 服务，通过 HTTP OVER JSON 访问 后端 REST 服务，获取数据然后渲染到页面。这是第十问题回答。接着页面的提示信息怎么返回？

肯定根据 API 的响应码（一般说错误码）：200 成功。具体《Spring Boot HTTP over JSON 的错误码异常处理》。地址： <https://www.bysocket.com/?p=1692>。

怎么验证用户身份？包括认证和授权，两块内容。具体《Spring Boot 之 RESRful API 权限控制》给出了方案： <https://www.bysocket.com/?p=1080>。

------

**问：Spring Boot2.0相比1.0最大优化点是什么？**

**答：**我认为Spring Boot2.0相比1.0最大优化点是：

1. 新增 Spring Data Reactive Repositories。
2. Java 9 & Spring Framework 5 支持。
3. 新增了 Reactive Spring Web 框架，即 WebFlux 支持。

不然不可能马上跳到 2.0 版本，大改动。

------

**问：关于 Spring flux的应用你有什么计划？**

**答：**还是从试用场景出发：

1. 请求海量实时调用。
2. 高并发消息处理。
3. 弹性网络。

那么我会考虑将某模块业务场景升级重构试试水，这样改动影响也不算大。

------

**问：通过命令行可以让Spring Boot加载不同的配置文件启动，在IDE里面可以实现吗？**

**答：**这是 spring boot 配置相关的问题。多环境配置，关键配置属性是：

```
#Spring Profiles Active
spring.profiles.active=dev

```

命令行的优先级比.properties 文件高，自然你在 IDE 配置，启动可以的。在这里我详细介绍了《Spring Boot 配置文件 》： <https://www.bysocket.com/?p=1786>。

------

**问：多个模块分给多个人开发的时候，怎么处理？要一个人把所有模块都启动起来吗，例如Spring Cloud这样多模块的？**

**答：**多个模块，不代表多个服务应用。多个模块是工程模块的划分，工程层次结构的体系。多个模块可以组成一个服务工程，并提供服务。

![enter image description here](http://images.gitbook.cn/943b60d0-bfd8-11e7-81d4-83c6d35804de)

老东家这么多模块也就构成了一个应用。所以，模块化，使得项目结构更清晰。

------

本文首发于GitChat，未经授权不得转载，转载需与GitChat联系。

------

在此感谢[异步社区](http://www.epubit.com.cn/)为本次活动提供的赠书《Java核心技术 卷II：高级特性（第10版•英文版）（上下册）》。

[异步社区](http://www.epubit.com.cn/)是人民邮电出版社旗下IT专业图书旗舰社区，也是国内领先的IT专业图书社区，致力于优质学习内容的出版和分享，实现了纸书电子书的同步上架。



![enter image description here](http://images.gitbook.cn/9dba8fa0-bfd8-11e7-81d4-83c6d35804de)



## 文章实录

```
本文目录：

即将的 Spring 2.0
- Spring 2.0 是什么
- 开发环境和 IDE
- 使用 Spring Initializr 快速入门
Starter 组件
- Web：REST API & 模板引擎
- Data：JPA -> H2
- ...
生产指标监控 Actuator
内嵌式容器 Tomcat / Jetty / Undertow
Spring 5 & Spring WebFlux

```

大家看到目录，这么多内容，简直一本书的节奏。如果很详细，确实是。 可是只有一篇文章我大致讲讲每个点，其是什么，其主要业界的使用场景是什么，然后具体会有对应的博客教程。恩，下面我们聊聊 Spring 2.0。

### 一、即将的 Spring 2.0

spring.io 官网有句醒目的话是：

BUILD ANYTHING WITH SPRING BOOT

Java 程序员都知道 Spring 是什么，Spring 走过了这么多个年头。Spring 是 Java 应用程序平台开发框架，肯定也是跨平台的。同样，它也是 Java EE 轻量级框架，为 Java 开发这提供了全面的基础设施支持。

从 SSH（Strusts / Spring / Hibernate） 到 SSM（Spring MVC / Spring / Mabtis） ，到现在一个 S （Spring）就够了的年代。 可见 Spring 越来越重要了。那 Spring Boot 是什么？

#### 1. Spring 2.0 是什么

先看看世界上最好的文档来源自官方的《Spring Boot Reference Guide》，是这样介绍的：

```
Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can “just run”... Most Spring Boot applications need very little Spring configuration.

```

Spring Boot(英文中是“引导”的意思)，是用来简化Spring应用的搭建到开发的过程。应用开箱即用，只要通过 “just run”（可能是 java -jar 或 tomcat 或 maven插件run 或 shell脚本），就可以启动项目。二者，Spring Boot 只要很少的Spring配置文件（例如那些xml，property）。

因为“习惯优先于配置”的原则，使得Spring Boot在快速开发应用和微服务架构实践中得到广泛应用。

Spring 目前是 2.0.0 M5 版本，马上 2.0 release 了。如图：Spring 2.0 架构图

![enter image description here](http://images.gitbook.cn/87db53c0-b936-11e7-b969-cb3cfaf54002)

最明显的是 Reactor 模型。

插个小故事，曾经有人问：Spring 这种阻塞的编程技术以及金字塔的结构已经开始被很多公司所摈弃？

我的回答自然是那个图。Spring Boot 2.0 基于 Spring 5 Framework ，提供了 异步非阻塞 IO 的响应式 Stream 、非堵塞的函数式 Reactive Web 框架 Spring WebFlux。本文最后我会详细介绍的。

```
响应式 Stream 
http://www.reactive-streams.org/
webflux 官方文档
https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#spring-webflux

```

#### 2. 开发环境和 IDE

大致介绍了 Spring Boot 2.0 是什么，下面我们快速入门下 Spring Boot 2.0。常言道，磨刀不误砍柴工砍柴工。在搭建一个 Spring Boot 工程应用前，需要配置好开发环境及安装好开发工具：

- JDK 1.8+ Spring Boot 2.x 要求 JDK 1.8 环境及以上版本。另外，Spring Boot 2.x 只兼容 Spring Framework 5.0 及以上版本。
- Maven 3.2+ 为 Spring Boot 2.x 提供了相关依赖构建工具是 Maven，版本需要 3.2 及以上版本。使用 Gradle 则需要 1.12 及以上版本。Maven 和 Gradle 大家各自挑选下喜欢的就好。
- IntelliJ IDEA IntelliJ IDEA （简称 IDEA）是常用的开发工具，也是本书推荐使用的。同样使用 Eclipse IDE 自然也是可以的。

这里额外介绍下，对于 Java 新手或者刚刚认识 Spring 的小伙伴。Spring Boot CLI 是一个学习 Spring Boot 的很好的工具。Spring Boot CLI 是 Spring Boot Commad Line 的缩写，是 Spring Boot 命令行工具。在 Spring Boot CLI 可以跑 Groovy 脚本，通过简单的 Java 语法就可以快速而又简单的学习 Spring Boot 原型。

Spring Boot CLI 具体快速入门看下我写的地址：https://www.bysocket.com/?p=1982

那最常用，最推荐的入门当然是使用 Spring Initializr 。下面介绍下如何使用 Spring Initializr。

#### 3. 使用 Spring Initializr 快速入门

打开 start.spring.io 进行很快速的，Spring Boot 骨架工程生成吧。

Spring 官方提供了名为 Spring Initializr 的网站，去引导你快速生成 Spring Boot 应用。网站地址为：https://start.spring.io，操作步骤如下：

第一步，选择 Maven 或者 Gradle 构建工具，开发语言 Java 、Kotlin 或者 Groovy，最后确定 Spring Boot 版本号。这里默认选择 Maven 构建工具、Java 开发语言和 Spring Boot 2.0.0。

第二步，输入 Maven 工程信息，即项目组 `groupId` 和名字 `artifactId`。这里对应 Maven 信息为：

- groupId：demo.springboot
- artifactId：spring-boot-quickstart 这里默认版本号 version 为 0.0.1-SNAPSHOT 。三个属性在 Maven 依赖仓库是唯一标识的。

第三步，选择工程需要的 Starter 组件和其他依赖。最后点击生成按钮，即可获得骨架工程压缩包。如图 ：

![enter image description here](http://images.gitbook.cn/c46cb2c0-b936-11e7-a2ba-479fe1d387e3)

在对应的 *Application 增加对应的代码如下：（这来自是官方 demo）

```
@Controller
@EnableAutoConfiguration
public class Application {

    @RequestMapping("/")
    @ResponseBody
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }
}

```

在 IDEA 中直接执行应用启动类，来运行 Spring Boot 应用，会得到下面成功的控制台输出：

```
... 省略
2017-10-15 10:05:19.994  INFO 17963 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http)
2017-10-15 10:05:20.000  INFO 17963 --- [           main] demo.springboot.QuickStartApplication    : Started QuickStartApplication in 5.544 seconds (JVM running for 6.802)

```

访问地址 localhost:8080 ，成功返回 "Hello World!" 的字符串。

这就是 Spring Boot 使用，是不是很方便。介绍下工程的目录结构：

```
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── demo
    │   │       └── springboot
    │   │           └── Application.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── demo
                └── springboot
                    └── QuickstartApplication Tests.java

```

这是默认的工程结构，java 目录中是编写代码的源目录，比如三层模型大致会新建三个包目录，web 包负责  web 服务，service 包负责业务逻辑，domain  包数据源服务。对应 java 目录的是 test 目录，编写单元测试的目录。

resources 目录会有 application.properties 应用配置文件，还有默认生成的 static 和 templates 目录，static 用于存放静态资源文件，templates 用于存放模板文件。可以在 application.properties 中自定义配置资源和模板目录

### 二、Starter 组件

什么是 Starter 组件？

Starter 组件是可被加载在应用中的 Maven 依赖项。Spring Boot 提供了很多 “开箱即用” 的 Starter 组件。只需要在 Maven 配置中添加对应的依赖配置，即可使用对应的 Starter 组件。例如，添加 `spring-boot-starter-web` 依赖，就可用于构建 RESTful Web 服务，其包含了 Spring MVC 和 Tomcat 内嵌容器等。

开发中，很多功能是通过添加 Starter 组件的方式来进行实现。下面都是常用的组件，还有很多事务、消息、安全、监控、大数据等支持，这里就不介绍了。大家可以同理可得去学习哈。

#### 1. Web：REST API & 模板引擎

在 Web 开发中，常见的场景有传统的 Web MVC 架构和前后端分离架构。

**Web MVC 架构**

Web MVC 模式很适合 Spring Boot 来开发，View 使用 JSP 或者其他模板引擎（默认支持：FreeMarker 、 Groovy 、 Thymeleaf 、 Mustache）。

传统模式比如获取用户，是从用户 view 层发送获取用户请求到 Spring Boot 提供的用户控制层，然后获取数据封装进 Model，最后将 model 返回到 View。因为 Spring Boot 基于 Spring ，所以 Spring 能做的，Spring Boot 就能做，而且更加方便，更近快速。

**前后端分离架构**

前后端分离架构，免不了的是 API 文档作为中间的桥梁。Spring Boot 很方便的开发 REST API，前端通过调用 REST API 获取数据。数据形式可能是 JSON 或者 XML 等。然后进行视图渲染，这里前端也有对应的前端模板引擎。其实 H5 ，PC，APP 都可采取类似的方式实现。这里的 API 文档可以使用 Swagger2 或者 APIDOC 来实现。

**那具体聊聊 REST API**

RESTful是什么？RESTful（Representational State Transfer）架构风格，是一个Web自身的架构风格，底层主要基于HTTP协议（ps:提出者就是HTTP协议的作者），是分布式应用架构的伟大实践理论。RESTful架构是无状态的，表现为请求-响应的形式，有别于基于Bower的SessionId不同。Spring Boot 的注解 `@RestController` 支持实现 RESTful 控制层。

那有个问题？权限怎么控制？ RESTful是无状态的，所以每次请求就需要对起进行认证和授权。

- 认证 身份认证，即登录验证用户是否拥有相应的身份。简单的说就是一个Web页面点击登录后，服务端进行用户密码的校验。
- 权限验证（授权） 也可以说成授权，就是在身份认证后，验证该身份具体拥有某种权限。即针对于某种资源的CRUD,不同用户的操作权限是不同的。 一般简单项目：做个sign（加密加盐参数）+ 针对用户的access_token 复杂的话，加入 SLL ，并使用OAuth2进行对token的安全传输。

**再聊聊模板引擎**

有人在我博客上评论 模板语言 现在没人用了吧。我们还是先了解下，什么是模板语言再说吧。常见的模板语言都包含以下几个概念：数据（Data）、模板（Template）、模板引擎（Template Engine）和结果文档（Result Documents）。

- 数据 数据是信息的表现形式和载体，可以是符号、文字、数字、语音、图像、视频等。数据和信息是不可分离的，数据是信息的表达，信息是数据的内涵。数据本身没有意义，数据只有对实体行为产生影响时才成为信息。
- 模板 模板，是一个蓝图，即一个与类型无关的类。编译器在使用模板时，会根据模板实参对模板进行实例化，得到一个与类型相关的类。
- 模板引擎 模板引擎（这里特指用于Web开发的模板引擎）是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的HTML文档。
- 结果文档 一种特定格式的文档，比如用于网站的模板引擎就会生成一个标准的HTML文档。

模板语言用途广泛，常见的用途如下：

- 页面渲染
- 文档生成
- 代码生成
- 所有 “数据+模板=文本” 的应用场景

所以大家看到这个用途，应该不会说没有用了吧。具体按模板语言 Thymeleaf 为例，使用如下

**pom.xml Thymeleaf 依赖**

使用模板引擎，就在 pom.xml 加入 Thymeleaf 组件依赖：

```
<!-- 模板引擎 Thymeleaf 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

```

**Thymeleaf 依赖配置**

在 Spring Boot 项目中加入 Thymeleaf 依赖，即可启动其默认配置。如果想要自定义配置，可以在 application.properties 配置如下

```
spring.thymeleaf.cache=true # Enable template caching.
spring.thymeleaf.check-template=true # Check that the template exists before rendering it.
spring.thymeleaf.check-template-location=true # Check that the templates location exists.
spring.thymeleaf.enabled=true # Enable Thymeleaf view resolution for Web frameworks.
spring.thymeleaf.encoding=UTF-8 # Template files encoding.
spring.thymeleaf.excluded-view-names= # Comma-separated list of view names that should be excluded from resolution.
spring.thymeleaf.mode=HTML5 # Template mode to be applied to templates. See also StandardTemplateModeHandlers.
spring.thymeleaf.prefix=classpath:/templates/ # Prefix that gets prepended to view names when building a URL.
spring.thymeleaf.reactive.max-chunk-size= # Maximum size of data buffers used for writing to the response, in bytes.
spring.thymeleaf.reactive.media-types= # Media types supported by the view technology.
spring.thymeleaf.servlet.content-type=text/html # Content-Type value written to HTTP responses.
spring.thymeleaf.suffix=.html # Suffix that gets appended to view names when building a URL.
spring.thymeleaf.template-resolver-order= # Order of the template resolver in the chain.
spring.thymeleaf.view-names= # Comma-separated list of view names that can be resolved.

```

Controller 的使用方式，和以前的 Spring 方式一致，具体的Tymeleaf 的语法糖，大家可以看看官方文档<http://www.thymeleaf.org/documentation.html>。本案例具体整合教程：<https://www.bysocket.com/?p=1973>。

#### 2. Data：JPA 、Mybatis

Data，顾名思义是数据。数据存储有 SQL 和 NoSQL：

- SQL：MySQL 、H2 等
- NoSQL：Redis、MongoDB、Cassandra、Elasticsearch 等 自然互联网常见使用的 SQL 关系型数据库是 MySQL。ORM 框架流行的有 Hibernate 和 Mybatis 等，JPA 底层实现使用 Hibernate。下面分别介绍下两者的使用方式

**Spring Data JPA 依赖 spring-boot-starter-data-jpa**

Spring Data JPA 是 Spring Data 的子项目，用于简化数据访问层的实现，可以方便的实现增删改查、分页、排序。还有很多常见的其他依赖：Spring Data MongoDB、Spring Data Redis 等。这里 Spring Boot 也有对应的 Starter 组件，名为 spring-boot-starter-data-jpa。

具体整合教程：https://www.bysocket.com/?p=1950

**Spring Boot Mybatis 依赖 mybatis-spring-boot-starter**

Mybatis 也是业界互联网流行的数据操作层框架。有两种形式进行使用 Mybatis。第一、纯 Annotation，第二、使用 xml 配置 SQL。个人推荐使用 xml 配置 SQL， SQL 和业务代码应该隔离，方便和 DBA 校对 SQL。二者 XML 对较长的 SQL 比较清晰。

虽然 XML 形式是我比较推荐的，但是注解形式也是方便的。尤其一些小系统，快速的 CRUD 轻量级的系统。这里可见，mybatis-spring-boot-starter依赖是非官方提供的。

Springboot 整合 Mybatis 的完整 Web 案例教程：<https://www.bysocket.com/?p=1610>

Spring Boot 整合 Mybatis Annotation 注解的完整 Web 案例教程：<https://www.bysocket.com/?p=1811>

另外，搜索常用 ES，spring-data-elasticsearch 是 Spring Data 的 Community modules 之一，是 Spring Data 对 Elasticsearch 引擎的实现。 Elasticsearch 默认提供轻量级的 HTTP Restful 接口形式的访问。相对来说，使用 HTTP Client 调用也很简单。

但 spring-data-elasticsearch 可以更快的支持构建在 Spring 应用上，比如在 application.properties 配置 ES 节点信息和 spring-boot-starter-data-elasticsearch 依赖，直接在 Spring Boot 应用上使用。

我也写了点系列博客在：<https://www.bysocket.com/?tag=elasticsearch>

还有很多组件无法一一介绍了，比如常用的 Redis 做缓存操作等。

### 三、生产指标监控 Actuator

Starter 组件 Actuator 提供了生产级监控的特性，可以使用 Actuator 来监控应用的一切。使用方式也很简单，加入对应的依赖，然后访问各个 HTTP 请求就可以获取需要的信息。具体提供的信息有：

```
autoconfig    自动配置相关信息
beans            Spring 容器中的 Bean 相关信息
configprops     配置项信息
dump            当前线程信息
env                当前环境变量信息
health            应用的健康信息
info            应用基本信息
metrics            性能指标信息
mappings        请求路径信息
trace            请求调用信息

```

至于可视化的话，可以使用 http/ssh/telnet 等方式拉监控数据到可视化 UI 上即可。进一步可以使用 prometheus 采集 springboot 应用指标，用 Grafana 可视化监控数据。

这里我师弟写一遍教程不错：<http://www.jianshu.com/p/7ecb57a3f326>

### 四、内嵌式容器 Tomcat / Jetty / Undertow

Spring Boot 运行的应用是独立的一个 Jar 应用，实际上在运行时启动了应用内部的内嵌容器，容器初始化 Spring 环境及其组件并启动应用。但我们也可以自定义配置 内嵌式容器，比如将 Tomcat 换成 Jetty。Spring Boot 能这样工作，主要靠下面两点：自动配置和外化配置。

**自动配置**

Spring Boot 在不需要任何配置情况下，就直接可以运行一个应用。实际上，Spring Boot 框架的 `spring-boot-autoconfigure` 依赖做了很多默认的配置项，即应用默认值。这种模式叫做 “自动配置”。Spring Boot 自动配置会根据添加的依赖，自动加载依赖相关的配置属性并启动依赖。例如，默认用的内嵌式容器是 Tomcat ，端口默认设置为 8080。

**外化配置**

Spring Boot 简化了配置，在 application.properties 文件配置常用的应用属性。Spring Boot 可以将配置外部化，这种模式叫做 “外化配置”。将配置从代码中分离外置，最明显的作用是只要简单地修改下外化配置文件，就可以在不同环境中，可以运行相同的应用代码。

所以，Spring Boot 启动应用，默认情况下是自动启动了内嵌容器 Tomcat，并且自动设置了默认端口为 8080。另外还提供了对 Jetty、Undertow 等容器的支持。开发者自行在添加对应的容器 Starter 组件依赖，即可配置并使用对应内嵌容器实例。具体操作如下：

第一步，将 tomcat 依赖排除：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

```

spring-boot-starter-web 依赖默认使用 Tomcat 作为内嵌容器

第二步，加入对应的 jetty 依赖（这里也可以是 Undertow 依赖）

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>

```

上面大致介绍了 Spring Boot 2.0 有的那些特性，也就是那些事，最后入门介绍下 WebFlux。

### 五、Spring 5 & Spring WebFlux

最后，Spring Boot 2.0 支持 Spring 5，Spring 5 具有了强大的特性：WebFlux/Reactor。所以最后来聊聊 WebFlux ，就算入个门吧。

对照下 Spring Web MVC ，Spring Web MVC 是基于 Servlet API 和 Servlet 容器设计的。那么 Spring WebFlux 肯定不是基于前面两者，它基于 Reactive Streams API 和 Servlet 3.1+ 容器设计。

**那 Reactive Streams API 是什么？**

先理解 Stream 流是什么？流是序列，是生产者生产，一个或多个消费者消费的元素序列。这种具体的设计模式成为发布订阅模式。常见的流处理机制是 pull / push 模式。背压是一种常用策略，使得发布者拥有无限制的缓冲区存储 item，用于确保发布者发布 item 太快时，不会去压制订阅者。

Reactive Streams （响应式流）是提供处理非阻塞背压异步流的一种标准。主要针对的场景是运行时环境（包括 JVM 和 JS）和网络。同样，JDK 9 java.util.concurrent 包提供了两个主要的 API 来处理响应流：

- Flow
- SubmissionPublisher<T>

**为啥只能运行在 Servlet 3.1+ 容器？**

大家知道，3.1 规范其中一个新特性是异步处理支持。

异步处理支持：Servlet 线程不需一直阻塞，即不需要到业务处理完毕再输出响应，然后结束 Servlet线程。异步处理的作用是在接收到请求之后，Servlet 线程可以将耗时的操作委派给另一个线程来完成，在不生成响应的情况下返回至容器。主要应用场景是针对业务处理较耗时的情况，可以减少服务器资源的占用，并且提高并发处理速度。

所以 WebFlux 支持的容器有 Tomcat、Jetty（Non-Blocking IO API） ，也可以像 Netty 和 Undertow 的本身就支持异步容器。在容器中 Spring WebFlux 会将输入流适配成 Mono 或者 Flux 格式进行统一处理。

#### Spring WebFlux 是什么

![enter image description here](http://images.gitbook.cn/8a0eb910-b937-11e7-bf6b-a3e311e5a596)

先看这张图，上面我们了解了容器、响应流。这里介绍下 Spring WebFlux 是什么？ Spring WebFlux 是 Spring 5 的一个新模块，包含了响应式 HTTP 和 WebSocket 的支持，另外在上层服务端支持两种不同的编程模型：

- 基于 Spring MVC 注解 @Controller 等
- 基于 Functional 函数式路由

下面是两个实现小案例，首先在 pom.xml 加入对应的依赖:

```
   <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

```

### 基于 Spring MVC 注解 RESTful API

官方案例很简单，如下：

```
@RestController
public class PersonController {

        private final PersonRepository repository;

        public PersonController(PersonRepository repository) {
                this.repository = repository;
        }

        @PostMapping("/person")
        Mono<Void> create(@RequestBody Publisher<Person> personStream) {
                return this.repository.save(personStream).then();
        }

        @GetMapping("/person")
        Flux<Person> list() {
                return this.repository.findAll();
        }

        @GetMapping("/person/{id}")
        Mono<Person> findById(@PathVariable String id) {
                return this.repository.findOne(id);
        }
}

```

但是 PersonRepository 这种 Spring Data Reactive Repositories 不支持 MySQL，进一步也不支持 MySQL 事务。所以用了 Reactivey 原来的 spring 事务管理就不好用了。jdbc jpa 的事务是基于阻塞 IO 模型的，如果 Spring Data Reactive 没有升级 IO 模型去支持 JDBC，生产上的应用只能使用不强依赖事务的。也可以使用透明的事务管理，即每次操作的时候以回调形式去传递数据库连接 connection。

Spring Data Reactive Repositories 目前支持 Mongo、Cassandra、Redis、Couchbase 。

如果应用只能使用不强依赖数据事务，依旧使用 MySQL ，可以使用下面的实现，代码如下：

```
@RestController
@RequestMapping(value = "/city")
public class CityRestController {

    @Autowired
    private CityService cityService;

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public Mono<City> findOneCity(@PathVariable("id") Long id) {
        return Mono.create(cityMonoSink -> cityMonoSink.success(cityService.findCityById(id)));
    }

    @RequestMapping(method = RequestMethod.GET)
    public Flux<City> findAllCity() {
        return Flux.create(cityFluxSink -> {
            cityService.findAllCity().forEach(city -> {
                cityFluxSink.next(city);
            });
            cityFluxSink.complete();
        });
    }

    @RequestMapping(method = RequestMethod.POST)
    public Mono<Long> createCity(@RequestBody City city) {
        return Mono.create(cityMonoSink -> cityMonoSink.success(cityService.saveCity(city)));
    }

    @RequestMapping(method = RequestMethod.PUT)
    public Mono<Long> modifyCity(@RequestBody City city) {
        return Mono.create(cityMonoSink -> cityMonoSink.success(cityService.updateCity(city)));
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    public Mono<Long> modifyCity(@PathVariable("id") Long id) {
        return Mono.create(cityMonoSink -> cityMonoSink.success(cityService.deleteCity(id)));
    }
}

```

findAllCity 方法中，利用 Flux.create 方法对响应进行创建封装成 Flux 数据。并且使用 lambda 写数据流的处理函数会十分的方便。

Service 层依旧是以前那套逻辑，业务服务层接口如下：

```
public interface CityService {

    /**
     * 获取城市信息列表
     *
     * @return
     */
    List<City> findAllCity();

    /**
     * 根据城市 ID,查询城市信息
     *
     * @param id
     * @return
     */
    City findCityById(Long id);

    /**
     * 新增城市信息
     *
     * @param city
     * @return
     */
    Long saveCity(City city);

    /**
     * 更新城市信息
     *
     * @param city
     * @return
     */
    Long updateCity(City city);

    /**
     * 根据城市 ID,删除城市信息
     *
     * @param id
     * @return
     */
    Long deleteCity(Long id);
}

```

具体案例在我的 Github：https://github.com/JeffLi1993/springboot-learning-example

### 基于 Functional 函数式路由实现 RESTful API

创建一个 Route 类来定义 RESTful HTTP 路由：

```
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RequestPredicates.accept;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;

@Configuration
public class Routes {
    private CityService cityService;

    public Routes(CityService cityService) {
        this.cityService = cityService;
    }

    @Bean
    public RouterFunction<?> routerFunction() {
        return route(
                       GET("/api/city").and(accept(MediaType.APPLICATION_JSON)), cityService:: findAllCity).and(route(
                       GET("/api/user/{id}").and(accept(MediaType.APPLICATION_JSON)), cityService:: findCityById)
                );
    }
}

```

RoouterFunction 类似 Spring Web MVC 的 @RequestMapping ，用来定义路由信息，每个路由会映射到一个处理方法，当接受 HTTP 请求时候会调用该处理方法。

创建 HttpServerConfig 自定义 Http Server，这里创建一个 Netty HTTP 服务器：

```
import org.springframework.http.server.reactive.HttpHandler;
import org.springframework.http.server.reactive.ReactorHttpHandlerAdapter;
import reactor.ipc.netty.http.server.HttpServer;

@Configuration
public class HttpServerConfig {
    @Autowired
    private Environment environment;

    @Bean
    public HttpServer httpServer(RouterFunction<?> routerFunction) {
        HttpHandler httpHandler = RouterFunctions.toHttpHandler(routerFunction);
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);
        HttpServer server = HttpServer.create("localhost", Integer.valueOf(environment.getProperty("server.port")));
        server.newHandler(adapter);
        return server;
    }
}

```

自然推荐 Netty 来运行 Reactive 应用，因为 Netty 是基于异步和事件驱动的。

从案例中，大家也简单入门了解 WebFlux

### 小结

谢谢大家的阅读，本文写的不是那么精致。期待过几天和大家微信沟通交流学习。by the way , 最近在写一本小小书，《Spring Boot 2.x 核心技术实战（上）基础篇》，针对基础实践，精致地写了 demo 和讲解。最后还是谢谢。

