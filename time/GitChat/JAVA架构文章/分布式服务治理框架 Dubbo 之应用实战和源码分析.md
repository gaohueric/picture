# 分布式服务治理框架 Dubbo 之应用实战和源码分析

查看本场Chat

### Dubbo 工作原理

![enter image description here](http://images.gitbook.cn/41573d10-7506-11e8-9016-addc5ac06bc8)

- init：启动初始化
- async：异步调用
- sync：同步调用

**调用关系说明：**

1. 注册中心启动。
2. 服务提供者（Provider）发送本机 IP，服务暴露端口，被调方法的类信息，接口信息，版本号信息到注册中心存储
3. 服务消费者（Consumer）连接注册中心，发送所请求的方法信息，订阅服务
4. 注册中心（Registry）根据消费者订阅的服务匹配定位，把对应的服务提供者地址列表返回给服务消费者内存中保存，服务提供者如果有变更，注册中心将基于长连接通知消费者变更的数据。
5. 服务消费者，基于软负载均衡算法在本地缓存的提供者列表，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中统计调用次数和调用时间等信息，定时每分钟发送一次统计数据到监控中心（Monitor）。

![enter image description here](http://images.gitbook.cn/5d08e8d0-7518-11e8-9016-addc5ac06bc8)

大数据量传输时适合用短连接，小数据量高并发适合用长连接。我们看到，消费方和提供方都是通过长连接与注册中心通信的，当消费方调用服务时，会创建一个连接，然后同时会创建一个心跳发送的定时线程池，每一分钟发送一次心跳包到注册中心，通过 ping-pong 来保持连接的存活，同时还会启动断线重连定时线程池，每两秒钟检查一次连接状态。如果断开就重连，而当注册中心断开连接后，会回调通知消费方销毁连接，同理，提供方也是通过长连接与注册中心通信。

### Dubbo 调用

通过搭建用户系统和订单系统演示 Dubbo 服务调用过程，采用的环境：

- 操作系统：Window 7，64 位

- `Eclipse: Spring Tool Suite`：Spring Tool Suite 对 Eclipse 做了二次封装，开发起来更方便【[下载](http://spring.io/tools/sts/)】

  ![enter image description here](http://images.gitbook.cn/8d3882b0-75ed-11e8-936a-bd63663e31c4)

- JDK 1.7： X86 对应 32 位，X64 对应 64 位，现在下载要先注册，可以用邮箱注册登录后下载【[下载](http://www.oracle.com/technetwork/java/javase/archive-139210.html)】![enter image description here](http://images.gitbook.cn/26a23460-75f2-11e8-9c7a-43197c8c6da2)

Dubbo 管理包 dubbo-admin-2.5.3.war 只支持到 Jdk1.7 ，所以用 Jdk1.8 是无法运行 Dubbo2.5.3 版管理系统的。

直接点击下载后的 jdk1.7.exe 文件，按默认路径安装，然后再点击计算机—>属性—>高级系统设置—>环境变量，在用户变量下点击新建，添加配置：

```
JAVA_HOME C:\Program Files\Java\jdk1.7.0_45
JRE_HOME C:\Program Files\Java\jre7

```

在系统变量下点击新建，添加配置：

```
CLASSPATH .;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar

```

选择Path，点击编辑，在值的末尾添加：

```
;C:%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

```

最后点击确定。

![enter image description here](http://images.gitbook.cn/f021b110-75f4-11e8-9def-35cb75b6091c)

通过 Window+R，输入 Cmd，再输入 java -version 能显示对应的 Java 版本信息则表示配置正确，然后解压 Spring Tool Suite ，简称 STS，点击绿色的树叶图标即可启动，启动后点击窗口 Window—>Preferences，调整字体，默认字体小了：

![enter image description here](http://images.gitbook.cn/5ea69460-75f6-11e8-936a-bd63663e31c4)

![enter image description here](http://images.gitbook.cn/79b46430-75f6-11e8-9c7a-43197c8c6da2)

然后在选择项目的运行环境：

![enter image description here](http://images.gitbook.cn/d1e9df90-75f6-11e8-936a-bd63663e31c4)

![enter image description here](http://images.gitbook.cn/0bfc82f0-75f7-11e8-9c7a-43197c8c6da2)

![enter image description here](http://images.gitbook.cn/42919850-75f7-11e8-9def-35cb75b6091c)

然后再配置代码的自动提示功能：

![enter image description here](http://images.gitbook.cn/64545360-75f7-11e8-b9a0-850e12cc69e8)

替换成 abcdefghijklmnopqrstuvwxyz. 当你输入任何一个字母都会有提示。

我们来创建提供方订单系统，菜单 File—>New—>New Maven project：

![enter image description here](http://images.gitbook.cn/752f7420-75fd-11e8-9def-35cb75b6091c)

不需要 Maven 骨架：

![enter image description here](http://images.gitbook.cn/6c531500-75fd-11e8-9c7a-43197c8c6da2)

订单系统是提供服务方，是运行在 Tomcat 容器上的，所以选择 war 打包方式：

![enter image description here](http://images.gitbook.cn/1f14dee0-76c0-11e8-b314-7db67650851a)

OrderServiceImpl：

```
public class OrderServiceImpl implements OrderService {

    // 查询订单实现类，模拟订单数据
    public List<Order> queryOrderList() {
        List<Order> list = new ArrayList<Order>();
        for (int i = 0; i < 10; i++) {
            Order Order = new Order();
            Order.setOrderId(UUID.randomUUID().toString());
            Order.setOrderPrice(new BigDecimal("99"));
            Order.setOrderDate(new Date());
            Order.setOrderAddresss("中国");
            list.add(Order);
        }
        return list;
    }

}

```

dubbo-server：

```
<beans xmlns="http://www.springframework.org/schema/beans"xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"xmlns:aop="http://www.springframework.org/schema/aop"xmlns:tx="http://www.springframework.org/schema/tx"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsdhttp://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsdhttp://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 应用名称，可显示依赖关系 -->
    <dubbo:application name="dubbo-order-server" />

    <!-- 注册中心是ZooKeeper，也可以选择Redis做注册中心 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"
        client="zkclient" />

    <!-- 通过dubbo协议在注册中心（127.0.0.1表示本机）的20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" host="127.0.0.1" port="20880" />

    <!-- 提供服务用地的是service标签，将该接口暴露到dubbo中 -->
    <dubbo:service interface="com.dubbo.service.OrderService"
        ref="orderService" />

    <!-- Spring容器加载具体的实现类-->
    <bean id="orderService" class="dubbo.service.impl.OrderServiceImpl" />

    <dubbo:monitor protocol="registry" />


</beans>

```

log4j 打印日志级别为 DEBUG：

```
log4j.rootLogger=DEBUG,A1
log4j.logger.org.mybatis = DEBUG
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n

```

web：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns="http://java.sun.com/xml/ns/javaee"xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"id="WebApp_ID" version="2.5">

    <display-name>dubbo-order</display-name>
    <!-- 加载src/main/resources下的xml文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:dubbo/dubbo-*.xml</param-value>
    </context-param>

    <!--代码启动时加载Spring上下文环境ApplicationContext -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

</web-app>

```

Pom：

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.dubbo.order</groupId>
    <artifactId>dubbo-order</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>


    <dependencies>
        <dependency>
            <groupId>com.dubbo.order</groupId>
            <artifactId>dubbo-order-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <!-- dubbo运行于spring配置方式，需要导入spring容器依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.1.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.6.4</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.3</version>
            <exclusions>
                <exclusion>
         <!-- dubbo里已有相同依赖，排除传递spring依赖 -->
                    <artifactId>spring</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.3.3</version>
        </dependency>

        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>

<!-- 通过tomcat插件的方式运行，不需要自己手动配置tomcat server -->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <port>8081</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

STS Eclipse 已经内嵌了 Maven，不需要我们手动配置，通过 Pom 里配置 plugin 使用 Tomcat 插件启动服务，也不需要手动配置 Tomcat Server，可直接通过 Maven 命令 `tomcat7:run` 运行项目：

![enter image description here](http://images.gitbook.cn/fb5a9230-76c2-11e8-83a1-c946ff369755)

![enter image description here](http://images.gitbook.cn/009f4ba0-76c3-11e8-8ded-3bf9334a1994)

选择更新项目，这样 Maven 就会从互联网的中央仓库下载依赖，下载完毕项目红叉消失。

搭建消费端系统 dubbo-user，和提供方创建过程一样，都是创建 Maven 项目，只是 packing 选择 jar，因为我们通过 main 函数启动测试用例，不需要通过 Tomcat 容器加载。

```
     groupId com.dubbo.user 
     artifactId dubbo-user 
     version 0.0.1-SNAPSHOT 

```

![enter image description here](http://images.gitbook.cn/f0c93aa0-76c3-11e8-b314-7db67650851a)

dubbo-consumer：

```
<beans xmlns="http://www.springframework.org/schema/beans"xmlns:context="http://www.springframework.org/schema/context"xmlns:p="http://www.springframework.org/schema/p"xmlns:aop="http://www.springframework.org/schema/aop"xmlns:tx="http://www.springframework.org/schema/tx"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsdhttp://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsdhttp://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/txhttp://www.springframework.org/schema/tx/spring-tx-4.0.xsdhttp://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 应用名称，可显示依赖关系 -->
    <dubbo:application name="dubbo-user-consumer" />

    <!-- zookeeper作为注册中心 ，也可以选择Redis做注册中心 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"
        client="zkclient" />

    <dubbo:protocol host="127.0.0.1" />

    <!-- 调用服务使用reference标签，从注册中心中查找服务 -->
    <dubbo:reference id="orderService" interface="com.dubbo.service.OrderService" />

</beans>

```

log4j：

```
log4j.rootLogger=DEBUG,A1
log4j.logger.org.mybatis = DEBUG
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n

```

OrderServiceTest：

```
public class OrderServiceTest {


    private OrderService orderService;

    @Before
    public void setUp() throws Exception {
        // 加载容器，完成OrderService的初始化
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:dubbo/dubbo-*.xml");
        this.orderService = context.getBean(OrderService.class);
    }

    @Test
    public void testQueryOrder() {

        List<Order> list = this.orderService.queryOrderList();
        for (Order Order : list) {
            System.out.println(Order.toString());
        }

    }
}

```

Pom：

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.dubbo.user</groupId>
    <artifactId>dubbo-user</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.dubbo.order</groupId>
            <artifactId>dubbo-order-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <!-- dubbo运行于spring配置方式，需要导入spring容器依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.1.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.6.4</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.3</version>
            <exclusions>
                <exclusion>
       <!-- dubbo里已有相同依赖，排除传递spring依赖 -->
                    <artifactId>spring</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.3.3</version>
        </dependency>

        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
        <!--通过junit启动测试用例 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

```

搭建消费端和提供端共同依赖的接口系统，无论是服务的提供者还是消费者，都需要依赖共同的接口包，如果有细心看提供者和消费者的 Pom 依赖配置，会发现都依赖了接口包。

```
        <dependency>
            <groupId>com.dubbo.order</groupId>
            <artifactId>dubbo-order-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

```

那我们现在就搭建这个接口包，同上，创建 Maven 项目，packing 为 jar。

```
   groupId com.dubbo.order 
   artifactId dubbo-order-api 
   version 0.0.1-SNAPSHOT 

```

![enter image description here](http://images.gitbook.cn/28e17b50-76c9-11e8-a5cf-8d4ee65675fa)

Order：

```
package com.dubbo.dto;

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.Date;

// 跨系统传输的对象必须实现序列化接口
public class Order implements Serializable {

    private static final long serialVersionUID = -9178414320407482966L;

    private String orderId;

    private BigDecimal orderPrice;

    private String orderAddresss;

    private Date orderDate;

    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }

    public BigDecimal getOrderPrice() {
        return orderPrice;
    }

    public void setOrderPrice(BigDecimal orderPrice) {
        this.orderPrice = orderPrice;
    }

    public String getOrderAddresss() {
        return orderAddresss;
    }

    public void setOrderAddresss(String orderAddresss) {
        this.orderAddresss = orderAddresss;
    }

    public Date getOrderDate() {
        return orderDate;
    }

    public void setOrderDate(Date orderDate) {
        this.orderDate = orderDate;
    }

    // 自动生成复写的toString方法
    @Override
    public String toString() {
        return "Order [orderId=" + orderId + ", orderPrice=" + orderPrice + ", orderAddresss=" + orderAddresss+ ", orderDate=" + orderDate + "]";
    }

}

```

OrderService：

```
package com.dubbo.service;

import java.util.List;

import com.dubbo.dto.Order;

public interface OrderService {

    /**
     * 查询所有订单
     * 
     * @return
     */
    public List<Order> queryOrderList();

}

```

Pom：

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.dubbo.order</groupId>
  <artifactId>dubbo-order-api</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</project>

```

选择 dubbo-order-api 项目，右键选择：

![enter image description here](http://images.gitbook.cn/82dcb0a0-76cb-11e8-83a1-c946ff369755)

安装接口包到本地仓库，以便于提供者和消费者共同依赖。

下载 Windows 端 zookeeper-3.4.6，进入 zookeeper-3.4.6\bin 目录下，双击启动。

![enter image description here](http://images.gitbook.cn/4e569a90-76ca-11e8-b314-7db67650851a)

注册中心启动成功

![enter image description here](http://images.gitbook.cn/74496ac0-76ca-11e8-b314-7db67650851a)

选择 dubbo-order 项目，右键选择：

![enter image description here](http://images.gitbook.cn/ec728680-76ca-11e8-83a1-c946ff369755)

![enter image description here](http://images.gitbook.cn/f18385c0-76ca-11e8-83a1-c946ff369755)

![enter image description here](http://images.gitbook.cn/f6cafe50-76ca-11e8-8ded-3bf9334a1994)

启动成功后，因为提供者和注册中心是长连接通信，所以提供者启动后会有定时线程间隔固定时间发送心跳包到注册中心保活连接，然后打开 Cmd.exe 输入：

```
telnet 127.0.0.1 20880

```

能连接上去而不是提示打开端口失败，表明提供者注册成功，再输入 ls 查看注册服务列表，输入 invoke OrderService.queryOrderList() 来直接调用服务。

![enter image description here](http://images.gitbook.cn/5ad49f90-76cc-11e8-b314-7db67650851a)

调用成功，返回十条订单数据，这中调用方法在调试阶段会用到，我们也可以通过消费端的测试用例来调用，选中 testQueryOrder() 右键选择：

![enter image description here](http://images.gitbook.cn/6adaeb80-76cf-11e8-b314-7db67650851a)

![enter image description here](http://images.gitbook.cn/c2ca8ba0-76cc-11e8-8ded-3bf9334a1994)

这种项目搭建方式在企业里已成为了最佳实践。

服务调用者：

```
<!-- 调用服务使用reference标签，从注册中心中查找服务 -->
    <dubbo:reference id="orderService" interface="com.dubbo.service.OrderService" />

```

服务提供者：

```
<!-- 提供服务用地的是service标签，将该接口暴露到dubbo中 -->
    <dubbo:service interface="com.dubbo.service.OrderService"
        ref="orderService" />

    <!-- Spring容器加载具体的实现类-->
    <bean id="orderService" class="dubbo.service.impl.OrderServiceImpl" />

```

远程服务调用就像调用本地服务一样，对调用者是透明的。

### Dubbo 调用高级功能

#### 并发控制（限流）

配置线程数和连接数，一般推荐配置在服务提供者端，不在客户端配置。

```
<!-- 限制接口OrderService里的每个方法，服务提供者端的执行线程不超过10个 -->
    <dubbo:service interface="com.bubbo.service.OrderService" executes="10" />
<!-- 限制接口OrderService里的queryOrderList方法，服务提供者端的执行线程不超过10个 -->
    <dubbo:service interface="com.bubbo.service.OrderService">
    <dubbo:method name="queryOrderList" executes="10" />
    </dubbo:service>

```

```
<!--限制使用dubbo协议时在服务提供者端启用的连接数不超过1000个-->
    <dubbo:provider protocol="dubbo" accepts="1000"/>

```

消费者和服务者默认通过一条共享的 TCP 长连接通信，连接成功请求线程交由 IO 线程池异步读写数据，数据被反序列化后交由业务线程池处理具体业务，也就是对应的 Impl 实现类的具体方法。

#### 服务隔离

```
<!--当同一个接口有多个实现时，可以通过group来隔离  -->
    <!--服务提供者  -->
    <dubbo:service group="ImplA" interface="com.bubbo.service.OrderService"/>
    <dubbo:service group="ImplB" interface="com.bubbo.service.OrderService"/>

    <!--服务调用者  -->
    <dubbo:reference id="MethodA" group="ImplA" interface="com.bubbo.service.OrderService"/>
    <dubbo:reference id="MethodB" group="ImplB" interface="com.bubbo.service.OrderService"/>

```

```
<!--当一个接口出现升级，新旧实现同时存在时，可以通过版本号来隔离，通常版本号隔离也用于联调阶段，不同版本号的服务无法调用，版本号相同的服务才能调用  -->
    <!--服务提供者  -->
    <dubbo:service interface="com.bubbo.service.OrderService" version="new2.0.0"/>
    <dubbo:service interface="com.bubbo.service.OrderService" version="old1.0.0"/>

    <!--服务调用者  -->
    <dubbo:reference id="NewMethodA" interface="com.bubbo.service.OrderService" version="new2.0.0"/>
    <dubbo:reference id="OldMethodB" interface="com.bubbo.service.OrderService" version="old1.0.0"/>

```

通过版本号，也可以实现消费者和提供者服务端直接连接，因为发起调用默认使用随机调用端负载均衡模式，当有多台提供者的时候，会随机选取，通常联调阶段都会调用指定服务进行联合调试，直连一般用在调试，开发阶段，只需要消费者和提供者 version 相同即可

#### 灰度发布

有三台服务器 A、B、C 要上线，现在三台服务器都是旧版本代码，那首先从 Ngnix 负载均衡列表里移除 A 服务器的配置，切断对 A 的访问，然后在 A 服务器不受新的代码，重新把 A 配置进 Ngnix 负载均衡列表。如果在线使用没有问题，则继续升级 B、C 服务器，否则会滚，恢复旧版本代码，这是针对三端（PC 端，微信端，移动端）跟网关系统的。

如果是针对子系统，譬如用户系统、订单系统等，可以通过分组 group 来实现子系统的灰度发布。服务提供者有两组，One、Two，将新版本代码 group 改为 Two，旧版本 group 还是 One，将新版本的消费者 group 改为 Two，这时请求定位到新的消费者再调用新的提供者，而且旧的消费者还是请求旧的提供者，如果线上没有问题，那就把提供者 group 为 One 的组改为 Two，并部署新代码，旧的消费者也改成 Two 并部署新代码如果有问题，那消费端和提供端都回滚到旧版本。

#### 异步调用

Dubbo 默认情况下是同步调用的，就是调用后立刻返回，但如果消费端调用服务端创建文件并转化成 PDF 格式的文件这种在 IO 密集操作时，消费端同步调用需要等待对方转换结束才返回，很消耗性能，这时选择异步调用和回调调用更合适。

```
<!--
async="true" 异步调用，调用后不用等待，继续往下执行
onreturn ="CallBack.onreturn" 返回后调用自定义的类CallBack类的onreturn方法
onthrow="CallBack.onthrow" 调用后，提供者抛出异常后，返回调用自定义的类CallBack类的onthrow方法
-->    
    <!--服务调用者  -->
    <dubbo:reference id="tranfromPDF" interface="com.bubbo.service.OrderService" >
    <dubbo:method name="tranPDF" async="true" 
    onreturn ="CallBack.onreturn" 
    onthrow="CallBack.onthrow"/>
     </dubbo:reference>

```

可以在 onthrow 事件里实现服务降级的方法，譬如遇到网络抖动，调用超时返回时可在 onthrow 里 return null。

调用方：

```
    @Test
    public void testQueryOrder() {
        // 此时调用会立即拿到null值
        List<Order> list = this.orderService.queryOrderList();
        // 拿到Future的引用，在提供方返回结果后，结果值会被设置进Future
        Future<String> orderFuture = RpcContext.getContext().getFuture();
        try {
            // 该方法是阻塞方法，在拿到值之前一直等待，直到拿到值才会被唤醒，该方法会抛出异常，可以捕获
            String returnValue = orderFuture.get();
        } catch (InterruptedException e) {

            e.printStackTrace();
        } catch (ExecutionException e) {

            e.printStackTrace();
        }

    }

```

回调方：要实现回调，要自定义一个接口以备提供者调用，根据具体场景在对应方法实现需要做的业务逻辑。

```
// 回调接口
interface ICallBack {
    // 第一个参数是返回值，第二个参数是原参数
    public void onreturn(String returnValue, String initParameter);

    // 第一个参数是异常，第二个参数是原参数
    public void onthrow(Throwable ex, String initParameter);
}

// 实现类
class CallBackImpl implements ICallBack {
    public void onreturn(String returnValue, String initParameter) {
        // do something
    };

    public void onthrow(Throwable ex, String initParameter) {
        // do something
    };
}

```

异步调用线程模型：

![enter image description here](http://images.gitbook.cn/eb348ba0-76f3-11e8-a5cf-8d4ee65675fa)

调用方会有一个用户线程池专门处理调用请求，请求转发到 IO 线程池，被分配到的 IO 线程发起对提供方的调用，然后立即设置 null 值金 RpcContext 里的 Future 对象，用户线程拿到 Future，相当于一个令牌，然后继续执行自己的业务逻辑，最后才调用 Future 的 get 方法阻塞等待，而服务端只需将结果返回给 IO 线程，由 IO 线程调用 notify 方法唤醒阻塞等待中的用户线程。

#### 服务降级

平常调用查询订单，可以查看所有订单不受限制，当遇到营销大促时，提 供方判断时间是促销时段，不查询所有订单，转而调用 mock 方法查询当月所有订单，降低服务级别，在高并发下保持服务可用不至于宕机。

```
 <dubbo:service interface="com.bubbo.service.OrderService" mock="com.dubbo.service.MonthOderMock"/>

```

再对订单接口多实现一个子类：

```
public class MonthOderMock implements OrderService {

    // 只查询最近三十天的订单数据
    public List<Order> queryOrderList() {
        List<Order> list = new ArrayList<Order>();

        return list;
    }

}

```

#### 热点缓存

```
    <!--服务调用者 -->
    <dubbo:reference id="queryCatalog" interface="com.bubbo.service.CatalogService">
        <dubbo:method name="queryCatalog" cache="lru" />
    </dubbo:reference>
    <dubbo:monitor protocol="registry" />

```

如果查询的对象改变很少但又数据量很大的时候，如首页目录，可以避免每次都频繁调用服务端，可以设置本地缓存，加快热点数据的访问，Dubbo 的缓存类型 LRU 缓存，最近最少使用的数据会被清除，使用频繁的数据被保留，Thredlocal 缓存，当前线程的缓存，假如当前线程有多次请求，每次请求都需要相同的用户信息，那就适用，避免每次都去查询用户基本信息。

### Dubbo 通信协议

#### Dubbo 协议

> - 连接个数：单连接
> - 连接方式：长连接
> - 传输协议：TCP
> - 传输方式：NIO 异步传输
> - 序列化：Hessian 二进制序列化
> - 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用 dubbo 协议传输大文件或超大字符串
> - 适用场景：常规远程服务方法调用
> - Dubbo 协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接

```
<!-- 表示该服务使用 JVM 共享长连接 -->
<dubbo:service connections="0"> 
<dubbo:reference connections="0">
<!-- 表示该服务使用独立长连接 -->
<dubbo:service connections="1"> 
<dubbo:reference connections="1">

```

**为什么要消费者比提供者个数多？**

因 dubbo 协议采用单一长连接，假设网络为千兆网卡 3，根据测试经验数据每条连接最多只能压满 7MByte(不同的环境可能不一样，供参考)，理论上 1 个服务提供者需要 20 个服务消费者才能压满网卡。

**为什么不能传大包?**

因 dubbo 协议采用单一长连接，如果每次请求的数据包大小为 500KByte，假设网络为千兆网卡 3，每条连接最大 7MByte(不同的环境可能不一样，供参考)，单个服务提供者的 TPS(每秒处理事务数)最大为：128MByte / 500KByte = 262。单个消费者调用单个服务提供者的 TPS(每秒处理事务数)最大为：7MByte / 500KByte = 14。如果能接受，可以考虑使用，否则网络将成为瓶颈。

**为什么采用异步单一长连接？**

因为服务的现状大都是服务提供者少，通常只有几台机器，而服务的消费者多，可能整个网站都在访问该服务，比如 Morgan 的提供者只有 6 台提供者，却有上百台消费者，每天有 1.5 亿次调用，如果采用常规的 hessian 服务，服务提供者很容易就被压跨，通过单一连接，保证单一消费者不会压死提供者，长连接，减少连接握手验证等，并使用异步 IO，复用线程池，防止 C10K 问题（服务器无法服务1w左右的并发连接）。

```
    <!-- 配置协议端口和服务提供方最大连接数，防止服务被压垮 -->
    <dubbo:protocol name="dubbo" port="20880" accepts="1000" />
    <!--配置dubbo默认协议 -->
    <dubbo:provider protocol="dubbo" />
    <!--配置dubbo设置服务协议 -->
    <dubbo:service protocol="dubbo" />

```

#### Hessian 协议

> - 连接个数：多连接
> - 连接方式：短连接
> - 传输协议：HTTP
> - 传输方式：同步传输
> - 序列化：Hessian 二进制序列化
> - 适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。
> - 适用场景：页面传输，文件传输，Hessian 是 Caucho 开源的一个 RPC 框架，其通讯效率高于 WebService 和 Java 自带的序列化，Hessian 底层采用 Http 通讯，采用 Servlet 暴露服务，Dubbo 缺省内嵌 Jetty 作为服务器实现。

```
    <!--定义 hessian 协议 -->
    <dubbo:protocol name="hessian" port="8080" server="jetty" />
    <!--设置默认协议 -->
    <dubbo:service protocol="hessian" />
    <!--设置 service 协议 -->
    <dubbo:service protocol="hessian" />

```

需要添加依赖：

```
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.33</version>
</dependency>

```

#### Http 协议

> - 连接个数：多连接
> - 连接方式：短连接
> - 传输协议：HTTP
> - 传输方式：同步传输
> - 序列化：表单序列化
> - 适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或 URL 传入参数，暂不支持传文件。
> - 适用场景：需同时给应用程序和浏览器 JS 使用的服务。

```
    <!--配置协议 -->
    <dubbo:protocol name="http" port="8080" />

```

#### Thrift 协议：

> 当前 dubbo 支持（2.3.0 以上版本支持）的 thrift 协议是对 thrift 原生协议（ Thrift 是 Facebook 捐给 Apache 的一个 RPC 框架）的扩展，在原生协议的基础上添加了一些额外的头信息，比如 service name，magic number 等。
>
> 使用 dubbo thrift 协议同样需要使用 thrift 的 idl compiler 编译生成相应的 java 代码。

需要添加依赖：

```
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.8.0</version>
</dependency>

```

所有服务共用一个端口（与原生Thrift不兼容 ）：

```
<dubbo:protocol name="thrift" port="3030" />

```

#### Rest 协议

> 基于 RESTful 风格的调用，REST 服务中虽然建议使用 HTTP 协议中四种标准方法 POST、DELETE、PUT、GET 来分别实现常见的“增删改查”，但实际中，我们一般情况直接用 POST 来实现“增改”，GET 来实现“删查”即可（DELETE 和 PUT 甚至会被一些防火墙阻挡）。
>
> 任何客户端都可以将包含用户信息的 JSON 字符串 POST 到以下 URL 来完成下单操作。

```
http://localhost:8080/orders/create

```

先开发服务的接口：

```
public class OrderService {    
   void createOrder(Order order);
}

```

再来实现接口：

```
@Path("orders") // 访问Url的相对路径
public class OrderServiceImpl implements OrderService {

    @POST
    @Path("create") // 访问Url的相对路径
    // 将传递过来的JSON数据反序列化为Order对象
    @Consumes({MediaType.APPLICATION_JSON}) 
    public void createOrder(Order order) {
        // create the order...
    }
}

```

最后在 Spring 配置文件中添加服务：

```
<!-- 用rest协议在8080端口暴露服务 -->
<dubbo:protocol name="rest" port="8080"/>

<!-- 声明需要暴露的服务接口 -->
<dubbo:service interface="com.service.OrderService" ref="orderService"/>

<!-- 和本地bean一样实现服务 -->
<bean id="orderService" class="com.service.OrderServiceImpl" />

```

更多的关于 Dubbo 协议的其他内容可以参考官方的 Dubbo 用户手册。

### Dubbo 集群容错和负载均衡

#### 集群容错

![enter image description here](http://images.gitbook.cn/dab37e80-76fd-11e8-a5cf-8d4ee65675fa)

##### **Failover**

失败自动切换，当出现失败，重试其它服务器 （该配置为默认配置）。通常用于读操作，但重试会带来更长延迟。

```
    <!--配置集群容错模式为失败自动切换 -->
    <dubbo:reference cluster="failover" />
    <!-- 调用queryOrder方法如果失败共调3次，重试2次，如果成功则只调1次 -->
    <dubbo:reference>
        <dubbo:method name="queryOrder" retries="2" />
    </dubbo:reference>

```

通常用于幂等操作，多次调用副作用相同，譬如只读请求，Failover 使用得较多，推荐使用，但重试会带来更长延迟，应用于消费者和提供者的服务调用。

##### **Failfast**

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录和修改数据，Failfast 使用得较多，但如果有机器正在重启，可能会出现调用失败，应用于消费者和提供者的服务调用。

##### **Failsafe**

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作，Failsafe使用得不多，但调用信息会丢失，应用于发送统计信息到监控中心。

##### **Failback**

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作，使用得很少，不可靠，重启会丢失，应用于注册服务到注册中心。

##### **Forking**

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，使用得很少，但需要浪费更多服务资源。

##### **Broadcast**

广播调用所有提供者，逐个调用，任意一台报错则报错 。通常用于通知所有提供者更新缓存或日志等本地资源信息，速度慢，任意一台报错则报错，使用得很少。

#### 负载均衡

##### **Random LoadBalance**

随机调用（默认配置），按权重设置随机概率，在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重，使用较多，推荐使用，但重试时，可能出现瞬间压力不均。

```
    <!-- 服务端方法基本负载均衡设置 -->
<dubbo:service interface="com.service.dubbo.queryOrder">
<dubbo:method name="queryOrder" loadbalance="roundrobin" />
</dubbo:service>

```

##### **RoundRobin LoadBalance**

轮循调用，按公约后的权重设置轮循比率，存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上，极端情况可能产生雪崩。

##### **LeastActive LoadBalance**

最少活跃数调用，相同活跃数的随机，活跃数指调用前后计数差，使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差（与时间有关）会越大，但不支持权重。

##### **ConsistentHash LoadBalance**

一致性 Hash，相同参数的请求总是发到同一提供者，当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。缺省只对第一个参数 Hash，如果要修改，请配置：

```
 <dubbo:parameter key="hash.arguments" value="0,1" />

```

缺省用 160 份虚拟节点，如果要修改，请配置：

```
<dubbo:parameter key="hash.nodes" value="320" />

```

由于是通过哈希算法分摊调用，有可能出现调用不均匀的情况

### Dubbo 源码分析

#### 1. 注册中心启动过程

ZooKeeper 和 Redis 都可以作为注册中心，ZooKeeper 是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，推荐使用。

![enter image description here](http://images.gitbook.cn/99efc790-7799-11e8-be1e-e3edaf406043)

- 服务提供者启动时: 向 /dubbo/com.foo.BarService/providers 目录下写入自己的 URL 地址
- 服务消费者启动时: 订阅 /dubbo/com.foo.BarService/providers 目录下的提供者 URL 地址。并向 /dubbo/com.foo.BarService/consumers 目录下写入自己的 URL 地址
- 监控中心启动时: 订阅 /dubbo/com.foo.BarService 目录下的所有提供者和消费者 URL 地址。

ZooKeeper 启动的时候会把配置信息加载进内存并持久化到数据库，然后启动定时器脏数据检查定时器 DirtyCheckTask，分别检查消费者和提供者的地址列表缓存、消费者和提供者地址列表的数据库数据，清理不存活的消费者和提供者数据，对于缓存中的存在的消费者和提供者而数据库不存在，提供者重新注册和消费者重新订阅。

ZooKeeper 支持的功能：

- 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
- 当注册中心重启时，能自动恢复注册数据，以及订阅请求
- 当会话过期时，能自动恢复注册数据，以及订阅请求
- 当设置 `<dubbo:registry check="false" />` 时，记录失败注册和订阅请求，后台定时重试
- 可通过 `<dubbo:registry username="admin" password="1234" />` 设置 ZooKeeper 登录信息
- 可通过 `<dubbo:registry group="dubbo" />` 设置 ZooKeeper 的根节点，不设置将使用无根树
- 支持 `*` 号通配符`<dubbo:reference group="*" version="*" />` ，可订阅服务的所有分组和所有版本的提供者

要了解更多其他关于注册中心的信息可以查阅 Dubbo 用户手册。

#### 2. 服务提供者服务暴露过程

```
// ServiceBean就是dubbo里dubbo:service interface="...">标签对应的实体Bean
/**
 * ServiceFactoryBean
 * 
 * @author william.liangf
 * @export
 */
public class ServiceBean<T> extends ServiceConfig<T>
        implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener, BeanNameAware {

    public void setApplicationContext(ApplicationContext applicationContext) {
        ......
    }

    // 实现ApplicationListener接口的监听事件，加载容器
    public void onApplicationEvent(ApplicationEvent event) {
  ......
  }

    private boolean isDelay() {
    ......
  }

    ......
    if(!isDelay()) {
      //调用到了暴露的方法export
          export();
      }
  }

    public void destroy() throws Exception {
        unexport();
    }

}

    public synchronized void export() {
        if (provider != null) {
            if (export == null) {
                export = provider.getExport();
            }
            if (delay == null) {
                delay = provider.getDelay();
            }
        }
        if (export != null && !export.booleanValue()) {
            return;
        }
        if (delay != null && delay > 0) {
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    try {
                        Thread.sleep(delay);
                    } catch (Throwable e) {
                    }
                    doExport();
                }
            });
            thread.setDaemon(true);
            thread.setName("DelayExportServiceThread");
            thread.start();
        } else {
            // 如果不是延迟暴露则直接暴露执行doExport
            doExport();
        }
    }

    protected synchronized void doExport() {
      if (unexported) {
          throw new IllegalStateException("Already unexported!");
      }
      if (exported) {
          return;
      }
      exported = true;
      ......
      //检查应用加载是否正常
      checkApplication();
      //检查注册中心加载是否正常
      checkRegistry();
      //检查协议配置
      checkProtocol();
      appendProperties(this);
      //检查Mock方法配置
      checkStubAndMock(interfaceClass);
      if (path == null || path.length() == 0) {
          path = interfaceName;
      }
      //最后调用doExportUrls
      doExportUrls();
  }

    @SuppressWarnings({ "unchecked", "rawtypes" })
    private void doExportUrls() {
        List<URL> registryURLs = loadRegistries(true);
        for (ProtocolConfig protocolConfig : protocols) {
            // 根据每个配置项进行逐个服务暴露
doExportUrlsFor1Protocol(protocolConfig, registryURLs);
        }
    }......

    // 导出服务
    String contextPath = protocolConfig.getContextpath();if((contextPath==null||contextPath.length()==0)&&provider!=null)
    {
        contextPath = provider.getContextpath();
    }

URL url = new URL(name, host, port,(contextPath == null || contextPath.length() == 0 ? "" : contextPath + "/") + path, map);    if(ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class).hasExtension(url.getProtocol()))
{
url=ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class).getExtension(url.getProtocol()).getConfigurator(url).configure(url);}
    String scope = url.getParameter(Constants.SCOPE_KEY);
    // 配置为none不暴露
if(!Constants.SCOPE_NONE.toString().equalsIgnoreCase(scope))
    {
// 配置不是remote的情况下做本地暴露 (配置为remote，则表示只暴露远程服务)
if (!Constants.SCOPE_REMOTE.toString().equalsIgnoreCase(scope)) {exportLocal(url);}
// 如果配置不是local则暴露为远程服务.(配置为local，则表示只暴露远程服务)
if (!Constants.SCOPE_LOCAL.toString().equalsIgnoreCase(scope)) {if (logger.isInfoEnabled()) {

            if (registryURLs != null && registryURLs.size() > 0 && url.getParameter("register", true)) {
                // 生成需要暴露的Url
                for (URL registryURL : registryURLs) {
                    url = url.addParameterIfAbsent("dynamic", registryURL.getParameter("dynamic"));
                    URL monitorUrl = loadMonitor(registryURL);

    Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass,registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));

    Exporter<?> exporter = protocol.export(invoker);
                    exporters.add(exporter);
                }
            } else {
                // 将接口实现类ref，接口interfaceClass，要暴露的url通过字节码插桩生成动态代理类invoker
                Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
                // 根据具体协议将invoker暴露成exporter，用以接收来自消费端的远程服务的调用
                Exporter<?> exporter = protocol.export(invoker);
                exporters.add(exporter);
            }
        }
    }this.urls.add(url);
}

```

#### 3. 服务消费者调用



```
// ReferenceBean对应的是标签<dubbo:reference id="..."
// interface="com.bubbo.service.OrderService"/>的实现类
public class ReferenceBean<T> extends ReferenceConfig<T>
        implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {

    @SuppressWarnings({ "unchecked"})
  public void afterPropertiesSet() throws Exception {
      ......
      if (b != null && b.booleanValue()) {
      //获取代理对象
          getObject();
      }
  }

}

    public Object getObject() throws Exception {
        return get();
    }

    public synchronized T get() {
        if (destroyed) {
            throw new IllegalStateException("Already destroyed!");
        }
        if (ref == null) {
            // 初始化
            init();
        }
        return ref;
    }

    // 接口类型
    private String interfaceName;

    private Class<?> interfaceClass;

    // 客户端类型
    private String client;

    // 点对点直连服务提供地址
    private String url;

    // 方法配置
    private List<MethodConfig> methods;

    // 缺省配置
    private ConsumerConfig consumer;

    private String protocol;

    // 接口代理类引用
    private transient volatile T ref;

private void init() {
        if (initialized) {
            return;
        }
        initialized = true;
      if (interfaceName == null || interfaceName.length() == 0) {
          throw new IllegalStateException("<dubbo:reference interface=\"\" /> interface not allow null!");
      }
      // 获取消费者全局配置
      checkDefault();
      appendProperties(this);
      if (getGeneric() == null && getConsumer() != null) {
          setGeneric(getConsumer().getGeneric());
      }
      if (ProtocolUtils.isGeneric(getGeneric())) {
          interfaceClass = GenericService.class;
      } else {
      ......
      //attributes通过系统context进行存储.
      StaticContext.getSystemContext().putAll(attributes);
      ref = createProxy(map);
  }

@SuppressWarnings({ "unchecked", "rawtypes", "deprecation" })
    private T createProxy(Map<String, String> map) {
        URL tmpUrl = new URL("temp", "localhost", 0, map);
        final boolean isJvmRefer;
      if (isInjvm() == null) {
          if (url != null && url.length() > 0) { 
          //指定URL的情况下，不做本地引用
              isJvmRefer = false;
          } else if (InjvmProtocol.getInjvmProtocol().isInjvmRefer(tmpUrl)) {
              //默认情况下如果本地有服务暴露，则引用本地服务.
              isJvmRefer = true;
          } else {
              isJvmRefer = false;
          }
      } else {
          isJvmRefer = isInjvm().booleanValue();
      }

        if (isJvmRefer) {
            URL url = new URL(Constants.LOCAL_PROTOCOL, NetUtils.LOCALHOST, 0, interfaceClass.getName()).addParameters(map);
            invoker = refprotocol.refer(interfaceClass, url);
          if (logger.isInfoEnabled()) {
              logger.info("Using injvm service " + interfaceClass.getName());
          }
        } else {
// 用户指定URL，指定的URL可能是对点对直连地址，也可能是注册中心URL
              String[] us = Constants.SEMICOLON_SPLIT_PATTERN.split(url);
              if (us != null && us.length > 0) {
                  for (String u : us) {
                      URL url = URL.valueOf(u);
                      if (url.getPath() == null || url.getPath().length() == 0) {
                          url = url.setPath(interfaceName);
                      }
                      if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
                          urls.add(url.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                      } else {
                          urls.add(ClusterUtils.mergeUrl(url, map));
                      }
                  }
              }
          } else { 
          // 通过注册中心配置拼装URL
              List<URL> us = loadRegistries(false);
              if (us != null && us.size() > 0) {
                  for (URL u : us) {
                      URL monitorUrl = loadMonitor(u);
                      if (monitorUrl != null) {
                          map.put(Constants.MONITOR_KEY, URL.encode(monitorUrl.toFullString()));
                      }
                      urls.add(u.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                  }
              }

          }

          if (urls.size() == 1) {
          //生成invoker 实例
              invoker = refprotocol.refer(interfaceClass, urls.get(0));
          } else {
              List<Invoker<?>> invokers = new ArrayList<Invoker<?>>();
              URL registryURL = null;
              for (URL url : urls) {
                  invokers.add(refprotocol.refer(interfaceClass, url));
                  if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
            // 用了最后一个registry url，注册Url
                  }
              }
              if (registryURL != null) { 
            // 有 注册中心协议的URL
            // 对有注册中心的Cluster 只用 AvailableCluster
                  URL u = registryURL.addParameter(Constants.CLUSTER_KEY, AvailableCluster.NAME); 
                  invoker = cluster.join(new StaticDirectory(u, invokers));
              }  else { // 不是 注册中心的URL
                  invoker = cluster.join(new StaticDirectory(invokers));
              }
          }
      }

      Boolean c = check;
      if (c == null && consumer != null) {
          c = consumer.isCheck();
      }
      if (c == null) {
          c = true; // default true
      }

      if (logger.isInfoEnabled()) {

      // 创建服务代理
      return (T) proxyFactory.getProxy(invoker);
  }

public <T> T getProxy(Invoker<T> invoker) throws RpcException {
      Class<?>[] interfaces = null;
      String config = invoker.getUrl().getParameter("interfaces");
      if (config != null && config.length() > 0) {
          String[] types = Constants.COMMA_SPLIT_PATTERN.split(config);
          if (types != null && types.length > 0) {
              interfaces = new Class<?>[types.length + 2];
              interfaces[0] = invoker.getInterface();
              interfaces[1] = EchoService.class;
              for (int i = 0; i < types.length; i ++) {
                  interfaces[i + 1] = ReflectUtils.forName(types[i]);
              }
          }
      }
      if (interfaces == null) {
          interfaces = new Class<?>[] {invoker.getInterface(), EchoService.class};
      }
      //根据invoker实例和接口名称返回供消费端可发起调用服务的对象
      return getProxy(invoker, interfaces);
  }

```

#### 4. SPI 机制

为了便于扩展 Dubbo 框架，提供了 SPI（Service Provider Interface）机制，软件质量的下降来源于不断的修改，良好的设计原则是对修改关闭，对扩展开放，也就是要么不修改，要么直接替换对应的模块，通过增量扩展的机制可以有效的管控代码的迭代。

实际中通过 SPI 扩展接口的步骤：

- 要求服务提供方的 src/main/resources 必须新建文件夹 META-INF/services，文件夹下面有以接口全路径命名的文本文件：![enter image description here](http://images.gitbook.cn/3abcd8b0-77b9-11e8-bdeb-6901fd742f04)
- 文本文件要有新扩展到实现类的全路径名：

```
com.protocol.HessianProtocolImpl
com.protocol.HttpProtocolImpl

```

![enter image description here](http://images.gitbook.cn/71fa14f0-77b9-11e8-ba56-e7b8e862ce40)

- 然后按照上图创建接口和实现类，包名随意。

```
package com.dubbo.rpc;

public interface IProtocol {
    // rpc协议接口定义
    public void rpcProtocol();

}

package com.protocol;

import com.alibaba.dubbo.common.Extension;
import com.dubbo.rpc.IProtocol;
//其中注解Extension里的值就是package com.protocol里的protocol
@Extension("protocol")
public class HessianProtocolImpl implements IProtocol {
    // hessian协议扩展接口实现
    public void rpcProtocol() {
        // do something
    };

}
package com.protocol;

import com.alibaba.dubbo.common.Extension;
import com.dubbo.rpc.IProtocol;
//其中注解Extension里的值就是package com.protocol里的protocol
@Extension("protocol")
public class HttpProtocolImpl implements IProtocol {

    // http协议扩展接口实现
    public void rpcProtocol() {
        // do something

    };

}

```

- 这样就可以扩展 Dubbo 框架新增 RPC 协议了。

### 参考文献

- [1][Apache.Dubbo用户手册[EB/OL\]](https://github.com/apache/incubator-dubbo).
- [2][Apache.Dubbo管理手册[EB/OL\]](https://github.com/apache/incubator-dubbo).