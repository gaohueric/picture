### 内容提要

内容提要：

- Spring Boot 如何设置支持跨域请求？如何设置支持自定义头部 Header 信息？ 如客户端 HTML5 页面通过 Ajax 的请求过程中 Header 头部设置了自定义的参数 Token，在 Spring Boot 如何配置支持并如何获取到这个 Token传到 Spring Boot 服务端的值？
- 你说了很多 Spring Boot 现在的应用情景，Spring Boot 这么广泛的应用。我们怎样掌握 Spring Boot呢？怎样跟着 Spring Boot 的发展？
- 按照你的说法，Spring Boot 根据约定生成了配置，如果有需求要修改配置该如何？另外 Spring Boot 如何支持上传？
- 在分散各处的微服务，网关调用微服务，微服务调用微服务都需要传用户凭证。那用户权限的鉴权是要在每个微服务都要做一遍吗？有什么最佳实践可以分享？
- 微服务的网关，注册中心的粒度怎么划分？大业务线还是小业务线？
- Spring Boot 配合 Druid 集成多数据源，需要每个数据源都配置 Druid 吗？有没有办法只配置一次 Druid？
- \#环境变量配置 spring.profiles.active=@profiles*active@ The following profiles are active: @profiles*active@，这个不能识别解析怎么办？
- 对于先有 Spring MVC 项目的改造有什么好的建议？
- Spring Boot+Token实现 Restful 接口，如何做？
- Spring Boot+Session的使用，怎么做？
- 在一个微服务体系内，应该如何进行契约测试？微服务的接口迭代有什么最佳实践吗？

------

**问：Spring Boot 如何设置支持跨域请求？如何设置支持自定义头部 Header 信息？ 如客户端 HTML5 页面通过 Ajax 的请求过程中 Header 头部设置了自定义的参数 Token，在 Spring Boot 如何配置支持并如何获取到这个 Token传到 Spring Boot 服务端的值？**

**答：**我自己研究并使用 Spring Boot 2年多，深切的体会到了 Spring Boot 开发项目的各种便利，因为也一直在呼吁 Java 工程师更早的学习和应用 Spring Boot。

**Spring Boot 如何设置支持跨域请求？**

现代浏览器出于安全的考虑 HTTP 请求时必须遵守同源策略，否则就是跨域的 HTTP 请求，默认情况下是被禁止的，IP（域名）不同、或者端口不同、协议不同（比如 HTTP、HTTPS）都会造成跨域问题。

一般前端的解决方案有：

1. 使用 JSONP 来支持跨域的请求，JSONP 实现跨域请求的原理简单的说，就是动态创建<script>标签，然后利用<script>的 SRC 不受同源策略约束来跨域获取数据。缺点是需要后端配合输出特定的返回信息。
2. 利用反应代理的机制来解决跨域的问题，前端请求的时候先将请求发送到同源地址的后端，通过后端请求转发来避免跨域的访问。

后来 HTML5 支持了 CORS 协议。CORS 是一个 W3C 标准，全称是”跨域资源共享”（Cross-origin resource sharing），允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 AJAX 只能同源使用的限制。它通过服务器增加一个特殊的 Header[Access-Control-Allow-Origin]来告诉客户端跨域的限制，如果浏览器支持 CORS、并且判断 Origin 通过的话，就会允许 XMLHttpRequest 发起跨域请求。

前端使用了 CORS 协议，就需要后端设置支持非同源的请求，Spring Boot 设置支持非同源的请求有两种方式。

第一，配置 CorsFilter。

```
@Configuration
public class GlobalCorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
          config.addAllowedOrigin("*");
          config.setAllowCredentials(true);
          config.addAllowedMethod("*");
          config.addAllowedHeader("*");
          config.addExposedHeader("*");

        UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
        configSource.registerCorsConfiguration("/**", config);

        return new CorsFilter(configSource);
    }
}

```

需要配置上述的一段代码。第二种方式稍微简单一些。

第二，在启动类上添加：

```
public class Application extends WebMvcConfigurerAdapter {  

    @Override  
    public void addCorsMappings(CorsRegistry registry) {  

        registry.addMapping("/**")  
                .allowCredentials(true)  
                .allowedHeaders("*")  
                .allowedOrigins("*")  
                .allowedMethods("*");  

    }  
}  

```

第三，也可以设置单个请求支持跨域。

```
@RequestMapping("/hello")
@ResponseBody
@CrossOrigin("http://localhost:8080") 
public String hello( ){
return "Hello World";
}

```

**如何设置支持自定义头部 Header 信息？**

如果我们使用的是 Ajax 可以这样使用：

```
$.ajax({  
        url: 'http://*******/api/xxx',  
        data: {  
            id: id,  
            xx: xx  
        },  
        beforeSend: function(request) {  
            request.setRequestHeader("token", token);  
        },  
        dataType: 'JSON',  
        type: 'GET',  
        success: function (list) {  
        },  
        error: function () {  
        }  
    }); 

```

重点在这块：

```
beforeSend: function(request) {  
            request.setRequestHeader("token", token);  

```

**如客户端 HTML5 页面通过 Ajax 的请求过程中 Header 头部设置了自定义的参数 Token，在 Spring Boot 如何配置支持并如何获取到这个 Token传到 Spring Boot 服务端的值？**

通过 Request 来获取：

```
request.getHeaders().getFirst("token");

```

也可以使用 @RequestHeader 来接收，类似下面这段代码：

```
@RequestMapping(value="/hello", method=RequestMethod.GET)
public String hello(@RequestHeader HttpHeaders headers) {
  System.out.println("token is:" + headers.getFirst("token"));
  return "hello";
}

```

------

**问：你说了很多 Spring Boot 现在的应用情景，Spring Boot 这么广泛的应用。我们怎样掌握 Spring Boot呢？怎样跟着 Spring Boot 的发展？**

**答：**首先，约定优于配置是一个设计理念，在很多年前就存在，是一种软件设计范式，通过一些约定的内容来大量的减少通用的配置，从而节省大量的开发工作。它不是真正指的是配置文件。Spring Boot 作为一个新的技术，并没有限定它的使用场景，其实现在各行各业都在使用 Spring Boot ，虽然使用的场景有所不同，但其实使用的技术体系基本一致。所以我们学习的时候，只要把握住它的主流使用方式即可。学习 Spring Boot 其实和学习其它技术的方式是一样：

第一阶段，我们需要首先会使用 Spring Boot 快速开发一个最基本的应用，比如简单的 Hello World，简单的增删改查，利用最简单的一些示例来感受 Spring Boot 的魅力。

第二个阶段，学习使用 Spring Boot 开发常用使用场景，比如使用 Spring Boot 进行 Web 操作读取配置文件、自定义 Filter ；学会使用 Spring Boot 操作数据库；学会使用 Spring Boot Jpa ；学习 Spring Boot 操作缓存、NoSQL 等。

第三个阶段：将掌握的 Spring Boot 应用在我们的项目中，在项目中的学习是最直接、最有效的，可以先从一个小项目开发，逐步的演化到大型复杂的项目中来，几个真实的项目开发经验过来，基本就掌握了 Spring Boot 的使用。

第四个阶段，可以尝试的使用 Spring Boot 定制一些解决方案，比如自定义 Starter、研究 Spirng Boot 的实现原理等。

当然大家可以关注我在 GitBook 上面的达人课，<http://gitbook.cn/gitchat/column/59f5daa149cd4330613605ba>。这个课程就是我根据自己的工作经验提炼出来工作中最容易使用的一些场景。

如何跟着 Spring Boot 发展？

因为 Spring Boot 一直也在不断的升级和改进，比如 Spring Boot 2.0 又采用了很多新的解决方案，一方面我们多关注新技术的发展方向，一方面可以多查看 Spring Boot 发布的 Releases 说明，每一个版本都会有说明更新了哪些技术。当然了也可以关注我的公号和博客，我会持续的跟进 Spring Boot 的发展，另外推荐一个网站 ：<http://springboot.fun/> 里面有最全的 Spring Boot 学习资源。

------

**问：按照你的说法，Spring Boot 根据约定生成了配置，如果有需求要修改配置该如何？另外 Spring Boot 如何支持上传？**

**答：**首先，约定优于配置是一个设计理念，在很多年前就存在，是一种软件设计范式，通过一些约定的内容来大量的减少通用的配置，从而节省大量的开发工作。它不是真正指的是配置文件。如果我们有需要来自定义一些约定的配置内容，就需要我们了解 Spring Boot 是如何来实现这样自定义配置，其实在我们使用的各个 Starter 中使用了大量的条件注解（Condition annotations）。在程序实际运行时动态判定初始化行为。比如根据某配置项、某个类是否存在判断是否加载某个 Bean。

比如下面这些：

@ConditionalOnClass/@ConditionalOnMissingClass 根据类的存在性判定；

@ConditionalOnBean/@ConditionalOnMissingBean 根据 Bean 的存在性判定；

@ConditionalOnProperty 根据配置项的存在性判定；

@ConditionalOnResource 根据文件的存在性判定；

@ConditionalOnWebApplication/@ConditionalOnNotWebApplication 根据是否为 Web 应用判定；

@@ConditionalOnExpression 根据 Spring EL 判定。

Spring Boot 其实就是根据这种大量的条件来选择在启动的时候使用哪些配置。其实就是将大量的默认配置封装到了各个 Starter 中。让我们使用的时候，感到很简单。我们自定义的时候只需要根据这些条件来做一些适配即可。或者完全自定义一个 Starter 来达到这个目的。

Spring Boot 如何支持上传？

只需要做两个地方的改动即可。

启动类：

```
@Bean
    public TomcatEmbeddedServletContainerFactory tomcatEmbedded() {
        TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
        tomcat.addConnectorCustomizers((TomcatConnectorCustomizer) connector -> {
            if ((connector.getProtocolHandler() instanceof AbstractHttp11Protocol<?>)) {
                //-1 means unlimited
                ((AbstractHttp11Protocol<?>) connector.getProtocolHandler()).setMaxSwallowSize(-1);
            }
        });
        return tomcat;
    }

```

这段代码是为了解决，上传文件大于10M出现连接重置的问题。此异常内容 GlobalException 也捕获不到。

Controller 层做如下处理：

```
@PostMapping("/upload") 
public String singleFileUpload(@RequestParam("file") MultipartFile file,
                               RedirectAttributes redirectAttributes) {
    if (file.isEmpty()) {
        redirectAttributes.addFlashAttribute("message", "Please select a file to upload");
        return "redirect:uploadStatus";
    }

    try {
        // Get the file and save it somewhere
        byte[] bytes = file.getBytes();
        Path path = Paths.get(UPLOADED_FOLDER + file.getOriginalFilename());
        Files.write(path, bytes);

        redirectAttributes.addFlashAttribute("message",
                "You successfully uploaded '" + file.getOriginalFilename() + "'");

    } catch (IOException e) {
        e.printStackTrace();
    }

    return "redirect:/uploadStatus";
}

```

上面代码的意思就是，通过 MultipartFile 读取文件信息，如果文件为空跳转到结果页并给出提示；如果不为空读取文件流并写入到指定目录，最后将结果展示到页面。

MultipartFile 是 Spring 上传文件的封装类，包含了文件的二进制流和文件属性等信息，在配置文件中也可对相关属性进行配置，基本的配置信息如下：

spring.http.multipart.enabled=true #默认支持文件上传；

spring.http.multipart.file-size-threshold=0 #支持文件写入磁盘；

spring.http.multipart.location= # 上传文件的临时目录；

spring.http.multipart.max-file-size=1Mb # 最大支持文件大小；

spring.http.multipart.max-request-size=10Mb # 最大支持请求大小。

------

**问：在分散各处的微服务，网关调用微服务，微服务调用微服务都需要传用户凭证。那用户权限的鉴权是要在每个微服务都要做一遍吗？有什么最佳实践可以分享？**

**答：**一般情况下网关之后的服务都集中在内网环境下，网关内部的服务都是相对较安全的，因此在实际使用的时候我们都是在网关这一层做权限的校验。一般的技术方案有 OAuth2、JWT、Shiro 等技术来实现。OAuth2 协议使用最多的场景还是用以给第三方来获取授权。JWT 最大的区别，就是用 Token 来进行认证，取代了常规的 Session。JWT 常用于前端的一些校验，比如前后分离的 Node 层 。Shiro 较多用于权限系统来控制 URL 的访问权限，我们公司内部的权限系统就是采用的 Shiro 技术来实现的。

------

**问：微服务的网关，注册中心的粒度怎么划分呢？大业务线还是小业务线？**

**答：**微服务的网关大多是对外的业务线来使用，如果公司比较大也可以将不同的业务群通过网关来切分。所以网关这块，一方面对外提供服务的时候使用，比如 APP 接口、Web 接口等。另一方方面，公司内部的业务群进行划分，不同的业务群通过不同的网站来暴露接口，封装网关后面服务的复杂性。同时也可以基于网关做一些权限交易、服务统计、流量计算等功能。注册中心的粒度，其实一般情况下：如果没有特殊的需求，中小公司使用一个集群的注册中心即可。搜索这种基础服务，只是一个单个的服务没必要进行单独的网关或者注册中心，网关和注册中心都是一些通用的服务支持。

------

**问：Spring Boot 配合 Druid 集成多数据源，需要每个数据源都配置 Druid 吗？有没有办法只配置一次 Druid？**

**答：**首先配置多数据源并不是一定要使用 Druid，Druid 只是一个淘宝开源出来的连接池工具而已。我在达人课里面有一节就是讲 MyBatis Druid 多数据源的解决方案：<http://gitbook.cn/gitchat/column/59f5daa149cd4330613605ba/topic/59f97ed968673133615f745f>。

以下内容摘录于此文：

> Druid 是阿里巴巴开源平台上的一个项目，整个项目由数据库连接池、插件框架和 SQL 解析器组成。该项目主要是为了扩展 JDBC 的一些限制，可以让程序员实现一些特殊的需求，比如向密钥服务请求凭证、统计 SQL 信息、SQL 性能收集、SQL 注入检查、SQL 翻译等，程序员可以通过定制来实现自己需要的功能。
>
> Druid 首先是一个数据库连接池，但它不仅仅是一个数据库连接池，它还包含一个 ProxyDriver，一系列内置的 JDBC 组件库，一个 SQL Parser。在 Javad 的世界中 Druid 是目前最好的数据库连接池，在功能、性能、扩展性方面，都超过其他数据库连接池，包括 DBCP、C3P0、BoneCP、Proxool、JBoss DataSource。Spring Boot 集成 Druid，阿里为 Druid 也提供了 Spring Boot Starter 的支持。官网这样解释：Druid Spring Boot Starter 用于帮助在 Spring Boot 项目中轻松集成 Druid 数据库连接池和监控。
>
> 我们使用的时候需要做以下操作：

![enter image description here](http://images.gitbook.cn/86efdff0-68ca-11e8-9826-f702b2e7f23c)

> 首先配置多数据源：

![enter image description here](http://images.gitbook.cn/8e705890-68ca-11e8-9826-f702b2e7f23c)

后面就是依次配置 SessionFactory 事务等信息配置进去。

------

**问：这个不能识别解析怎么办？**

环境变量配置 spring.profiles.active=@profiles*active@ The following profiles are active: @profiles*active@

**答：** spring.profiles.active 的作用是在服务启动的时候，指定采用哪个配置文件来加载，在使用这个配置的时候需要确保有两个地址有配置。

第一，Pom 包中类似下面：

```
<resources>
   <resource>
      <directory>src/main/resources</directory>
      <filtering>true</filtering>
      <excludes>
         <exclude>application-dev.properties</exclude>
         <exclude>application-uat.properties</exclude>
         <exclude>application-pro.properties</exclude>
         <exclude>application.properties</exclude>
      </excludes>
   </resource>
   <resource>
      <directory>src/main/resources</directory>
      <filtering>true</filtering>
      <includes>
         <include>application-${env}.properties</include>
         <include>application.properties</include>
      </includes>
   </resource>
</resources>

```

指定有 UAT 和生产的三个配置文件，同时在项目的 Resources 目录下要存在这三个配置文件。

![enter image description here](http://images.gitbook.cn/d560bf10-68ca-11e8-9826-f702b2e7f23c)

类似这样。

如果我在 application.properties 文件中这样配置：

```
spring.profiles.active=dev

```

就代表了使用这个 application-dev.properties 的内容来加载。同样在启动服务器 ，启动 Spring Boot 项目时也可以动态的来指定那个配置文件。

```
nohup java -jar /usr/local/favorites-web-1.0.0.jar --spring.profiles.active=pro &

```

类似这样 --spring.profiles.active=pro 代表加载application-pro.properties的配置信息启动。

------

**问：对于先有 Spring MVC 项目的改造有什么好的建议？**

**答：**首先 Spring Boot 是基于 Spring 来开发的，就自然包含了 Spring Mvc，所以以前 Spring Mvc 支持的功能，Spring Boot 都有，并且 Spring Boot 提供了组件 spring-boot-starter-web ，针对 Web 开发提供了大量的支持，只会让开发 Web 项目更简单实用。

不过需要注意的是 Spring Boot Web 的用法确实和以前有一些不一致，因此仅仅只是平移过来的话，肯定会有报错。因此我的建议是，首先需要了解我们以前项目中都使用了 Spring Mvc 的哪些功能，在学习 Spring Boot 的时候关注在 Spring Boot 中的使用方式是否是保持了一致。如果不一致，需要在平移的时候做适当的修改，以适应 Spring Boot 项目的风格，其实大家不用担心，Spring Boot 只会简化了 Web 的相关操作，详细大家通过一些学习，就会很快的掌握新的配置方式。

------

**问：Spring Boot+Token实现 Restful 接口，如何做？**

**答：** Spring Boot 天生就是支持 Restful 方式的接口，在我们使用的时候只需要做不同的注解就可以支持，甚至 Spring Boot 还提供了一组简化的注解来支持。

- `@GetMapping`，处理 Get 请求。
- `@PostMapping`，处理 Post 请求。
- `@PutMapping`，用于更新资源。
- `@DeleteMapping`，处理删除请求。
- `@PatchMapping`，用于更新部分资源。

其实这些组合注解就是我们使用的`@RequestMapping`的简写版本，下面是 Java 类中的使用示例：

```
java
@GetMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.GET)

@PostMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.POST)

@PutMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.PUT)

@DeleteMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.DELETE)

@PatchMapping(value="/xxx")
等价于
@RequestMapping(value = "/xxx",method = RequestMethod.PATCH)

```

接下来其实就是 Token 验证的问题。在这里我介绍一个开源项目：<https://github.com/cloudfavorites/favorites-web>。其实就是 Token 验证的机制来做的，在登录的似乎根据用户信息生成 Token，并且将 Token 信息推送的浏览器的 Cookie 中。利用 Spring Boot 的 Filter 机制来校验 Token 的有效性。上面讲的 JWT 其实也是利用 Token 机制来做的。

------

**问：Spring Boot+Session的使用，怎么做？**

**答：**在这里就要介绍 Spring 另外一个组建了 spring-session，Spring Session 作为 Spring 官方推荐的分布式 Session 解决方案，它提供了一种独立于应用服务器的方案，这种方案能够在 Servlet 规范之内配置可插拔的 Session 数据存储，不依赖于任何应用服务器的特定 API。这就意味着 Spring Session 能够用于实现了 Servlet 规范的所有应用服务器之中（Tomcat、Jetty、 WebSphere、WebLogic、JBoss 等），它能够非常便利地在所有应用服务器中以完全相同的方式进行配置。我们还可以选择任意最适应需求的外部 Session 数据存储，例如 Redis、Hazelcast、MongoDB。

Spring Session 提供了 API 和实现，用于管理用户的 Session 信息。除此之外，它还提供了如下特性：

- 将 Session 所保存的状态卸载到特定的外部 Session 存储汇总，如 Redis 中，他们能够以独立于应用服务器的方式提供高质量的集群。
- 控制 Sessionid 如何在客户端和服务器之间进行交换，这样的话就能很容易地编写 Restful API ，因为它可以从 HTTP 头信息中获取 Sessionid ，而不必再依赖于 Cookie。
- 在非 Web 请求的处理代码中，能够访问 Session 数据，比如在 JMS 消息的处理代码中。
- 支持每个浏览器上使用多个 Session，从而能够很容易地构建更加丰富的终端用户体验。
- 当用户使用 WebSocket 发送请求的时候，能够保持 HttpSession 处于活跃状态。

通常我们在使用的时候 一般是 Spring session +Redis 组合的方式来管理 Session。这样做有以下几方面的好处：

第一，可以让服务无状态化，方便服务水平扩展，因为 Session 信息独立在第三方中的信息，所以请求打到任何一个后端服务都可以做支撑。

第二，方便 Session 集中管理，我可以利用这些信息来统计在线用户数等内容。

第三、容灾性，就算某个微服务挂掉了或者重启了，Session 信息不会失效。

Spring Boot 使用 Spring-Session 非常简单，只需要在 Pom 包中添加依赖，添加一个配置类 @EnableRedisHttpSession public class HttpSessionConfig { }。在添加 Redis 的相关信息即可，具体可以参考我的博客。

------

**问：在一个微服务体系内，应该如何进行契约测试？微服务的接口迭代有什么最佳实践吗？**

**答：**契约测试 ，又称之为消费者驱动的契约测试(Consumer-Driven Contracts，简称 CDC)，根据消费者驱动契约 ，我们可以将服务分为消费者端和生产者端，而消费者驱动的契约测试的核心思想在于是从消费者业务实现的角度出发，由消费者自己会定义需要的数据格式以及交互细节，并驱动生成一份契约文件。然后生产者根据契约文件来实现自己的逻辑，并在持续集成环境中持续验证。Spring 有一个组建来支持 Spring Cloud Contract。Spring Cloud Contract 验证程序是一种能够启用消费者驱动合同（CDC）开发基于JVM的应用程序的工具。它与合同定义语言（DSL）一起提供。下来大家可以找找 Spring Cloud Contract 使用教程。



### 文章实录

Spring Boot 2.0 的推出又激起了一阵学习 Spring Boot 热，就单从我个人博客的访问量大幅增加就可以感受到大家对学习 Spring Boot 的热情，那么在这么多人热衷于学习 Spring Boot 之时，我自己也在思考： Spring Boot 诞生的背景是什么？Spring 企业又是基于什么样的考虑创建 Spring Boot？传统企业使用 Spring Boot 会给我们带来什么样变革？

带着这些问题，我们一起来了解下 Spring Boot 到底是什么。

### Spring 历史

说起 Spring Boot 我们不得不先了解一下 Spring 这个企业，不仅因为 Spring Boot 来源于 Spirng 大家族，而且 Spring Boot 的诞生和 Sping 框架的发展息息相关。

时间回到 2002 年，当时正是 Java EE 和 EJB 大行其道的时候，很多知名公司都是采用此技术方案进行项目开发。这时候有一个美国的小伙子认为 EJB 太过臃肿，并不是所有的项目都需要使用 EJB 这种大型框架，应该会有一种更好的方案来解决这个问题。

为了证明他的想法是正确的，于 2002 年 10 月甚至写了一本书《 Expert One-on-One J2EE 》，介绍了当时 Java 企业应用程序开发的情况，并指出了 Java EE 和 EJB 组件框架中存在的一些主要缺陷。在这本书中，他提出了一个基于普通 Java 类和依赖注入的更简单的解决方案。

在书中，他展示了如何在不使用 EJB 的情况下构建高质量，可扩展的在线座位预留系统。为了构建应用程序，他编写了超过 30,000 行的基础结构代码，项目中的根包命名为 com.interface21，所以人们最初称这套开源框架为 interface21，也就是 Spring 的前身。

他是谁呢，他就是大名鼎鼎的 Rod Johnson （下图）, Rod Johnson 在悉尼大学不仅获得了计算机学位，同时还获得了音乐学位，更令人吃惊的是在回到软件开发领域之前，他还获得了音乐学的博士学位。现在 Rod Johnson 已经离开了 Spring ，成为了一个天使投资人，同时也是多个公司的董事，早已走上人生巅峰。

![enter image description here](http://images.gitbook.cn/13003d90-6287-11e8-9231-137314da4e00)

在这本书发布后，一对一的 J2EE 设计和开发一炮而红。这本书免费提供的大部分基础架构代码都是高度可重用的。 2003 年 Rod Johnson 和同伴在此框架的基础上开发了一个全新的框架命名为 Spring，据 Rod Johnson 介绍 Spring 是传统 J2EE 新的开始。随后 Spring 发展进入快车道。

- 2004 年 03 月，1.0 版发布。
- 2006 年 10 月，2.0 版发布。
- 2007 年 11 月更名为 SpringSource，同时发布了 Spring 2.5。
- 2009 年 12 月，Spring 3.0 发布。
- 2013 年 12 月，Pivotal 宣布发布 Spring 框架 4.0。
- 2017 年 09 月，Spring 5.0 发布。

### Spring Boot 的诞生

随着使用 Spring 进行开发的个人和企业越来越多，Spring 也慢慢从一个单一简洁的小框架变成一个大而全的开源软件，Spring 的边界不断的进行扩充，到了后来 Spring 几乎可以做任何事情了，市面上主流的开源软件、中间件都有 Spring 对应组件支持，人们在享用 Spring 的这种便利之后，也遇到了一些问题。

Spring 每集成一个开源软件，就需要增加一些基础配置，慢慢的随着人们开发的项目越来越庞大，往往需要集成很多开源软件，因此后期使用 Spirng 开发大型项目需要引入很多配置文件，太多的配置非常难以理解，并容易配置出错，到了后来人们甚至称 Spring 为配置地狱。

Spring 似乎也意识到了这些问题，急需有这么一套软件可以解决这些问题，这个时候微服务的概念也慢慢兴起，快速开发微小独立的应用变得更为急迫，Spring 刚好处在这么一个交叉点上，于 2013 年初开始的 Spring Boot 项目的研发，2014 年 4 月，Spring Boot 1.0.0 发布。

Spring Boot 诞生之初，就受到开源社区的持续关注，陆续有一些个人和企业尝试着使用了 Spring Boot，并迅速喜欢上了这款开源软件。直到 2016 年，在国内 Spring Boot 才被正真使用了起来，期间很多研究 Spring Boot 的开发者在网上写了大量关于 Spring Boot 的文章，同时有一些公司在企业内部进行了小规模的使用，并将使用经验分享了出来。从 2016 年到 2018 年，使用 Spring Boot 的企业和个人开发者越来越多，我们从 Spring Boot 关键字的百度指数就可以看出。

![enter image description here](http://images.gitbook.cn/3a324480-6287-11e8-9231-137314da4e00)

上图为 2014 年到 2018 年 Spring Boot 的百度指数，可以看出 Spring Boot 2.0 的推出引发了搜索高峰。

当然 Spring Boot 不是为了取代 Spring，Spring Boot 基于 Spring 开发，是为了让人们更容易的使用 Spring。看到 Spring Boot 的市场反应，Spring 官方也非常重视 Spring Boot 的后续发展，已经将 Spring Boot 作为公司最顶级的项目来推广，放到了官网上第一的位置，因此后续 Spring Boot 的持续发展也被看好。

![enter image description here](http://images.gitbook.cn/54eec000-6287-11e8-9231-137314da4e00)

### 什么是 Spring Boot

#### Spring Boot 介绍

Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化新 Spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是 Spring Boot 其实不是什么新的框架，它默认配置了很多框架的使用方式，就像 Maven 整合了所有的 Jar 包，Spring Boot 整合了所有的框架（不知道这样比喻是否合适）。

Spring Boot 简化了基于 Spring 的应用开发，通过少量的代码就能创建一个独立的、产品级别的 Spring 应用。 Spring Boot 为 Spring 平台及第三方库提供开箱即用的设置，这样你就可以有条不紊地开始。Spring Boot 的核心思想就是约定大于配置，多数 Spring Boot 应用只需要很少的 Spring 配置。采用 Spring Boot 可以大大的简化你的开发模式，所有你想集成的常用框架，它都有对应的组件支持。

#### Spring Boot 特性

- 使用 Spring 项目引导页面可以在几秒构建一个项目
- 方便对外输出各种形式的服务，如 REST API、WebSocket、Web、Streaming、Tasks
- 非常简洁的安全策略集成
- 支持关系数据库和非关系数据库
- 支持运行期内嵌容器，如 Tomcat、Jetty
- 强大的开发包，支持热启动
- 自动管理依赖
- 自带应用监控
- 支持各种 IED，如 IntelliJ IDEA 、NetBeans

Spring Boot 这些特性会给我们研发带来非常大的优势，下面我们可以分开来介绍。

### 使用 Spring Boot 的优势

使用 Spring Boot 开发项目，会给我们带来非常美妙的开发体验，可以从以下几个方面展开来说明：

#### Spring Boot 让开发变得更简单

##### **构建开发环境**

Spring Boot 对开发效率的提升是全方位的，我们可以简单做一下对比：

在没有使用 Spring Boot 之前我们开发一个 Web 项目需要做哪些工作：

- 1）配置 web.xml，加载 Spring 和 Spring mvc
- 2）配置数据库连接、配置 Spring 事务
- 3）配置加载配置文件的读取，开启注解
- 4）配置日志文件
- …
- n) 配置完成之后部署 Tomcat 调试

可能你还需要考虑各个版本的兼容性，Jar 包冲突的各种可行性。

那么使用 Spring Boot 之后我们需要开发一个 Web 项目需要哪些操作呢？

- 1）登录网址 http://start.spring.io/ 选择对应的组件直接下载
- 2）导入项目，直接开发

上面的 N 步和下面的 2 步形成巨大的反差，这仅仅只是在开发环境搭建的这个方面。

##### **Spring DevTools**

Spring Boot 还专门提供了一个组件包：Spring DevTools， DevTools 包括一组额外的工具，可以使应用程序开发体验更加愉快。spring-boot-devtools 为应用提供一些开发时特性，包括默认值设置，自动重启，livereload 等。

**1）属性默认值**

Spring Boot 支持的一些库中会使用缓存来提高性能，例如模版引擎将缓存编译后的模板，以避免重复解析模板文件。 此外，Spring mvc 可以在服务静态资源时向响应中添加 HTTP 缓存头。

虽然缓存在生产中非常有益，但它在开发过程中可能会产生反效果，它会阻止你看到刚刚在应用程序中进行的更改。 因此，`spring-boot-devtools` 将默认禁用这些缓存选项。

**2）自动重启**

在使用了 `spring-boot-devtools` 依赖包的 Spring Boot 项目，我们只需要简单配置就可以让项目具有自动重启的功能，这样我们在开发过程中调试代码就变得更加高效和自然。

自动重启的原理在于 Spring Boot 使用两个 classloader：不改变的类（如第三方jar）由 base 类加载器加载，正在开发的类由 restart 类加载器加载。应用重启时，restart 类加载器被扔掉重建，而 base 类加载器不变，这种方法意味着应用程序重新启动通常比“冷启动”快得多，因为 base 类加载器已经可用并已填充。所以，当我们开启 devtools 后，classpath 中的文件变化会导致应用自动重启.

以上只是 `spring-boot-devtools` 我们关注的两个核心功能，还有很多优化开发过程的支持。

##### **Starters**

Spring Boot 拥有强大融合社区开源软件的能力，在没有使用 Spring Boot 之前，我们需要按照每个开源软件的特性，将对应的组件包集成到我们的开发项目中，因为每个组件的设计理念和开发团队都不一致，因此会有很多不同的调用风格在我们的项目中。

Spring Boot 整合了主流的开源软件形成了一系列的 Starter，让我们有了一致的编程体验来集成各种软件，Spring Boot 在集成的时候做了大量的优化，让我们在集成的时候往往只需要很少的配置和代码就可以完成。可以说各种 Starters 就是 Spring Boot 最大的优势之一。

以下为我们常用的 Spring Boot Starter 列表：

| 名称                                     | 描述                                       | Pom                                      |
| -------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| spring-boot-starter                    | 核心starter，包括自动配置支持，日志和YAML               | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| spring-boot-starter-activemq           | 用于使用Apache ActiveMQ实现JMS消息               | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| spring-boot-starter-amqp               | 用于使用Spring AMQP和Rabbit MQ                | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| spring-boot-starter-cache              | 用于使用Spring框架的缓存支持                        | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
| spring-boot-starter-data-elasticsearch | 用于使用Elasticsearch搜索，分析引擎和Spring Data Elasticsearch | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
| spring-boot-starter-data-jpa           | 用于使用Hibernate实现Spring Data JPA           | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| spring-boot-starter-data-mongodb       | 用于使用基于文档的数据库MongoDB和Spring Data MongoDB  | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
| spring-boot-starter-data-redis         | 用于使用Spring Data Redis和Jedis客户端操作键—值数据存储Redis | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
| spring-boot-starter-jta-atomikos       | 用于使用Atomikos实现JTA事务                      | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |
| sring-boot-starter-mail                | 用于使用Java Mail和Spring框架email发送支持          | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
| spring-boot-starter-quartz             | 用于定时任务 quartz 的支持                        | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
| spring-boot-starter-security           | 对Spring Security的支持                      | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| spring-boot-starter-test               | 用于测试Spring Boot应用，支持常用测试类库，包括JUnit, Hamcrest和Mockito | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| spring-boot-starter-thymeleaf          | 用于使用Thymeleaf模板引擎构建MVC web应用             | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| spring-boot-starter-validation         | 用于使用Hibernate Validator实现Java Bean校验     | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| spring-boot-starter-web                | 用于使用Spring MVC构建web应用，包括RESTful。Tomcat是默认的内嵌容器 | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| spring-boot-starter-websocket          | 用于使用Spring框架的WebSocket支持构建WebSocket应用    | [Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |

> 这里只节选了我们最常使用的 Starter，完整的 Starter 参考这里:**Spring Boot application starters**

因为 Spring Boot 足够的强大，很多第三方社区都进行了主动的集成比如：Mybats、RabbitMQ（高级用法）等，第三方社区支持的列表，可以在这里查看[Community Contributions](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters)，可以看到社区贡献的其他 Starters 列表。

看完这些 Starters 会不会瞬间觉得 Spring Boot 很强大，几乎我们涉及的开源软件都做了支持，在 Spring Boot 环境下使用这些软件，仅仅只需要引入对应的 Starters 包即可。

#### Spring Boot 使测试变得更简单

Spring Boot 对测试的支持不可谓不强大，Spring Boot 内置了 7 种强大的测试框架：

- JUnit： 一个 Java 语言的单元测试框架
- Spring Test & Spring Boot Test：为 Spring Boot 应用提供集成测试和工具支持
- AssertJ：支持流式断言的 Java 测试框架
- Hamcrest：一个匹配器库
- Mockito：一个 Java Mock 框架
- JSONassert：一个针对 JSON 的断言库
- JsonPath：JSON XPath 库

我们只需要在项目中引入 `spring-boot-start-test` 依赖包，就可以对数据库、Mock、 Web 等各种情况进行测试。

比如我们想测试一个方法，只需要引入一个 `@Test` 即可:

```
@Test
public void hello() {
    System.out.println("hello world");
}

```

很多时候我们需要上下文环境，比如依赖注入对象，在测试类的上面添加 `@RunWith(SpringRunner.class)` 和 `@SpringBootTest` 注解即可。

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloTest {

    @Resource
    HelloService helloService;

    @Test
    public void  sayHelloTest(){
        helloService.sayHello();
    }
}

```

如果我们想测试 Web 项目，Spring Boot Test 还提供了 MockMvc 支持，MockMvc 实现了对 Http 请求的模拟，能够直接使用网络的形式，转换到 Controller 的调用，这样可以使得测试速度更快、不依赖网络环境，而且提供了一套验证的工具，这样可以使得请求的验证统一而且更方便。

一个简单的示例：

```
@SpringBootTest
public class HelloTest {
    private MockMvc mockMvc;
    @Before
    public void setUp() throws Exception {
        //测试 HelloWorldController
        mockMvc = MockMvcBuilders.standaloneSetup(new HelloWorldController()).build();
    }
    @Test
    public void getHello() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.post("/hello?name=小明")
                .accept(MediaType.APPLICATION_JSON_UTF8)).andDo(print());
    }
}

```

`getHello()` 方法的含义是：发送 post 请求到 `/hello?name=小明`，将 Web 层返回的结果详细的打印出来。

这里只是罗列了我们最常用最基本的一些用法，Spring Boot Test 中包含了我们需要使用的各种测试场景，满足我们日常项目的测试需求。

#### Spring Boot 让配置变得更简单

Spring Boot 让配置变简单，说到这里我们就需要了解一下 Spring Boot 的核心思想：约定优于配置。那么什么是约定优于配置呢？

约定优于配置（convention over configuration），也称作按约定编程，是一种软件设计范式，旨在减少软件开发人员需做决定的数量，获得简单的好处，而又不失灵活性。

本质是说，开发人员仅需规定应用中不符约定的部分。例如，如果模型中有个名为 User 的类，那么数据库中对应的表就会默认命名为 user。只有在偏离这一约定时，例如将该表命名为“user_info”，才需写有关这个名字的配置。

我们可以按照这个思路来设想，我们约定 controller 层就是 web 请求层可以省略了 MVC 的配置；我们约定在 Service 结尾的类自动注入事务，就可以省略了 Spring 的切面事务配置......

在 Spring 体系中，Spring Boot JPA 就是约定优于配置最佳实现之一，不需要关注表结构，我们约定类名即是表名，属性名即是表的字段，String 对应 varchar，long 对应 bigint , 只有需要一些特殊要求的属性，我们再单独进行配置，按照这个约定我们可以将以前的工作大大的简化。



Spring Boot 体系将约定优于配置的思想展现得淋淋尽致，小到配置文件，中间件的默认配置，大到内置容器、生态中的各种 Starters 无不遵循此设计规则。Spring Boot 鼓励各软件组织方创建自己的 Starters ，创建 Starter 的核心组件之一就是 autoconfigure 模块，也是 Starter 的核心功能就是在启动的时候进行自动装配，属性默认化配置。

可以说正是因为 Spring Boot 简化的配置和众多的 Starters 才让 Spring Boot 变得简单、易用、快速上手，也可以说正是约定优于配置的思想的彻底落地才让 Spring Boot 走向辉煌。Spring Boot 让编程变得更简单，其实编程本该很简单，简单才是编程的美。

#### Spring Boot 让部署变得更简单

说起 Spring Boot 让部署变简单，就不得不说 Spring Boot 内嵌容器。内嵌容器不只让部署变得简单，其实在开发调试阶段也会带来非常大的便利性，对比以往开发 Web 项目时配置 Tomcat 的繁琐，会让大家使用 Spring Boot 内嵌容器开发时有更深的感触。使用 Spring Boot 开发 Web 项目，让我们不需要关心容器的环境问题，专心写业务代码即可。

但内嵌容器对部署带来的改变其实更多，现在 Maven 、Gradle 已经成了我们日常开发必不可少的构建工具，使用这些工具很容易的将项目打包成 Jar 或者 War 包，在服务器上我们仅仅只需要一条命令就可以启动项目，我们以 Maven 为例进行演示。

Maven 打包会根据 pom 包中的 packaging 配置来决定生成是 Jar 包或者 War 包：

```
mvn clean package 

```

在使用 Maven 进行打包的时候，会自动对 test 包下面的测试用例进行测试，测试用例失败会给出错误提示，并停止打包，我们可以利用这个功能来做项目的自动化测试，检测最基本的功能是否正确，当然我们也可以选择不执行这些测试用例：

```
mvn clean package  -Dmaven.test.skip=true

```

我们将打包好的文件上传到我们的目标服务器之后，仅仅只需要一条命令即可启动项目：

```
nohup java -jar target/spring-boot-scheduler-1.0.0.jar &

```

也可以在启动的时候选择不同的配置文件进行启动：

```
nohup java -jar target/spring-boot-scheduler-1.0.0.jar  --spring.profiles.active=dev  &

```

Spring Boot 支持在启动的时候添加定制，比如设置应用的堆内存、垃圾回收机制、日志路径等等。

当然以上只是单个项目部署，假如我们有100多个项目，如果再按照这种方案来进行打包上传启动，整个平台部署完成估计需要几个小时，并且非常容易出错。**所以在大量部署应用时，给大家介绍另外一款神器 Jenkins。**

Jenkins 是目前持续构建领域使用最广泛的工具之一，Jenkins 是一个独立的开源自动化服务器，可用于自动化各种任务，如构建，测试和部署软件。Jenkins 可以通过本机系统包 Docker 安装，甚至可以通过安装 Java Runtime Environment 的任何机器独立运行。

Jenkins 可以很好的支持各种语言（比如：java, c#, php等）的项目构建，也完全兼容 ant、maven、gradle 等多种第三方构建工具，同时跟 svn、git 能无缝集成，也支持直接与知名源代码托管网站，比如 github、bitbucket 直接集成。

说直白一点 Jenkins 就是专门来负责如何将代码变成可执行的程序包，将它部署到目标服务器中，并对其运营状态（日志）进行监控的软件。自动化、性能、打包、部署、发布、发布结果自动化验证、接口测试、单元测试等等关于我们打包测试部署的方方面面 Jenkins 都可以很友好的支持。

使用 Jenkins 部署 Spring Boot 项目非常简单，大家想继续了解可以参考我的文章：[使用Jenkins部署Spring Boot](http://www.mooooc.com/springboot/2017/11/11/springboot-jenkins.html)，只需要前期做一些简单的配置，当我们需要发布项目时只需要点击项目对应的发布按钮，就可以将项目从版本库中拉取、打包、发布到目标服务器中，大大简化了运维后期的部署工作。

虚拟化技术的发展给我们带来了更多的可能性，我们可以利用容器化技术，将 Spring Boot 项目做成镜像，根据容器集群的策略来实现弹性扩容、动态部署等。所以 Spring Boot + Docker + Jenkins 会将 Spring Boot 项目的部署做得更简单化、智能化。

#### Spring Boot 让监控变得更简单

可以说 Spring Boot 就是一款自带监控的开源软件，在设计之初就考虑到应用的监控问题，专门提供了一款监控组件来完成这个工作，这个组件就是 Spring Boot Actuator 。

Spring Boot 使用“约定优于配置的理念”，采用包扫描和自动化配置的机制来加载依赖 Jar 中的 Spring Bean，不需要任何 Xml 配置，就可以实现 Spring 的所有配置。虽然这样做能让我们的代码变得非常简洁，但是整个应用的实例创建和依赖关系等信息都被离散到了各个配置类的注解上，这使得我们分析整个应用中资源和实例的各种关系变得非常的困难。

Spring Boot Actuator 是 Spring Boot 提供的对应用系统监控的集成功能，可以查看应用配置的详细信息，例如自动化配置信息、创建的 Spring beans 以及一些环境属性等。

Spring Boot Actuator 监控分成两类：原生端点和用户自定义端点；自定义端点主要是指扩展性，用户可以根据自己的实际应用，定义一些比较关心的指标，在运行期进行监控。

原生端点是在应用程序里提供众多 Web 接口，通过它们了解应用程序运行时的内部状况。原生端点又可以分成三类：

- 应用配置类：可以查看应用在运行期的静态信息：例如自动配置信息、加载的 Spring Bean 信息、yml 文件配置信息、环境信息、请求映射信息；
- 度量指标类：主要是运行期的动态信息，例如堆栈、请求连、一些健康指标、metrics 信息等；
- 操作控制类：主要是指 shutdown，用户可以发送一个请求将应用的监控功能关闭。

所以说使用 Spring Boot Actuator 几乎可以监控一个 Spring Boot 应用的各方面信息，包括内存中堆栈的使用、应用的状态如何、加载了哪些 Spring Beans、应用的环境信息等等。可以说 Spring Boot Actuator 就是 Spring Boot 的透视镜，这样当 Spring Boot 应用出现问题的时候，方便我们根据这些信息监测、跟踪、解决问题。

当然 Spring Boot Actuator 虽然可以监控一个 Spring Boot 应用的健康情况，实际上现在的系统都是需要很多的服务相互配合来完成工作，如何通过一个监控软件来监控所以的 Spring Boot 项目将变得比较紧迫。

在开源界也有人意识到了这个问题，并且基于 Spring boot actuator 做出了一款强大的监控软件，这个软件就是 Spring Boot admin 。

Spring Boot Admin 是一个管理和监控 Spring Boot 应用程序的开源软件。每个应用都认为是一个客户端，通过 HTTP 或者使用 Eureka 注册到 admin server 中进行展示，Spring Boot Admin UI 部分使用 AngularJs 将数据展示在前端。

Spring Boot Admin 是一个针对 spring-boot 的 actuator 接口进行UI美化封装的监控工具。他可以：在列表中浏览所有被监控 spring-boot 项目的基本信息，详细的 Health 信息、内存信息、JVM 信息、垃圾回收信息、各种配置信息（比如数据源、缓存列表和命中率）等，还可以直接修改logger的level。

使用 Spring Boot Admin 不仅可以监控 Spring Boot 项目，还可以监控 Spring Cloud 项目，因此使用了 Spring Boot 项目之后我们监控 Spring Boot 集群效果如下：

![img](https://github.com/codecentric/spring-boot-admin/raw/master/images/screenshot.png)
![img](https://github.com/codecentric/spring-boot-admin/raw/master/images/screenshot-details.png)
![img](https://github.com/codecentric/spring-boot-admin/raw/master/images/screenshot-logging.png)![img](https://github.com/codecentric/spring-boot-admin/raw/master/images/screenshot-hystrix.png)

简单、直观、易用是它的特点，针对一些特殊情况还可以提供报警服务。所以说使用 Spring Boot Actuator 解决了单个 Spring Boot 的监控问题，使用 Spring Boot Admin 就是解决了整个集群监控的问题。

### Spring 、Spring Boot 和 Spring Cloud 的关系

Spring 最初最核心的两大核心功能 Spring Ioc 和 Spring Aop 成就了 Spring，Spring 在这两大核心的功能上不断的发展，才有了 Spring 事务、Spirng Mvc 等一系列伟大的产品，最终成就了 Spring 帝国，到了后期 Spring 几乎可以解决企业开发中的所有问题。

Spring Boot 是在强大的 Spring 帝国生态基础上面发展而来，发明 Spring Boot 不是为了取代 Spring，是为了让人们更容易的使用 Spring 。所以说没有 Spring 强大的功能和生态，就不会有后期的 Spring Boot 火热，Spring Boot 使用约定优于配置的理念，重新重构了 Spring 的使用，让 Spring 后续的发展更有生命力。

Spring Cloud 是一系列框架的有序集合。它利用 Spring Boot 的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用 Spring Boot 的开发风格做到一键启动和部署。

Spring 并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过 Spring Boot 风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

根据上面的说明我们可以看出来，Spring Cloud 是为了解决微服务架构中服务治理而提供的一系列功能的开发框架，并且 Spring Cloud 是完全基于 Spring Boot 而开发，Spring Cloud 利用 Spring Boot 特性整合了开源行业中优秀的组件，整体对外提供了一套在微服务架构中服务治理的解决方案。

综上我们可以这样来理解，正是由于 Spring Ioc 和 Spring Aop 两个强大的功能才有了 Spring，Spring 生态不断的发展才有了 Spring Boot ，使用 Spring Boot 让 Spring 更易用更有生命力，Spring Cloud 是基于 Spring Boot 开发的一套微服务架构下的服务治理方案。

用一组不太合理的包含关系来表达它们之间的关系。

> Spring ioc/aop > Spring > Spring Boot > Spring Cloud

### 使用 Spring Boot 构建微服务的优势

微服务架构是在互联网高速发展，技术日新月异的变化以及传统架构无法适应快速变化等多重因素的推动下诞生的产物。互联网时代的产品通常有两类特点：需求变化快和用户群体庞大，在这种情况下，如何从系统架构的角度出发，构建灵活、易扩展的系统，快速应对需求的变化；同时，随着用户的增加，如何保证系统的可伸缩性、高可用性，成为系统架构面临的挑战。

如果还按照以前传统开发模式，开发一个大型而全的系统已经很难满足市场对技术的需求，这时候分而治之的思想被提了出来，于是我们从单独架构发展到分布式架构，又从分布式架构发展到 SOA 架构，服务不断的被拆分和分解，粒度也越来越小，直到微服务架构的诞生。

大约 2009 年开始，Netflix 完全重新定义了它的应用程序开发和操作模型，拉开了微服务探索的第一步，直到2014年3月 Martin Fowler 写的一篇文章 Microservices 以更加通俗易懂的形式为大家定义了什么是微服务架构。Martin Fowler 在文中阐述了对微服务架构的设想，认为微服务架构是一种架构模式，它提倡将单一应用程序划分成一组小的服务，服务之间互相协调、互相配合，为用户提供最终价值。

每个服务运行在其独立的进程中，服务和服务间采用轻量级的通信机制互相沟通（通常是基于 HTTP 的 RESTful API）。每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建。

Spring Boot 诞生时，正处于微服务概念在慢慢酝酿中，Spring Boot 的研发融合了微服务架构的理念，实现了在 Java 领域内微服务架构落地的技术支撑。Spring Boot 在开发、测试、部署、运维等方面都做了大量的优化，使用 Spring Boot 开发项目，可以快速响应需求、独立完成开发部署上线。

Spring Boot 的一系列特性容易实现微服务架构的落地，从目前众多的技术栈对比来看 Spring Boot 是 Java 领域微服务架构最优落地技术。

### 总结

不知道什么时候起，行业里一些开发人员愿意相信，使用复杂的软件就意味着采用了高深的技术；使用了大量的配置，就意味着软件有着很多比较强大的功能。在产品设计的时候有一个理念就是让产品操作足够的傻瓜化，假设用户是一个智商并不高的群体，却可以使他很容易的学会使用其产品，将此特性做为产品设计的一项标准之一。

其实我们的开源软件也是一款产品，繁琐并不意味着功能强大，反而有可能是设计不够合理；简洁也并不意味着简单，很有可能它只是将众多复杂的功能进行了封装，让我们在使用的时候足够的简单。好的产品如此，好的开源软件也应该如此，Spring Boot 的出现就是让编程变得更简单一些。

在此引用 Python 的经典设计格言，格言来源于 Python 但不限于 Python。

> 美丽优于丑陋，清楚优于含糊，简单优于复杂，复杂优于繁琐，平坦优于曲折，宽松优于密集。
>
> 重要的是可读性，特殊的案例不足以特殊到破坏规则。尽管实践可以打破真理，错误却不可置之不理。
>
> 除非另有明确要求。面对模棱两可，拒绝猜测。
>
> 总会有一个——最好是只有一个——显而易见的方式来明辨，哪怕这种方式在开始的时候可能并不明显。
>
> 现在有比没有好，尽管没有经常好于现在。
>
> 如果如何实现很难被解释清楚，那么这个想法就是一个坏想法；如果如何实现可以被很好的解释，那么这是一个好想法。