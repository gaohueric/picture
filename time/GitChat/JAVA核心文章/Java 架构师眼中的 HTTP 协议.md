

#### 内容提要

内容提要：

- 能否举例说明 Spring Data Rest 的实际用途？
- 请问你在实际项目中，做过文章里的哪些架构？
- 面试时会问有关 http 协议的内容吗，一般会涉及哪些内容？
- 请问你之前是怎么做 Etag 缓存的？
- http2 有实际使用吗，它的优点是什么？
- 在 Node.JS 使用 Swagger，修改接口的同时可以自动更新接口文档。请问 JAVA Spring 生态中有没有类似的技术？
- 请问 http 协议在微服务中起到了什么作用？
- http 协议的安全，需要考虑哪方面内容？
- 现在的证书去哪买比较合适？
- Springcloud 和 dubbo 谁得天下？
- 如何理解 http 请求过程中的长短连接？
- 如何学习 http 协议？
- 能否介绍一下 https 流量的解密，特别是在没有证书又做不了中间人代理的时候怎么办？

------

**问：能否举例说明 Spring Data Rest 的实际用途？**

**答：**我给大家做了一个 [demo](https://github.com/zhangzhenhuajack/spring_data_rest_demo)，运行效果如下图：

![enter image description here](http://images.gitbook.cn/ba5736a0-17d9-11e8-bfaa-e1b3dadd077d)

你可以看到，所有 rest 的服务 api 尽在眼底，并且其与 spring-data-rest-hal-browser 结合得非常好，直接可以有类似 swagger-ui.html 的效果。

代码如下：

![enter image description here](http://images.gitbook.cn/c1a554a0-17d9-11e8-bfaa-e1b3dadd077d)

它对小型服务非常友好。随着微服务越来越广泛地使用，如果可以让它快速搭起一个微服务，和 spring boot2.0 配合起来，它未来的发展肯定会越来越好。

另外 spring data rest 的好处是：它天然与 spring data jpa 集成。因为它是基于 spring 的，可以有很多扩展。

另一个好处是，它帮我们定义了返回的 json 格式，不需要我们去定义规范。这和 json api 规范的实现者有些异曲同工之效。

------

**问：请问你在实际项目中，做过文章里的哪些架构？**

**答：**在个别的 app 的 api 服务上面，做过 json api 的封装；对 api 的 json 返回格式做了统一的封装；框架层面解决 exception 的各种状态码的封装；http2 的网管层面的设置。

------

**问：面试时会问有关 http 协议的内容吗，一般会涉及哪些内容？**

**答：**首先，会问 http 协议的基本内容，考验地是大家的思维组织能力。其次，可能根据公司招聘的岗位不同，问一些针对性的问题。

最常见也是必问的是 spring 对 http 进行了什么封装。cookie 和 session 的关系是什么也是必考的。另外，还经常问到 http 的缓存原理是什么，你是怎么做的。

如果你能按文章中的条例说上 80%。你通过面试的可能性很高。

------

**问：请问你之前是怎么做 Etag 缓存的？**

**答：**做法如下：

1. 自定义了 EtagCache 注解。
2. 通过拦截器判断带 EtagCache 注解的 controller。
3. 通过拦截器判断带 EtagCache 注解的 controller。
4. 将其值放在 http 的 header 里面，即可。

另一种做法是针对内容进行 hash code 编码。不过这是之前比较老的做法，不建议这么复杂的来做。

其实，明白 etag 的原理后，缓存就非常好做了。

------

**问：http2 有实际使用吗，它的优点是什么？**

**答：**这个问题非常好。不知道你有没有关注 serverapi 是 http 的哪个版本？

（答案：通常是 1.1）

![enter image description here](http://images.gitbook.cn/2f0423f0-17da-11e8-bfaa-e1b3dadd077d)

Google 采用的就是 http2 的协议版本。aws 亚马逊的云平台，它的 lbs 也默认是 http2.0 的协议。

http2.0 有个业内不成文的约定，一般都是必须要上 https 的。所以从这一点出发，很多大公司其实都是在推 http2.0。

还有个好处，就是浏览器的内核大部分支持 http2.0，就像我们文章里面说的，无阻塞的 multiplexing 请求队列。

![enter image description here](http://images.gitbook.cn/380dd4a0-17da-11e8-bfaa-e1b3dadd077d)

还有个好处，http2.0 规定了 server 端可以 push 给 client 端资源，不过它一般不需要我们自己去实现的。除非像各种云平台，你自己去做网关这一层。

------

**问：在 Node.JS 使用 Swagger，修改接口的同时可以自动更新接口文档。请问 JAVA Spring 生态中有没有类似的技术？**

**答：** Swagger 其实是个整体解决方案，是跨语言的。Swagger Spring 社区也有相应的开源产品。swagger2，又叫 springfox，它与 Spring boot 2.0 结合得非常好。

还有一个容易被大家忽略的，就是 spring data rest hal-browser。它们两者都是自动更新文档的。

但是 swagger-ui 更加完善，可以支持得更广，不过两者不能同时存在。

最后，强烈推荐 swagger-ui，神器。

------

**问：请问 http 协议在微服务中起到了什么作用？**

**答：**首先要明白什么是微服务，什么是微服务架构？

http 只是实现为服务架构的一种协议约定方案，正如我们文章里面所说的 spring cloud 的整体技术体系就是为微服务架构提供了一套技术解决方案，是基于 http 协议进行服务发现与管理的。

与之相对应的就是 rpc 了。阿里的开员工具就是基于 rpc 协议的微服务管理框架。

我个人认为，未来会逐渐将 http 协议微服务化。因为 http 协议的生态开源产品比较多，可选性比较大。

并且现在监控、心跳等更多微服务自理，越来越离不开 http 的 api 接口。整个生态 rpc 干不过 http，特大好处就是传输效率，性能高。

读者补充：看使用场景，基建类型的服务追求效率就使用 RPC，业务层面的场景为了前端调用的方便使用 REST 或者 GraphQL。

------

**问：http 协议的安全，需要考虑哪方面内容？**

**答：**其实只要用 http 协议，别人总能破解你能想到的方案。我就说几个注意事项吧：

1. 如果出于对数据安全的考虑建议走 https 协议，通过证书对内容进行加密。
2. 常见的就是：非对称加密，公钥，私钥，加解密。
3. 脚本注入，filter 过滤。
4. 每个接口可以约定一些必填的 Header，紧急关头可以很好地做好限流。
5. 要提前考虑黑白名单的 IP 地址。

------

**问：现在的证书去哪买比较合适？**

**答：**我遇到的几个运维都是到 aliyun 和 asw 国际站买证书比较多。

如果自己玩玩，免费的证书就可以，只要是个云平台一般都会免费的让你玩玩。如果是生产环境，还是不要见用免费的，这样会导致你的 api 不可信任。

------

**问：Springcloud 和 dubbo 谁得天下？**

**答：**我感觉 spring cloud 更好协议，不过 dubbo 使用起来更加便利。Spring cloud 主要是国外贡献开源的比较多，比如 netflix、推特等。国内用 dubbo 的人占大多数。

天下大事，目前很难看到苗头，不过感觉一般这两个在公司混用的比较多。如果你去阿里面试一定要看 dubbo。

------

**问：如何理解 http 请求过程中的长短连接？**

**答：**你知道目前 http1.1 默认是长连接还是短连接吗？

http1.1 本来是无状态的短链接，这是最早之前的做法。

目前最常见的 NIO 的做法：在个别的 app 的 api 服务上做 json api 的封装；对 api 的 json 返回格式做统一的封装；框架层面解决 exception 的各种状态码的封装；http2 的网管层面的设置。

短连接每次请求都是一次连接，要三次握手，效率低。你自己写api调用的时候需要自己注意连接池的配置问题。

------

**问：如何学习 http 协议？**

**答：**最好的方法就是抓包。看看别人是怎么写的，特别是大厂的 http 资源的规划和架构，很多抓包工具都是可以破解 https 的。

比如 mac 用 chs，win 用 wireshark 或者 fidder。

![enter image description here](http://images.gitbook.cn/5aefe260-17da-11e8-bfaa-e1b3dadd077d)

分析 http 流量，特别是带 etag 的数据的时候，burp 很好用。如果只是普通的分析，chrome 配合 postman 也是一个好组合。

firefox 上的抓包插件就是 httprequest，轻量又好用。

其次，看 spring boot 的源码，它对 http 封装得非常棒。多看源码，就会知道自己的关注重点应该在哪里。

不管对不对，先发表自己的意见，因为被别人反驳的时候也是你学习的好机会。

------

**问：能否介绍一下 https 流量的解密，特别是在没有证书又做不了中间人代理的时候怎么办？**

**答：**看看这篇文章[《抓包工具Charles的使用心得》](https://www.jianshu.com/p/fdd7c681929c)，关于如何查看 https 内容，这位老师写得挺好的。



#### 文章实录

### HTTP 协议的基本内容

#### 什么是 HTTP 协议

首先我们来看协议是什么？协议是指计算机通信网络中两台计算机之间进行通信所必须共同遵守有规则的文本格式。一但有了协议，就可以使很多公司分工起来，有些公司做 Server 端，如 Tomcat，而有些公司就可以做浏览器了。这样大家只要一套约定，彼此的通讯就会相互兼容。

接下来我们看什么是 HTTP？HTTP 是基于 TCP/IP 的应用层通信协议，它是客户端和服务器之间相互通信的标准。它规定了如何在互联网上请求和传输内容。通过应用层协议，我的意思是，它只是一个规范了主机（客户端和服务器）如何通信的抽象层，并且它本身依赖于 TCP/IP 来获取客户端和服务器之间的请求和响应。默认的 TCP 端口是80端口，当然，使用其他端口也是可以的。然而，HTTPS 使用的端口是443端口。

#### HTTP 协议的简单历史

![http 协议历史](http://images.gitbook.cn/48b9c280-10a2-11e8-b3e2-cdd55b7df45d)

根据上图，我们可将 HTTP 协议的发展历程分为五个阶段。

**第一阶段，1996年之前。**第一版的 HTTP 文档是1991年提出来的 HTTP/0.9，其主要特点有：（1）它仅有一个 GET 方法。（2）没有 header 数据块。（3）必须以HTML格式响应。

**第二阶段，HTTP/1.0 - 1996。**HTML 格式响应，HTTP/1.0 能够处理其他的响应格式，例如：图像、视频文件、纯文本或其他任何的内容类型（Content-Type 来区分）。它增加了更多的方法（即 POST 和 HEAD），请求/响应的格式也发生了改变，请求和响应中均加入了 HTTP 头信息，响应数据还增加了状态码标识，还介绍了字符集的支持、多部分发送、权限、缓存、内容编码等很多内容。HTTP/1.0 的主要缺点之一是，你不能在每个连接中发送多个请求。也就是说，每当客户端要向服务器端请求东西时，它都会打开一个新的 TCP 连接，并且在这个单独请求完成后，该连接就会被关闭。每一次连接里面都包含了著名的三次握手协议。于是有些 HTTP/1.0 的实现试图通过引入一个新的头信息 Connection: keep-alive，来解决这个问题。

**第三个阶段，HTTP/1.1 - 1999。**HTTP/1.0 发布之后，随着 HTTP 开始普及之后，它的缺点也开始展现。时隔三年，HTTP/1.1 便在1999年问世，它在之前的基础上做了很多的改进。主要内容包含：

- 新增的 HTTP 方法有 PUT、PATCH、HEAD、OPTIONS、DELETE。
- 主机名标识。在 HTTP/1.0 中，Host 头信息不是必须项，但 HTTP/1.1 中要求必须要有 Host 头信息。
- 持久性连接。正如前面所说，在 HTTP/1.0 中每个连接只有一个请求，且在这个请求完成后该连接就会被关闭，从而会导致严重的性能下降及延迟问题。HTTP/1.1 引入了对持久性连接的支持，例如：默认情况下连接不会被关闭，在多个连续的请求下它会保存连接的打开状态。想要关闭这些连接，需要将 Connection: close 加入到请求的头信息中。客户端通常会在最后一次请求中发送这个头信息用来安全的关闭连接。
- 管道机制。HTTP/1.1 也引入了对管道机制的支持，客户端可以向服务器发送多个请求，而无需等待来自同一连接上的服务器响应，并且当收到请求时服务器必须以相同的顺序来响应。但你可能会问客户端是怎么知道第一个响应下载完成和下一个响应内容开始的？要解决这个问题，必须要有 Content-Length 头信息，客户端可以用它来确定响应结束，然后开始等待下一个响应。

**第四个阶段，SPDY - 2009。**Google 走在前面，它开始试验一种可替换的协议来减少网页的延迟，使得网页加载更快、提升 Web 安全性。2009年，他们称这种协议为 SPDY。SPDY 的功能包含多路复用、压缩、优先级、安全等。2015年，谷歌不想存在两个相互竞争的标准，因此他们决定把它合并到 HTTP 中成为 HTTP/2，同时放弃 SPDY。

**第五个阶段，HTTP/2 - 2015。**HTTP/2 是专为低延迟传输的内容而设计。关键特征或与 HTTP / 1.1 旧版本的差异，如下。

- 二进制协议。HTTP/2 倾向于使用二进制协议来减少 HTTP/1.x 中的延迟。二进制协议更容易解析，而不具有像 HTTP/1.x 中那样对人的可读性。HTTP/2 中的数据块是帧和流。
  帧和流：

HTTP 消息是由一个或多个帧组成的。有一个叫做 HEADERS 的帧存放元数据，真正的数据是放在 DATA 帧中的，帧类型定义在the HTTP/2 specs（HTTP/2规范），如 HEADERS、DATA、`RST_STREAM`、SETTINGS、PRIORITY 等。每个 HTTP/2 请求和响应都被赋予一个唯一的流 ID 且放入了帧中。帧就是一块二进制数据。一系列帧的集合就称为流。每个帧都有一个流 id，用于标识它属于哪一个流，每一个帧都有相同的头。同时，除了流标识是唯一的，值得一提的是，客户端发起的任何请求都使用奇数和服务器的响应是偶数的流 id。除了 HEADERS 和 DATA， 另外一个值得说一说帧类型是 `RST_STREAM`，它是一个特殊的帧类型，用于中止流，如客户端发送这儿帧来告诉服务器我不再需要这个流了。在 HTTP/1.1 中只有一种方式来实现服务器停止发送响应给客户端，那就是关闭连接引起延迟增加，因为后续的请求就需要打开一个新的连接。 在 HTTP/2 中，客户端可以使用 RST_FRAME 来停止接收指定的流而不关闭连接且还可以在此连接中接收其它流。

- 多路复用。由于 HTTP/2 现在是一个二进制协议，且是使用帧和流来实现请求和响应，一旦 TCP 连接打开了，所有的流都通过这一连接来进行异步的发送而不需要打开额外的连接。反过来，服务器的响应也是异步的方式，如响应是无序的、客户端使用流 id 来标识属于流的包。这就解决了存在于 HTTP/1.x 中 head-of-line 阻塞问题，如客户端将不必耗时等待请求，而其他请求将被处理。如下图所示。

![http2.0 Multiplexing](http://images.gitbook.cn/8df0b930-10a2-11e8-b3e2-cdd55b7df45d)

- HPACK 头部压缩。它是一个单独的用于明确优化发送 Header RFC 的一部分。它的本质是，当我们同一个客户端不断的访问服务器时，在 header 中发送很多冗余的数据，有时 cookie 就增大 header，且消耗带宽和增加了延迟。为了解决这个问题， HTTP/2 引入了头部压缩。与请求和响应不同，header 不是使用 gzip 或 compress 等压缩格式，它有不同的机制，它使用了霍夫曼编码和在客户端和服务器维护的头部表来消除重复的 headers（如 User Agent)，在后续的请求中就只使用头部表中引用。它与 HTTP/1.1 中的一样，不过增加了伪 header，如 :method、:scheme、:host 和:path。
- 服务器推送。在服务器端，Server Push 是 HTTTP/2 的另外一个重要功能，我们知道，客户端是通过请求来获取资源的，它可以通过推送资源给客户端而不需客户端主动请求。例如，浏览器载入了一个页面，浏览器解析页面时发现了需要从服务器端载入的内容，接着它就发送一个请求来获取这些内容。Server Push允许服务器推送数据来减少客户端请求。它是如何实现的呢，服务器在一个新的流中发送一个特殊的帧 PUSH_PROMISE，来通知客户端：“嘿，我要把这个资源发给你!你就不要请求了。”
- 请求优先级。客户端可以在一个打开的流中在流的 HEADERS 帧中放入优先级信息。在任何时间，客户端都可以发送一个 PRIORITY 的帧来改变流的优先级。如果没有优先级信息，服务器就会异步的处理请求，比如无序处理。如果流被赋予了优先级，它就会基于这个优先级来处理，由服务器决定需要多少资源来处理该请求。
- 安全。大家对 HTTP/2 是否强制使用安全连接（通过 TLS）进行了充分的讨论。最后的决定是不强制使用。然而，大多数厂商表示，他们将只支持基于 TLS 的 HTTP/2。所以，尽管 HTTP/2 规范不需要加密，但它已经成为默认的强制执行的。在这种情况下，基于 TLS 实现的 HTTP/2 需要的 TLS 版本最低要求是1.2。 因此必须有最低限度的密钥长度、临时密钥等。

通过开发者工具我们看一下Google的请求协议。

![enter image description here](http://images.gitbook.cn/ac25b9a0-10a2-11e8-b3e2-cdd55b7df45d)

而我们大多数的网站的协议的版本都是 HTTP 1.1。

![enter image description here](http://images.gitbook.cn/b194de70-10a2-11e8-b3e2-cdd55b7df45d)

#### HTTP 协议的具体内容

而我们平时老生常谈的 HTTP 的协议大都是指的是 HTTP 1.1 协议的内容，接下去我们一起看一下 HTTP 1.1 协议的结构。如下图所示。![enter image description here](http://images.gitbook.cn/cfdcf1b0-10a2-11e8-b3e2-cdd55b7df45d)

接下来，我将通过四部分大概介绍一下 HTTP 协议的基本内容。

**1.URL & URI**

![enter image description here](http://images.gitbook.cn/e423add0-10a2-11e8-b3e2-cdd55b7df45d)

```
schema://host[:port#]/path/.../[;url-params][?query-string][#anchor]

```

URL（Uniform Resource Locator）主要包括以下几部分。

- scheme：指定低层使用的协议，一般是 HTTP，如果强调安全的话可以是 HTTPS。
- host：HTTP 服务器的 IP 地址或者域名。
- port：HTTP 服务器的默认端口是80，这种情况下端口号可以省略。如果使用了别的端口，必须指明。
- path：访问资源的路径。
- url-params：URL 的参数。
- query-string：发送给 HTTP 服务器的数据。
- anchor：锚。

URI，在 Java 的 Servlet 中指的是 resource path 部分。

**2.请求方法 Method**

主要包括以下几种请求方法。

- GET：向指定的资源发出“显示”请求。使用 GET 方法应该只用在读取数据，而不应当被用于产生“副作用”的操作中，例如在 Web Application 中。其中一个原因是 GET 可能会被网络蜘蛛等随意访问。
- POST：向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求本文中。这个请求可能会创建新的资源或修改现有资源，或二者皆有。
- PUT：向指定资源位置上传其最新内容。
- DELETE：请求服务器删除 Request-URI 所标识的资源。
- OPTIONS：这个方法可使服务器传回该资源所支持的所有 HTTP 请求方法。用“*”来代替资源名称，向 Web 服务器发送 OPTIONS 请求，可以测试服务器功能是否正常运作。
- HEAD：与 GET 方法一样，都是向服务器发出指定资源的请求。只不过服务器将不传回资源的本文部分。它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中“关于该资源的信息”（元信息或称元数据）。
- TRACE：回显服务器收到的请求，主要用于测试或诊断。
- CONNECT：HTTP/1.1 协议中预留给能够将连接改为渠道方式的代理服务器。通常用于 SSL 加密服务器的链接（经由非加密的 HTTP 代理服务器）。

Method 名称是区分大小写的。当某个请求所针对的资源不支持对应的请求方法的时候，服务器应当返回状态码 405（Method Not Allowed），当服务器不认识或者不支持对应的请求方法的时候，应当返回状态码 501（Not Implemented）。

**3.HTTP 之状态码**

状态代码有三位数字组成，第一个数字定义了响应的类别，共分五种类别:

- 1xx：指示信息--表示请求已接收，继续处理。
- 2xx：成功--表示请求已被成功接收、理解、接受。
- 3xx：重定向--要完成请求必须进行更进一步的操作。
- 4xx：客户端错误--请求有语法错误或请求无法实现。
- 5xx：服务器端错误--服务器未能实现合法的请求。

常见状态码有：

```
200 OK                        //客户端请求成功
400 Bad Request               //客户端请求有语法错误，不能被服务器所理解
401 Unauthorized              //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用 
403 Forbidden                 //服务器收到请求，但是拒绝提供服务
404 Not Found                 //请求资源不存在，eg：输入了错误的URL
500 Internal Server Error     //服务器发生不可预期的错误
503 Server Unavailable        //服务器当前不能处理客户端的请求，一段时间后可能恢复正常

```

**4.请求体&响应体**

请求体&响应体，这个没有特殊规定，需要配合不同的 Content-Type 来使用。

唯一需要注意的是 multipart/form-data、application/x-www-from-urlencoded、raw、binary 的区别。

（1）multipart/form-data

它将表单的数据组织成 Key-Value 形式，用分隔符 boundary（boundary 可任意设置）处理成一条消息。由于有 boundary 隔离，所以当即上传文件，又有参数的时候，必须要用这种 content-type 类型。如下图所示。

![enter image description here](http://images.gitbook.cn/9daaa4c0-10a3-11e8-b3e2-cdd55b7df45d)

（2）x-www-form-urlencoded

即 application/x-www-from-urlencoded，将表单内的数据转换为 Key-Value。这种和 Get 方法把参数放在 URL 后面一样的想过，这种不能文件上传。

![enter image description here](http://images.gitbook.cn/b59ba430-10a3-11e8-b3e2-cdd55b7df45d)

（3）raw

可以上传任意格式的“文本”，可以上传 Text、JSON、XML、HTML 等。

![enter image description here](http://images.gitbook.cn/c55a8e40-10a3-11e8-b3e2-cdd55b7df45d)

（4）binary

即 Content-Type:application/octet-stream，只可以上传二进制数据流，通常用来上传文件。由于没有键值，所以一次只能上传一个文件。

（5）Header

![enter image description here](http://images.gitbook.cn/d802dca0-10a3-11e8-b3e2-cdd55b7df45d)

HTTP 消息的 Headers 共分为三种，分别是 General Headers、Entity Headers、Request/Response Headers。

- General Headers

我把被 Request 和 Response 共享的 Headers 成为General Headers，具体有：

```
general-header = Cache-Control           
               | Connection       
               | Date             
               | Pragma           
               | Trailer          
               | Transfer-Encoding
               | Upgrade          
               | Via              
               | Warning

```

其中，Cache-Control 指定请求和响应遵循的缓存机制；Connection 允许客户端和服务器指定与请求/响应连接有关的选项；Date 提供日期和时间标志，说明报文是什么时间创建的；Pragma 头域用来包含实现特定的指令，最常用的是 Pragma:no-cache；Trailer，如果报文采用了分块传输编码(chunked transfer encoding) 方式，就可以用这个首部列出位于报文拖挂（trailer）部分的首部集合；Transfer-Encoding 告知接收端为了保证报文的可靠传输，对报文采用了什么编码方式；Upgrade 给出了发送端可能想要“升级”使用的新版本和协议；Via 显示了报文经过的中间节点（代理，网嘎un）。

- Entity Headers

Entity Headers 主要用来描述消息体（message body）的一些元信息，具体有：

```
entity-header  = Allow                   
               | Content-Encoding 
               | Content-Language 
               | Content-Length   
               | Content-Location 
               | Content-MD5      
               | Content-Range    
               | Content-Type     
               | Expires          
               | Last-Modified

```

其中，以 Content 为前缀的 Headers 主要描述了消息体的结构、大小、编码等信息，Expires 描述了 Entity 的过期时间，Last-Modified 描述了消息的最后修改时间。

- Request/Response Headers

Request-Line 是 Request 消息体的第一部分，其具体定义如下：

```
Request-Line = Method SP URI SP HTTP-Version CRLF
Method = "OPTIONS"
       | "HEAD"  
       | "GET"  
       | "POST"  
       | "PUT"  
       | "DELETE"  
       | "TRACE"

```

其中 SP 代表字段的分隔符，HTTP-Version 一般就是"http/1.1"，后面紧接着是一个换行。

在 Request-Line 后面紧跟着的就是 Headers。我们在上面已经介绍了 General Headers 和 Entity Headers，下面便是 Request Headers的定义。

```
request-header = Accept                   
               | Accept-Charset    
               | Accept-Encoding   
               | Accept-Language   
               | Authorization     
               | Expect            
               | From              
               | Host              
               | If-Match          
               | If-Modified-Since 
               | If-None-Match     
               | If-Range          
               | If-Unmodified-Since
               | Max-Forwards       
               | Proxy-Authorization
               | Range              
               | Referer            
               | TE                 
               | User-Agent

```

Request Headers 扮演的角色其实就是一个 Request 消息的调节器。需要注意的是若一个 Headers 名称不在上面列表中，则默认当做 Entity Headers 的字段。前缀为 Accept 的 Headers 定义了客户端可以接受的媒介类型、语言和字符集等。From、Host、Referer 和 User-Agent 详细定义了客户端如何初始化 Request。前缀为 If 的 Headers 规定了服务器只能返回符合这些描述的资源，若不符合，则会返回 304 Not Modified。

Request Body，若 Request-Line 中的 Method 为 GET，请求中不包含消息体，若为 POST，则会包含消息体。

一个具体的 Request 消息实例，如下。

```
GET /articles/http-basics HTTP/1.1
Host: www.articles.com
Connection: keep-alive
Cache-Control: no-cache
Pragma: no-cache
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

```

- Response 消息体

Response 消息格式和 Request 类似，也分为三部分，即 Response-Line、Response Headers、Response Body。

Response-Line 具体定义如下：

```
Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
HTTP-Version字段值一般为HTTP/1.1
Status-Code前面已经讨论过了
Reason-Phrase 是对status code的具体描述

```

一个最常见的 Response 响应为:

```
HTTP/1.1 200 OK    

```

Response Headers的定义如下。

```
response-header = Accept-Ranges
                | Age
                | ETag              
                | Location          
                | Proxy-Authenticate
                | Retry-After       
                | Server            
                | Vary              
                | WWW-Authenticate

```

其中，Age 表示消息自 server 生成到现在的时长，单位是秒；ETag 是对 Entity 进行 MD5 hash 运算的值，用来检测更改；Location 是被重定向的 URL；Server 表示服务器标识。

Http 更加详细的介绍，请参考[这里](http://www.runoob.com/http/http-status-codes.html) 。

### 架构师关注 HTTP 协议的重点

HTTP 协议内容其实也挺多的，架构师其实也应该有重点，哪些是我们必须重点关注的，心里要清楚。

#### 采用哪个 HTTP 协议版本及其 Java 里面如何配置

**1.Tomcat 的原始配置在 server.xml 里面。**

```
 <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="150" SSLEnabled="true" >
        <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                         certificateFile="conf/localhost-rsa-cert.pem"
                         certificateChainFile="conf/localhost-rsa-chain.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>

```

**2.Spring boot2.0 项目的配置方法。**

只需要在 properties 里面选择如下配置即可。

```
server.ssl.enabled=true
server.ssl.****//等等
server.http2.enabled=true
server.tomcat.protocol-header-https-value=https

```

需要注意的是一般 HTTP 2的使用都要伴随着证书。详细的配置直接参考 ServerProperties 类里面的具体描述即可。

#### HTTPS 配置注意事项

一般很少针对单个项目去设置的，一般都是通过开源的云如 ali、asw里面的 SLB 配置 HTTP2和证书，在网关那层统一做掉。客户端一般是各个浏览器或者 App 的浏览器内核库来支持的。其实也很少需要开发来关心具体如何按照 HTTP2 来实现一些代码逻辑。

#### 缓存机制 HTTP 缓存

**1.如何缓存**

降低网络上发送 HTTP 请求的次数，这里采用“过期”机制。

HTTP 服务器通过两种实体头（Entity-Header）来实现“过期”机制：Expires 头和 Cache-Control 头的 max-age 子项。

Expires/Cache-Control 控制浏览器是否直接从浏览器缓存取数据还是重新发请求到服务器取数据。只是 Cache-Control 比 Expires 可以控制的多一些，而且 Cache-Control 会重写 Expires 的规则。

降低网络上完整回复 HTTP 请求包的次数，这里采用“确证”机制。

HTTP服务器通过两种方式实现“确证”机制：ETag 以及 Last-Modified。

**2.相关的 Header**

主要包括以下几个。

- Cache-Control

常用的值有：

（1）max-age（单位为 s）指定设置缓存最大的有效时间，定义的是时间长短。当浏览器向服务器发送请求后，在 max-age 这段时间里浏览器就不会再向服务器发送请求了。 （2）s-maxage（单位为 s）同 max-age，只用于共享缓存（比如 CDN 缓存），也就是说 max-age 用于普通缓存，而 s-maxage 用于代理缓存。如果存在 s-maxage，则会覆盖掉 max-age 和 Expires header。 （3）public 指定响应会被缓存，并且在多用户间共享。如果没有指定 public 还是 private，则默认为 public。 （4）private 响应只作为私有的缓存，不能在用户间共享。如果要求 HTTP 认证，响应会自动设置为 private。 （5）no-cache 指定不缓存响应，表明资源不进行缓存，比如，设置了 no-cache 之后并不代表浏览器不缓存，而是在缓存前要向服务器确认资源是否被更改。因此有的时候只设置 no-cache 防止缓存还是不够保险，还可以加上 private 指令，将过期时间设为过去的时间。 （6）no-store 表示绝对禁止缓存。一看就知道，如果用了这个命令，当然就是不会进行缓存啦！每次请求资源都要从服务器重新获取。 （7）must-revalidate 指定如果页面是过期的，则去服务器进行获取。这个指令并不常用，就不做过多的讨论了。

- Expires

缓存过期时间，用来指定资源到期的时间，是服务器端的具体时间点。也就是说，Expires=max-age + 请求时间，需要和 Last-modified 结合使用。但在上面我们提到过 cache-control 的优先级更高。Expires 是 Web 服务器响应消息头字段，在响应 HTTP 请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。

- Last-modified

服务器端文件的最后修改时间，需要和 cache-control 共同使用，是检查服务器端资源是否更新的一种方式。当浏览器再次进行请求时，会向服务器传送 If-Modified-Since 报头，询问 Last-Modified 时间点之后资源是否被修改过。如果没有修改，则返回码为304，使用缓存；如果修改过，则再次去服务器请求资源，返回码和首次请求相同为200，资源为服务器最新资源。

- Etag

根据实体内容生成一段 hash 字符串，标识资源的状态，由服务端产生。浏览器会将这串字符串传回服务器，验证资源是否已经修改。

为什么要使用 Etag 呢?Etag 主要为了解决 Last-Modified 无法解决的一些问题。

一些文件也许会周期性的更改，但是它的内容并不改变（仅仅改变的修改时间），这个时候我们并不希望客户端认为这个文件被修改了，而重新 Get。

某些文件修改非常频繁，比如在秒以下的时间内进行修改（比方说1s内修改了 N 次），If-Modified-Since 能检查到的粒度是 s 级的，这种修改无法判断（或者说 UNIX 记录 MTIME 只能精确到秒）。

某些服务器不能精确的得到文件的最后修改时间。

缓存过程如下图所示。
![enter image description here](http://images.gitbook.cn/7552f7a0-10a5-11e8-b3e2-cdd55b7df45d)

#### Session 与 Cookie 必知必会

很好的解决了 HTTP 通讯中状态问题，但其本身也存在一些问题，比如：

- 客户端存储，可能会被修改或删除。
- 发送请求时，Cookie 会被一起发送到服务器，当 Cookie 数据量较大时也会带来额外的请求数据量。
- 客户端对 Cookie 数量及大小有一定的限制，Session 解决了 Cookie 的一些缺点。Session 同样是为了记录用户状态，对于每个用户来说都会有相应的一个状态值保存在服务器中，而只在客户端记录一个 sessionID 用于区分是哪个用户的 Session。

与 Cookie 相比，Session有一定的优势，如：

- Session 值存储在服务器，相对来说更安全。
- 客户端发送给服务器的只有一个 sessionID，数据量更小。Session同样需要在客户端存储一个 sessionID。可以这个值存储在 Cookie，每次发送请求时通过 Cookie 请求头将其发送到服务器；也可以不使用 Cookie，而将 sessionID 作为一个额外的请求参数，通过 URL 或请求体发送到服务器。

基于 Cookie 实现 Session 的实现原理如下图的示。

![enter image description here](http://images.gitbook.cn/ba90cc20-10a5-11e8-b3e2-cdd55b7df45d)

由上可见，基于 Cookie 实现 Session 时，其本质上还是在客户端保存一个 Cookie 值。这个值就是 sessionID，sessionID 的名称也可按需要设置，为保存安全，其值也可能会在服务器端做加密处理。服务器在收到 sessionID 后，就可以对其解密及查找对应的用户信息等。

#### 协议格式如何统一（见文章后面内容）

### Spring 对 HTTP 协议的支持

我为什么想提一下这个呢，我看到太多的开发者遇到 HTTP 协议都喜欢自定义变量，自定义类，其实完全没有必要。且看下面的分析。

#### Spring MVC Web

在 spring-web**.jar 里面我们可以找到如下几个类：

![enter image description here](http://images.gitbook.cn/fa01cb70-10a5-11e8-b3e2-cdd55b7df45d)

需要我们重点关注的有 HttpStatus、MediaType等。

![enter image description here](http://images.gitbook.cn/2e41cd40-10a6-11e8-b3e2-cdd55b7df45d)

Spring web bind中对应的注解，如下图所示。

![enter image description here](http://images.gitbook.cn/2671fc20-10a6-11e8-b3e2-cdd55b7df45d)

上面是一些主要的（需要注意的是 @RequestParam、@PostMapping、@****Mapping 与我们上面的 HttpMethod 相对应。而里面还有 Headers、Consumes 和 produces 等参数来确定一些 Mapping 的条件），下面的可以根据需要查看源码里面有哪些注解。

![enter image description here](http://images.gitbook.cn/3f3d2ef0-10a6-11e8-b3e2-cdd55b7df45d)

#### Spring Data Rest

Spring Data Rest 是基于 Spring Data repositories，分析实体之间的关系。为我们生成Hypermedia API(HATEOAS)风格的Http Restful API接口。

Spring Data REST通过构建在Spring Data repositories之上，自动将其导出为REST资源的api，减少了大量重复代码和无聊的样板代码。它利用超媒体来允许客户端查找存储库暴露的功能，并将这些资源自动集成到相关的超媒体功能中。

Spring Data REST本身就是一个Spring MVC应用程序，它的设计方式应该是尽可能少的集成到现有的Spring MVC应用程序中。现有的（或将来的）服务层可以与Spring Data REST一起运行，只有较小的考虑。

#### Spring RestTemplate 的实际使用

Spring RestTemplate 是 Spring 提供的用于访问 Rest 服务的客户端，RestTemplate 提供了多种便捷访问远程 HTTP 服务的方法，能够大大提高客户端的编写效率。

简单例子，如下。

```
RestTemplate restTemplate = new RestTemplate();
String fooResourceUrl = "http://localhost:8080/spring-rest/foos";
ResponseEntity<String> response = restTemplate.getForEntity(fooResourceUrl + "/1", String.class);
assertThat(response.getStatusCode(), equalTo(HttpStatus.OK));

```

更详尽例子，如下。

```
/**
 * 发送一个get请求，并接受封装成map
 */
@Test
public void restTemplateMap() {
    RestTemplate restTemplate = new RestTemplate();
    Map map=restTemplate.getForObject("https://api.weixin.qq.com/cgi-bin/getcallbackip?access_token=ACCESS_TOKEN",Map.class);
    System.out.println(map.get("errmsg"));
}

/**
 * 发送一个get请求，并接受封装成string
 */
@Test
public void restTemplateString() {
    RestTemplate restTemplate = new RestTemplate();
    String str=restTemplate.getForObject("https://api.weixin.qq.com/cgi-bin/getcallbackip?access_token=ACCESS_TOKEN",String.class);
    System.out.println(str);
}
/**
 * 添加消息头
 */
@Test
public void httpHeaders() {
    final String uri = "https://api.weixin.qq.com/cgi-bin/getcallbackip?access_token=ACCESS_TOKEN";
    RestTemplate restTemplate = new RestTemplate();
    HttpHeaders headers = new HttpHeaders();
    headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
    HttpEntity<String> entity = new HttpEntity<String>("parameters", headers);
    ResponseEntity<String> result = restTemplate.exchange(uri, HttpMethod.GET, entity, String.class);
    System.out.println(result);
}

```

我们在实际工作中会覆盖默认的 RestTemplate。

```
/**
 * 替代默认的SimpleClientHttpRequestFactory
 * 设置超时时间重试次数
 * 还可以设置一些拦截器以便监控
 *
 * @return
 */
@Bean
public RestTemplate restTemplate() {
    //生成一个设置了连接超时时间、请求超时时间、异常重试次数3次
    RequestConfig config = RequestConfig.custom().setConnectionRequestTimeout(10000).setConnectTimeout(10000).setSocketTimeout(30000).build();
//实际工作中，这个地方还会加上filter来抓取每次restTemplate的日志信息。
    HttpClientBuilder builder = HttpClientBuilder.create().setDefaultRequestConfig(config).setRetryHandler(new DefaultHttpRequestRetryHandler(3, false));
    HttpClient httpClient = builder.build();
    ClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
    RestTemplate restTemplate = new RestTemplate(requestFactory);
    return restTemplate;
}

```

使用的地方会变成如下即可。

```
@Autowired
public RestTemplate restTemplate;

```

#### Spring Data Jpa

随着使用 Spring Data Jpa 的人越来越多，它里面也对 Spring Web 做了很好的支持。

![enter image description here](http://images.gitbook.cn/20ee4870-10a7-11e8-b3e2-cdd55b7df45d)

我们可以重点看一下 Pageable 和 Page，以及 PageImpl 和 PageRequest 对分页和排序做了很好的封装，及其返回的 JSON 格式也做了很好的约定。

#### Spring Cloud 中的知识点

Spring Cloud 的微服务管理都是基于 HTTP 协议的 Rest 风格的 API 来管理的，所以我们详细了解 HTTP 协议还是非常有必要的。

![enter image description here](http://images.gitbook.cn/3083d890-10a7-11e8-b3e2-cdd55b7df45d)

### JSON API

我们都知道了约定的好处。如果你和你的团队曾经争论过使用什么方式构建合理 JSON 响应格式， 那么 JSON API 就是你的 anti-bikeshedding 武器。

通过遵循共同的约定，可以提高开发效率，利用更普遍的工具，可以是你更加专注于开发重点：你的程序。

点击[这里](http://jsonapi.org/)，访问 JSON API 官方。

（1）JSON API 介绍

JSON API 是数据交互规范，用以定义客户端如何获取与修改资源，以及服务器如何响应对应请求。

JSON API 设计用来最小化请求的数量，以及客户端与服务器间传输的数据量。在高效实现的同时，无需牺牲可读性、灵活性和可发现性。

JSON API 需要使用 JSON API 媒体类型（application/vnd.api+json）进行数据交互。

JSON API 服务器支持通过 GET 方法获取资源。而且必须独立实现 HTTP POST，PUT 和 DELETE 方法的请求响应，以支持资源的创建、更新和删除。

JSON API 服务器也可以选择性支持 HTTP PATCH 方法 [RFC5789]和 JSON Patch 格式 [RFC6902]，进行资源修改。JSON Patch 支持是可行的，因为理论上来说，JSON API 通过单一 JSON 文档，反映域下的所有资源，并将 JSON 文档作为资源操作介质。在文档顶层，依据资源类型分组。每个资源都通过文档下的唯一路径辨识。

（2）规则约定

文档中的关键字MUST、MUST NOT、REQUIRED、SHALL、SHALL NOT、SHOULD、 SHOULD NOT、RECOMMENDED、MAY 和 OPTIONAL。依据 RFC 2119 [RFC2119] 规范解释。

（3）内容约定

客户端职责，即客户端必须在包含 Content-Type: application/vnd.api+json 头并且不包含媒体类型参数的请求文档中发送所有 JSON API 数据。在 Accept 头中包含 JSON API 媒体类型并且不包含媒体类型参数的客户端必须在 Accept 头中指定媒体类型至少一次。

客户端必须忽略任何从响应文档的 Content-Type 头中获取的 application/vnd.api+json 媒体类型参数。

服务器职责，即服务器必须在包含 Content-Type: application/vnd.api+json 头并且不包含媒体类型参数的请求文档中发送所有 JSON API 数据。如果接收到一个用任何媒体类型参数指定 Content-Type: application/vnd.api+json 头的请求，服务器必须返回一个 415 Unsupported Media Type 状态码响应。如果接收到一个在 Accept 头中包含任何 JSON API 媒体类型并且所有实体都以媒体类型参数更改的请求，服务器必须返回一个 406 Not Acceptable状态码响应。

（4）文档结构

我们将[描述 JSON API 文档结构](http://jsonapi.org/format/) ，通过媒体类型 application/vnd.api+json 标示。JSON API 文档使用 JavaScript 对象（JSON）[RFC4627]定义。

尽管同种媒体类型用以请求和响应文档，但某些特性只适用于其中一种。差异在下面呈现。

除非另有说明，根据本规范定义的对象都不应该包含任何其他键。客户端和服务器实现必须忽略本规范未指定的键。

（5）Top Level

JSON 对象必须位于每个 JSON API 文档的根级。这个对象定义文档的“top level”。

文档必须包含以下至少一种 top-level 键。

- data: 文档的”primary data”。
- errors: 错误对象列表。
- meta: 包含非标准元信息的元对象。

data 键和 errors 键不能再一个文档中同时存在。

文档可能包含以下任何 top-level 键。

- jsonapi: 描述服务器实现的对象。
- links: 与primary data相关的链接对象。
- include: 与 primary data 或其他资源相关的资源对象（included resources）列。

如果文档不包含 top-level data 键，included 键也不应该出现。

文档的 top-level 链接对象可能包含以下键。

- self: 生成当前响应文档的链接。
- related: 当primary data代表资源关系时，表示相关资源链接。
- Primary data的分页链接。

文档中的“primary data”代表一个请求所要求的资源或资源集合。

Primary data 必须是以下列举的一种。

- 如果请求要求单一资源，应该是一个单一资源对象，或一个单一资源标识符，或 null。
- 如果请求要求资源集合，应该是一个资源对象列表，或一个空列表([])。

例如，以下 primary data 表示一个单一资源对象。

```
{
  "data": {
    "type": "articles",
    "id": "1",
    "attributes": {
      // ... this article's attributes
    },
    "relationships": {
      // ... this article's relationships
    }
  }
}

```

以下 primary data 表示一个指向同样资源的单一资源标识符。

```
{
  "data": {
    "type": "articles",
    "id": "1"
  }
}

```

即使只包含一个元素或为空，资源的一个逻辑集合也必须表示为一个列表。

资源对象，即 JSON API 文档中的“Resuorce objects”，代表资源。

一个资源对象必须至少包含以下 top-level 键。

- id
- `type'

例外，当资源对象来自客户端并且代表一个将要在服务器创建的新资源时，id键不是必须的。

此外，资源对象可能包含以下 top-level 键。

- 'attribute': 属性对象代表资源的某些数据。
- relationshiops: 关联对象描述该资源与其他JSON API资源之间的关系。
- links: 链接资源包含与资源相关的链接。
- meta: 元数据资源包含与资源对象相关的非标准元信息，这些信息不能被作为属性或关联对象表示。

一篇文献（即一个“文献”类型的资源）在文档中这样表示:

```
{
  "type": "articles",
  "id": "1",
  "attributes": {
    "title": "Rails is Omakase"
  },
  "relationships": {
    "author": {
      "links": {
        "self": "/articles/1/relationships/author",
        "related": "/articles/1/author"
      },
      "data": { "type": "people", "id": "9" }
    }
  }
}

```

标识符，即每个资源对象包含一个 id 键和一个 type 键。id 键和 typ e键的值必须是字符串类型。

对于每一个既定 API，每个资源对象的 type 和 id 对必须定义一个单独且唯一的资源（由一个或多个但行为表现为一个服务器的服务器控制的 URI 集合构成一个API）。

type 键用于描述共享相同属性和关联的资源对象。type键的值必须与遵循键名称相同的约束条件。

字段，即资源对象的属性和关联被统称为“fields”。

一个资源对象的所有字段必须与 type 和 id 在同一命名空间中。即一个资源不能拥有名字相同的属性与关联，也不能拥有被命名为 type 或 id 的属性和关联。

属性，即 attribute，键的值必须是一个对象（一个“attributes object”)。属性对象的键（“attributes”）代表与资源对象中定义的与其有关的信息。

属性可以包含任何合法 JSON 值。JSON 对象和列表涉及的复杂数据结构可以作为属性的值。但是一个组成或被包含于属性中的对象不能包含 relationships 或 links 键，因为这些键为此规范未来的用途所预留。

虽然一些 has-one 关系的外键（例如author_id）被在内部与其他将要在资源对象中表达的信息一起储存，但是这些键不能作为属性出现。

关联，即relationships，键的值必须是一个对象（“relationships object”）。关联对象（“relationships”）的键表示在资源对象中定义的与其相关的其他资源对象。

关联可以是单对象关联或多对象关联。

一个“relationship object”必须包含以下至少一种键：

- links: 一个链接对象至少包含以下一种键：
  - self: 指向关联本身的链接（“relationship link”）。此链接允许客户端直接修改关联。例如，通过一个 articale 的关联 URL 移除一个 author 将会解除一个人与 article 的关系，而不需要删除这个 people 资源本身。获取成功后，这个链接将返回一个相关资源之间的连接，将其作为 primary data（见获取关联）。
  - related: 相关资源链接。
- data: 资源连接。
- meta: 包含关于此关联的非标准元信息的元对象。

更多介绍见[官方文档](http://jsonapi.org/format/)。

#### Yahoo elide 对 JSON API 的支持

点击[这里](http://elide.io/pages/guide/01-start.html)，访问 Yahoo elide官方网站。

**1. elide介绍**

elide 通过 Spring Data Jpa 的 Entity，加上自定义的 @Include(rootLevel = true) 注解，来完成 JSON API 标准的输出。

使用方法如下图所示。

![enter image description here](http://images.gitbook.cn/4bc2fc20-10a8-11e8-b3e2-cdd55b7df45d)

效果如下：

![enter image description here](http://images.gitbook.cn/51586da0-10a8-11e8-b3e2-cdd55b7df45d)

### 生产环境中 HTTP 协议有哪些架构

#### RestTemplate 的重试和监控

代码如下。

```
@SpringBootApplication
public class DubbingApiApplication {
   /**
   * 使用全局的RestTemplate
    * 设置了连接超时时间、请求超时时间、异常重试次数3次
    * 并且会记录所有请求的详细日志
    *
    * @return
    */
   @Bean
   public RestTemplate restTemplate() {
      RequestConfig config = RequestConfig.custom().setConnectionRequestTimeout(10000).setConnectTimeout(10000).setSocketTimeout(30000).build();
      HttpClentBuilder builder = HttpClientBuilder.create().setDefaultRequestConfig(config).setRetryHandler(new DefaultHttpRequestRetryHandler(3, false));
      HttpClient httpClient = builder.build();
      ClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
      RestTemplate restTemplate = new RestTemplate(requestFactory);
      restTemplate.setInterceptors(Collections.singletonList(new LoggingRestTemplate()));
      restTemplate.setRequestFactory(new HttpComponentsAsyncClientHttpRequestFactory());
      return restTemplate;
   }
   public static void main(String[] args) {
      SpringApplication.run(DubbingApiApplication.class, args);
   }
}

```

#### 请求格式和返回结果的格式约定。

1.实现 ResponseBodyAdvice 对 controller 返回的 Result 进行统一的包装，如下代码。

```
public class ElideResponseBodyAdvice implements ResponseBodyAdvice {
    @Autowired
    private ElideProperties elideProperties;
    /**
     * 配置注解可以跳过去，类上，方法上都行
     *
     * @param returnType
     * @param converterType
     * @return
     */
    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        if (converterType != null && converterType.isAssignableFrom(StringHttpMessageConverter.class)) {
            return false;
        }
        ElideSkippable elideSkippable = returnType.getMethodAnnotation(ElideSkippable.class);
        if (elideSkippable == null) {
            elideSkippable = returnType.getDeclaringClass().getAnnotation(ElideSkippable.class);
        }
        return !(elideSkippable != null && elideSkippable.value());
    }
    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
        HttpServletRequest httpServletRequest = ((ServletServerHttpRequest) request).getServletRequest();
       //only process elide persistable
        return ResultElideWrapHandler.process(httpServletRequest, body);
    }
}

```

2.异常统一格式的处理，返回固定的 error 级别的格式结果，如下代码。

```
/**
 * 框架级别的通用异常处理
 */
@ControllerAdvice
@Slf4j
@Order(4)
public class ExceptionAdvice {
   @ExceptionHandler({BindException.class})
   @ResponseBody
   public JsonApiErrorDocument exception(BindException e, HttpServletResponse response) {
      log.warn(e.getMessage(), e);
      ErrorResponse errors = new ErrorResponse(Constant.ERROR_VALIDATION, response.getStatus(), e.getMessage());
      errors.setMeta(e.getAllErrors());
      return new JsonApiErrorDocument(errors);
   }
}

```

#### 对缓存 Etag 的支持。

针对 HTTP 里 Etag 的支持，也需要我们框架层面去支持 Etag 缓存，这里就不给大家贴代码了，大家可以思考一下。

### 微服务中 HTTP 与 RPC 的权衡

#### HTTP 与 RPC 比较

如下表所示。

| 比较项      | HTTP                 | RPC                      |
| -------- | -------------------- | ------------------------ |
| 微服务治理    | Spring Cloud         | ali dubbo                |
| 耦合度      | 代码无耦合                | service jar依赖            |
| api升级    | 完全无影响                | 需要注意java Serializable的使用 |
| 周边开源监控产品 | 多                    | 少                        |
| 开发效率     | 快                    | 更快                       |
| 语言要求     | 可以不一个体系的语音           | 只能同一个体系如Java             |
| 性能       | 可以                   | 非常好                      |
| 带宽       | http<http1.1<http2.0 | 更小带宽                     |

实际工作中建议两者都用，API 对外，Ali Dubbo的 RPC 对内部使用，这样两个的优点都能使用到。

#### 总结：面试中起到的关键作用是什么

如果面试中问到你这个问题，主要的考验的点有：

- 思路是否清晰。
- 实战解决那些问题如缓存可能会让你说的非常细。
- 知识是否全面，及其是否针对一个问题了解的足够多。

基本上面的东西如果你都能提到，面试官的印象基本上是非常好的，加分会加很多。

