

# SpringBoot 启动源码解析

### 准备

在正式讲解 SpringBoot 启动源码解析之前，先来回顾下搭建 SpringBoot 环境的流程：

1. 手动创建一个 maven 项目；
2. 在 pom.xml 配置文件中导入以下 SpringBoot 依赖包：

```
     <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
     </parent>
     <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
     </dependency>

```

除了以上手动建立 SpringBoot 项目方式，还可使用以下两种方式：

1. 通过 IDEA 或 Eclipse 的 Spring Initializr 创建；
2. 访问 https://start.spring.io 创建 SpringBoot 的骨架

当然 SpringBoot 应用的基本骨架你也有必要知道：

```
    pom.xml
    src
        main
            java
                DemoApplication
            resources
                application.properties
                templates
                static
        test
            java
            resources

```

好了，话不多说，现在开始进入正题。

首先来看一下 SpringBoot 启动的正确方式（注：本实例采用的是 SpringBoot1.4.3 版本进行说明，最新版本可能部分源代码不同）：

```
public static void main(String[] args) {
    //首先创建SpringApplication类
   SpringApplication sa = new SpringApplication(Application.class);
   //可添加自定义的监听器  如CustomListener
   sa.addListeners(new CustomListener());
   sa.run(args);//开始启动运行
}

```

### SpringApplication 类的源码

主要看下其构造方法

```
public SpringApplication(Object... sources) {
    initialize(sources);
}

```

在其构造方法中，传入资源数组，然后通过 initialize 方法进行初始化，initialize 方法源码如下：

```
private void initialize(Object[] sources) {
    if (sources != null && sources.length > 0) {
    //1，将传入的Application放入Set<Object>sources集合中
        this.sources.addAll(Arrays.asList(sources));
    }
    //2，判断是否是Web环境
    this.webEnvironment = deduceWebEnvironment();
    //3，创建并初始化ApplicationInitializer列表
    setInitializers((Collection) getSpringFactoriesInstances(
            ApplicationContextInitializer.class));
    //4，创建并初始化ApplicationListener列表
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //5,初始化主类mainApplicationClass
    this.mainApplicationClass = deduceMainApplicationClass();
}

```

下面依次来看下该方法中的每一步骤。

#### 1. 将传入的 Application 放入 `Set< Object >sources` 集合中

#### 2. 判断是否是 Web 环境，即 deduceWebEnvironment 方法源码：

```
    private boolean deduceWebEnvironment() {
        for (String className : WEB_ENVIRONMENT_CLASSES) {
            if (!ClassUtils.isPresent(className, null)) {
                return false;
            }
        }
        return true;
    }

```

通过在 classpath 中查看是否存在 `WEB_ENVIRONMENT_CLASSES` 这个数组中所包含的所有类（实际上就是 2 个类）：

```
javax.servlet.Servlet
org.springframework.web.context.ConfigurableWebApplicationContext)

```

如果两个类都存在那么当前程序即是一个 Web 应用程序，反之则不是。

#### 3. 创建并初始化 ApplicationInitializer 列表：

```
setInitializers((Collection)getSpringFactoriesInstances(ApplicationContextInitializer.class));

```

其中 setInitializers 方法源码为：

```
    public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {
        this.initializers = new ArrayList<ApplicationContextInitializer<?>>();
        this.initializers.addAll(initializers);
    }

```

该方法表示创建一个 `ArrayList<ApplicationContextInitializer<?>>` 实例 initializers，之后将所有传入的初始化器都添加到 initializers 实例中。

那么这些 initializers 参数是怎么获取的呢？通过方法 getSpringFactoriesInstances 可以找到答案：

```
    private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type) {
        return getSpringFactoriesInstances(type, new Class<?>[] {});
    }

```

在 getSpringFactoriesInstances 方法中，getSpringFactoriesInstances 方法源码为：

```
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();// Use names and ensure unique to protect against duplicates
        //获取要加载的initializer的名字
    Set<String> names = new LinkedHashSet<String>(
                SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        //实例化所有initializer集合并返回
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes,classLoader, args, names);
        AnnotationAwareOrderComparator.sort(instances);
        return instances;
    }

```

其中最重要的就两句，获取要加载的 initializer 的名字（并且使用 set 集合去重）；之后将所有 initilaizer 实例化并返回。实例化代码没什么好说的，就是简单的反射创建对象，代码如下：

```
private <T> List<T> createSpringFactoriesInstances(Class<T> type,Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,Set<String> names) {
    List<T> instances = new ArrayList<T>(names.size());
    for (String name : names) {
     try {
        Class<?> instanceClass = ClassUtils.forName(name,classLoader);
        Assert.isAssignable(type, instanceClass);
        Constructor<?> constructor =instanceClass.getDeclaredConstructor(parameterTypes);
        T instance = (T) BeanUtils.instantiateClass(constructor, args);
        instances.add(instance);
    }catch (Throwable ex) {
        throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
    }
 }
    return instances;
 }

```

关键是怎么获取这些 initializer 的名字呢？

来看一下 SpringFactoriesLoader.loadFactoryNames(type, classLoader) 的源码：

```
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    try {
        Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) : ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        List<String> result = new ArrayList<String>();
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            Properties properties =PropertiesLoaderUtils.loadProperties(new UrlResource(url));
            String factoryClassNames = properties.getProperty(factoryClassName);
            result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
        }
            return result;
    }catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() + "] factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}

```

其实就是从 Maven 仓库中的 spring-boot-1.4.3.RELEASE.jar 以及 spring-boot-autoconfigure-1.4.3.RELEASE.jar 的 META-INF/spring.factories 中获取 key 为 org.springframework.context.ApplicationContextInitializer 的所有 initializer 全类名。

看一下 spring-boot-1.4.3.RELEASE.jar 中的 spring.factories 中的 initializers：

```
# Application Context Initializers
org.springframework.context.ApplicationContextIntializer=\org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\org.springframework.boot.context.ContextIdApplicationContextInitializer,\org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer

```

再看一下 spring-boot-autoconfigure-1.4.3.RELEASE.jar 中的 spring.factories 中的 initializers：

```
# Initializers
org.springframework.context.ApplicationContextInitializer=\org.springframework.boot.autoconfigure.ShareMetadataReaderFactoryContextInitializer,\org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer

```

所以，初始化后的`List<ApplicationContextInitializer<?>>intializer`包含如上6个initializer实例。

#### 4. 创建并初始化 ApplicationListener 列表：

```
public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
    this.listeners = new ArrayList<ApplicationListener<?>>();
    this.listeners.addAll(listeners);
}

```

初始化 `List<ApplicationListener<?>> listener` 实例的过程与初始化 intializer 实例的过程一样。来看一下 spring-boot-1.4.3.RELEASE.jar 中的 spring.factories 中的 listeners：

```
# Application Listeners:
org.springframework.context.ApplicationListener=\org.springframework.boot.ClearCachesApplicationListener,\org.springframework.boot.builder.ParentContextCloserApplicationListener,\org.springframework.boot.context.FileEncodingApplicationListener,\org.springframework.boot.context.config.ConfigFileApplicationListener,\org.springframework.boot.context.config.DelegatingApplicationListener,\org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener,\org.springframework.boot.logging.ClasspathLoggingApplicationListener,\org.springframework.boot.logging.LoggingApplicationListener

```

再看一下 spring-boot-autoconfigure-1.4.3.RELEASE.jar 中的 spring.factories 中的 listeners：

```
# Application Listeners
org.springframework.context.ApplicationListener=\org.springframework.boot.autoconfigure.BackgroundPreinitializer

```

所以，初始化后的 `List<ApplicationListener<?>> listeners` 包含如上 10 个 listener 实例。

#### 5. 初始化主类 mainApplicationClass 源码分析：

```
private Class<?> deduceMainApplicationClass() {
    try {
        StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
        for (StackTraceElement stackTraceElement : stackTrace) 
        {
        if ("main".equals(stackTraceElement.getMethodName())) 
            {
                return Class.forName(stackTraceElement.getClassName());
            }
        }
    }catch (ClassNotFoundException ex) {
        // Swallow and continue
    }
        return null;
}

```

该方法也很简单，首先获取方法调用栈，之后循环遍历该调用栈，最后一个 stackTraceElement 的方法名就是 main（因为方法是从 main 方法发起的）。初始化完主类之后，`Class<?> mainApplicationClass` 的值为 xxxxx.Application。

### 添加自定义监听器

看下添加自定义监听器的正确方法：

```
sa.addListeners(new CustomListener());

```

看下 addListeners 方法源码：

```
public void addListeners(ApplicationListener<?>... listeners) 
    {
        this.listeners.addAll(Arrays.asList(listeners));
    }

```

就是向之前的 listener 列表中添加一个自己的监听器，这里就不做过多说明了。

### 启动核心 run 方法源码解析

```
public ConfigurableApplicationContext run(String... args) {
        //1,创建并启动计时器StopWatch
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    FailureAnalyzers analyzers = null;
        //2，配置awt系统属性
    configureHeadlessProperty();
        //3，获取SpringApplicationRunListeners
    SpringApplicationRunListeners listeners = getRunListeners(args);
        //4，启动SpringApplicationRunListener
    listeners.starting();
    try {
            //5，创建ApplicationArguments
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            //6，创建并初始化ConfigurableEnvironment
        ConfigurableEnvironment environment = prepareEnvironment(listeners,applicationArguments);
            //7,打印Banner
        Banner printedBanner = printBanner(environment);
            //8，创建ConfigurableApplicationContext
        context = createApplicationContext();
        analyzers = new FailureAnalyzers(context);
            //9，准备ConfigurableApplicationContext
        prepareContext(context, environment, listeners, applicationArguments,printedBanner);
            //10,刷新ConfigurableApplicationContext
        refreshContext(context);
            //11，容器刷新后动作
        afterRefresh(context, applicationArguments);
            //12，SpringApplicationRunListener发布finish事件
        listeners.finished(context, null);
            //13，计时器停止计时
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        return context;
    }catch (Throwable ex) {
        handleRunFailure(context, listeners, analyzers, ex);
        throw new IllegalStateException(ex);
    }
}

```

#### 创建并启动停止计时器

StopWatch类表示停止计时器，主要有两个方法：start和stop，在start方法中只是记录了当前时间，而在stop中计算了总的启动时间，通过属性running来判断是否启动，running初始值为false。

start方法源码：

```
public void start(String taskName) throws IllegalStateException {
    if (this.running) {
        throw new IllegalStateException("Can't start StopWatch: it's already running");
    }
    this.running = true;
    this.currentTaskName = taskName;
    this.startTimeMillis = System.currentTimeMillis();
}

```

stop 方法源码：

```
public void stop() throws IllegalStateException {
    if (!this.running) {
        throw new IllegalStateException("Can't stop StopWatch: it's not running");
    }
    long lastTime = System.currentTimeMillis() - this.startTimeMillis;
    this.totalTimeMillis += lastTime;
    this.lastTaskInfo = new TaskInfo(this.currentTaskName, lastTime);
    if (this.keepTaskList) {
        this.taskList.add(lastTaskInfo);
    }
    ++this.taskCount;
    this.running = false;
    this.currentTaskName = null;
}

```

#### 配置 awt 系统属性源码分析：

设置 awt 系统属性，与我们的服务关系不大，这里只列出源码：

```
private void configureHeadlessProperty() {
    System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, System.getProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
}

```

#### 获取 SpringApplicationRunListeners 源码分析：

```
private SpringApplicationRunListeners getRunListeners(String[] args) {
    Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
    return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
}

```

与之前初始化 initializer 和 listener 一样，从 spring-boot-1.4.3.RELEASE.jar 中的 spring.factories 中读取的 SpringApplicationRunListener 全类名，之后创建该对象。来看一下 spring.factories 中有哪些 SpringApplicationRunListeners：

```
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\org.springframework.boot.context.event.EventPublishingRunListener

```

实际上就是创建了个仅含 EventPublishingRunListener 实例的列表。之后将该列表赋值到 SpringApplicationRunListeners 中的 listener 实例中，代码如下：

```
private final Log log;
private final List<SpringApplicationRunListener> listeners;
SpringApplicationRunListeners(Log log,Collection<? extends SpringApplicationRunListener> listeners) {
    this.log = log;
    this.listeners = new ArrayList<SpringApplicationRunListener>(listeners);
}

```

EventPublishingRunListener 类是一个非常重要的类，用来发布事件，触发监听器，我们看一下该对象的创建源码：

```
private final SpringApplication application;
private final String[] args;
private final ApplicationEventMulticaster initialMulticaster;
public EventPublishingRunListener(SpringApplication application,String[] args){
   this.application = application;
   this.args = args;
   this.initialMulticaster = new SimpleApplicationEventMulticaster();
   for(ApplicationListener<?> listener : application.getListeners()){
        this.initialMulticaster.addApplicationListener(listener);
      }
}

```

这里将之前所有的 listener 都添加到了 initialMulticaster 实例中，当 initialMulticaster 发布事件时，监听器监听到该事件，就会做出相应动作。

#### 启动 SpringApplicationRunListener 源码：

```
public void started(){
     for(SpringApplicationRunListener listener: this.listeners)
        {
           listener.started();
        }
    }

```

此时 EventPublishingRunListener 就会发布 ApplicationStartedEvent 事件：

```
@Override
public void started(){
    this.initialMulticaster.multicasterEvent(new ApplicationStartedEvent(this.application,this.args));
}

```

所有监听了该事件的 listener 就会执行相应的方法，这里布列出来了。

#### 创建 ApplicationArguments 源码分析：

```
ApplicationArguments applicationArguments = new defaultApplicationArguments(args);

```

其中传入的 args 参数就是 main 方法中传入的参数，这里传入的是 spring.output.ansi.enabled=always，该参数指定了日志输出使用多彩输出。

看一下源码：

```
private final Source source;
private final String[] args;
public DefaultApplicationArguments(String[] args){
     Assert.notNull(args,"Args must not be null");
     this.source = new Source(args);
     this.args = args;
}

```

其中 Source 内部类是 SimpleCommandLinePropertySource 的子类， SimpleCommandLinePropertySource 是 CommandLinePropertySource 的子类， CommandLinePropertySource 是 EnumerablePropertySource 子类， EnumerablePropertySOurce 是 PropertySource 子类。

如果把源码列出来，会很占篇幅，这里只列出最后 DefaultApplicationArguments 的初始化结果（形象化的表示，不考虑是否符合 Java 语法），两个属性：args 数组和 Source source，其中 args 数组如下：

```
String[] args = {"--spring.output.ansi.enabled=always"};
Source source如下：
String name = "commandLineArgs";
T source = commandLineArgs;

```

再看看一下 CommandLineArgs：

```
private final Map<String,List<String>> optionArgs = new HashMap<String,List<String>>();
private final List<String> nonOptionArgs = new ArrayList<String>();

```

CommandLineArgs 包含以上两个参数，其中 optionArgs 在代码执行完之后，包含一个 key-value 对，其中 key 为 "spring.output.ansi.enabled"，value 是一个只包含一个元素"always"的列表。nonOptionArgs 没有变化。

#### 创建并初始化 ConfigurableEnvironment 源码解析：

```
ConfigurableEnvironment environment = prepareEnvironment(listeners,applicationArguments);

```

准备环境阶段 prepareEnvironment 方法源码：

```
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,ApplicationArguments applicationArguments) {
        // Create and configure the environment
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    configureEnvironment(environment,applicationArguments.getSourceArgs());
    listeners.environmentPrepared(environment);
    if (!this.webEnvironment) {
        environment = new EnvironmentConverter(getClassLoader()).convertToStandardEnvironmentIfNecessary(environment);
    }
    return environment;
}

```

总的来书，一共做了三件事：

1. 创建一个 COnfigurableEnvironment 实例；
2. 配置 ConfigurableEnvironment 实例；
3. 发布 ApplicationEnvironmentPreparedEvent 事件，通知所有监听了该事件的监听器做相应操作。

接下来我们在逐一对这三件事源码做分析。

##### **创建 ConfigurableEnvironment 实例**

```
        private ConfigurableEnvironment getOrCreateEnvironment() {
        if (this.environment != null) {
                return this.environment;
            }
            if (this.webEnvironment) {
                return new StandardServletEnvironment();
            }
            return new StandardEnvironment();
        }

```

创建 ConfigurableEnvironment 实例时，由于是 web 环境，因此 this.webEnvironment为true，所以创建了一个 StanderdServletEnvironment。在调用 StandersEnvironment 的构造器的时候，会先调用其父类 AbstractEnvironment 的构造器，这里截取部分 AbstractEnvironment 中的重要代码，代码如下：

```
        private final MutablePropertySources propertySources = new MutablePropertySources(this.logger);
        public AbstractEnvironment() {
            customizePropertySources(this.propertySources);
            if (logger.isDebugEnabled()) {
                logger.debug("Initialized " + getClass().getSimpleName() + " with PropertySources " + this.propertySources);
            }
        }

```

其中 MutablePropertySources 是 PropertySources 的实现类，其含有一个很重要的属性，如下：

```
private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<PropertySource<?>>();

```

propertySourceLis t属性会存放所有的属性源 PropertySource。接下来及做这个事儿。在 AbstractEnvironment 的无参构造器中调用 StandardServletEnvironment 的 customizePropertySources 方法，StandardServletEnvironment 的 customizePropertySources 方法源码如下：

```
protected void customizePropertySources(MutablePropertySources propertySources) {
    propertySources.addLast(new StubPropertySource("servletConfigInitParams"));
    propertySources.addLast(new StubPropertySource("servletContextInitParams"));
    if (JndiLocatorDelegate.isDefaultJndiEnvironmentAvailable()) {
        propertySources.addLast(new JndiPropertySource("jndiProperties"));
    }
        super.customizePropertySources(propertySources);
}

```

这里为 propertySourceListpropertySourceList 添加了两个 PropertySource：一个是 name 为“servletConfigInitParams”的属性源；另一个是 name 为"servletContextInitParams"的属性源。这两个属性源暂且都没有属性值。之后调用了 StandardServletEnvironment 的父类的 customizePropertySources，如下：

```
protected void customizePropertySources(MutablePropertySources propertySources) {
    propertySources.addLast(new MapPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
    propertySources.addLast(new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
}

```

这里为 propertySourceList 又添加两个 PropertySource，一个是 MapPropertySource，其 name 为“systemProperties”，值为一个Map的数据结构，主要用于存放 JVM 的一些系统属性，例如："java.runtime.name"-"Java(TM)SE Runtime Envirnment"这样的 key-value 对。

值得注意的是，其中在启动的 JVM 参数中设置了"-Dserver.port=8088"，这个值就被存放在了该 Map 中，server.port=8088。此时，里边暂时存了 55 个 key-value 对。另一个属性源 name 为 "systemEnvironment"，值为 Map 的数据结构，主要用于存放一些系统环境属性，通常我们配置的环境变量，例如在 ~/.bash*profile 等文件中设置的都会被读取到，比如 M2*HOME 等。里边暂时存放了 21 个 key-value 对。

此时，一个ConfigurableEnvironment实例就被创建成功了。看一下创建完成后的该实例包含的一些东西：

- 一个不包含任何元素的LinkedHashSet，name为"activeProfile"。
- 一个包含一个元素"default"的LinkedHashSet，name为"defaultProfiles"。
- 一个MutablePropertySources实例propertySources，其内部的属性propertySourceList包含4个属性源。
- 一个PropertySourcesPropertyResolver属性，主要用于解析属性源。其内部包含一个包含上述MutablePropertySources实例的属性，以及用于解析属性源的其他属性，例如placeholderPrefix=${、placeholderSuffix=}、valueSeparator=: 等。

##### **配置 ConfigurableEnvironment 实例源码分析：**

```
protected void configureEnvironment(ConfigurableEnvironment environment,String[] args) {
    configurePropertySources(environment, args);
    configureProfiles(environment, args);
}

```

```
   对于ConfigurableEnvironment的配置，其实就是配置其两个属性：propertySources和activeProfiles。

```

```
protected void configurePropertySources(ConfigurableEnvironment environment,String[] args) {
    MutablePropertySources sources = environment.getPropertySources();
    if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
        sources.addLast(new MapPropertySource("defaultProperties", this.defaultProperties));
    }
    if (this.addCommandLineProperties && args.length > 0) {//addCommandLineProperties=true
        String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
     if (sources.contains(name)) {
        PropertySource<?> source = sources.get(name);
        CompositePropertySource composite = new CompositePropertySource(name);
        composite.addPropertySource(new SimpleCommandLinePropertySource(name + "-" + args.hashCode(), args));
        composite.addPropertySource(source);
        sources.replace(name, composite);
      }else {
            sources.addFirst(new SimpleCommandLinePropertySource(args));
      }
   }
}

```

该方法传入的参数 args 是包含一个参数"--spring.output.ansi.enabled=always"的数组，所以整个流程就是创建一个 SimpleCommandLinePropertySource（其实该属性源就是读取main函数传入参数并做相应的封装），并将该属性源放置在 ConfigurableEnvironment 的 propertySources 属性的首部，也就是说该属性源的优先级最高。propertySources 属性中的属性源的存放位置从前到后优先级依次降低。此时 propertySources 属性中包含了 5 个属性源。

再配置 activeProfiles：

```
protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
      environment.getActiveProfiles(); // ensure they are initialized
      // But these ones should go first (last wins in a property key clash)
      Set<String> profiles = new LinkedHashSet<String>(this.additionalProfiles);
      profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
      environment.setActiveProfiles(profiles.toArray(new String[profiles.size()]));
}

```

该配置文件其实就是读取"spring.profiles.active"所指定的配置文件，该配置值通常用于指定在不同的环境下使用不同的配置文件，但是在配置外存放之后几乎不会再使用该配置了。这里我们没有配置它，所以 activeProfiles 属性的 size 值依然能为0。可以简单的看一下 environment.getActiveProfiles() 的源码：

```
@Override
public String[] getActiveProfiles() {
      return StringUtils.toStringArray(doGetActiveProfiles());
}
        /**
         * Return the set of active profiles as explicitly set through
         * {@link #setActiveProfiles} or if the current set of active profiles
         * is empty, check for the presence of the {@value #ACTIVE_PROFILES_PROPERTY_NAME}
         * property and assign its value to the set of active profiles.
         * @see #getActiveProfiles()
         * @see #ACTIVE_PROFILES_PROPERTY_NAME
         */
protected Set<String> doGetActiveProfiles() {
     synchronized (this.activeProfiles) {
        if (this.activeProfiles.isEmpty()) {
            String profiles = getProperty(ACTIVE_PROFILES_PROPERTY_NAME);
            if (StringUtils.hasText(profiles)) {
                  setActiveProfiles(StringUtils.commaDelimitedListToStringArray(
                      StringUtils.trimAllWhitespace(profiles)));
            }
        }
        return this.activeProfiles;
     }
}

```

##### **发布 ApplicationEnvironmentPreparedEvent 事件**

发布 ApplicationEnvironmentPreparedEvent 事件源码

```
listeners.contextPrepared(context);

```

进入该方法：

```
public void environmentPrepared(ConfigurableEnvironment environment) {
     for (SpringApplicationRunListener listener : this.listeners) {
        listener.environmentPrepared(environment);
      }
}

```

在 listener 中只有一个元素，就是之前的 EventPublishingRunListener，此时该监听器发布了 ApplicationEnvironmentPreparedEvent 事件，看一下源码：

```
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
      this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(
      this.application, this.args, environment));
}

```

所有监听了该事件的监听器执行相应的逻辑。这里就不列出了。这些监听器监听之后，最主要做的一件事，就是读取配置文件 application.properties，并把这些信息存储在一个 name 为"applicationCOnfigurationProperties"的属性源中，并将该属性源置于整个属性源列表的最后。如果将来只是从外放的配置文件中读取配置，最好将该配置源删掉，然后自己读取外部配置文件构建数据源，之后再将该数据源放到数据源列表中。

#### 打印 Banner 源码：

```
Banner printedBanner = printBanner(environment);

```

该行主要用于打印出 SpringBoot 的图标和版本号，可以自己定制，这里不做解析。

#### 创建 ConfigurableApplicationContext 源码分析：

```
context = createApplicationContext();
protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
        try {
            contextClass = Class.forName(this.webEnvironment ? DEFAULT_WEB_CONTEXT_CLASS : DEFAULT_CONTEXT_CLASS);
        }catch (ClassNotFoundException ex) {
        throw new IllegalStateException("Unable create a default ApplicationContext, "+ "please specify an ApplicationContextClass",ex);
        }
    }
    return (ConfigurableApplicationContext) BeanUtils.instantiate(contextClass);
}

```

这里主要是为 contextClass 赋值，使用了 SpringBoot 自己的一个 AnnotationConfigEmbeddedWebApplicationContext。之后使用反射实例化该 ApplicationContext，

看一下 instantiate 的源码：

```
public static <T> T instantiate(Class<T> clazz) throws BeanInstantiationException {
    Assert.notNull(clazz, "Class must not be null");
    if (clazz.isInterface()) {
    throw new BeanInstantiationException(clazz, "Specified class is an interface");
    }
    try {
        return clazz.newInstance();
    }catch (InstantiationException ex) {
        throw new BeanInstantiationException(clazz, "Is it an abstract class?", ex);
    }catch (IllegalAccessException ex) {
        throw new BeanInstantiationException(clazz, "Is the constructor accessible?", ex);
    }
}

```

#### 准备 ConfigurableApplicationContext 源码分析：

```
prepareContext(context, environment, listeners, applicationArguments,printedBanner);

```

进入该方法：

```
private void prepareContext(ConfigurableApplicationContext context,ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,ApplicationArguments applicationArguments, Banner printedBanner) {
    context.setEnvironment(environment);
    postProcessApplicationContext(context);
        //1，执行初始化器
    applyInitializers(context);
    listeners.contextPrepared(context);
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }
        // Add boot specific singleton beans
context.getBeanFactory().registerSingleton("springApplicationArguments",applicationArguments);
    if (printedBanner != null) {
        context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
    }
        // Load the sources
    Set<Object> sources = getSources();
        Assert.notEmpty(sources, "Sources must not be empty");
        //2，加载配置
    load(context, sources.toArray(new Object[sources.size()]));
    listeners.contextLoaded(context);
}

```

该方法重点包含两部分：执行初始化器和加载配置

##### **执行初始化器源码分析：**

```
protected void applyInitializers(ConfigurableApplicationContext context) {
    for (ApplicationContextInitializer initializer : getInitializers()) {
        Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(), ApplicationContextInitializer.class);
        Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
        initializer.initialize(context);
        }
}

```

遍历所有的初始化器，执行每一个初始化器的initialize方法。这里可以通过实现ApplicationContextInitializer接口来创建自己的初始化器。例如，可以在初始化器中读取外部配置文件。

##### **加载配置源码分析：**

```
load(context, sources.toArray(new Object[sources.size()]));

```

其中，sources 是一个只包含主类的 set 集合。在该方法中设置实例属性、资源和环境，最后进行统一设置，源码如下：

```
protected void load(ApplicationContext context, Object[] sources) {
     if (logger.isDebugEnabled()) {
         logger.debug("Loading source " + StringUtils.arrayToCommaDelimitedString(sources));
     }
     BeanDefinitionLoader loader = createBeanDefinitionLoader(getBeanDefinitionRegistry(context), sources);
     if (this.beanNameGenerator != null) {
         loader.setBeanNameGenerator(this.beanNameGenerator);
     }
     if (this.resourceLoader != null) {
         loader.setResourceLoader(this.resourceLoader);
     }
     if (this.environment != null) {
         loader.setEnvironment(this.environment);
     }
         loader.load();
}

```

对于 loader.load() 方法进入后会有有一系列的 load 方法链调用，这里只介绍一个比较重要的load方法，找到 `load((Class<?>) source)` 方法，源码如下：

```
private int load(Class<?> source) {
      if (isGroovyPresent()) {
// Any GroovyLoaders added in beans{} DSL can contribute beans here
      if (GroovyBeanDefinitionSource.class.isAssignableFrom(source)){
        GroovyBeanDefinitionSource loader = BeanUtils.instantiateClass(source,GroovyBeanDefinitionSource.class);
        load(loader);
       }
    }
        if (isComponent(source)) {
        this.annotatedReader.register(source);
            return 1;
        }
        return 0;
}

```

isGroovyPresent() 方法为判断 Groovy 部分，看下下面这个 isComponent(source) 方法，查看源码：

```
private boolean isComponent(Class<?> type) {
// This has to be a bit of a guess. The only way to be sure that this type is
// eligible is to make a bean definition out of it and try to instantiate it.
      if (AnnotationUtils.findAnnotation(type, Component.class) != null) {
        return true;
       }
// Nested anonymous classes are not eligible for registration, nor are groovy
// closures
      if (type.getName().matches(".*\\$_.*closure.*") || type.isAnonymousClass()|| type.getConstructors() == null || type.getConstructors().length == 0) {
            return false;
        }
     return true;

```

该方法主要用来判断某个类是否加了 @Component 注解，如果加了返回 true，没有返回 false，最后将该资源注册到 annotatedReader 中，返回 1 表示成功。

#### 刷新 ConfigurableApplicationContext 源码分析：

```
refreshContext(context);

```

由于该调用链比较长，这里分析比较重要的 ((AbstractApplicationContext) applicationContext).refresh() 方法，源码如下：

```
@Override
public void refresh() throws BeansException, IllegalStateException {
       synchronized (this.startupShutdownMonitor) {
// Prepare this context for refreshing.
        prepareRefresh();
// Tell the subclass to refresh the internal bean factory.
       ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
// Prepare the bean factory for use in this context.
       prepareBeanFactory(beanFactory);
       try {
// Allows post-processing of the bean factory in context subclasses.
        postProcessBeanFactory(beanFactory);
// Invoke factory processors registered as beans in the context.
        invokeBeanFactoryPostProcessors(beanFactory);
// Register bean processors that intercept bean creation.
          registerBeanPostProcessors(beanFactory);
// Initialize message source for this context.
        initMessageSource();
// Initialize event multicaster for this context.
        initApplicationEventMulticaster();
// Initialize other special beans in specific context subclasses.
        onRefresh();
// Check for listener beans and register them.
        registerListeners();
// Instantiate all remaining (non-lazy-init) singletons.
        finishBeanFactoryInitialization(beanFactory);
// Last step: publish corresponding event.
        finishRefresh();
     }catch (BeansException ex) {
        if (logger.isWarnEnabled()) {
        logger.warn("Exception encountered during context initialization - " +"cancelling refresh attempt: " + ex);
        }
// Destroy already created singletons to avoid dangling resources.
            destroyBeans();
// Reset 'active' flag.
            cancelRefresh(ex);
// Propagate exception to caller.
            throw ex;
        }finally {
// Reset common introspection caches in Spring's core, since we
// might not ever need metadata for singleton beans anymore...
        resetCommonCaches();
        }
      }
}

```

这个方法是 Spring 最重要的一个方法，其中比较重要的有 this.onRefresh()和this.finishRefresh() 方法。

onRefresh() 方法有很多实现方法，我们找到 EmbeddedWebApplicationContext 类中的 onRefresh 方法，源码如下：

```
@Override
protected void onRefresh() {
      super.onRefresh();
      try {
        createEmbeddedServletContainer();
       }catch (Throwable ex) {
        throw new ApplicationContextException("Unable to start embedded container",ex);
        }
}

```

这个方法创建了一个内嵌的 Servlet 容器，用于执行 Web 应用，默认使用 TomcatEmbedded-ServletContainer。

finishRefresh() 方法同样有很多实现方法，这里找到 EmbeddedWebApplicationContext类中的finishRefresh() 加以说明：

```
@Override
protected void finishRefresh() {
     super.finishRefresh();
     EmbeddedServletContainer localContainer = startEmbeddedServletContainer();
     if (localContainer != null) {
        publishEvent(new EmbeddedServletContainerInitializedEvent(this, localContainer));
      }
}

```

首先调用了父类 AbstractApplicationContext的finishRefresh() 方法，在该方法中发布了 ContextRefreshedEvent 事件，所有监听了该事件的监听器开始执行逻辑；然后启动 TomcatEmbeddedServletContainer，最后发布 EmbeddedServletContainerInitializedEvent，所有监听了该事件的监听器执行相应的逻辑。

#### 容器刷新后动作源码：

```
afterRefresh(context, applicationArguments);

```

该方法主要用来调取 run 方法，在 run 方法中执行一些 job，这里不做过多说明。

#### SpringApplicationRunListeners 发布 finish 事件源码：

```
listeners.finished(context, null);

```



这里发布 ApplicationReadyEvent 或 ApplicationFailedEvent 事件，所有监听了该事件的监听器去执行相应的逻辑。

#### 计时器停止计时

```
stopWatch.stop();

```

这里主要用来计算总的时长，不再做过多说明。 综上 SpringBoot 启动源码流程说明大致可以用以下图形进行说明：

![enter image description here](http://images.gitbook.cn/7d922490-77ac-11e8-b44f-d74525bdada7)

demo 地址

> <https://download.csdn.net/download/qq_35255384/10385514>

