Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化新 Spring 应用的初始搭建以及开发过程。该框架通过约定由于配置的原则，来进行简化配置。Spring Boot致力于在蓬勃发展的快速应用开发领域成为领导者。Spring Boot 目前广泛应用与各大互联网公司，有以下特点：

1. 创建独立的 Spring 应用程序
2. 嵌入的 Tomcat，无需部署 WAR 文件
3. 简化 Maven 配置
4. 自动配置 Spring
5. 提供生产就绪型功能，如指标，健康检查和外部配置
6. 绝对没有代码生成，对 XML 没有要求配置

并且 Spring Boot 可以与Spring Cloud、Docker完美集成，所以我们非常有必要学习 Spring Boot 。并且了解其内部实现原理。通过本次分享，您不仅可以学会如何使用 Spring Boot，还可以学习到其内部实现原理，并深入理解：

1. Spring Boot 项目结构，starter 结构
2. 常用注解分析
3. Spring Boot 启动过程梳理（含：Spring 事件监听与广播；自定义事件； SpringFactoriesLoader 工厂加载机制等）
4. 自定义 starter
5. 自定义 condition

# 1. 项目初始化过程

## springboot启动类

springboot启动类非常简单，就一句话：

```
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

```

要想了解springboot的启动过程，肯定要从这句代码开始了。

跟进去可以看到有两步，一个是初始化，一个是run方法的执行：

```
public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
        return new SpringApplication(sources).run(args);
    }

```

先看初始化方法：

```
private void initialize(Object[] sources) {
//这里的sources其实就只有一个，我们的启动类DemoApplication.class。
        if (sources != null && sources.length > 0) {
            this.sources.addAll(Arrays.asList(sources));
        }
        //是否是web应用程序。通过判断应用程序中是否可以加载（class.forname）【"javax.servlet.Servlet","org.springframework.web.context.ConfigurableWebApplicationContext"】这两个类。
        this.webEnvironment = deduceWebEnvironment();
        //设置初始化类：从配置文件spring.factories中查找所有的key=org.springframework.context.ApplicationContextInitializer的类【加载，初始化，排序】
        //SpringFactoriesLoader:工厂加载机制
        setInitializers((Collection) getSpringFactoriesInstances(
                ApplicationContextInitializer.class));
                //设置Listeners：从配置文件spring.factories中查找所有的key=org.springframework.context.ApplicationListener的类.【加载，初始化，排序】
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        //从当前调用栈中，查找主类
        this.mainApplicationClass = deduceMainApplicationClass();
    }

```

## SpringFactoriesLoader工厂加载机制

上面代码中 ，Initializers和Listeners的加载过程都是使用到了`SpringFactoriesLoader`工厂加载机制。我们进入到`getSpringFactoriesInstances`这个方法中：

```
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,
            Class<?>[] parameterTypes, Object... args) {
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
//获取META-INF/spring.factories文件中key为type类型的所有的类全限定名。注意是所有jar包内的。
        Set<String> names = new LinkedHashSet<String>(
                SpringFactoriesLoader.loadFactoryNames(type, classLoader));
//通过上面获取到的类的全限定名，这里将会使用Class.forName加载类，并调用构造方法实例化类
        List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
                classLoader, args, names);
//根据类上的org.springframework.core.annotation.Order注解，排序
        AnnotationAwareOrderComparator.sort(instances);
        return instances;
    }

```

配置文件中的内容：

![enter image description here](http://images.gitbook.cn/62e00410-ed59-11e7-8069-d98afee5b493)

了解过dubbo的同学会觉得这个非常熟悉，是的，没错，和dubbo中的扩展点有异曲同工之妙。

`SpringFactoriesLoader.loadFactoryNames(type, classLoader));`方法的执行内容入下，下图中展示了我们自定义的一个配置类的加载过程（后面自定义starter的实现一节中会讲）：

![enter image description here](http://images.gitbook.cn/6feacaa0-ed59-11e7-8056-471c3e83e0a6)

ApplicationContextInitializer的类图：

![enter image description here](http://images.gitbook.cn/784e2930-ed59-11e7-bdf6-ad804cfda5cb)

下面是listener的类图（太多了，不全，只列出了部分）

![enter image description here](http://images.gitbook.cn/80bcf970-ed59-11e7-bdf6-ad804cfda5cb)

## 总结初始化initialize过程

1. 判断是否是web应用程序
2. 从所有类中查找META-INF/spring.factories文件，加载其中的初始化类和监听类。
3. 查找运行的主类 默认初始化Initializers都继承自ApplicationContextInitializer。

![enter image description here](http://images.gitbook.cn/a6b547e0-ed59-11e7-8056-471c3e83e0a6)

默认Listeners有：

![enter image description here](http://images.gitbook.cn/b04fedf0-ed59-11e7-8056-471c3e83e0a6)

## run方法:

```
public ConfigurableApplicationContext run(String... args) {
//stopWatch 用于简单监听run启动过程
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        FailureAnalyzers analyzers = null;
        configureHeadlessProperty();
        //获取监听器。springboot中只有一个SpringApplicationRunListener监听器【参考spring的事件与广播】
        SpringApplicationRunListeners listeners = getRunListeners(args);
        listeners.starting();
        try {
        //下面两句是加载属性配置，执行完成后，所有的environment的属性都会加载进来，包括application.properties和外部的属性配置。
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(
                    args);
            ConfigurableEnvironment environment = prepareEnvironment(listeners,
                    applicationArguments);
            Banner printedBanner = printBanner(environment);
            //创建spring容器【Class.forName加载ing实例化类："org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext"或"org.springframework.context.annotation.AnnotationConfigApplicationContext。前者是web应用程序时使用】
            context = createApplicationContext();
            //错误分析器
            analyzers = new FailureAnalyzers(context);
            //主要是调用所有初始化类的initialize方法
            prepareContext(context, environment, listeners, applicationArguments,
                    printedBanner);
            //初始化spring容器【后面写到】
            refreshContext(context);
            //主要是执行ApplicationRunner和CommandLineRunner的实现类【后面会写到】
            afterRefresh(context, applicationArguments);
            //通知监听器
            listeners.finished(context, null);
            stopWatch.stop();
            if (this.logStartupInfo) {
                new StartupInfoLogger(this.mainApplicationClass)
                        .logStarted(getApplicationLog(), stopWatch);
            }
            return context;
        }
        catch (Throwable ex) {
            handleRunFailure(context, listeners, analyzers, ex);
            throw new IllegalStateException(ex);
        }
    }

private void prepareContext(ConfigurableApplicationContext context,
            ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
            ApplicationArguments applicationArguments, Banner printedBanner) {
        context.setEnvironment(environment);//设置环境变量
        postProcessApplicationContext(context);
        //SpringApplication的的初始化器开始工作。主要是循环遍历调用所有Initializers的initialize方法
        applyInitializers(context);
        //目前是空实现
        listeners.contextPrepared(context);
        if (this.logStartupInfo) {
            logStartupInfo(context.getParent() == null);
            logStartupProfileInfo(context);
        }

        // 注册单例：applicationArguments
        context.getBeanFactory().registerSingleton("springApplicationArguments",
                applicationArguments);
        if (printedBanner != null) {
            context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
        }

        // Load the sources: 只有一个主类：DemoApplication
        Set<Object> sources = getSources();
        Assert.notEmpty(sources, "Sources must not be empty");
        //加载bean，目前只有一个主类：DemoApplication
        load(context, sources.toArray(new Object[sources.size()]));
        //广播容器加载完成事件
        listeners.contextLoaded(context);
    }

```

## spring事件

### 自定义spring事件

spring事件为Bean与bean之间的通讯提供了支持。我们可以发送某个事件，然后通过监听器来处理该事件。

> spring事件继承自`ApplicationEvent`, 监听器实现了接口`ApplicationListener<E extends ApplicationEvent>`

下面我们自定义一个事件和监听器

```
import org.springframework.context.ApplicationEvent;

/**
 * Created by hzliubenlong on 2017/12/17.
 */
@Data
public class MyEvent extends ApplicationEvent{

    private String msg;

    /**
     * Create a new ApplicationEvent.
     *
     * @param source the object on which the event initially occurred (never {@code null})
     */
    public MyEvent(Object source, String msg) {
        super(source);
        this.msg = msg;
    }
}

```

```
package com.example.demo.event;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

/**
 * Created by hzliubenlong on 2017/12/17.
 */
@Slf4j
@Component
public class MyListener implements ApplicationListener<MyEvent>{

    @Override
    public void onApplicationEvent(MyEvent event) {
        log.info("MyListener收到了MyEvent的消息：{}", event.getMsg());
    }
}

```

编写一个controller方法：

```
    @Autowired
    ApplicationContext applicationContext;


    @RequestMapping("testPublishMsg")
    public void testPublishMsg(String msg) {
        applicationContext.publishEvent(new MyEvent(this, msg));
    }

```

启动服务，访问`testPublishMsg`接口，就可以在控制台看到日志打印：`MyListener收到了MyEvent的消息：。。。。。`

## springboot启动过程中的事件广播

SpringApplicationRunListener类图:

![enter image description here](http://images.gitbook.cn/f9256820-ed59-11e7-8056-471c3e83e0a6)

上述run过程广泛应用了spring事件机制（主要是广播）。上述代码中首先获取SpringApplicationRunListeners。这就是在`spring.factories`文件中配置的所有监听器。然后整个run 过程使用了listeners的5个方法，每个方法对应一个事件Event：

- starting() run方法执行的时候立马执行；对应事件的类型是`ApplicationStartedEvent`
- environmentPrepared() `ApplicationContext`创建之前并且环境信息准备好的时候调用；对应事件的类型是`ApplicationEnvironmentPreparedEvent`
- contextPrepared() `ApplicationContext`创建好并且在source加载之前调用一次；没有具体的对应事件
- contextLoaded() `ApplicationContext`创建并加载之后并在refresh之前调用；对应事件的类型是`ApplicationPreparedEvent`
- finished() run方法结束之前调用；对应事件的类型是`ApplicationReadyEvent`或`ApplicationFailedEven`

`SpringApplicationRunListeners`是`SpringApplicationRunListener`的集合，`SpringApplicationRunListener`只有一个实现类：`EventPublishingRunListener`，在这个实现类中，有一个`SimpleApplicationEventMulticaster`类型的属性`initialMulticaster`，所有的事件都是通过这个属性的`multicastEvent`方法广播出去的。以`environmentPrepared()`方法为例，展示一下环境变了的加载过程：

```
@Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(
                this.application, this.args, environment));
    }

```

上述广播了一个`ApplicationEnvironmentPreparedEvent`事件。我们知道所有的事件都会被监听器捕获处理，spring的监听器都是`ApplicationListener`的子类。根据事件的类型，找到处理这个时间的监听器类`ConfigFileApplicationListener`:

```
@Override
    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationEnvironmentPreparedEvent) {
            onApplicationEnvironmentPreparedEvent(
                    (ApplicationEnvironmentPreparedEvent) event);
        }
        if (event instanceof ApplicationPreparedEvent) {
            onApplicationPreparedEvent(event);
        }
    }

    private void onApplicationEnvironmentPreparedEvent(
            ApplicationEnvironmentPreparedEvent event) {
        List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
        postProcessors.add(this);
        AnnotationAwareOrderComparator.sort(postProcessors);
        for (EnvironmentPostProcessor postProcessor : postProcessors) {
            postProcessor.postProcessEnvironment(event.getEnvironment(),
                    event.getSpringApplication());
        }
    }
    //省略部分代码
@Override
    public void postProcessEnvironment(ConfigurableEnvironment environment,
            SpringApplication application) {
        addPropertySources(environment, application.getResourceLoader());
        configureIgnoreBeanInfo(environment);
        bindToSpringApplication(environment, application);
    }
    //省略部分代码
/**
     * Add config file property sources to the specified environment.
     * @param environment the environment to add source to
     * @param resourceLoader the resource loader
     * @see #addPostProcessors(ConfigurableApplicationContext)
     */
    protected void addPropertySources(ConfigurableEnvironment environment,
            ResourceLoader resourceLoader) {
        RandomValuePropertySource.addToEnvironment(environment);
//加载属性配置的最关键的一行代码
        new Loader(environment, resourceLoader).load();
    }

```

`ConfigFileApplicationListener`类中可以看到一些其他有用的信息：[DEFAULT*SEARCH*LOCATIONS = “classpath:/,classpath:/config/,file:./,file:./config/”][ACTIVE*PROFILES*PROPERTY = “spring.profiles.active”]等等。

OK，spring的事件机制以及springboot启动过程中的事件广播机制讲完了。

## FailureAnalyzers错误分析器

> 查找并加载spring.factories中所有的org.springframework.boot.diagnostics.FailureAnalyzer。

FailureAnalyzers的类图：

![enter image description here](http://images.gitbook.cn/6482e930-ed5a-11e7-8069-d98afee5b493)

当启动过程中发生错误，会广播事件并且执行分析器。 目前主要有analyze和report两个方法，用途是打印日志。

比如：当我们的启动类`DemoApplication`不是放到最外层的时候，报错,这就是`FailureAnalyzers`中打印出的结果：

```
Error starting ApplicationContext. To display the auto-configuration report re-run your application with 'debug' enabled.
2017-12-17 17:04:23.107 ERROR 9356 --- [           main] o.s.b.d.LoggingFailureAnalysisReporter   : 

***************************
APPLICATION FAILED TO START
***************************

Description:

Field environmentService in com.example.demo.environment.impl.DemoApplication required a bean of type 'com.example.demo.environment.EnvironmentService' that could not be found.


Action:

Consider defining a bean of type 'com.example.demo.environment.EnvironmentService' in your configuration.


Process finished with exit code 1

```

## refresh过程

比较复杂，主要是spring的加载过程，本文的目标是springboot而不是spring，所以不再深入，我会专门写篇文件介绍。现在只需要了解：

- spring启动过程
- bean的加载初始化都是在该方法中完成

## afterRefresh与CommandLineRunner、ApplicationRunner

在springboot启动过程中的run方法的最后，有一句·afterRefresh(context, applicationArguments);·。主要是执行·ApplicationRunner·和·CommandLineRunner·两个接口的实现类，这两个类都是在springboot启动完成后执行的一点代码，类似于普通bean中的init方法，开机自启动。 两者唯一不同是获取参数方式不同，后面的小例子会讲。callRunners的源码：

```
        List<Object> runners = new ArrayList<Object>();
        runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
        runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
        AnnotationAwareOrderComparator.sort(runners);
        for (Object runner : new LinkedHashSet<Object>(runners)) {
            if (runner instanceof ApplicationRunner) {
                callRunner((ApplicationRunner) runner, args);
            }
            if (runner instanceof CommandLineRunner) {
                callRunner((CommandLineRunner) runner, args);
            }
        }
    }

```

如果有类似的需求：springboot启动完成后执行一点代码逻辑，则可以通过实现上述两个类来完成。下面写个小例子： 定义4个类：

```
/**
 * Created by hzliubenlong on 2017/12/17.
 */
@Component
@Order(7)
@Slf4j
public class MyApplicationRunner1 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        StringBuffer stringBuffer = new StringBuffer();
        args.getOptionNames().forEach(s -> stringBuffer.append(s).append("=").append(args.getOptionValues(s)).append(","));
        log.info("MyApplicationRunner1    "+stringBuffer.toString());
    }
}

/**
 * Created by hzliubenlong on 2017/12/17.
 */
@Component
@Order(9)
@Slf4j
public class MyApplicationRunner2 implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        StringBuffer stringBuffer = new StringBuffer();
        args.getOptionNames().forEach(s -> stringBuffer.append(s).append("=").append(args.getOptionValues(s)).append(","));
        log.info("MyApplicationRunner2    "+stringBuffer.toString());
    }
}

/**
 * Created by hzliubenlong on 2017/12/17.
 */
@Component
@Order(4)
@Slf4j
public class MyCommandLineRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        log.info("MyCommandLineRunner1    " + Arrays.asList(args));
    }
}

/**
 * Created by hzliubenlong on 2017/12/17.
 */
@Component
@Order(2)//数字越小，优先级越高
@Slf4j
public class MyCommandLineRunner2 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        log.info("MyCommandLineRunner2    " + Arrays.asList(args));
    }
}

```

执行结果：

```
2017-12-17 15:40:13.387  INFO 6244 --- [           main] c.e.demo.runners.MyCommandLineRunner2    : MyCommandLineRunner2    [--a=1, --b=2]
2017-12-17 15:40:13.387  INFO 6244 --- [           main] c.e.demo.runners.MyCommandLineRunner1    : MyCommandLineRunner1    [--a=1, --b=2]
2017-12-17 15:40:13.388  INFO 6244 --- [           main] c.e.demo.runners.MyApplicationRunner1    : MyApplicationRunner1    a=[1],b=[2],
2017-12-17 15:40:13.388  INFO 6244 --- [           main] c.e.demo.runners.MyApplicationRunner2    : MyApplicationRunner2    a=[1],b=[2],

```

结果分析：

- `CommandLineRunner`与`ApplicationRunner`的实现类是在springboot初始化完成后执行的
- 可以设置执行的顺序，数字越小，优先级越高
- 两者都可以从外界获取参数。唯一不同是：`CommandLineRunner`的参数类型是字符串数组。而`ApplicationRunner`的参数类型是`ApplicationArguments`。它可以解析--a=1 --b=2类型的参数为key-value形式。

## @order

上述过程中，我们看到了好几个地方使用到了order，比如`CommandLineRunner`与`ApplicationRunner`，初始化器Initializer，监听器Listener。

order比较简单，主要是为了控制这些实现类的执行顺序。规则是数值越小，优先级越高。

在通过spring工厂模式，从`spring.factories`中加载这些实现之后，会通过`AnnotationAwareOrderComparator.sort(instances);`方法来进行排序。

```
Collections.sort(list, INSTANCE);

//上述第二个参数是比较器，是AnnotationAwareOrderComparator的实例，从Order注解中获取对象的排序数值。
protected Integer findOrder(Object obj) {
        // Check for regular Ordered interface
        Integer order = super.findOrder(obj);
        if (order != null) {
            return order;
        }

        // Check for @Order and @Priority on various kinds of elements
        if (obj instanceof Class) {
            return OrderUtils.getOrder((Class<?>) obj);
        }
        else if (obj instanceof Method) {
            Order ann = AnnotationUtils.findAnnotation((Method) obj, Order.class);
            if (ann != null) {
                return ann.value();
            }
        }
        else if (obj instanceof AnnotatedElement) {
            Order ann = AnnotationUtils.getAnnotation((AnnotatedElement) obj, Order.class);
            if (ann != null) {
                return ann.value();
            }
        }
        else if (obj != null) {
        //真正获取order的代码
            order = OrderUtils.getOrder(obj.getClass());
            if (order == null && obj instanceof DecoratingProxy) {
                order = OrderUtils.getOrder(((DecoratingProxy) obj).getDecoratedClass());
            }
        }

        return order;
    }

//跳转到getOrder方法中，其实现是获取注解Order的值
public static Integer getOrder(Class<?> type, Integer defaultOrder) {
        Order order = AnnotationUtils.findAnnotation(type, Order.class);
        if (order != null) {
            return order.value();
        }
        Integer priorityOrder = getPriority(type);
        if (priorityOrder != null) {
            return priorityOrder;
        }
        return defaultOrder;
    }

```

## 总结启动run过程

1. 注册一个StopWatch，用于监控启动过程
2. 获取监听器SpringApplicationRunListener，用于springboot启动过程中的事件广播
3. 设置环境变量environment
4. 创建spring容器
5. 创建FailureAnalyzers错误分析器，用于处理记录启动过程中的错误信息
6. 调用所有初始化类的initialize方法
7. 初始化spring容器
8. 执行ApplicationRunner和CommandLineRunner的实现类
9. 启动完成

# springboot中重要的注解

前面讲到了@order等注解，下面看一下springboot中最重要的、最常用的几个注解。

## @SpringBootApplication

他是一个组合注解:

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    @AliasFor(annotation = EnableAutoConfiguration.class, attribute = "exclude")
    Class<?>[] exclude() default {};
    @AliasFor(annotation = EnableAutoConfiguration.class, attribute = "excludeName")
    String[] excludeName() default {};
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
    String[] scanBasePackages() default {};
    @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
    Class<?>[] scanBasePackageClasses() default {};

}

```

## @Inherited

Inherited作用是，使用此注解声明出来的自定义注解，在使用此自定义注解时，如果注解在类上面时，子类会自动继承此注解，否则的话，子类不会继承此注解。这里一定要记住，使用Inherited声明出来的注解，只有在类上使用时才会有效，对方法，属性等其他无效。

参考[关于java 注解中元注解Inherited的使用详解](http://blog.csdn.net/snow_crazy/article/details/39381695)

## @SpringBootConfiguration

源码：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}

```

`@SpringBootConfiguration`继承自`@Configuration`，二者功能也一致，标注当前类是配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到srping容器中，并且实例名就是方法名。

## @EnableAutoConfiguration

```
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};

}

```

`@EnableAutoConfiguration`用于启动自动的配置，是springboot的核心注解。上面import了`EnableAutoConfigurationImportSelector`,这个类继承自`AutoConfigurationImportSelector.AutoConfigurationImportSelector`是关键类。

`EnableAutoConfigurationImportSelector`类使用了Spring Core包的`SpringFactoriesLoade`r类的`loadFactoryNamesof()`方法。 `SpringFactoriesLoader`会查询`META-INF/spring.factories`文件中包含的JAR文件。

当找到`spring.factories`文件后，`SpringFactoriesLoader`将查询配置文件命名的属性`org.springframework.boot.autoconfigure.EnableAutoConfiguration`的值。

![enter image description here](http://images.gitbook.cn/04a723e0-efae-11e7-91fc-5fa70a4dd375)

然后在spring启动过程中的refresh方法中进行真正的bean加载。

## @ComponentScan

`@ComponentScan`，扫描当前包及其子包下被`@Component`，`@Controller`，`@Service`，`@Repository`注解标记的类并纳入到spring容器中进行管理。这也是为什么我们的启动类`DemoApplication`要放到项目的最外层的原因。

# springboot自动化配置原理及自定义starter

前面的文章已经讲了springboot的实现原理，无非就是通过spring的condition条件实现的，还是比较简单的（感谢spring设计的开放性与扩展性）。

在实际工作过程中会遇到需要自定义starter的需求，那么我们接下来就自己实现一个starter。

先看一下目录结构：

![enter image description here](http://images.gitbook.cn/241874e0-efae-11e7-ac24-853217cb2095)

- MyConfig是自定义的配置类
- HelloService是自定义的bean
- HelloServiceProperties是自定义的类型安全的属性配置
- MEYA-INF/spring.factories文件是springboot的工厂配置文件

本项目就是自定义的starter。假设我们这里需要一些配置项，使用者在使用该starter时，需要在`application.properties`文件中配置相关属性。这里我们使用了`@ConfigurationProperties`来将属性配置到一个POJO类中。这样做的好处是：可以检测数据的类型，并且可以对数据值进行校验，详情请参考我的另一篇博客：

`HelloServiceProperties`类内容如下：

```
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * Created by hzliubenlong on 2017/12/13.
 */
@Data
@Component
@ConfigurationProperties(prefix = "hello")
public class HelloServiceProperties {
    private String a;
    private String b;
}

```

这样使用这个starter的时候，就可以配置`hello.a=**`来设置属性了。

`HelloService`是我们的bean，这里实现比较简单，获取外部的属性配置，打印一下日志。内容如下：

```
/**
 * Created by hzliubenlong on 2017/12/12.
 */
@Slf4j
public class HelloService {

    String initVal;

    public void setInitVal(String initVal) {
        this.initVal = initVal;
    }

    public String getInitVal() {
        return initVal;
    }

    public String hello(String name) {
        log.info("initVal={}, name={}", initVal, name);
        return "hello " + name;
    }

}

```

接下来是比较重要的配置类MyConfig,前面已经讲过，springboot是通过条件注解来判断是否要加载bean的，这些内容都是在我们自定义的配置类中来实现：

```
import com.example.myservice.HelloService;
import com.example.properties.HelloServiceProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(HelloServiceProperties.class)
public class MyConfig {

    @Autowired
    private HelloServiceProperties helloServiceProperties;

    @ConditionalOnProperty(prefix="hello", value="enabled", matchIfMissing = true)
    @ConditionalOnClass(HelloService.class)
    @Bean
    public HelloService initHelloService() {
        HelloService helloService = new HelloService();
        helloService.setInitVal(helloServiceProperties.getA());
        return helloService;
    }
}

```

`@Configuration`表明这是一个配置类`@EnableConfigurationProperties(HelloServiceProperties.class)`表示该类使用了属性配置类`HelloServiceProperties` `initHelloService()`方法就是实际加载初始化helloServicebean的方法了，它上面有三个注解：

1. `@ConditionalOnProperty(prefix="hello", value="enabled", matchIfMissing = true): hello.enabled=true`时才加载这个bean，配置没有的话，默认为true，也就是只要引入了这个依赖，不做任何配置，这个bean默认会加载。
2. `@ConditionalOnClass(HelloService.class)`:当HelloService这个类存在时才加载bean。
3. `@Bean`:表明这是一个产生bean的方法，改方法生成一个HelloService的bean，交给spring容器管理。

好了，到这里，我们的代码已经写完。根据前面讲的springboot的原理我们知道，springboot是通过扫描`MEYA-INF/spring.factories`这个工厂配置文件来加载配置类的， 所以我们还需要创建这个文件。其内容如下：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.demo.MyConfig

```

上面的`\`是换行符，只是为了便于代码的阅读。通过这个文件，springboot就可以读取到我们自定义的配置类MyConfig。 接下来我们只需要打个jar包即可供另外一个项目使用了。下面贴一下pom.xml的内容：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>springboot-starter-hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>springboot-starter-hello</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-release-plugin</artifactId>
                <version>2.4.2</version>
                <dependencies>
                    <dependency>
                        <groupId>org.apache.maven.scm</groupId>
                        <artifactId>maven-scm-provider-gitexe</artifactId>
                        <version>1.8.1</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <skip>false</skip>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.7</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

上面定义了一个starter，下面我们将写一个新的工程，来引用我们自定义的starter。还是先看一下项目目录：

![enter image description here](http://images.gitbook.cn/7814da70-efae-11e7-baca-bbeda3a76da6)

要想引用我们自定义的starter，自然是先引入依赖了：

```
<dependency>
            <groupId>com.example</groupId>
            <artifactId>springboot-starter-hello</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

```

然后再application.properties文件中添加如下配置：

```
hello.enabled=true
hello.a=hahaha

```

好了，现在就可以在项目中注入HelloService使用了。是的，就是那么简单.DemoApplication主类如下：

```
import com.example.demo.environment.EnvironmentService;
import com.example.myservice.HelloService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
@SpringBootApplication
public class DemoApplication {

    @Autowired
    HelloService helloService;

    @RequestMapping("/")
    public String word(String name) {
        return helloService.hello(name);
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

```

启动项目，访问URLhttp://127.0.0.1:8080/?name=hhh:

![enter image description here](http://images.gitbook.cn/91d346e0-efae-11e7-baca-bbeda3a76da6)

后台打印日志为`initVal=hahaha, name=hhh`其中`hahaha`就是我们没在配置文件中配置的属性值，这句日志是我们上面starter项目中`com.example.myservice.HelloService#hello`方法打印出来的。

# 自定义Conditional注解

前面已经讲过condition的原理。其实自定义一个condition很简单，只需要实现`SpringBootCondition`类即可，并重写`com.example.demo.condition.OnLblCondition#getMatchOutcome`方法。

下面我们简单写个实例：**根据属性配置文件中的内容，来判断是否加载bean。**

首先定义一个注解，有两个内容：一个是属性文件的KEY，一个是value。我们要实现的是，属性文件的value与指定的value一致，才创建bean。

```
package com.example.demo.condition;

import org.springframework.context.annotation.Conditional;

import java.lang.annotation.*;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnLblCondition.class)
public @interface ConditionalOnLbl {
    String key();
    String value();
}

```

自定义SpringBootCondition的实现类OnLblCondition：

```
package com.example.demo.condition;

import org.springframework.boot.autoconfigure.condition.ConditionOutcome;
import org.springframework.boot.autoconfigure.condition.SpringBootCondition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

import java.util.Map;

public class OnLblCondition extends SpringBootCondition {

    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Map<String, Object> annotationAttributes = metadata.getAnnotationAttributes(ConditionalOnLbl.class.getName());
        Object key = annotationAttributes.get("key");//
        Object value = annotationAttributes.get("value");
        if(key == null || value == null){
            return new ConditionOutcome(false, "error");
        }

        //获取environment中的值
        String key1 = context.getEnvironment().getProperty(key.toString());
        if (value.equals(key1)) {//如果environment中的值与指定的value一致，则返回true
            return new ConditionOutcome(true, "ok");
        }
        return new ConditionOutcome(false, "error");
    }
}

```

然后随便定义一个类：

```
@Slf4j
public class MyConditionService {
    public void say() {
        log.info("MyConditionService init。");
    }
}

```

好了，condition已经自定义完成。接下来就是如何使用了：

```
@Configuration
@Slf4j
public class MySpringBootConfig {
    @ConditionalOnLbl(key = "com.lbl.mycondition", value = "lbl")
    @ConditionalOnClass(MyConditionService.class)
    @Bean
    public MyConditionService initMyConditionService() {
        log.info("MyConditionService已加载。");
        return new MyConditionService();
    }

```

要想加载`MyConditionService`的bean到spring容器中，需要满足以下两个条件：

1. 属性文件中配置了`com.lbl.mycondition=lbl`
2. 可以加载`MyConditionService`类

好了，让我们在`application.properties`文件中添加配置`com.lbl.mycondition=lbl` 启动项目，可以看到日志: `MyConditionService`已加载。 把`application.properties`文件中的`com.lbl.mycondition`去掉，或者更改个值，则上述日志不会打印，也就是不会创建`MyConditionService`这个bean .

# spring @Conditional注解

前面讲了springboot的实现基础是spring的@Conditional注解。介绍原理前我们来看看怎么用。后面介绍其原理。

![enter image description here](http://images.gitbook.cn/e05b3070-efae-11e7-baca-bbeda3a76da6)

我们实现这么一个小功能：**根据不同的环境，实例化不同的bean。 ** springboot通常都是通过-Dspring.profiles.active=dev来区分环境的，如果我们想实现线上的代码逻辑与开发或者测试环境不同，那么这是一个解决方案。

使用java的多态，先定义一个接口：

```
public interface EnvironmentService {

    void printEnvironment();
}

```

然后定义两个不同环境的实现类，内容比较简单，只是输出一句log。

```
@Slf4j
public class ProdService  implements EnvironmentService {

    @Override
    public void printEnvironment() {
        log.info("我是生产环境。");
    }
}

```

```
@Slf4j
public class DevService implements EnvironmentService{

    @Override
    public void printEnvironment() {
        log.info("我是开发环境。");
    }
}

```

好了，，准备工作已经做完，接下来编写配置类，使用spring的@Conditional注解来根据不同的环境配置加载不同的类：

```
import com.example.demo.environment.impl.DevService;
import com.example.demo.environment.impl.ProdService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnExpression;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


/**
 * 根据不同的环境，初始化不同的bean
 */
@Configuration
@Slf4j
public class MySpringBootConfig {

    @ConditionalOnExpression("#{'${spring.profiles.active}'.equals('dev')}")//使用了spring的SPEL表达式
    @ConditionalOnClass(DevService.class)
    @Bean
    public DevService initDevService() {
        log.info("DevService已加载。");
        return new DevService();
    }

    @ConditionalOnExpression("#{'${spring.profiles.active}'.equals('prod')}")
    @ConditionalOnClass(ProdService.class)
    @Bean
    public ProdService initProdService() {
        log.info("ProdService已加载。");
        return new ProdService();
    }
}

```

官方的![enter image description here](http://images.gitbook.cn/08f41a60-efaf-11e7-ac24-853217cb2095)condition有很多： 这里我们使用的是@ConditionalOnExpression。它根据SPEL表达式返回的结果作为条件判断。 这里判断条件为：spring.profiles.active=dev时，创建DevService。为prod时，创建ProdService。

接下来看如何使用：

```
import com.example.demo.environment.EnvironmentService;
import com.example.myservice.HelloService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
@SpringBootApplication
public class DemoApplication {

    @Autowired
    HelloService helloService;
    @Autowired
    EnvironmentService environmentService;

    @Value("#{'${spring.profiles.active}'.equals('dev')}")
    String springProfilesActive;

    @RequestMapping("/")
    public String word(String name) {
        environmentService.printEnvironment();
        log.info("springProfilesActive={}", springProfilesActive);
        return helloService.hello(name);
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

```

很简单，只需要注入`EnvironmentService`即可。注意IDEA可能会提示错误，因为这个接口有两个实现类。不过不用去管他，因为我们通过condition只实例化了一个bean。![enter image description here](http://images.gitbook.cn/1f627170-efaf-11e7-baca-bbeda3a76da6)

运行结果：

```
我是开发环境。
springProfilesActive=true
initVal=hahaha, name=hhh

```

将配置修改为prod后：

```
我是生产环境。
springProfilesActive=true
initVal=hahaha, name=hhh

```

使用非常简单，那么`@Conditional`是怎么实现的呢？

官方文档的说明是“**只有当所有指定的条件都满足是，组件才可以注册**”。主要的用处是在创建bean时增加一系列限制条件。 他的核心类是：

```
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}

```

所有的condition都是Condition接口的实现类，条件判断是通过matches方法返回的布尔值来判断的。

以`@ConditionalOnExpression`注解为例：

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnExpressionCondition.class)
public @interface ConditionalOnExpression {

    /**
     * The SpEL expression to evaluate. Expression should return {@code true} if the
     * condition passes or {@code false} if it fails.
     * @return the SpEL expression
     */
    String value() default "true";
}

```

`ConditionalOnExpression`注解上添加了另一个注解Conditional,指明是哪个condition类处理改注解。 我们打开`OnExpressionCondition`这个类：

```
@Override
    public ConditionOutcome getMatchOutcome(ConditionContext context,
            AnnotatedTypeMetadata metadata) {
            //获取到我们配置的EL表达式的值
        String expression = (String) metadata
                .getAnnotationAttributes(ConditionalOnExpression.class.getName())
                .get("value");
        expression = wrapIfNecessary(expression);
        String rawExpression = expression;
        expression = context.getEnvironment().resolvePlaceholders(expression);
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        BeanExpressionResolver resolver = (beanFactory != null)
                ? beanFactory.getBeanExpressionResolver() : null;
        BeanExpressionContext expressionContext = (beanFactory != null)
                ? new BeanExpressionContext(beanFactory, null) : null;
        if (resolver == null) {
            resolver = new StandardBeanExpressionResolver();
        }
        boolean result = (Boolean) resolver.evaluate(expression, expressionContext);
        return new ConditionOutcome(result, ConditionMessage
                .forCondition(ConditionalOnExpression.class, "(" + rawExpression + ")")
                .resultedIn(result));
    }

```

有这么一个方法，获取到我们设置的EL表达式的值后，进行一些处理，比如渠道`environment`中的值，然后计算整体的结果。 其中关键的是`boolean result = (Boolean) resolver.evaluate(expression, expressionContext);` 这么一行代码，通过springEL计算最终结果。

至此，spring的condition算是介绍完了，我们可以通过实现`org.springframework.context.annotation.Condition#matches`来**自定义condition。**

当然抛开@Condition注解，实现不同环境加载不同的类，既可以使用`ConditionalOnExpression`也可以使用`@Profile`注解

> 两种都可以实现不同环境加载不同的类，写法不同，但是他们的实现原理都是一样的，都是获取环境变量`spring.profiles.active`的值，与value进行比较。读者可以去看一下`ProfileCondition`源码

```
@Bean
    @Profile("dev")
    public DevService initDevService1() {
        log.info("DevService已加载---Profile");
        return new DevService();
    }

    @Bean
    @Profile("prod")
    public ProdService initProdService1() {
        log.info("ProdService已加载---Profile");
        return new ProdService();
    }

```

# 备注：springboot的核心注解

spring.factories文件里每一个xxxAutoConfiguration文件一般都会有下面的条件注解:

@ConditionalOnBean：当容器里有指定Bean的条件下

@ConditionalOnClass：当类路径下有指定类的条件下

@ConditionalOnExpression：基于SpEL表达式作为判断条件

@ConditionalOnJava：基于JV版本作为判断条件

@ConditionalOnJndi：在JNDI存在的条件下差在指定的位置

@ConditionalOnMissingBean：当容器里没有指定Bean的情况下

@ConditionalOnMissingClass：当类路径下没有指定类的条件下

@ConditionalOnNotWebApplication：当前项目不是Web项目的条件下

@ConditionalOnProperty：指定的属性是否有指定的值

@ConditionalOnResource：类路径是否有指定的值

@ConditionalOnSingleCandidate：当指定Bean在容器中只有一个，或者虽然有多个但是指定首选Bean

@ConditionalOnWebApplication：当前项目是Web项目的条件下。

上面@ConditionalOnXXX都是组合@Conditional元注解，使用了不同的条件Condition

