## 内容提要

内容提要：

- 请讲讲搞懂这些源码的实际作用，和之间的串联关系，越详细越好，联系实际去讲解可以吗？
- 根据Spring举例，比如思想举例说明，设计模式举例？
- 阅读源码的正确姿势/最佳实践是什么？
- Spring源码哪本书比较好？
- 设计模式应该主要掌握哪几种？
- 如何进行统一加解密使用Spring（还要考虑到上传文件返回文件数据的请款）？
- 谈谈模块化思想？
- 请问Spring源码那么多的类，类之间调用比较复杂，从哪里开始看比较好，如何理清头绪？是从低版本开始看吗？

------

**问：请讲讲搞懂这些源码的实际作用，和之间的串联关系，越详细越好，联系实际去讲解可以吗？**

**答：**读懂源码的作用在我看来有两点好处，面试/实践，目前编程变得越来越简单，初级程序员越来越多，读源码可以很好的让你理解设计模式，具备更加良好的编程能力。

------

**问：根据Spring举例，比如思想举例说明，设计模式举例？**

**答：**比如Spring，核心思想就是IOC/AOP，这种编程思想已经被很多语言复用，不详细解释，Spring用到了很多的设计模式。简单工厂，Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得bean对象。工厂方法，Spring并没有采用new方式创建对象，而是将一系列创建bean的动作交给工厂，工厂通过反射来创建对象，一层一层的将bean的每个功能模块化，更有利于代码维护。代理模式cglib和jdk动态代理。面试的话代理是必须问的，所以说一定要好好理解。还有观察者、观察者模式等。

------

**问：阅读源码的正确姿势/最佳实践是什么？**

**答：**阅读源码首先要了解每个类的作用，了解完了之后可以读一读别人的博客。如果想要深入理解一定要读大神写的书。实践的话，说实话，日常工作用到的很少，但是如果理解一定有益无害。

------

**问：Spring源码哪本书比较好？**

**答：**个人比价喜欢Spring源码深度解析和Spring技术内幕。《 J2EE with out ejb》这个是Spring作者写的，英文翻译过来，可能阅读体验不是特别好。

------

**问：设计模式应该主要掌握哪几种？**

**答：**面试而言，单例、代理、观察者足够了。能有其他的一定是加分项，作为技术积累的话，责任链模式、策略模式、模板方法模式、访问者模式，能了解多一些一定是好的。

------

**问：如何进行统一加解密使用Spring（还要考虑到上传文件返回文件数据的请款）？**

**答：**加密的话如果是密码，可以调用Spring security内置的5中方式。不过需要注意的是，这五种方式加密是不可解密的，同样在不同时间加密相同字符串等到的结果也是不同的。进行比较调用加密算法的equal进行比较。如果是文件是有专门的加密算法，将byte[]调用指定算法就ok了。目前我了解到的文件加密算法一般都是花钱的。

------

**问：谈谈模块化思想？**

**答：**提到模块化我第一想法就是java9作为java9新特性。我尝试着下载安装，发现以前的项目都无法运行了，所以直接卸载了，因为java并不是按照模块化思想设计的。不过模块化有助于但测试和开发的便捷性。

------

**问：请问Spring源码那么多的类，类之间调用比较复杂，从哪里开始看比较好，如何理清头绪？是从低版本开始看吗？**

**答：**Spring确实很多类，我觉得一定要从3.0的版本开始看，因为3.0时候还是xml的配置方式。从最简单的BeanFactory逐渐看他的父类一层一层了解这个类作用，使用这个类。到最后就可以掌握了，如果没有思路你可以先看看hashmap源码，看完hashmap之后你自然会知道Spring源码如何看。Spring4.0之后基于注解的方式一定要了解，毕竟新技术会逐渐成为主流。



## 文章实录

### 目录

- 二、IOC入门篇之概念理解
- 三、深入理解IOC核心类
- 四、IOC体系结构源码剖析


- 入门篇 -- 读文章 对这个知识有一定的了解，了解大概的发展趋势以及使用复杂程度，达到基本可以使用程度。
- 掌握篇 -- 看网上的教程 对这个知识使用并了解一些出现bug解决方案，扩展使用途径。
- 精通篇 -- 对知识体系结构已有完整认知 需要读别人写的书 加上自己亲自打代码时间 完全了解这个知识发展渊源。
- ​

### IOC入门篇之概念理解

控制反转(IoC)：概念的描述：

> Martin Fowler提出，那些方面的控制被反转了？ - 他给出的结论：依赖对象的获取被反转了，基于这个结论他为控制反转创造了一个由两个或多各类批次之间的合作来实现业务逻辑的，这是的每一个对象都需要与其合作的对象（也就是他依赖的对象）的引用。如果这个获取过程要考自身实现，那么如果如你所见，这将导致代码高度耦合难以测试。--《摘自维基百科》

我是这样理解IOC容器：以水桶为例，有大的，小的，有金的，银的，各种各样。IOC容器也如此，每种不同的容器有自己功能，但是他们有一个是不能改变的，那就是装水，我们所学习的IOC容器也一样。

IOC是Inversion of Control的缩写，由于引进了中间位置的第三方，也就是IOC容器使得没有关联关系的类有了一个共同的关系--被ICO容器所管理，所以说IOC容器起到了粘合剂的作用。

DI是Dependency Injection的所写，是指通过引入IOC容器，利用类与类相关的依赖关系，注入的方式，实现代码重用以及对象之间的解耦合。（说到解耦合，三大框架都是为了解耦合，面试时候千万别说解耦合 ，说具体，哈哈。）

IOC的好处，初始化的过程中就不可避免的会写大量的new，只需要维护XML或注解，不需要大量的修改代码，IOC容器是分层的，从最底层BeanFactory网上找（后面源码解读会详细讲解），实现类与类之间的解耦合，可以将代码分离，每个人只需要写自己的部分，利于团队协作。

复合IOC规范的产品: Spring、Guice、Pico Container、Avalon、HiveMind；重量级的有EJB，JBoss，Jdon等等。以上除了Spring别的都没用过，出去面试吹一下也没啥^-^

使用IOC容器的小缺点: 引入第三方工具，对性能，初始化等速度均有影响，需要配置一大堆（springboot简化了好多，推荐大家从基于xml的spring学起），通过反射机制创建对象，效率较低~~~实际没影响。

### 深入理解IOC核心类

BeanFactory的基本类体系结构：

![BeanFactory的基本类体系结构](http://images.gitbook.cn/89808360-c84c-11e7-ad0d-af5dea044937)

如图所示：BeanFactory是最基础的IOC底层，定义了一些bean的基本属性，XMLBeanFactory算比较高级的IOC容器了，IOC容器是分层的，每一层都有自己需要做的功能，一下将简略展示IOC各个容器的功能。

- BeanFactory 负责 获取bean，封装判断bean容器是否包含bean，判断bean是否为单利/指定类型，获取类的别名，类型匹配等。
- ListanleBeanFactory 继承自BeanFactory 封装bean的一些基本属性信息，如类的个数，类型，别名等，可以根据条件获取指定bean。
- AutowireCapableBeanFactory 继承自BeanFactory 封装提供一系列自动装噢诶bean的策略，自动注入初始化以及bean的前/后处理器，分解依赖等。
- HierarchicalBeanFactory 继承自BeanFactory 封装两个方法，一个获取bean工厂的父工厂，另一个判断本地工厂是否包含指定bean。
- SinglentonRegistry 定义对单例注册及获取是否包含等判断。
- ConfigurableBeanFactory 提供配置Factory的方法。
- AliaRegistry 提供对bean的别名的增删改查操作。
- BeanDefinitionRegister 使用BeanDefinition修饰bean，提供对BeanDefinition的增删改操作。
- ConfigurableBeanFactory 提供配置bean的方法 。
- ConfigurableListableBeanFactory 配置清单BeanFactory指定忽略类型，接口等。
- AbstractAutowireCapableBeanFactory 综合AbstractBeanFactory，对接自动装配bean。
- DefaultListableBeanFactory 综合上面的功能，主要针对bean的注册。

### IOC体系结构源码剖析

XmlBeanFactory 实现类体系结构:

![XmlBeanFactory 实现类体系结构](http://images.gitbook.cn/9b7de210-c9c8-11e7-ae3f-4d866022d471)

[Spring 5.x 源码下载](http://download.csdn.net/download/crpxnmmafq/10109434)

你也可以到github上下载gradle构架的Spring 然后把你的eclipse配置gradle 安装插件项目。

**SimpleAliasRegistry**

```
public class SimpleAliasRegistry implements AliasRegistry {

// 包含别名的map 就我了解 所有包含并发操作的都是用currentHashMap 推荐大家也使用
private final Map<String, String> aliasMap = new ConcurrentHashMap<String, String>(16);


@Override
public void registerAlias(String name, String alias) {
    Assert.hasText(name, "'name' must not be empty");
    Assert.hasText(alias, "'alias' must not be empty");
    // 若别名和实际bean名字相同，移除别名
    if (alias.equals(name)) {
        this.aliasMap.remove(alias);
    }
    else {
        // 获取bean的真实名
        String registeredName = this.aliasMap.get(alias);
        if (registeredName != null) {
            if (registeredName.equals(name)) {
                // An existing alias - no need to re-register
                return;
            }
            if (!allowAliasOverriding()) {
                throw new IllegalStateException("Cannot register alias '" + alias + "' for name '" +
                        name + "': It is already registered for name '" + registeredName + "'.");
            }
        }
        checkForAliasCircle(name, alias);// 当bean名字与name相同 检查别名是否相同 相同抛出异常 已存在的别名
        this.aliasMap.put(alias, name); 
    }
}
// SO WHAT ?    
protected boolean allowAliasOverriding() {
    return true;
}
// checkForAliasCircle调用的 上文指出
public boolean hasAlias(String name, String alias) {
    for (Map.Entry<String, String> entry : this.aliasMap.entrySet()) {
        String registeredName = entry.getValue();
        if (registeredName.equals(name)) {
            String registeredAlias = entry.getKey();
            return (registeredAlias.equals(alias) || hasAlias(registeredAlias, alias));
        }
    }
    return false;
}
// 移除别名
@Override
public void removeAlias(String alias) {
    String name = this.aliasMap.remove(alias);
    if (name == null) {
        throw new IllegalStateException("No alias '" + alias + "' registered");
    }
}
// 是否是别名
@Override
public boolean isAlias(String name) {
    return this.aliasMap.containsKey(name);
}
// 获取所有别名
@Override
public String[] getAliases(String name) {
    List<String> result = new ArrayList<String>();
    synchronized (this.aliasMap) {
        retrieveAliases(name, result);
    }
    return StringUtils.toStringArray(result);
}

// 找出所有name对应的别名 存入List<String>
private void retrieveAliases(String name, List<String> result) {
    for (Map.Entry<String, String> entry : this.aliasMap.entrySet()) {
        String registeredName = entry.getValue();
        if (registeredName.equals(name)) {
            String alias = entry.getKey();
            result.add(alias);
            retrieveAliases(alias, result);
        }
    }
}

// 处理所有的别名，如果处理正确，把原来的用解析后的替换
public void resolveAliases(StringValueResolver valueResolver) {
    Assert.notNull(valueResolver, "StringValueResolver must not be null");
    synchronized (this.aliasMap) {
        Map<String, String> aliasCopy = new HashMap<String, String>(this.aliasMap);
        for (String alias : aliasCopy.keySet()) {
            String registeredName = aliasCopy.get(alias);
            String resolvedAlias = valueResolver.resolveStringValue(alias);
            String resolvedName = valueResolver.resolveStringValue(registeredName);
            if (resolvedAlias == null || resolvedName == null || resolvedAlias.equals(resolvedName)) {
                this.aliasMap.remove(alias);
            }
            else if (!resolvedAlias.equals(alias)) {
                String existingName = this.aliasMap.get(resolvedAlias);
                if (existingName != null) {
                    if (existingName.equals(resolvedName)) {
                        // Pointing to existing alias - just remove placeholder
                        this.aliasMap.remove(alias);
                        break;
                    }
                    throw new IllegalStateException(
                            "Cannot register resolved alias '" + resolvedAlias + "' (original: '" + alias +
                            "') for name '" + resolvedName + "': It is already registered for name '" +
                            registeredName + "'.");
                }
                checkForAliasCircle(resolvedName, resolvedAlias);
                this.aliasMap.remove(alias);
                this.aliasMap.put(resolvedAlias, resolvedName);
            }
            else if (!registeredName.equals(resolvedName)) {
                this.aliasMap.put(alias, resolvedName);
            }
        }
    }
}

// 若当前map中存在别名和name相同 抛异常
protected void checkForAliasCircle(String name, String alias) {
    if (hasAlias(alias, name)) {
        throw new IllegalStateException("Cannot register alias '" + alias +
                "' for name '" + name + "': Circular reference - '" +
                name + "' is a direct or indirect alias for '" + alias + "' already");
    }
}


//根据name这个Key，在aliasMap中不断循环的取对应的value，如果取得到，就继续根据这个value取值，不断循环继续。直到取不到，就把这个在aliasMap中无对应值的key返回。

public String canonicalName(String name) {
    String canonicalName = name;
    // Handle aliasing...
    String resolvedName;
    do {
        resolvedName = this.aliasMap.get(canonicalName);
        if (resolvedName != null) {
            canonicalName = resolvedName;
        }
    }
    while (resolvedName != null);
    return canonicalName;
    }
}

```

**DefaultSingletonBeanRegistry**

```
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
    // 看看人家 代码写的多规范 我要学习
    protected static final Object NULL_OBJECT = new Object();

    // 定义记录log日志
    protected final Log logger = LogFactory.getLog(getClass());

    // 单例bean缓存
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

    // 单例工厂缓存
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

    // 没注册之前存放单例记录
    private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

    // 注册过的单例
    private final Set<String> registeredSingletons = new LinkedHashSet<String>(256);

    // 使用ConcurrentHashMap<T,Boolean> -- set<T> 底层我看过 那个Boolean属性true 包含
    // 这样的Set集合是采用锁分段的机制 对高并发操作优化
    // 即将创建的单例类
    private final Set<String> singletonsCurrentlyInCreation =
            Collections.newSetFromMap(new ConcurrentHashMap<String, Boolean>(16));

    // 正在创建的单例类
    private final Set<String> inCreationCheckExclusions =
            Collections.newSetFromMap(new ConcurrentHashMap<String, Boolean>(16));

    // 异常集合
    private Set<Exception> suppressedExceptions;

    // 单例类是否真正被销毁
    private boolean singletonsCurrentlyInDestruction = false;

    // disposable接口的实例
    private final Map<String, Object> disposableBeans = new LinkedHashMap<String, Object>();

    // bean名称和bean所有包含的Bean的名称的map
    private final Map<String, Set<String>> containedBeanMap = new ConcurrentHashMap<String, Set<String>>(16);

    // bean名称和所有依赖于Bean的名称的map
    private final Map<String, Set<String>> dependentBeanMap = new ConcurrentHashMap<String, Set<String>>(64);

    // bean名称和bean所依赖的所有名称的map
    private final Map<String, Set<String>> dependenciesForBeanMap = new ConcurrentHashMap<String, Set<String>>(64);

    // 注册单例类 重复注册抛异常
    @Override
    public void registerSingleton(String beanName, Object singletonObject) throws IllegalStateException {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            Object oldObject = this.singletonObjects.get(beanName);
            if (oldObject != null) {
                throw new IllegalStateException("Could not register object [" + singletonObject +
                        "] under bean name '" + beanName + "': there is already object [" + oldObject + "] bound");
            }
            addSingleton(beanName, singletonObject);
        }
    }

    // 从earlySingletonObjects，singletonFactories移除
    // 注册bean以及将bean添加到earlySingletonObjects
    protected void addSingleton(String beanName, Object singletonObject) {
        synchronized (this.singletonObjects) {
            this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));
            this.singletonFactories.remove(beanName);// 
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }

    // 注册一个单例工厂类，注册后从earlySingletonObjects移除
    protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(singletonFactory, "Singleton factory must not be null");
        synchronized (this.singletonObjects) {
            if (!this.singletonObjects.containsKey(beanName)) {
                this.singletonFactories.put(beanName, singletonFactory);
                this.earlySingletonObjects.remove(beanName);
                this.registeredSingletons.add(beanName);
            }
        }
    }

    @Override
    public Object getSingleton(String beanName) {
        return getSingleton(beanName, true);
    }

    // 获取单例类
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            synchronized (this.singletonObjects) {
                singletonObject = this.earlySingletonObjects.get(beanName);
                if (singletonObject == null && allowEarlyReference) {// 不存在且正在创建中
                    ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);// 从缓存工厂获取单例工厂
                    if (singletonFactory != null) {
                        //  单例工厂创建单例对象
                        singletonObject = singletonFactory.getObject();
                        // 压进去
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }
        return (singletonObject != NULL_OBJECT ? singletonObject : null);
    }

    // 获取指定的单例Bean
    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            Object singletonObject = this.singletonObjects.get(beanName);
            // 如果从当前的单例缓存中没找到指定bean
            if (singletonObject == null) {
                if (this.singletonsCurrentlyInDestruction) {//当前单例正在摧毁
                    throw new BeanCreationNotAllowedException(beanName,
                            "Singleton bean creation not allowed while singletons of this factory are in destruction " +
                            "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
                }
                if (logger.isDebugEnabled()) {
                    logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
                }
                beforeSingletonCreation(beanName);// 单例类创建之前执行
                boolean newSingleton = false;
                boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = new LinkedHashSet<Exception>();
                }
                try {
                    singletonObject = singletonFactory.getObject();
                    newSingleton = true;
                }
                catch (IllegalStateException ex) {
                    // Has the singleton object implicitly appeared in the meantime ->
                    // if yes, proceed with it since the exception indicates that state.
                    singletonObject = this.singletonObjects.get(beanName);
                    if (singletonObject == null) {
                        throw ex;
                    }
                }
                catch (BeanCreationException ex) {
                    if (recordSuppressedExceptions) {
                        for (Exception suppressedException : this.suppressedExceptions) {
                            ex.addRelatedCause(suppressedException);
                        }
                    }
                    throw ex;
                }
                finally {
                    if (recordSuppressedExceptions) {
                        this.suppressedExceptions = null;
                    }
                    afterSingletonCreation(beanName);
                }
                if (newSingleton) {
                    addSingleton(beanName, singletonObject);
                }
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    // 添加异常
    protected void onSuppressedException(Exception ex) {
        synchronized (this.singletonObjects) {
            if (this.suppressedExceptions != null) {
                this.suppressedExceptions.add(ex);
            }
        }
    }

    // 根据名称移除本容器中缓存的对应的单例Bean以及所有关联移除
    protected void removeSingleton(String beanName) {
        synchronized (this.singletonObjects) {
            this.singletonObjects.remove(beanName);
            this.singletonFactories.remove(beanName);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.remove(beanName);
        }
    }
    // 判断容器是否包含单例bean
    @Override
    public boolean containsSingleton(String beanName) {
        return this.singletonObjects.containsKey(beanName);
    }
    // 获取所有单例类名字
    @Override
    public String[] getSingletonNames() {
        synchronized (this.singletonObjects) {
            return StringUtils.toStringArray(this.registeredSingletons);
        }
    }
    // 获取单例类个数
    @Override
    public int getSingletonCount() {
        synchronized (this.singletonObjects) {
            return this.registeredSingletons.size();
        }
    }

    // 设置bean创建状态
    public void setCurrentlyInCreation(String beanName, boolean inCreation) {
        Assert.notNull(beanName, "Bean name must not be null");
        if (!inCreation) {
            this.inCreationCheckExclusions.add(beanName);
        }
        else {
            this.inCreationCheckExclusions.remove(beanName);
        }
    }
    // 当前是否正在被创建
    public boolean isCurrentlyInCreation(String beanName) {
        Assert.notNull(beanName, "Bean name must not be null");
        return (!this.inCreationCheckExclusions.contains(beanName) && isActuallyInCreation(beanName));
    }
    // 是否在被创建
    protected boolean isActuallyInCreation(String beanName) {
        return isSingletonCurrentlyInCreation(beanName);
    }
    // 
    public boolean isSingletonCurrentlyInCreation(String beanName) {
        return this.singletonsCurrentlyInCreation.contains(beanName);
    }

    // 单例类创建之前调用
    protected void beforeSingletonCreation(String beanName) {
        // 若即将创建的类不在正在创建的集合中，向将要即将创建的类加入
        if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }
    }

    // 单例类创建之后调用
    protected void afterSingletonCreation(String beanName) {
        // 若即将创建的类正在创建的集合中，向将要即将创建的类移除
        if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
            throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
        }
    }


    // 向disposable注册实例
    public void registerDisposableBean(String beanName, DisposableBean bean) {
        synchronized (this.disposableBeans) {
            this.disposableBeans.put(beanName, bean);
        }
    }

    // 注册一个子类 -> 父类
    public void registerContainedBean(String containedBeanName, String containingBeanName) {
        // A quick check for an existing entry upfront, avoiding synchronization...
        Set<String> containedBeans = this.containedBeanMap.get(containingBeanName);
        // 已存在 return
        if (containedBeans != null && containedBeans.contains(containedBeanName)) {
            return;
        }

        // No entry yet -> fully synchronized manipulation of the containedBeans Set
        synchronized (this.containedBeanMap) {
            containedBeans = this.containedBeanMap.get(containingBeanName);
            if (containedBeans == null) {
                containedBeans = new LinkedHashSet<String>(8);// 初始化
                // containingBeanName 父类 containedBeans 孩子集合
                this.containedBeanMap.put(containingBeanName, containedBeans);
            }
            containedBeans.add(containedBeanName);// 孩子进窝
        }
        registerDependentBean(containedBeanName, containingBeanName);// 添加依赖关系·
    }

    // 给beanName添加dependentBeanName依赖
    public void registerDependentBean(String beanName, String dependentBeanName) {
        // A quick check for an existing entry upfront, avoiding synchronization...
        String canonicalName = canonicalName(beanName);
        Set<String> dependentBeans = this.dependentBeanMap.get(canonicalName);
        // 已存在 ruturn
        if (dependentBeans != null && dependentBeans.contains(dependentBeanName)) {
            return;
        }

        // 和上面一样
        synchronized (this.dependentBeanMap) {
            dependentBeans = this.dependentBeanMap.get(canonicalName);
            if (dependentBeans == null) {
                dependentBeans = new LinkedHashSet<String>(8);
                this.dependentBeanMap.put(canonicalName, dependentBeans);
            }
            dependentBeans.add(dependentBeanName);
        }
        synchronized (this.dependenciesForBeanMap) {
            Set<String> dependenciesForBean = this.dependenciesForBeanMap.get(dependentBeanName);
            if (dependenciesForBean == null) {
                dependenciesForBean = new LinkedHashSet<String>(8);
                this.dependenciesForBeanMap.put(dependentBeanName, dependenciesForBean);
            }
            dependenciesForBean.add(canonicalName);
        }
    }

    // 是否依赖关系
    protected boolean isDependent(String beanName, String dependentBeanName) {
        return isDependent(beanName, dependentBeanName, null);
    }

    private boolean isDependent(String beanName, String dependentBeanName, Set<String> alreadySeen) {
        if (alreadySeen != null && alreadySeen.contains(beanName)) {
            return false;
        }
        String canonicalName = canonicalName(beanName);
        Set<String> dependentBeans = this.dependentBeanMap.get(canonicalName);
        if (dependentBeans == null) {
            return false;
        }
        if (dependentBeans.contains(dependentBeanName)) {
            return true;
        }
        for (String transitiveDependency : dependentBeans) {
            if (alreadySeen == null) {
                alreadySeen = new HashSet<String>();
            }
            alreadySeen.add(beanName);
            if (isDependent(transitiveDependency, dependentBeanName, alreadySeen)) {
                return true;
            }
        }
        return false;
    }

    // 是否有依赖bean
    protected boolean hasDependentBean(String beanName) {
        return this.dependentBeanMap.containsKey(beanName);
    }

    // 根据beanName获取所有依赖bean
    public String[] getDependentBeans(String beanName) {
        Set<String> dependentBeans = this.dependentBeanMap.get(beanName);
        if (dependentBeans == null) {
            return new String[0];
        }
        return StringUtils.toStringArray(dependentBeans);
    }

    // 根据beanName获取所有所依赖
    public String[] getDependenciesForBean(String beanName) {
        Set<String> dependenciesForBean = this.dependenciesForBeanMap.get(beanName);
        if (dependenciesForBean == null) {
            return new String[0];
        }
        return dependenciesForBean.toArray(new String[dependenciesForBean.size()]);
    }
    // 销毁所有单例
    public void destroySingletons() {
        if (logger.isDebugEnabled()) {
            logger.debug("Destroying singletons in " + this);
        }
        synchronized (this.singletonObjects) {
            this.singletonsCurrentlyInDestruction = true;
        }

        String[] disposableBeanNames;
        synchronized (this.disposableBeans) {
            disposableBeanNames = StringUtils.toStringArray(this.disposableBeans.keySet());
        }
        for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
            destroySingleton(disposableBeanNames[i]);
        }
        // 清空相关数据
        this.containedBeanMap.clear();
        this.dependentBeanMap.clear();
        this.dependenciesForBeanMap.clear();

        synchronized (this.singletonObjects) {
            this.singletonObjects.clear();
            this.singletonFactories.clear();
            this.earlySingletonObjects.clear();
            this.registeredSingletons.clear();
            this.singletonsCurrentlyInDestruction = false;
        }
    }

    // 销毁某个单例类
    public void destroySingleton(String beanName) {
        // Remove a registered singleton of the given name, if any.
        removeSingleton(beanName);

        // Destroy the corresponding DisposableBean instance.
        DisposableBean disposableBean;
        synchronized (this.disposableBeans) {
            disposableBean = (DisposableBean) this.disposableBeans.remove(beanName);
        }
        destroyBean(beanName, disposableBean);
    }

    // 删除之前添加的各种依赖
    protected void destroyBean(String beanName, DisposableBean bean) {
        // Trigger destruction of dependent beans first...
        Set<String> dependencies = this.dependentBeanMap.remove(beanName);
        if (dependencies != null) {
            if (logger.isDebugEnabled()) {
                logger.debug("Retrieved dependent beans for bean '" + beanName + "': " + dependencies);
            }
            for (String dependentBeanName : dependencies) {
                destroySingleton(dependentBeanName);
            }
        }

        // Actually destroy the bean now...
        if (bean != null) {
            try {
                bean.destroy();
            }
            catch (Throwable ex) {
                logger.error("Destroy method on bean with name '" + beanName + "' threw an exception", ex);
            }
        }

        // Trigger destruction of contained beans...
        Set<String> containedBeans = this.containedBeanMap.remove(beanName);
        if (containedBeans != null) {
            for (String containedBeanName : containedBeans) {
                destroySingleton(containedBeanName);
            }
        }

        // Remove destroyed bean from other beans' dependencies.
        synchronized (this.dependentBeanMap) {
            for (Iterator<Map.Entry<String, Set<String>>> it = this.dependentBeanMap.entrySet().iterator(); it.hasNext();) {
                Map.Entry<String, Set<String>> entry = it.next();
                Set<String> dependenciesToClean = entry.getValue();
                dependenciesToClean.remove(beanName);
                if (dependenciesToClean.isEmpty()) {
                    it.remove();
                }
            }
        }

        // Remove destroyed bean's prepared dependency information.
        this.dependenciesForBeanMap.remove(beanName);
    }

    // 
    public final Object getSingletonMutex() {
        return this.singletonObjects;
    }

}

```

**AbstractBeanFactory**

```
   =
    public abstract class FactoryBeanRegistrySupport extends DefaultSingletonBeanRegistry {

    // 存放工厂生产的单例集（查询的结果暂存，像hibernate或mybatis的调用缓存思路一致）
    private final Map<String, Object> factoryBeanObjectCache = new ConcurrentHashMap<>(16);



    // 返回指定beanFactory工厂的类型
    /**
        public interface FactoryBean<T> {
        T getObject() throws Exception;
        boolean isSingleton();      
        }
    */
    protected Class<?> getTypeForFactoryBean(final FactoryBean<?> factoryBean) {
        try {
            // 安全相关检查 我不去了解了
            if (System.getSecurityManager() != null) {
                return AccessController.doPrivileged((PrivilegedAction<Class<?>>) () ->
                        factoryBean.getObjectType(), getAccessControlContext());
            }
            else {
                return factoryBean.getObjectType();
            }
        }
        catch (Throwable ex) {
            // Thrown from the FactoryBean's getObjectType implementation.
            logger.warn("FactoryBean threw exception from getObjectType, despite the contract saying " +
                    "that it should return null if the type of its object cannot be determined yet", ex);
            return null;
        }
    }

    // 根据工厂名返回其生产的对象
    @Nullable
    protected Object getCachedObjectForFactoryBean(String beanName) {
        return this.factoryBeanObjectCache.get(beanName);
    }

    // 从Factory取对象 第三个参数 是否触发Post处理
    protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
        if (factory.isSingleton() && containsSingleton(beanName)) {
            synchronized (getSingletonMutex()) {
                Object object = this.factoryBeanObjectCache.get(beanName);
                if (object == null) {
                    // 从BeanFactory调用方法获取bean
                    object = doGetObjectFromFactoryBean(factory, beanName);
                    Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
                    if (alreadyThere != null) {
                        object = alreadyThere;
                    }
                    else {
                        if (shouldPostProcess) {
                            try {
                                object = postProcessObjectFromFactoryBean(object, beanName);
                            }
                            catch (Throwable ex) {
                                throw new BeanCreationException(beanName,
                                        "Post-processing of FactoryBean's singleton object failed", ex);
                            }
                        }
                        // 将对象放入缓存单例对象
                        this.factoryBeanObjectCache.put(beanName, object);
                    }
                }
                return object;
            }
        }
        else {
            Object object = doGetObjectFromFactoryBean(factory, beanName);
            if (shouldPostProcess) {
                try {
                    object = postProcessObjectFromFactoryBean(object, beanName);
                }
                catch (Throwable ex) {
                    throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
                }
            }
            return object;
        }
    }

    // 从单例工厂获取bean
    private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
            throws BeanCreationException {

        Object object;
        try {
            // 安全判断
            if (System.getSecurityManager() != null) {
                AccessControlContext acc = getAccessControlContext();
                try {
                    object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () ->
                            factory.getObject(), acc);
                }
                catch (PrivilegedActionException pae) {
                    throw pae.getException();
                }
            }
            else {
                object = factory.getObject();
            }
        }
        catch (FactoryBeanNotInitializedException ex) {
            throw new BeanCurrentlyInCreationException(beanName, ex.toString());
        }
        catch (Throwable ex) {
            throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", ex);
        }

        if (object == null) {
            if (isSingletonCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(
                        beanName, "FactoryBean which is currently in creation returned null from getObject");
            }
            object = new NullBean();
        }
        return object;
    }

    // 交给子类处理
    protected Object postProcessObjectFromFactoryBean(Object object, String beanName) throws BeansException {
        return object;
    }

    // beanInstance是BeanFactory类型？是返回
    protected FactoryBean<?> getFactoryBean(String beanName, Object beanInstance) throws BeansException {
        if (!(beanInstance instanceof FactoryBean)) {
            throw new BeanCreationException(beanName,
                    "Bean instance of type [" + beanInstance.getClass() + "] is not a FactoryBean");
        }
        return (FactoryBean<?>) beanInstance;
    }

    // 移除 beanName
    @Override
    protected void removeSingleton(String beanName) {
        super.removeSingleton(beanName);
        this.factoryBeanObjectCache.remove(beanName);
    }

    // 销毁 所有
    @Override
    public void destroySingletons() {
        super.destroySingletons();
        this.factoryBeanObjectCache.clear();
    }

    protected AccessControlContext getAccessControlContext() {
        return AccessController.getContext();
    }
}

```

**AbstractBeanFactory**

```
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport
    implements ConfigurableBeanFactory {

// 父工厂的引用
private BeanFactory parentBeanFactory;

// 类加载器
private ClassLoader beanClassLoader = ClassUtils.getDefaultClassLoader();

// 临时类加载器
private ClassLoader tempClassLoader;

// 是否缓存加载bean
private boolean cacheBeanMetadata = true;

// bean的表达形式
private BeanExpressionResolver beanExpressionResolver;

// 转换服务，用来替代属性编辑器的
private ConversionService conversionService;

// 属性编辑登记员集合，容量为4的LinkedHashSet
private final Set<PropertyEditorRegistrar> propertyEditorRegistrars = new LinkedHashSet<PropertyEditorRegistrar>(
        4);

// 类型转换器
private TypeConverter typeConverter;

// 默认的属性编辑器集合
private final Map<Class<?>, Class<? extends PropertyEditor>> customEditors = new HashMap<Class<?>, Class<? extends PropertyEditor>>(
        4);

// 嵌入值转换器集合
private final List<StringValueResolver> embeddedValueResolvers = new LinkedList<StringValueResolver>();

// bean后置处理器集合
private final List<BeanPostProcessor> beanPostProcessors = new ArrayList<BeanPostProcessor>();

// 标记是否有处理器被注册
private boolean hasInstantiationAwareBeanPostProcessors;

// 标记是否处理器被销毁
private boolean hasDestructionAwareBeanPostProcessors;

// 范围标识符和Scope实例的对应的Map
private final Map<String, Scope> scopes = new HashMap<String, Scope>(8);

// 安全相关(不看了)
private SecurityContextProvider securityContextProvider;

// 合并后的Bean根定义的集合
private final Map<String, RootBeanDefinition> mergedBeanDefinitions = new ConcurrentHashMap<String, RootBeanDefinition>(
        64);

// 被创建过得bean集合
private final Map<String, Boolean> alreadyCreated = new ConcurrentHashMap<String, Boolean>(
        64);

// 当前正在创建的原型，当前线程相关
private final ThreadLocal<Object> prototypesCurrentlyInCreation = new NamedThreadLocal<Object>(
        "Prototype beans currently in creation");


public AbstractBeanFactory() {
}


public AbstractBeanFactory(BeanFactory parentBeanFactory) {
    this.parentBeanFactory = parentBeanFactory;
}

// 获取bean
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}

// 获取bean
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
    return doGetBean(name, requiredType, null, false);
}

// 获取bean
public Object getBean(String name, Object... args) throws BeansException {
    return doGetBean(name, null, args, false);
}

// 提供创建时需要参数列表的getBean
public <T> T getBean(String name, Class<T> requiredType, Object... args)
        throws BeansException {
    return doGetBean(name, requiredType, args, false);
}

// 从容器中获取bean的基本方法。
@SuppressWarnings("unchecked")
protected <T> T doGetBean(final String name, final Class<T> requiredType,
        final Object[] args, boolean typeCheckOnly) throws BeansException {
    // 获取转化后的beanName
    final String beanName = transformedBeanName(name);
    Object bean;

    // 尝试从单例集合取这个bean
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        // 允许degbug输出日志
        if (logger.isDebugEnabled()) {
            if (isSingletonCurrentlyInCreation(beanName)) {
                logger.debug("Returning eagerly cached instance of singleton bean '"
                        + beanName
                        + "' that is not fully initialized yet - a consequence of a circular reference");
            }
            else {
                logger.debug("Returning cached instance of singleton bean '"
                        + beanName + "'");
            }
        }
        //// 根据给定的实例是否为工厂Bean，返回它自己或工厂Bean创建的实例
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }

    else {
        if (isPrototypeCurrentlyInCreation(beanName)) {// 如果正在被创建，就抛出异常
            throw new BeanCurrentlyInCreationException(beanName);
        }

        BeanFactory parentBeanFactory = getParentBeanFactory();// 取本容器的父容器
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {// 若存在父容器，且本容器不存在对应的Bean定义
            String nameToLookup = originalBeanName(name);// 取原始的Bean名
            if (args != null) {// 若参数列表存在
                // 那么用父容器根据原始Bean名和参数列表返回
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
            else {
                // 参数列表不要求，那就直接根据原始名称和要求的类型返回
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
        }

        // 如果不需要类型检查，标记其已经被创建
        if (!typeCheckOnly) {
            markBeanAsCreated(beanName);
        }

        // 根据beanName取其根Bean定义
        final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
        checkMergedBeanDefinition(mbd, beanName, args);

        String[] dependsOn = mbd.getDependsOn();// 得到这个根定义的所有依赖
        if (dependsOn != null) {
            for (String dependsOnBean : dependsOn) {
                getBean(dependsOnBean);// 注册这个Bean
                // 注册一个Bean和依赖于它的Bean（后参数依赖前参数）
                registerDependentBean(dependsOnBean, beanName);
            }
        }

        // 如果Bean定义是单例，就在返回单例
        if (mbd.isSingleton()) {
            sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {

                public Object getObject() throws BeansException {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {
                        destroySingleton(beanName);
                        throw ex;
                    }
                }
            });
            // 根据给定的实例是否为工厂Bean，返回它自己或工厂Bean创建的实例
            bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
        }
        // 如果是原型
        else if (mbd.isPrototype()) {
            // It's a prototype -> create a new instance.
            Object prototypeInstance = null;
            try {
                beforePrototypeCreation(beanName);// 原型创建前，与当前线程绑定
                prototypeInstance = createBean(beanName, mbd, args);
            }
            finally {
                afterPrototypeCreation(beanName);// 原型创建后，与当前线程解除绑定
            }
            bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
        }

        else {// 既不是单例又不是原型的情况
            String scopeName = mbd.getScope();
            final Scope scope = this.scopes.get(scopeName);// 得到范围
            if (scope == null) {
                throw new IllegalStateException(
                        "No Scope registered for scope '" + scopeName + "'");
            }
            try {// 根据范围创建实例
                Object scopedInstance = scope.get(beanName,
                        new ObjectFactory<Object>() {

                            public Object getObject() throws BeansException {
                                beforePrototypeCreation(beanName);
                                try {
                                    return createBean(beanName, mbd, args);// 原型创建前，与当前线程绑定
                                }
                                finally {
                                    //// 原型创建后，与当前线程解除绑定
                                    afterPrototypeCreation(beanName);
                                }
                            }
                        });
                // 根据给定的实例是否为工厂Bean，返回它自己或工厂Bean创建的实例
                bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
            }
            catch (IllegalStateException ex) {
                throw new BeanCreationException(beanName,
                        "Scope '" + scopeName
                                + "' is not active for the current thread; "
                                + "consider defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                        ex);
            }
        }
    }

    // 判断要求的类型是否和Bean实例的类型正在匹配
    if (requiredType != null && bean != null
            && !requiredType.isAssignableFrom(bean.getClass())) {
        try {
            return getTypeConverter().convertIfNecessary(bean, requiredType);// 转换类型，不抛出异常就说明类型匹配
        }
        catch (TypeMismatchException ex) {
            if (logger.isDebugEnabled()) {
                logger.debug(
                        "Failed to convert bean '" + name + "' to required type ["
                                + ClassUtils.getQualifiedName(requiredType) + "]",
                        ex);
            }
            throw new BeanNotOfRequiredTypeException(name, requiredType,
                    bean.getClass());
        }
    }
    return (T) bean;
}

// 判断本容器是否包含指定bean
public boolean containsBean(String name) {
    String beanName = transformedBeanName(name);
    // （如果是否包含单例 或 包含Bean定义）且 （为工厂Bean的产物 或 本身就是工厂bean），就返回true
    if (containsSingleton(beanName) || containsBeanDefinition(beanName)) {
        return (!BeanFactoryUtils.isFactoryDereference(name) || isFactoryBean(name));
    }
    // 如果不包含单例且不包含Bean定义，就从父类去查找
    BeanFactory parentBeanFactory = getParentBeanFactory();
    return (parentBeanFactory != null
            && parentBeanFactory.containsBean(originalBeanName(name)));
}

// 判断指定Bean是否为单例
public boolean isSingleton(String name) throws NoSuchBeanDefinitionException {
    String beanName = transformedBeanName(name);

    Object beanInstance = getSingleton(beanName, false);// 首先从单例集合中取
    if (beanInstance != null) {// 取不到，就判断它是不是FactoryBean的实例
        if (beanInstance instanceof FactoryBean) { // 如果是，要求它是工厂Bean产生的实例或这个工厂bean是单例
            return (BeanFactoryUtils.isFactoryDereference(name)
                    || ((FactoryBean<?>) beanInstance).isSingleton());
        }
        else {// 如果不是，要求它不是工厂Bean产生的实例
            return !BeanFactoryUtils.isFactoryDereference(name);
        }
    } // 若虽然取不到，但是单例集合中包含它的名字，说明它是单例
    else if (containsSingleton(beanName)) {
        return true;
    }

    else {
        // 从父工厂中去查询Bean定义
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            // 父工厂找不到Bean定义，那就在父工厂根据原始名去查是否为单例
            return parentBeanFactory.isSingleton(originalBeanName(name));
        }
        // 返回一个合并后的根Bean定义
        RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);

        // In case of FactoryBean, return singleton status of created object if not a
        // dereference.
        // 若该根定义是单例
        if (mbd.isSingleton()) {
            if (isFactoryBean(beanName, mbd)) { // 若该根定义为工厂Bean
                if (BeanFactoryUtils.isFactoryDereference(name)) {// 判断是否为工厂产生的实例
                    return true;
                }
                // 取对应的工厂，判断该工厂Bean是否为单例
                FactoryBean<?> factoryBean = (FactoryBean<?>) getBean(
                        FACTORY_BEAN_PREFIX + beanName);
                return factoryBean.isSingleton();
            }
            else { // 是否不为工厂Bean产生的实例（此时，即，该根定义不为工厂Bean，且不为工厂Bean产生的实例的时候，由于根定义是单例，那么它就是单例）
                return !BeanFactoryUtils.isFactoryDereference(name);
            }
        }
        else {
            return false;
        }
    }
}

// 判断是否为原型
public boolean isPrototype(String name) throws NoSuchBeanDefinitionException {
    String beanName = transformedBeanName(name);

    BeanFactory parentBeanFactory = getParentBeanFactory();// 得到父工厂
    // 若父工厂中的定义为原型，就为原型
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
        return parentBeanFactory.isPrototype(originalBeanName(name));
    }

    // 若合并后的根定义为原型，且不是工厂Bean产生的实例、或本身是工厂Bean，那么就为原型
    RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
    if (mbd.isPrototype()) {
        return (!BeanFactoryUtils.isFactoryDereference(name)
                || isFactoryBean(beanName, mbd));
    }
    else {
        if (BeanFactoryUtils.isFactoryDereference(name)) {// 若为工厂Bean产生的实例
            return false;
        }
        if (isFactoryBean(beanName, mbd)) {// 若为工厂Bean，取它产生的Bean，判断SmartFactoryBean
            final FactoryBean<?> factoryBean = (FactoryBean<?>) getBean(
                    FACTORY_BEAN_PREFIX + beanName);
            if (System.getSecurityManager() != null) {
                return AccessController.doPrivileged(new PrivilegedAction<Boolean>() {

                    public Boolean run() {
                        return ((factoryBean instanceof SmartFactoryBean
                                && ((SmartFactoryBean<?>) factoryBean).isPrototype())
                                || !factoryBean.isSingleton());
                    }
                }, getAccessControlContext());
            }
            else {
                return ((factoryBean instanceof SmartFactoryBean
                        && ((SmartFactoryBean<?>) factoryBean).isPrototype())
                        || !factoryBean.isSingleton());
            }
        }
        else {
            return false;
        }
    }
}

// 判断类型是否匹配
public boolean isTypeMatch(String name, Class<?> targetType)
        throws NoSuchBeanDefinitionException {
    String beanName = transformedBeanName(name);
    Class<?> typeToMatch = (targetType != null ? targetType : Object.class);

    Object beanInstance = getSingleton(beanName, false);// 取name对应的单例
    if (beanInstance != null) {
        if (beanInstance instanceof FactoryBean) {// 若为工厂Bean
            // 若不是工厂Bean产生的实例
            if (!BeanFactoryUtils.isFactoryDereference(name)) {
                // 取工厂Bean的类型与targetType进行对比
                Class<?> type = getTypeForFactoryBean((FactoryBean<?>) beanInstance);
                return (type != null && ClassUtils.isAssignable(typeToMatch, type));
            }
            else {
                return ClassUtils.isAssignableValue(typeToMatch, beanInstance);
            }
        }
        // 不是工厂Bean，那就直接判断
        else {
            return !BeanFactoryUtils.isFactoryDereference(name)
                    && ClassUtils.isAssignableValue(typeToMatch, beanInstance);
        }
    }
    // 单例表中，对应的Key没有值，也不包含Bean定义，说明没有注册，返回false
    else if (containsSingleton(beanName) && !containsBeanDefinition(beanName)) {
        return false;
    }
    // 以下是包含Bean定义的情况
    else {
        // 先查父类的Bean定义
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            // No bean definition found in this factory -> delegate to parent.
            return parentBeanFactory.isTypeMatch(originalBeanName(name), targetType);
        }

        // 直接查合并后的根定义
        RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);

        // 构建类型数组
        Class[] typesToMatch = (FactoryBean.class.equals(typeToMatch)
                ? new Class[] { typeToMatch }
                : new Class[] { FactoryBean.class, typeToMatch });

        // Check decorated bean definition, if any: We assume it'll be easier
        // to determine the decorated bean's type than the proxy's type.
        // 得到Bean定义的持有者
        BeanDefinitionHolder dbd = mbd.getDecoratedDefinition();
        if (dbd != null && !BeanFactoryUtils.isFactoryDereference(name)) {// 若为Bean工厂生成的实例，先得到根定义
            RootBeanDefinition tbd = getMergedBeanDefinition(dbd.getBeanName(),
                    dbd.getBeanDefinition(), mbd);
            Class<?> targetClass = predictBeanType(dbd.getBeanName(), tbd,
                    typesToMatch);// 得到预测的根定义
            if (targetClass != null
                    && !FactoryBean.class.isAssignableFrom(targetClass)) {
                return typeToMatch.isAssignableFrom(targetClass);
            }
        }

        Class<?> beanType = predictBeanType(beanName, mbd, typesToMatch);// 预测后的类型
        if (beanType == null) {
            return false;
        }

        if (FactoryBean.class.isAssignableFrom(beanType)) {// BeanFactory是否为其子类
            if (!BeanFactoryUtils.isFactoryDereference(name)) {// 若不为工厂Bean的产物
                // If it's a FactoryBean, we want to look at what it creates, not the
                // factory class.
                beanType = getTypeForFactoryBean(beanName, mbd);
                if (beanType == null) {
                    return false;
                }
            }
        }
        else if (BeanFactoryUtils.isFactoryDereference(name)) {// 若为工厂类Bean的产物
            beanType = predictBeanType(beanName, mbd, FactoryBean.class);// 预测类型
            if (beanType == null || !FactoryBean.class.isAssignableFrom(beanType)) {
                return false;
            }
        }

        return typeToMatch.isAssignableFrom(beanType);
    }
}

// 返回类型
public Class<?> getType(String name) throws NoSuchBeanDefinitionException {
    String beanName = transformedBeanName(name);

    // Check manually registered singletons.
    Object beanInstance = getSingleton(beanName, false);
    if (beanInstance != null) {
        if (beanInstance instanceof FactoryBean
                && !BeanFactoryUtils.isFactoryDereference(name)) {
            return getTypeForFactoryBean((FactoryBean<?>) beanInstance);
        }
        else {
            return beanInstance.getClass();
        }
    }
    else if (containsSingleton(beanName) && !containsBeanDefinition(beanName)) {
        // null instance registered
        return null;
    }

    else {
        // No singleton instance found -> check bean definition.
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            // No bean definition found in this factory -> delegate to parent.
            return parentBeanFactory.getType(originalBeanName(name));
        }

        RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);

        // Check decorated bean definition, if any: We assume it'll be easier
        // to determine the decorated bean's type than the proxy's type.
        BeanDefinitionHolder dbd = mbd.getDecoratedDefinition();
        if (dbd != null && !BeanFactoryUtils.isFactoryDereference(name)) {
            RootBeanDefinition tbd = getMergedBeanDefinition(dbd.getBeanName(),
                    dbd.getBeanDefinition(), mbd);
            Class<?> targetClass = predictBeanType(dbd.getBeanName(), tbd);
            if (targetClass != null
                    && !FactoryBean.class.isAssignableFrom(targetClass)) {
                return targetClass;
            }
        }

        Class<?> beanClass = predictBeanType(beanName, mbd);

        // Check bean class whether we're dealing with a FactoryBean.
        if (beanClass != null && FactoryBean.class.isAssignableFrom(beanClass)) {
            if (!BeanFactoryUtils.isFactoryDereference(name)) {
                // If it's a FactoryBean, we want to look at what it creates, not at
                // the factory class.
                return getTypeForFactoryBean(beanName, mbd);
            }
            else {
                return beanClass;
            }
        }
        else {
            return (!BeanFactoryUtils.isFactoryDereference(name) ? beanClass : null);
        }
    }
}

// 重写了，得到别名的方法。
@Override
public String[] getAliases(String name) {
    String beanName = transformedBeanName(name);
    List<String> aliases = new ArrayList<String>();
    boolean factoryPrefix = name.startsWith(FACTORY_BEAN_PREFIX);
    String fullBeanName = beanName;
    if (factoryPrefix) {
        fullBeanName = FACTORY_BEAN_PREFIX + beanName;
    }
    if (!fullBeanName.equals(name)) {
        aliases.add(fullBeanName);
    }
    String[] retrievedAliases = super.getAliases(beanName);
    for (String retrievedAlias : retrievedAliases) {
        String alias = (factoryPrefix ? FACTORY_BEAN_PREFIX : "") + retrievedAlias;
        if (!alias.equals(name)) {
            aliases.add(alias);
        }
    }
    if (!containsSingleton(beanName) && !containsBeanDefinition(beanName)) {
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null) {
            aliases.addAll(Arrays.asList(parentBeanFactory.getAliases(fullBeanName)));
        }
    }
    return StringUtils.toStringArray(aliases);
}

// ---------------------------------------------------------------------
// Implementation of HierarchicalBeanFactory interface
// ---------------------------------------------------------------------

// 返回本Bean工厂的父Bean工厂
public BeanFactory getParentBeanFactory() {
    return this.parentBeanFactory;
}

// 是否在本容器中（就是说，并不是工厂bean生产出来的）
public boolean containsLocalBean(String name) {
    String beanName = transformedBeanName(name); // 转换后的名字
    // （是否为单例或有对应的Bean定义） 且（不是工厂Bean生产出来的 或 本身就是工厂bean）
    return ((containsSingleton(beanName) || containsBeanDefinition(beanName))
            && (!BeanFactoryUtils.isFactoryDereference(name)
                    || isFactoryBean(beanName)));
}

// ---------------------------------------------------------------------
// Implementation of ConfigurableBeanFactory interface
// ---------------------------------------------------------------------

public void setParentBeanFactory(BeanFactory parentBeanFactory) {
    if (this.parentBeanFactory != null
            && this.parentBeanFactory != parentBeanFactory) {
        throw new IllegalStateException("Already associated with parent BeanFactory: "
                + this.parentBeanFactory);
    }
    this.parentBeanFactory = parentBeanFactory;
}

public void setBeanClassLoader(ClassLoader beanClassLoader) {
    this.beanClassLoader = (beanClassLoader != null ? beanClassLoader
            : ClassUtils.getDefaultClassLoader());
}

public ClassLoader getBeanClassLoader() {
    return this.beanClassLoader;
}

public void setTempClassLoader(ClassLoader tempClassLoader) {
    this.tempClassLoader = tempClassLoader;
}

public ClassLoader getTempClassLoader() {
    return this.tempClassLoader;
}

public void setCacheBeanMetadata(boolean cacheBeanMetadata) {
    this.cacheBeanMetadata = cacheBeanMetadata;
}

public boolean isCacheBeanMetadata() {
    return this.cacheBeanMetadata;
}

public void setBeanExpressionResolver(BeanExpressionResolver resolver) {
    this.beanExpressionResolver = resolver;
}

public BeanExpressionResolver getBeanExpressionResolver() {
    return this.beanExpressionResolver;
}

public void setConversionService(ConversionService conversionService) {
    this.conversionService = conversionService;
}

public ConversionService getConversionService() {
    return this.conversionService;
}

public void addPropertyEditorRegistrar(PropertyEditorRegistrar registrar) {
    Assert.notNull(registrar, "PropertyEditorRegistrar must not be null");
    this.propertyEditorRegistrars.add(registrar);
}

/**
 * Return the set of PropertyEditorRegistrars.
 */
public Set<PropertyEditorRegistrar> getPropertyEditorRegistrars() {
    return this.propertyEditorRegistrars;
}

public void registerCustomEditor(Class<?> requiredType,
        Class<? extends PropertyEditor> propertyEditorClass) {
    Assert.notNull(requiredType, "Required type must not be null");
    Assert.isAssignable(PropertyEditor.class, propertyEditorClass);
    this.customEditors.put(requiredType, propertyEditorClass);
}

public void copyRegisteredEditorsTo(PropertyEditorRegistry registry) {
    registerCustomEditors(registry);
}

/**
 * Return the map of custom editors, with Classes as keys and PropertyEditor classes
 * as values.
 */
public Map<Class<?>, Class<? extends PropertyEditor>> getCustomEditors() {
    return this.customEditors;
}

public void setTypeConverter(TypeConverter typeConverter) {
    this.typeConverter = typeConverter;
}

// 得到通用的类型转换器
protected TypeConverter getCustomTypeConverter() {
    return this.typeConverter;
}

// 得到类型转换器
public TypeConverter getTypeConverter() {
    TypeConverter customConverter = getCustomTypeConverter();
    if (customConverter != null) {
        return customConverter;
    }
    else {// 若本容器未注册类型转换器，就创建一个简单的类型转换器
        SimpleTypeConverter typeConverter = new SimpleTypeConverter();
        typeConverter.setConversionService(getConversionService());
        registerCustomEditors(typeConverter);
        return typeConverter;
    }
}

public void addEmbeddedValueResolver(StringValueResolver valueResolver) {
    Assert.notNull(valueResolver, "StringValueResolver must not be null");
    this.embeddedValueResolvers.add(valueResolver);
}

public String resolveEmbeddedValue(String value) {
    String result = value;
    for (StringValueResolver resolver : this.embeddedValueResolvers) {
        if (result == null) {
            return null;
        }
        result = resolver.resolveStringValue(result);
    }
    return result;
}

public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) {
    Assert.notNull(beanPostProcessor, "BeanPostProcessor must not be null");
    this.beanPostProcessors.remove(beanPostProcessor);
    this.beanPostProcessors.add(beanPostProcessor);
    if (beanPostProcessor instanceof InstantiationAwareBeanPostProcessor) {
        this.hasInstantiationAwareBeanPostProcessors = true;
    }
    if (beanPostProcessor instanceof DestructionAwareBeanPostProcessor) {
        this.hasDestructionAwareBeanPostProcessors = true;
    }
}

public int getBeanPostProcessorCount() {
    return this.beanPostProcessors.size();
}

/**
 * Return the list of BeanPostProcessors that will get applied to beans created with
 * this factory.
 */
public List<BeanPostProcessor> getBeanPostProcessors() {
    return this.beanPostProcessors;
}

/**
 * Return whether this factory holds a InstantiationAwareBeanPostProcessor that will
 * get applied to singleton beans on shutdown.
 * 
 * @see #addBeanPostProcessor
 * @see org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor
 */
protected boolean hasInstantiationAwareBeanPostProcessors() {
    return this.hasInstantiationAwareBeanPostProcessors;
}

/**
 * Return whether this factory holds a DestructionAwareBeanPostProcessor that will get
 * applied to singleton beans on shutdown.
 * 
 * @see #addBeanPostProcessor
 * @see org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor
 */
protected boolean hasDestructionAwareBeanPostProcessors() {
    return this.hasDestructionAwareBeanPostProcessors;
}

public void registerScope(String scopeName, Scope scope) {
    Assert.notNull(scopeName, "Scope identifier must not be null");
    Assert.notNull(scope, "Scope must not be null");
    if (SCOPE_SINGLETON.equals(scopeName) || SCOPE_PROTOTYPE.equals(scopeName)) {
        throw new IllegalArgumentException(
                "Cannot replace existing scopes 'singleton' and 'prototype'");
    }
    this.scopes.put(scopeName, scope);
}

public String[] getRegisteredScopeNames() {
    return StringUtils.toStringArray(this.scopes.keySet());
}

public Scope getRegisteredScope(String scopeName) {
    Assert.notNull(scopeName, "Scope identifier must not be null");
    return this.scopes.get(scopeName);
}

/**
 * Set the security context provider for this bean factory. If a security manager is
 * set, interaction with the user code will be executed using the privileged of the
 * provided security context.
 */
public void setSecurityContextProvider(SecurityContextProvider securityProvider) {
    this.securityContextProvider = securityProvider;
}

/**
 * Delegate the creation of the access control context to the
 * {@link #setSecurityContextProvider SecurityContextProvider}.
 */
@Override
public AccessControlContext getAccessControlContext() {
    return (this.securityContextProvider != null
            ? this.securityContextProvider.getAccessControlContext()
            : AccessController.getContext());
}

public void copyConfigurationFrom(ConfigurableBeanFactory otherFactory) {
    Assert.notNull(otherFactory, "BeanFactory must not be null");
    setBeanClassLoader(otherFactory.getBeanClassLoader());
    setCacheBeanMetadata(otherFactory.isCacheBeanMetadata());
    setBeanExpressionResolver(otherFactory.getBeanExpressionResolver());
    if (otherFactory instanceof AbstractBeanFactory) {
        AbstractBeanFactory otherAbstractFactory = (AbstractBeanFactory) otherFactory;
        this.customEditors.putAll(otherAbstractFactory.customEditors);
        this.propertyEditorRegistrars.addAll(
                otherAbstractFactory.propertyEditorRegistrars);
        this.beanPostProcessors.addAll(otherAbstractFactory.beanPostProcessors);
        this.hasInstantiationAwareBeanPostProcessors = this.hasInstantiationAwareBeanPostProcessors
                || otherAbstractFactory.hasInstantiationAwareBeanPostProcessors;
        this.hasDestructionAwareBeanPostProcessors = this.hasDestructionAwareBeanPostProcessors
                || otherAbstractFactory.hasDestructionAwareBeanPostProcessors;
        this.scopes.putAll(otherAbstractFactory.scopes);
        this.securityContextProvider = otherAbstractFactory.securityContextProvider;
    }
    else {
        setTypeConverter(otherFactory.getTypeConverter());
    }
}

// 返回合并后的bean定义（父Bean定义和子Bean定义合并）
public BeanDefinition getMergedBeanDefinition(String name) throws BeansException {
    String beanName = transformedBeanName(name);

    // Efficiently check whether bean definition exists in this factory.
    // 若Bean定义不存在，且本容器父工厂为ConfigurableBeanFactory的实例，让父工厂来调用这个方法
    if (!containsBeanDefinition(beanName)
            && getParentBeanFactory() instanceof ConfigurableBeanFactory) {
        return ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(
                beanName);
    }
    // 否则直接从本地合并后的Bean定义中取
    return getMergedLocalBeanDefinition(beanName);
}

public boolean isFactoryBean(String name) throws NoSuchBeanDefinitionException {
    String beanName = transformedBeanName(name);

    Object beanInstance = getSingleton(beanName, false);
    if (beanInstance != null) {
        return (beanInstance instanceof FactoryBean);
    }
    else if (containsSingleton(beanName)) {
        // null instance registered
        return false;
    }

    // No singleton instance found -> check bean definition.
    if (!containsBeanDefinition(beanName)
            && getParentBeanFactory() instanceof ConfigurableBeanFactory) {
        // No bean definition found in this factory -> delegate to parent.
        return ((ConfigurableBeanFactory) getParentBeanFactory()).isFactoryBean(name);
    }

    return isFactoryBean(beanName, getMergedLocalBeanDefinition(beanName));
}

@Override
public boolean isActuallyInCreation(String beanName) {
    return isSingletonCurrentlyInCreation(beanName)
            || isPrototypeCurrentlyInCreation(beanName);
}

// 判断指定的原型是否正在被创建
protected boolean isPrototypeCurrentlyInCreation(String beanName) {
    Object curVal = this.prototypesCurrentlyInCreation.get();
    return (curVal != null && (curVal.equals(beanName)
            || (curVal instanceof Set && ((Set<?>) curVal).contains(beanName))));
}

// 原型创建前回调，需要子类重写
@SuppressWarnings("unchecked")
protected void beforePrototypeCreation(String beanName) {
    Object curVal = this.prototypesCurrentlyInCreation.get();
    if (curVal == null) {// 原型创建状态与当前线程绑定
        this.prototypesCurrentlyInCreation.set(beanName);
    }
    else if (curVal instanceof String) {
        Set<String> beanNameSet = new HashSet<String>(2);
        beanNameSet.add((String) curVal);
        beanNameSet.add(beanName);
        this.prototypesCurrentlyInCreation.set(beanNameSet);
    }
    // 这里多余了。。。
    else {
        Set<String> beanNameSet = (Set<String>) curVal;
        beanNameSet.add(beanName);
    }
}

// 创建原型后，从当前线程解除绑定
@SuppressWarnings("unchecked")
protected void afterPrototypeCreation(String beanName) {
    Object curVal = this.prototypesCurrentlyInCreation.get();
    if (curVal instanceof String) {
        this.prototypesCurrentlyInCreation.remove();
    }
    else if (curVal instanceof Set) {
        Set<String> beanNameSet = (Set<String>) curVal;
        beanNameSet.remove(beanName);
        if (beanNameSet.isEmpty()) {
            this.prototypesCurrentlyInCreation.remove();
        }
    }
}

public void destroyBean(String beanName, Object beanInstance) {
    destroyBean(beanName, beanInstance, getMergedLocalBeanDefinition(beanName));
}

/**
 * Destroy the given bean instance (usually a prototype instance obtained from this
 * factory) according to the given bean definition.
 * 
 * @param beanName the name of the bean definition
 * @param beanInstance the bean instance to destroy
 * @param mbd the merged bean definition
 */
protected void destroyBean(String beanName, Object beanInstance,
        RootBeanDefinition mbd) {
    new DisposableBeanAdapter(beanInstance, beanName, mbd, getBeanPostProcessors(),
            getAccessControlContext()).destroy();
}

public void destroyScopedBean(String beanName) {
    RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
    if (mbd.isSingleton() || mbd.isPrototype()) {
        throw new IllegalArgumentException("Bean name '" + beanName
                + "' does not correspond to an object in a mutable scope");
    }
    String scopeName = mbd.getScope();
    Scope scope = this.scopes.get(scopeName);
    if (scope == null) {
        throw new IllegalStateException(
                "No Scope SPI registered for scope '" + scopeName + "'");
    }
    Object bean = scope.remove(beanName);
    if (bean != null) {
        destroyBean(beanName, bean, mbd);
    }
}

// ---------------------------------------------------------------------
// Implementation methods
// ---------------------------------------------------------------------

// 变换后的Bean名称（先去掉BeanFactory前缀，然后在aliasMap中取标准名）
protected String transformedBeanName(String name) {
    return canonicalName(BeanFactoryUtils.transformedBeanName(name));
}

// 返回原始的Bean名
protected String originalBeanName(String name) {
    String beanName = transformedBeanName(name);
    if (name.startsWith(FACTORY_BEAN_PREFIX)) {
        beanName = FACTORY_BEAN_PREFIX + beanName;
    }
    return beanName;
}

/**
 * Initialize the given BeanWrapper with the custom editors registered with this
 * factory. To be called for BeanWrappers that will create and populate bean
 * instances.
 * <p>
 * The default implementation delegates to {@link #registerCustomEditors}. Can be
 * overridden in subclasses.
 * 
 * @param bw the BeanWrapper to initialize
 */
protected void initBeanWrapper(BeanWrapper bw) {
    bw.setConversionService(getConversionService());
    registerCustomEditors(bw);
}

/**
 * Initialize the given PropertyEditorRegistry with the custom editors that have been
 * registered with this BeanFactory.
 * <p>
 * To be called for BeanWrappers that will create and populate bean instances, and for
 * SimpleTypeConverter used for constructor argument and factory method type
 * conversion.
 * 
 * @param registry the PropertyEditorRegistry to initialize
 */
protected void registerCustomEditors(PropertyEditorRegistry registry) {
    PropertyEditorRegistrySupport registrySupport = (registry instanceof PropertyEditorRegistrySupport
            ? (PropertyEditorRegistrySupport) registry : null);
    if (registrySupport != null) {
        registrySupport.useConfigValueEditors();
    }
    if (!this.propertyEditorRegistrars.isEmpty()) {
        for (PropertyEditorRegistrar registrar : this.propertyEditorRegistrars) {
            try {
                registrar.registerCustomEditors(registry);
            }
            catch (BeanCreationException ex) {
                Throwable rootCause = ex.getMostSpecificCause();
                if (rootCause instanceof BeanCurrentlyInCreationException) {
                    BeanCreationException bce = (BeanCreationException) rootCause;
                    if (isCurrentlyInCreation(bce.getBeanName())) {
                        if (logger.isDebugEnabled()) {
                            logger.debug("PropertyEditorRegistrar ["
                                    + registrar.getClass().getName()
                                    + "] failed because it tried to obtain currently created bean '"
                                    + ex.getBeanName() + "': " + ex.getMessage());
                        }
                        onSuppressedException(ex);
                        continue;
                    }
                }
                throw ex;
            }
        }
    }
    if (!this.customEditors.isEmpty()) {
        for (Map.Entry<Class<?>, Class<? extends PropertyEditor>> entry : this.customEditors.entrySet()) {
            Class<?> requiredType = entry.getKey();
            Class<? extends PropertyEditor> editorClass = entry.getValue();
            registry.registerCustomEditor(requiredType,
                    BeanUtils.instantiateClass(editorClass));
        }
    }
}

// 返回一个合并后的根Bean定义（父Bean定义和子Bean定义合并）（从当前容器取）
protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName)
        throws BeansException {
    // Quick check on the concurrent map first, with minimal locking.
    RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName);// 首先直接从合并根定义集合中取
    if (mbd != null) {
        return mbd;
    }
    // 根据bean名和其对应的Bean定义，取其根Bean根定义
    return getMergedBeanDefinition(beanName, getBeanDefinition(beanName));
}

// 根据Bean名和Bean定义取其Bean根定义
protected RootBeanDefinition getMergedBeanDefinition(String beanName,
        BeanDefinition bd) throws BeanDefinitionStoreException {

    return getMergedBeanDefinition(beanName, bd, null);// 调用重载方法
}

// 根据Bean名称返回根定义（若给定的Bean定义为子Bean定义，那么合并它的父Bean定义）
protected RootBeanDefinition getMergedBeanDefinition(String beanName,
        BeanDefinition bd, BeanDefinition containingBd)
        throws BeanDefinitionStoreException {

    synchronized (this.mergedBeanDefinitions) {
        RootBeanDefinition mbd = null;

        // 若给定的Bean定义并没有包含子Bean定义，那么直接根据Bean名取根定义
        if (containingBd == null) {
            mbd = this.mergedBeanDefinitions.get(beanName);
        }

        if (mbd == null) {// 若取不到
            if (bd.getParentName() == null) {// 若Bean定义没有父类，就很简单了
                if (bd instanceof RootBeanDefinition) {// 若Bean定义是RootBeanDefinition的实例，克隆、强转后返回
                    mbd = ((RootBeanDefinition) bd).cloneBeanDefinition();
                }
                else {// 否则，根据Bean定义，来构造一个根Bean定义
                    mbd = new RootBeanDefinition(bd);
                }
            }
            else {// 若Bean定义有父类
                    // Child bean definition: needs to be merged with parent.
                BeanDefinition pbd;
                try {
                    String parentBeanName = transformedBeanName(bd.getParentName());// 取其父Bean定义的名字
                    if (!beanName.equals(parentBeanName)) {// 若Bean名字并不是bd的父Bean的名字
                        pbd = getMergedBeanDefinition(parentBeanName);// 根据父Bean定义名称来返回合并后的bean定义
                    }
                    else {// 如果beanName对应的Bean就是bd的父Bean
                        if (getParentBeanFactory() instanceof ConfigurableBeanFactory) {// 若父Bean工厂为ConfigurableBeanFactory的实例
                            // 那么强转成ConfigurableBeanFactory后再调用合并方法
                            pbd = ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(
                                    parentBeanName);
                        }
                        else {// 若父Bean工厂不是ConfigurableBeanFactory的实例，就抛出异常
                            throw new NoSuchBeanDefinitionException(
                                    bd.getParentName(),
                                    "Parent name '" + bd.getParentName()
                                            + "' is equal to bean name '" + beanName
                                            + "': cannot be resolved without an AbstractBeanFactory parent");
                        }
                    }
                }
                catch (NoSuchBeanDefinitionException ex) {
                    throw new BeanDefinitionStoreException(
                            bd.getResourceDescription(), beanName,
                            "Could not resolve parent bean definition '"
                                    + bd.getParentName() + "'",
                            ex);
                }
                // 深度复制
                mbd = new RootBeanDefinition(pbd);// 根据Bean定义生成一个根Bean定义
                mbd.overrideFrom(bd);// 将Bean定义的属性复制进自己的定义（根Bean定义）中
            }

            if (!StringUtils.hasLength(mbd.getScope())) {// 如果根Bean定义未设置范围
                mbd.setScope(RootBeanDefinition.SCOPE_SINGLETON);// 那么设置其范围为单例
            }

            // 若本根Bean定义包含Bean定义、本根Bean定义为单例且包含的Bean定义并不是单例
            if (containingBd != null && !containingBd.isSingleton()
                    && mbd.isSingleton()) {
                mbd.setScope(containingBd.getScope());// 那么将本根Bean定义的范围设置为包含的Bean定义的范围
            }

            // 若本根Bean定义不包含Bean定义，且是缓存Bean元数据（重写前均为true）且Bean定义是否有资格缓存（默认实现是，这个Bean已经创建便有资格）
            if (containingBd == null && isCacheBeanMetadata()
                    && isBeanEligibleForMetadataCaching(beanName)) {
                this.mergedBeanDefinitions.put(beanName, mbd);// 放进mergedBeanDefinitions中
            }
        }

        return mbd;
    }
}

// 检查Bean定义，抛出异常
protected void checkMergedBeanDefinition(RootBeanDefinition mbd, String beanName,
        Object[] args) throws BeanDefinitionStoreException {

    if (mbd.isAbstract()) {
        throw new BeanIsAbstractException(beanName);
    }

    if (args != null && !mbd.isPrototype()) {
        throw new BeanDefinitionStoreException(
                "Can only specify arguments for the getBean method when referring to a prototype bean definition");
    }
}

/**
 * Remove the merged bean definition for the specified bean, recreating it on next
 * access.
 * 
 * @param beanName the bean name to clear the merged definition for
 */
protected void clearMergedBeanDefinition(String beanName) {
    this.mergedBeanDefinitions.remove(beanName);
}

// 解析类型，处理异常
protected Class<?> resolveBeanClass(final RootBeanDefinition mbd, String beanName,
        final Class<?>... typesToMatch) throws CannotLoadBeanClassException {
    try {
        if (mbd.hasBeanClass()) {
            return mbd.getBeanClass();
        }
        if (System.getSecurityManager() != null) {
            return AccessController.doPrivileged(
                    new PrivilegedExceptionAction<Class<?>>() {

                        public Class<?> run() throws Exception {
                            return doResolveBeanClass(mbd, typesToMatch);
                        }
                    }, getAccessControlContext());
        }
        else {
            return doResolveBeanClass(mbd, typesToMatch);
        }
    }
    catch (PrivilegedActionException pae) {
        ClassNotFoundException ex = (ClassNotFoundException) pae.getException();
        throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName,
                mbd.getBeanClassName(), ex);
    }
    catch (ClassNotFoundException ex) {
        throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName,
                mbd.getBeanClassName(), ex);
    }
    catch (LinkageError err) {
        throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName,
                mbd.getBeanClassName(), err);
    }
}

// 真正的解析类型
private Class<?> doResolveBeanClass(RootBeanDefinition mbd, Class<?>... typesToMatch)
        throws ClassNotFoundException {
    if (!ObjectUtils.isEmpty(typesToMatch)) {
        ClassLoader tempClassLoader = getTempClassLoader();// 找到临时的类加载器
        if (tempClassLoader != null) {
            if (tempClassLoader instanceof DecoratingClassLoader) {// 若为装饰类加载器
                DecoratingClassLoader dcl = (DecoratingClassLoader) tempClassLoader;
                for (Class<?> typeToMatch : typesToMatch) {
                    dcl.excludeClass(typeToMatch.getName());
                }
            }
            String className = mbd.getBeanClassName();
            return (className != null ? ClassUtils.forName(className, tempClassLoader)
                    : null);
        }
    }
    return mbd.resolveBeanClass(getBeanClassLoader());
}

/**
 * Evaluate the given String as contained in a bean definition, potentially resolving
 * it as an expression.
 * 
 * @param value the value to check
 * @param beanDefinition the bean definition that the value comes from
 * @return the resolved value
 * @see #setBeanExpressionResolver
 */
protected Object evaluateBeanDefinitionString(String value,
        BeanDefinition beanDefinition) {
    if (this.beanExpressionResolver == null) {
        return value;
    }
    Scope scope = (beanDefinition != null
            ? getRegisteredScope(beanDefinition.getScope()) : null);
    return this.beanExpressionResolver.evaluate(value,
            new BeanExpressionContext(this, scope));
}

// 预测类型
protected Class<?> predictBeanType(String beanName, RootBeanDefinition mbd,
        Class<?>... typesToMatch) {
    // 若根Bena定义的工厂方法名存在，说明它是工厂Bean创建的，无法预测类型？
    if (mbd.getFactoryMethodName() != null) {
        return null;
    }
    // 否则，解析Bean的Class
    return resolveBeanClass(mbd, beanName, typesToMatch);
}

/**
 * Check whether the given bean is defined as a {@link FactoryBean}.
 * 
 * @param beanName the name of the bean
 * @param mbd the corresponding bean definition
 */
protected boolean isFactoryBean(String beanName, RootBeanDefinition mbd) {
    Class<?> beanType = predictBeanType(beanName, mbd, FactoryBean.class);
    return (beanType != null && FactoryBean.class.isAssignableFrom(beanType));
}

// 返回工厂Bean的类型
protected Class<?> getTypeForFactoryBean(String beanName, RootBeanDefinition mbd) {
    if (!mbd.isSingleton()) {
        return null;
    }
    try {
        FactoryBean<?> factoryBean = doGetBean(FACTORY_BEAN_PREFIX + beanName,
                FactoryBean.class, null, true);
        return getTypeForFactoryBean(factoryBean);
    }
    catch (BeanCreationException ex) {
        // Can only happen when getting a FactoryBean.
        if (logger.isDebugEnabled()) {
            logger.debug(
                    "Ignoring bean creation exception on FactoryBean type check: "
                            + ex);
        }
        onSuppressedException(ex);
        return null;
    }
}

// 标记这个Bean已经被创建
protected void markBeanAsCreated(String beanName) {
    this.alreadyCreated.put(beanName, Boolean.TRUE);
}

/**
 * Determine whether the specified bean is eligible for having its bean definition
 * metadata cached.
 * 
 * @param beanName the name of the bean
 * @return {@code true} if the bean's metadata may be cached at this point already
 */
// 若本根Bean定义包含Bean元定义作为缓存，这个方法应被之类覆盖，这里仅判断Bean是否已经被创建
protected boolean isBeanEligibleForMetadataCaching(String beanName) {
    return this.alreadyCreated.containsKey(beanName);
}

/**
 * Remove the singleton instance (if any) for the given bean name, but only if it
 * hasn't been used for other purposes than type checking.
 * 
 * @param beanName the name of the bean
 * @return {@code true} if actually removed, {@code false} otherwise
 */
protected boolean removeSingletonIfCreatedForTypeCheckOnly(String beanName) {
    if (!this.alreadyCreated.containsKey(beanName)) {
        removeSingleton(beanName);
        return true;
    }
    else {
        return false;
    }
}

// 返回它自己或工厂Bean创建的实例
protected Object getObjectForBeanInstance(Object beanInstance, String name,
        String beanName, RootBeanDefinition mbd) {

    // 如果这个Bean是工厂Bean创建的 且 这个Bean实例并不是FactoryBean实例，抛异常
    if (BeanFactoryUtils.isFactoryDereference(name)
            && !(beanInstance instanceof FactoryBean)) {
        throw new BeanIsNotAFactoryException(transformedBeanName(name),
                beanInstance.getClass());
    }

    // 如果这个Bean实例并不是FactoryBean实例 或 这个Bean是工厂Bean创建的，直接返回
    if (!(beanInstance instanceof FactoryBean)
            || BeanFactoryUtils.isFactoryDereference(name)) {
        return beanInstance;
    }

    // ——————————以下都是 这个Bean实例是FactoryBean实例的情况
    Object object = null;
    if (mbd == null) {// 若根Bean定义为空，取这个BeanFactory所生产的实例
        object = getCachedObjectForFactoryBean(beanName);
    }
    if (object == null) {// 若取不到，那么手动取
        FactoryBean<?> factory = (FactoryBean<?>) beanInstance;// 把这个实例转化成一个FactoryBean
        // Caches object obtained from FactoryBean if it is a singleton.
        if (mbd == null && containsBeanDefinition(beanName)) {// 若根Bean定义为空，但是容器内有Bean定义
            mbd = getMergedLocalBeanDefinition(beanName);// 返回合并后的Bean定义
        }
        boolean synthetic = (mbd != null && mbd.isSynthetic());// 标记这个Bean定义是合并的
        object = getObjectFromFactoryBean(factory, beanName, !synthetic);// 从工厂Bean中取
    }
    return object;
}

// 判断给定的Bean是否被使用过
public boolean isBeanNameInUse(String beanName) {
    // 若是别名 或 并非工厂bean生产出来的 或 被其他某个bean所依赖，那么判断其被使用过
    return isAlias(beanName) || containsLocalBean(beanName)
            || hasDependentBean(beanName);
}

/**
 * Determine whether the given bean requires destruction on shutdown.
 * <p>
 * The default implementation checks the DisposableBean interface as well as a
 * specified destroy method and registered DestructionAwareBeanPostProcessors.
 * 
 * @param bean the bean instance to check
 * @param mbd the corresponding bean definition
 * @see org.springframework.beans.factory.DisposableBean
 * @see AbstractBeanDefinition#getDestroyMethodName()
 * @see org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor
 */
protected boolean requiresDestruction(Object bean, RootBeanDefinition mbd) {
    return (bean != null && (DisposableBeanAdapter.hasDestroyMethod(bean, mbd)
            || hasDestructionAwareBeanPostProcessors()));
}

/**
 * Add the given bean to the list of disposable beans in this factory, registering its
 * DisposableBean interface and/or the given destroy method to be called on factory
 * shutdown (if applicable). Only applies to singletons.
 * 
 * @param beanName the name of the bean
 * @param bean the bean instance
 * @param mbd the bean definition for the bean
 * @see RootBeanDefinition#isSingleton
 * @see RootBeanDefinition#getDependsOn
 * @see #registerDisposableBean
 * @see #registerDependentBean
 */
protected void registerDisposableBeanIfNecessary(String beanName, Object bean,
        RootBeanDefinition mbd) {
    AccessControlContext acc = (System.getSecurityManager() != null
            ? getAccessControlContext() : null);
    if (!mbd.isPrototype() && requiresDestruction(bean, mbd)) {
        if (mbd.isSingleton()) {
            // Register a DisposableBean implementation that performs all destruction
            // work for the given bean: DestructionAwareBeanPostProcessors,
            // DisposableBean interface, custom destroy method.
            registerDisposableBean(beanName, new DisposableBeanAdapter(bean, beanName,
                    mbd, getBeanPostProcessors(), acc));
        }
        else {
            // A bean with a custom scope...
            Scope scope = this.scopes.get(mbd.getScope());
            if (scope == null) {
                throw new IllegalStateException(
                        "No Scope registered for scope '" + mbd.getScope() + "'");
            }
            scope.registerDestructionCallback(beanName, new DisposableBeanAdapter(
                    bean, beanName, mbd, getBeanPostProcessors(), acc));
        }
    }
}

// ---------------------------------------------------------------------
// Abstract methods to be implemented by subclasses
// ---------------------------------------------------------------------

// 标记是否包含Bean定义的方法
protected abstract boolean containsBeanDefinition(String beanName);

// 根据Bean名返回其BeanDefinition
protected abstract BeanDefinition getBeanDefinition(String beanName)
        throws BeansException;

// 根据指定的bean定义和bean名、参数，创建对象
protected abstract Object createBean(String beanName, RootBeanDefinition mbd,
        Object[] args) throws BeanCreationException;

}
```





