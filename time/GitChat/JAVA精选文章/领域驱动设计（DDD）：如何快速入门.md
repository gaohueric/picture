### 一、背景与问题

无论是企业内部系统还是互联网产品，多年来开发这种基于业务与数据库的系统都是 IT 领域一个重要的内容。作为一个 IT 开发团队，无论是做外包还是自己的产品，都面临从产品经理拿到需求，然后需要进行两方面重要的工作：一是把需求转换为设计、二是从设计开始编写代码。

以前我们开发这种基于业务的软件产品，通常都是基于开发人员自己的经验和编写代码的习惯开始系统的设计与代码的编写。常见的方式是项目经理或架构师分析需求后，开始进行数据库表结构的设计，然后按照自己熟悉的分层架构，比如四层架构（数据访问层-业务逻辑层-WebApi-前端）进行代码组织。从上述的实现可以看出几个问题：

1. 没有描述需求的设计模型；而是直接通过数据库表的方式体现，也就是需求与设计是脱节的。
2. 编码的架构也没有与设计和需求对应起来。
3. 业务逻辑与技术混在一起；业务逻辑可能直接调用的数据访问，这样把业务逻辑与数据访问的技术混在一起。
4. 开发没有层次感和节奏感；系统没有一个统一的约束，开发人员没有一个统一的节奏，这主要体现在随意的编码。
5. Bug 定位困难：当系统出现业务异常行为时，无法快速准确的定位出现问题的位置，因为系统不同开发人员的代码放置随意性。
6. 需求变更响应缓慢：在大型的系统或产品中，当需要增加功能或修改现有功能时，因为代码架构的随意性，可能会出现改了功能可能会影响到其他的功能，造成系统极不稳定。 大家知道，万事万物都有其内在规律，这就是套路，软件设计与开发也是如此。软件设计与开发的套路就是领域驱动设计（简称 DDD）。DDD 这个套路能够灵活解决以上的问题，为我们提供一个好的设计、架构以及高质量的代码。

### 二、为什么要学习 DDD

DDD 是软件行业的一种成熟的方法论和模式。通过应用 DDD，我们能够很好的将需求应对到设计，能够让开发聚焦业务本身而不是技术，能够让代码体现我们设计，能够让团队在一个框架内有节奏的开发。

#### 1. DDD 核心概念

首先给大家描述一下 DDD 的概念：DDD 是一种开发理念，核心是维护一个反应领域概念的模型（领域模型是软件最核心的部分，反应了软件的业务本质），然后通过大量模式来指导模型设计与开发。

从上述定义可以看出，DDD 方法首先是需要将需求分析后，形成一个反应需求的领域模型。领域模型就是大家平常理解的类、类的属性、类之间的关系等。当然在 DDD 中，为了更好的将领域模型反应需求，对类、类的属性、类之间的关系等有一些模式的指导。比如类的属性可能是一般属性，也可能是值对象；比如有关系的类之间是否是代表一个整体概念、有相同生命周期、需要统一持久化等。所以我们的领域模型除了能够跑通需求外，还要考虑聚合根、实体、值对象、聚合等概念的应用，这样领域模型的设计才能更好的反应需求，也能够更好的将设计对应成有约束力的代码。

另外 DDD 也提供了大量模式，告诉我们应该如何编写对应设计的代码，能够将我们的代码真正映射到设计；如何进行业务逻辑与持久化机制的剥离；如何进行更好的架构设计等。

#### 2. DDD 能应对复杂性

在软件设计与开发过程中，复杂性主要体现在三个方面。一是技术维度，有业务代码的实现、有与数据库或其他持久化存储交互的实现、有消息队列的实现、有身份验证与授权的实现、有 WebAPI 暴露的实现等；二是业务维度，有太多的模块和功能需要去做；三是时间维度，需要快速的开发，快速的响应需求的变更，快速的修正 Bug。那么 DDD 是如何解决上述三个维度的复杂性呢？

- **a. 技术维度：**通过合理的架构分层，能够让每层关注自己的事情，比如领域层只关注业务逻辑的事情，仓储实现层只关注持久化数据与查询的事情，应用服务层只关注协调领域层与仓储实现层完成用例的事情，接口层只关注暴露给前端的事情。
- **b. 业务维度：**通过将大系统划分成多个界限上下文，可以让不同团队和不同人只关注当前上下文的开发。在当前界限上下文中的领域层、仓储实现层、应用服务层、接口层都与其他界限上下文独立开来，这样可以专注开发，并且在修改代码与发布产品时，影响面较小。
- **c. 时间维度：**通过敏捷式迭代快速验证，快速修正。

### 三、什么人适合学习 DDD

从文章前面的部分大家可能有这样的初步感觉：DDD 是一套设计、架构和编码指导的方法，通过灵活运用 DDD，能够设计出很好反应产品需求的领域模型，另外通过更好的分层指导能够让我们的代码很好的反映设计，其实最终解决的问题就是我们第一个部分提出的 6 大问题。所以只要具有业务属性的产品或项目，在这个团队中的架构师、主要后端开发人员都是比较适合去学习 DDD 的。DDD 与语言无关，无论是 C#、Java 还是其他的语言，只要我们是遵循 DDD 套路的设计与编码架构，那就代表我们很好的实践了 DDD。当然为了你更好的入门 DDD，本文第五部分会有代码的演示。在代码演示部分使用了 C# 语言和 .net Core、EF Core、Unity、Asp.net Core WebAPI 等语言或基础框架，如果你是 Java 体系，也能够找到对应的框架实现的。再次强调，DDD 与语言和技术无关。

### 四、如何高效学习 DDD

其实要学习一种方法论和架构模式，就两个步骤：一是了解它核心的概念和组件，二是将这些概念和组件灵活的运用到产品开发中。

要高效的学习 DDD，首先我们要搞清楚两个方面的内容：一是核心的组件和概念，二是 DDD 分层架构。

#### **核心组件和概念**

1. **界限上下文：**首先要将大系统划分为多个界限上下文，比如一个经销商电商系统可以划分为产品、经销商、订单等几个界限上下文，每个界限上下文有自己的领域逻辑、数据持久化、用例、接口等。每个界限上下文根据特点，具体实现方式又不同，比如有些界限上下文基本没有业务逻辑，就是增删改查，则可以使用 CRUD 最简单的模式；有些界限上线文有一定的业务逻辑，但对高并发、高性能没要求，则可以使用经典 DDD 模式；有些界限上下文有一定的业务逻辑，而且有高性能要求，则可以使 CQRS 模式。 划分界限上下文并且在每个上下文实现自己的业务逻辑、持久化、用例和接口作用是巨大大，一是可以让不同开发小组专注与此界限上下文的开发，二是可以分别部署，三是如果一个界限上下文出现问题，并不影响其他界限上下文功能的使用。
2. **实体：**有业务生命周期的对象，采用业务标识符进行跟踪。比如一个订单就是实体，订单有生命周期的，而且有一个订单号唯一的标识它自己，如果两个订单所有属性值全部相同，但订单号不同，也是不同的实体。
3. **值对象：**无业务生命周期，无业务标识符，通常用于描述实体。比如订单的收货地址、订单支付的金额等就是值对象。值对象在数据库中表的表现形式可以是两种，一种是作为一个列或多个列与所属的实体对象所对应的表放在一起，另一种是单独一个表，通过ID与所属的实体对象关联。
4. **领域服务：**无状态，有行为，通常就是一个用例来协调多个领域逻辑完成功能。
5. **聚合：**通常将多个实体和值对象组合到一个聚合中来表达一个完整的概念，比如订单实体、订单明细实体、订单金额值对象就代表一个完整的订单概念，而且生命周期是相同的，并且需要统一持久化到数据库中。
6. **聚合根：**将聚合中表达总概念的实体做成聚合根，比如订单实体就是聚合根，对聚合中所有实体的状态变更必须经过聚合根，因为聚合根协调了整个聚合的逻辑，保证一致性。当然其他实体可以被外部直接临时查询调用。
7. **服务：**协调聚合之间的业务逻辑，并且完成用例。
8. **仓储：**用于对聚合进行持久化，通常为每个聚合根配备一个仓储即可。仓储能够很好的解耦领域逻辑与数据库。

#### **DDD 经典分层架构**

只有了解了经典 DDD 的架构，你才能知道具体在哪层要实现哪些功能，编写哪些代码，具体在开发支撑 DDD 的轻量级框架与具体模块代码实现时，才能做到有的放矢。

传统三层架构以及问题：![传统三层架构](http://images.gitbook.cn/6f7a4b60-62d2-11e8-b977-f33e31f528f0)

问题：

1. 过分注重数据访问层，而不重视领域。
2. 业务逻辑直接与数据访问层耦合，与领域为核心的 DDD 思想背道而驰。
3. 没有一系列的模式与方法论指导这种分层架构的开发约束。

经典 DDD 架构：![经典DDD架构](http://images.gitbook.cn/ed598000-62d2-11e8-b864-0bd1f4b74dfb)

**1. 基础结构层：整个产品或系统的底层支撑。**

- a. **常用工具、支撑功能：**这个 .net core 项目至少要实现以下的功能：Json 配置文件的读取、WebAPI 返回给前端的基本格式对象的定义、Json 序列化与反序列化、加密功能、依赖注入框架的二次封装等。
- b. **支持 DDD 框架：**这个 .net core 项目至少要实现以下的功能：聚合根接口定义、实体接口定义、值对象接口定义、仓储接口定义、仓储接口的 EF Core 顶层实现（工作单元模式）。
- c. **聚合根仓储实现：**这个 .net core 项目严格来讲其实不属于基础结构层部分，只是由于习惯，把它放到基础结构层这个解决方案文件夹中。它其实是引用了领域层的领域对象，并且从领域层对应的聚合根仓储接口中继承，然后实现领域对象持久化到数据库，这样，仓储实现是依赖领域对象，领域对象与领域逻辑就不需要依赖仓储。领域模型才是系统真正的核心。

**2. 领域层：界限上下文的领域逻辑**

- a. 首先要实现这个界限上下文的领域对象的 POCO 模型。
- b. 然后针对这个界限上下文的所有领域对象，建立每个领域对象自己的业务逻辑，注意的是，领域对象的业务逻辑最好不与仓储直接发生交互，就算领域逻辑要临时查询数据库也不要这样。
- c. 定义该界限上下文聚合根的仓储接口，这个接口代表的是聚合根与持久化打交道的基础约束，具体实现还是在基础结构层的聚合根仓储中实现，这样就实现了解耦。把聚合根仓储接口定义在领域层的意义是可以为领域层的调用方（应用服务层）的用例提供对聚合持久化支持。
- d. 定义该界限上下文的 EF Core 上下文接口并实现，这样就通过映射关系，EF Core 就可以处理领域对象与数据库表之间的映射了。

**3. 应用服务层：界限上下文的用例**

- a. 某个上下文的应用服务层的某个用例，通过调用领域对象的领域逻辑，完成相关领域逻辑的实现。
- b. 领域逻辑完成后，应用服务层用例调用领域层的聚合根的仓储接口的方法，完成领域对象的预持久化。（应用服务通过基础结构层的依赖注入框架与 Json 配置文件找到聚合根仓储接口对应的实现）
- c. 应用服务层用例然后调用基础结构层的 EF Core 仓储接口的工作单元方式，完成真正的持久化。（应用服务通过基础接口层的依赖注入框架与 Json 配置文件找到顶层仓储接口对应的工作单元实现）
- d. 用例返回给接口层需要的前端所需的 Json 对象格式。

**4. 接口层：非常薄的一层**

- a. 只需要调用应用服务层用例
- b. 向前端返回所需的json对象格式

### 五、DDD 如何落地

了解了 DDD 的好处、基本概念以及经典分层后，我们这里用一个简单的产品上架功能来演示 DDD 是如何落地的。这里需要说明的是，因为是 DDD 入门文章，并没有涉及到复杂的业务逻辑，只是让大家看到如何落的概念和 DDD 经典架构的一个最简单的例子。 首先我们需要有一个支持 DDD 概念的一个初级版本的 DDD 轻量级框架来体现 DDD 一些基本思想。

#### 实体、聚合根与值对象的顶层体现

##### **1. 实体定义**

```
public interface IEntity
{
string Code { get; set; }
Guid Id { get; set; }
}

```

Id 是一个未来存储到数据库表中的技术主键，Code 是领域对象的唯一业务标识符。你也可以扩展这个接口，定义两个实体比较接口（未来实现就是比较两个实体如果 Code 一致，则代表两个实体相等）。

##### **2. 聚合根定义**

```
public interface IAggregationRoot:IEntity
{   }

```

聚合根接口就是从实体接口继承，只是未来的用法可以在仓储中定义持久化时的领域对象必须从这个接口或继承了这个接口的抽象类继承下来。

##### **3. 值对象定义**

```
public interface IValueObject
{
Guid Id { get; set; }
}

```

值对象接口只需要保留一个技术主键即可，它没有业务标识符。在数据库中，值对象可能作为单独表存储，也可以作为实体的一部分存储。你也可以扩展这个接口，定义两个值对象比较接口（未来实现就是比较两个值对象如果所有属性值一致，则代表两个值对象相等）。

##### **4. 工作单元定义**

```
public interface IUnitOfWork
{
void Commit();
}

```

工作单元接口就定义了一个提交方法，在具体实现时，其实就是对应的 EF Core 的整个聚合的事务提交方法，这样就保证了整个聚合中聚合根、所有实体和值对象的强一致性。

##### **5. 仓储接口定义**

```
public interface IRepository:IUnitOfWork,IDisposable
{  }

```

仓储接口从工作单元接口与资源释放接口继承，为未来的数据访问框架和可替换性提供顶层约束。

##### **6. EF Core 仓储持久化实现**

```
public class EFCoreRepository : IRepository
{
private readonly DbContext context;
public EFCoreRepository(DbContext context)
{
this.context = context;
}
public void Commit()
{
try
{
context.SaveChanges();
}
catch(Exception error)
{
throw error;
}
}
public void Dispose()
{
context.Dispose();
}
}

```

从上述代码中可以看到，主要实现了仓储接口的 Commit 方法，其实就是使用了 EF Core 的 DbContext 数据访问上下文类的 SaveChanges() 事务提交方法，应用服务层的用例就可以获取到某个聚合根的当前状态，然后调用仓储接口的 Commit 方法，实现了整个聚合所有对象的一次性事务提交。

##### **7. 其他支撑工具类的实现**

我们还应该实现一些配置文件的读取，返回给前端 Json 格式对象的一些定义等，这里不再累述这些内容。

#### 产品界限上下文落地实现

这部分主要讲产品上下文中的领域层的主体实现。

先简单讲下业务方面的需求：产品 SPU 与产品 SKU，产品 SPU 主要是产品的名字和相关描述，产品 SKU 包括产品 SPU 的多个规格，每个规格有不同的价格与 PV 值。从我们对 DDD 概念的理解，产品 SPU 与产品 SKU 属于同一个聚合，产品 SPU 是聚合根。![产品界限上下文领域模型](http://images.gitbook.cn/7d5504c0-62d5-11e8-b977-f33e31f528f0)

##### **领域层实现**

1.产品SKU领域对象POCO代码：

```
public partial class ProductSKU : IEntity
{
public ProductSKU() { }
[Key]
public Guid Id { get; set; }
public string Code { get; set; }
public string Spec { get; set; }
public Unit Unit { get; set; }
public decimal PV { get; set; }
public decimal DealerPrice { get; set; }
public byte[] Image { get; set; }
public Guid ProductSPUId { get; set; }
public string ProductSPUName { get; set; }
}

```

2.产品SPU领域对象POCO代码：

```
public partial class ProductSPU : IAggregationRoot
{
[Key]
public Guid Id { get; set; }
public string Code { get; set; }
public string ProductSPUName { get; set; }
public string ProductSPUDes { get; set; }
public List<ProductSKU> ProductSKUS { get; set; }   
}

```

从上面代码可以看到，ProductSPU 从聚合根接口继承，ProductSKU 从实体接口继承，ProductSPU 包含了一个 ProductSKU 的集合（也就是引用），这就代表它们同属一个聚合，在具体使用 EF Core 做持久化时，会作为一个事务统一持久化。领域对象除了包含自身的属性，也应该包括自身的业务逻辑，产品上架的功能比较简单，业务逻辑也比较简单，主要就是如何生成整个领域对象，以及聚合根与实体业务标识符 Code 的生成规则。

3.产品 SPU 领域对象业务逻辑代码：

```
 public partial class ProductSPU
{
public ProductSPU CreateProductSPU(Guid id,string spuname,string spudesc,List<ProductSKU> productskus)
{
this.Id = id;
//业务标识符的生成规则
this.Code = "Code " + spuname;
this.ProductSPUName = spuname;
this.ProductSKUS = productskus;
this.ProductSPUDes = spudesc;
return this;
}
}

```

4.产品 SKU 领域对象业务逻辑代码：

```
public partial class ProductSKU
{
public ProductSKU CreateProductSKU(string productspuname,Guid productspuid,
byte[] image,decimal dealerprice,decimal pv,string unit,string spec)
{
this.Id = Guid.NewGuid();
this.ProductSPUId = productspuid;
this.Code = "Code " + productspuname + spec;
this.ProductSPUName = productspuname;
this.Image = image;
this.DealerPrice = dealerprice;
this.PV = pv;
switch (unit)
{
case "盒":
this.Unit = Unit.盒;
break;
case "包":
this.Unit = Unit.包;
break;
case "瓶":
this.Unit = Unit.瓶;
break;
}
this.Spec = spec;
return this;
}
}

```

将领域对象的属性与领域对象的逻辑分到不同的 cs 文件中，便于不同职责人开发与管理，而且采用相同的名称空间和 Partial 关键字。

除了要实现领域逻辑之外，还要定义 ProductSPU 的仓储接口、通过 EF Core 定义产品上下文与数据库上下文之间的映射关系。

5.仓储接口定义：

```
public interface IProductRepository
{
void CreateProduct<T>(T productspu) where T : class, IAggregationRoot;
}

```

从上面可以看到，这个接口其实就是定义了将 ProductSPU 这个聚合根持久化到数据库与的接口。

6.产品上下文与数据库上下文映射关系：

因为映射关系使用 EF Core 实现，未来可能被替换掉，所以先定义一个产品上下文接口：

```
public interface IProductContext
{  }

```

EF Core 映射实现：

```
public class ProductEFCoreContext:DbContext,IProductContext
{
public DbSet<ProductSPU> ProductSPU { get; set; }
public DbSet<ProductSKU> ProductSKU { get; set; }
protected override void OnConfiguring(DbContextOptionsBuilder optionBuilder)
{
optionBuilder.UseSqlServer("数据库连接字符串");
}
}

```

7.利用数据访问框架工具生成对应的数据库和表

到这里，我们就基本实现了产品上下文的领域层，可以看到领域层主要是领域逻辑，定义了一个仓储接口，将数据库技术解耦，当然要定义领域对象与数据库之间的映射关系，否则用例无法完成真正对领域对象的持久化。

##### **仓储与应用服务实现**

我们已经有了关于产品方面的简单领域逻辑，我们接着来实现产品上下文关于仓储持久化与应用层的用例如何来协调领域逻辑与仓储持久化。

首先大家需要明确的是，产品上下文的领域逻辑是系统的核心，它不应该依赖仓储，而仓储应该要依赖领域层，这样仓储可以把领域逻辑执行完后，才可能将领域对象持久化到数据库中，这一点与传统的架构有本质的区别。一般我们会在解决方案中建立一个项目，这个项目就是包含了所有聚合的仓储实现，具体不同上下文的仓储实现，可以在这个项目下建立不同的文件夹。

1.产品上下文仓储实现：

```
public class ProductEFCoreRepository : IProductRepository
{
private readonly DbContext context;
public ProductEFCoreRepository(DbContext context)
{
this.context = context;
}
public void CreateProduct<T>(T productspu) where T:class,IAggregationRoot
{
var productdbcontext = this.context as ProductEFCoreContext;
var productspunew = productspu as ProductSPU;
try
{
productdbcontext.ProductSPU.Add(productspunew);
}
catch(Exception error)
{
throw error;
}
}  
}

```

上面的代码有几个要注意的方面：

- a. 首先会从产品的仓储接口做继承，通过 EF Core 的机制，实现了仓储接口的 CreateProduct方法。
- b. 使用了产品上下文的 EF Core 数据访问上下文 ProductEFCoreContext 完成了 Productspu 的数据库预添加。
- c. 上一个说法中，可能大家有两个疑惑，一是为什么不使用 productdbcontext 标记 ProductSPU 为 Added 状态，而是使用.Add 方法，二是为什么只是完成了添加状态，而不再后续调用 Commit 或 SaveChange 方法真正持久化到数据库中？首先，因为未来持久化要将这个聚合中的 ProductSPU 聚合根与 ProductSKU 实体作为一个整体持久化到数据库中，而 Added 状态只能将当前聚合根作为添加状态，而不能同时将引用的 ProductSKU 对象作为添加状态，所以不能使用 Added 状态而使用.Add 方法；其次仓储实现聚合提交时，只进行数据库预添，是因为协调领域逻辑与仓储的应用服务层用例可能涉及到多个聚合，所以可能要同时调用多个领域对象的业务逻辑，多个仓储，完成后，将多聚合作为一个整体事务做提交，所以真正的提交应该放到应用服务层更合适，而不是仓储层。

2.产品上架应用服务实现：

应用服务层实际就是完成用例，通过应用服务层调用领域逻辑，然后通过应用服务层调用仓储，最后应用服务层做真正的提交，这样就把职责分的非常清楚，也在领域逻辑不依赖仓储的前提下，完成了整个用例和持久化。

- a. 首先我们在产品上下文的应用服务层项目中，建立需要添加的产品 SPU 与对应产品 SKU 的 DTO 对象：

```
public class AddProductSPUDTO
{
public string SPUName { get; set; }
public string SPUDesc { get; set; }
public List<string> SKUSpecs { get; set; }
public List<string> SKUUnits { get; set; }
public List<decimal> SKUDealerPrices { get; set; }
public List<byte[]> SKUImages { get; set; }
public List<decimal> SKUPvs { get; set; }
}

```

- b. 上架产品的用例服务，协调领域逻辑与仓储完成用例：

```
public class AddProductSPUUseCase:BaseAppSrv
{
private readonly IRepository irepositorycontext;
private readonly IProductRepository iproductrepository;
public AddProductSPUUseCase(IRepository irepositorycontext,IProductRepository iproductrepository)
{
this.irepositorycontext = irepositorycontext;
this.iproductrepository = iproductrepository;
}
public ResultEntity<bool> AddProduct(AddProductSPUDTO addproductspudto)
{
var productspuid = Guid.NewGuid();
var productskus = new List<ProductSKU>();
for(int i = 0; i < addproductspudto.SKUSpecs.Count; i++)
{
var productsku = new ProductSKU().CreateProductSKU(addproductspudto.SPUName,
productspuid, addproductspudto.SKUImages[i], addproductspudto.SKUDealerPrices[i],
addproductspudto.SKUPvs[i], addproductspudto.SKUUnits[i], addproductspudto.SKUSpecs[i]);
productskus.Add(productsku);
}
var productspu = new ProductSPU().CreateProductSPU(productspuid, addproductspudto.SPUName,
addproductspudto.SPUDesc, productskus);
try
{
using (irepositorycontext)
{
iproductrepository.CreateProduct(productspu);
irepositorycontext.Commit();
}
return GetResultEntity(true);
}
catch(Exception error)
{
throw error;
}
}
}

```

BaseAppSrv 是你要定义的一个类，它的 GetResultEntity 方法功能是完成用例后后，返回接口层的数据格式，这个数据格式会进一步通过接口层返回给前端，返回的数据格式就是 `ResultEntity<T>`，这两个部分大家可以自己去实现。

##### **产品上下文接口实现**

在实际的项目中，多种前端的形态比如 PC Web、微信小程序、原生 APP 等要调用后端的功能，通常要将后端的功能包装成 RESTFUL 风格，这样前端就可以使用 Http Get 或 Post 方式调用后端的功能，所以这篇文章我们先来完成后端的 Asp.net Core WebAPI，通过 WebAPI 将上架产品的功能暴露出去。

实现产品上架接口：

```
[Produces("application/json")]
[Route("api/Product")]
public class ProductController : Controller
{
ServiceLocator servicelocator = new ServiceLocator();
[HttpPost]
[Route("AddProduct")]
public ResultEntity<bool> AddProduct([FromBody] AddProductSPUDTO addproductspudto)
{
var result = new ResultEntity<bool>();
var productdbcontext = servicelocator.GetService<IProductContext>();
var irepository = servicelocator.GetService<IRepository>(new ParameterOverrides { { "context", productdbcontext } });
var iproductrepository=servicelocator.GetService<IProductRepository>(new ParameterOverrides { { "context", productdbcontext } });
var addproductspuusecase = new AddProductSPUUseCase(irepository,iproductrepository);
try
{
result = addproductspuusecase.AddProduct(addproductspudto);
result.IsSuccess = true;
result.Count = 1;
result.Msg = "上架产品成功!";
}
catch(Exception error)
{
result.ErrorCode = 100;
result.Msg = error.Message;
}
return result;
}
}

```

- a. 首先大家看到接口层是非常薄的一层，它并不包含业务逻辑和数据访问，它只是初始化一些对象，然后完成应用服务的调用，返回前端所需要的格式的对象。
- b. 产品数据访问上下文、仓储接口、产品上下文仓储接口等需要通过依赖注入框架来获取特定的实现类，依赖注入框架可以采用 Asp.net Core 自带的，也可以采用 Unity 等框架。这里略去了依赖注入框架的具体实现。
- c. 如果在调用应用服务可能抛出异常时，需要详细指明每个 catch 与抛出的内容。

这篇文章只是 DDD 快速入门，大家通过概念和代码已经了解了 DDD 的核心思想以及如何实现一个简单的落地.相信大家通过概念和一个简单的开发案例，已经初步了解了 DDD 为什么能解决软件开发中的 6 大难题，期望未来再给大家奉献上进一步复杂业务逻辑以及值对象应用的案例,让大家更深入得了解 DDD。

