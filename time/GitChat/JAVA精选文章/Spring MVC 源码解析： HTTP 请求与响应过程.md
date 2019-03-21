1. 常见的两种错误场景介绍；
2. HTTP 请求与处理源码解析；
3. 两种错误场景解决方案；
4. 涉及的设计模式介绍。

相关版本:

- Maven : apache-maven-3.3.9
- SpringMVC :4.1.1.RELEASE
- Tomcat-Maven-Plugin: 2.2

介绍方式：代码 + 文字说明 + 源码截图（为减小篇幅，因此源码部分采用截图的方式）。读者阅读时，结合前面列出的流程图/主要操作步骤，再浏览。

> 本文中涉及的代码[下载](https://github.com/wlmshuaia/JsonDemo)

## 一、常见的两种错误场景

### 1.场景1

jQuery 以 ajax 方式访问 SpringMVC 接口时，如未显示指定 Content-Type，则会显示 '415 (Unsupported Media Type)' 错误。如:

![415 错误](http://images.gitbook.cn/35bb5530-f032-11e7-bfec-fbc6e2a608a5)

前端代码片段：

```
function fSave(url) {
    var obj = {};
    obj['cateId'] = $("input[name=cateId]").val();
    obj['cateName'] = $("input[name=cateName]").val();

    $.ajax({
        url: url,
        method: 'post',
        data: JSON.stringify(obj), // 以json字符串方式传递
        success: function(data) {
            console.log(data);
        },
        error: function(data) {
            console.log("error...");
            console.log(data);
        }
    });
}

```

后端接口代码片段为：

```
@RequestMapping(value = "/save-by-model-2", method = RequestMethod.POST)
@ResponseBody
public Category saveByModel2(@RequestBody Category category){
    categoryService.save(category);
    return category;
}

```

### 2.场景2

SpringMVC 接口定义返回 Json 格式数据时，一般有 **字符串和对象** 两种方式。而相同条件下，返回 Json 格式字符串时，中文会出现乱码。如:

![中文乱码](http://images.gitbook.cn/55bdb610-f047-11e7-bc73-87cf5851ce7a)

前端代码片段：

```
function fSave(url) {
    var obj = {};
    obj['cateId'] = $("input[name=cateId]").val();
    obj['cateName'] = $("input[name=cateName]").val();

    $.ajax({
        url: url,
        method: 'post',
        contentType: 'application/json', 
        data: JSON.stringify(obj), // 以json字符串方式传递
        success: function(data) {
            console.log(data);
        },
        error: function(data) {
            console.log("error...");
            console.log(data);
        }
    });
}

```

后端接口片段：

```
@RequestMapping(value = "/save-by-map", method = RequestMethod.POST)
@ResponseBody
public String saveByMap(@RequestBody Map<String, Object> valMap) {
    categoryService.save(valMap);
    return valMap.toString();
}

@RequestMapping(value = "/save-by-map-2", method = RequestMethod.POST)
@ResponseBody
public Map<String, Object> saveByMap2(@RequestBody Map<String, Object> valMap) {
    categoryService.save(valMap);
    return valMap;
}

```

## 二、HTTP 请求与响应处理源码解析

SpringMVC 中 HTTP 请求处理流程时序图如下：

![HTTP 请求处理时序图](http://images.gitbook.cn/e9b4de40-f482-11e7-aa0c-95761436cde1)

从上图可以看出，所有的 HTTP 请求都会进入到 `DispatcherServlet`类的 `doDispatch()`方法。该方法中的主要工作为:

1. 获取处理执行链对象: `HandlerExecutionChain mappedHandler = getHandler(processedRequest);`
2. 获取处理适配器: `HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());`
3. 调用拦截器的 `preHandle` 方法；
4. 调用具体的接口方法，并返回模型视图对象: `mv = ha.handle(processedRequest, response,mappedHandler.getHandler());`
5. 调用拦截器的 `postHandle` 方法；
6. 处理结果: `processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);` 。

**注：本次源码解析为通过 @EnableWebMvc 方式启动，与配置文件方式启动时的源码略有不同（该方式在 Spring 3.2 以后已被弃用），如 HandlerMapping, HandlerAdapter 的实现 。**

接下来深入 SpringMVC 处理 HTTP 请求的过程。

### 1.获取处理执行链对象

**处理执行链类 HandlerExecutionChain : 由处理对象 handler 和 拦截器列表 interceptorList 组成，通过 HandlerMapping.getHandler() 方法返回。**

`DispatcherServlet` 类中获取处理链对象的 `getHandler(HttpServletRequest request)` 方法实现为:

![getHandler()](http://images.gitbook.cn/766d1540-f05b-11e7-bfec-fbc6e2a608a5)

从上图可以看到，框架遍历所有的 `HandlerMapping` 对象，调用对应的 `hm.getHandler(request)` 方法，如果获取的处理执行链对象不为 `null` 则返回该处理执行链对象。

**HandlerMapping: 定义请求和处理对象之间的映射关系接口。HandlerMapping 列表由 SpringMVC 初始化时寻找所有的 HandlerMapping 类组成。**

SpringMVC 中，默认的 `HandlerMapping` 实现有：

![HandlerMapping](http://images.gitbook.cn/171cd600-f05d-11e7-bfec-fbc6e2a608a5)

上图 `HandlerMapping` 列表中，`RequestMappingHandlerMapping` 类为 `@Controller` 注解的类中的 **@RequestMapping 注解方法**创建 `RequestMappingInfo` 对象。而前面示例中定义的接口方法为 `@RequestMapping` 类型，因此此处取得的 `HandlerMapping` 类为该类。简化的类图如下：

![RequestMappingHandlerMapping](http://images.gitbook.cn/6cb3b1b0-f05d-11e7-bfec-fbc6e2a608a5)

`RequestMappingInfo` 封装了 `@RequestMapping` 注解方法相关的状态如下:

![RequestMappingInfo](http://images.gitbook.cn/22a19f30-f065-11e7-99ec-1991674b03a0)

而在`RequestMappingHandlerMapping` 类中获取处理链对象则由父类`AbstractHandlerMapping` 实现:

![getHandler()](http://images.gitbook.cn/ecd8c990-f05b-11e7-bfec-fbc6e2a608a5)

**在 AbstractHandlerMapping 中，主要操作为2步:**

1. 获取处理器对象；
2. 获取处理执行链。

#### 1.1 获取处理器对象

由子类 `AbstractHandlerMethodMapping` 实现:

![getHandlerInternal](http://images.gitbook.cn/867299b0-f065-11e7-bfec-fbc6e2a608a5)

上图实现中返回的处理器对象类型为处理方法 (`HandlerMethod`)。主要操作为:

1. 调用 `UrlPathHelper` 类从 `request` 中获取请求路径 `lookupPath`；
2. 根据请求路径获取处理方法。

在第2步中，**SpringMVC 会根据 lookupPath 作为 key 值从缓存的 urlMap 对象中获取 HandlerMethod ，如果根据 key 值未获取到对应的数据，则会遍历所有的 urlMap 数据列表。**

![lookupHandlerMethod()](http://images.gitbook.cn/44344230-f068-11e7-99ec-1991674b03a0)

**注： urlMap 中的数据在 SpringMVC 初始化时填入。**

`addMatchingMappings()` 方法中的实现为:

![addMatchingMappings()](http://images.gitbook.cn/6ac10870-f068-11e7-bfec-fbc6e2a608a5)

由于该机制的存在，如果定义的 `@RequestMapping` 与访问的路径不相等，则框架会遍历所有的 `@RequestMapping` 方法。如果数量很多时，则该部分会消耗较多时间。因此如该部分有性能问题，应尽量使 `@RequestMapping` 路径与访问路径匹配以减少遍历开销。

**优化建议:**

1. SpringMVC-4.0.3 以后的版本可通过`WebMvcConfigurerAdapter.configurePathMatch(PathMatchConfigurer configurer)` 方法自定义实现 `UrlPathHelper, AntPathMatcher` 优化该问题；
2. 修改 `RequestMappingHandlerMapping` 类中 `UrlPathHelper, AntPathMatcher` 的实现 ([代码参考](https://github.com/wlmshuaia/JsonDemo))；
3. 定义 `@RequestMapping` 路径与访问路径一致，如访问路径为 'index.do'，则相应的 `@RequestMapping(value = "/index.do")`；
4. 减少单个项目中的 `@RequestMapping` 的数量。

#### 1.2 获取处理执行链

该方法根据传入的处理器对象建立处理执行链，将传入的处理对象和匹配的拦截器添加到处理执行链对象中，并返回：![获取处理执行链]](http://images.gitbook.cn/c6ed9660-f480-11e7-9976-43dd6592a81a)

### 2.获取处理适配器

**HandlerAdapter:** MVC 框架的 SPI 接口，允许核心MVC工作流的参数化（这句话不是很懂...有会的读者欢迎指教）。每一种处理请求的处理器类型必须实现该接口，`DispatcherServlet` 类可根据该接口无限扩展，且 `DispatcherServlet` **根据该接口访问所有已安装的处理器对象**。而处理器类型设置为 `Object`，其他的框架不用修改源码就能和 SpringMVC 整合。完整的接口介绍如下：

![HandlerAdapter](http://images.gitbook.cn/4619f580-f091-11e7-bfec-fbc6e2a608a5)

传入处理器对象，SpringMVC 遍历所有的 `HandlerAdapter` 类，如果某个处理适配器支持该处理器类型，则返回该处理器:

![handlerAdapters](http://images.gitbook.cn/152c4640-f081-11e7-85cc-1fa4202332c7)

`supports(handler)` 接口判断某个具体的处理适配器是否支持传入的处理器对象，定义如下:

![supports()](http://images.gitbook.cn/460bacf0-f0fa-11e7-97ee-f59a3681bc6a)

在 `RequestMappingHandlerAdapter` 类中，支持的处理器类型为 `HandlerMethod` ：

![supports()](http://images.gitbook.cn/607e3220-f081-11e7-bfec-fbc6e2a608a5)

因此此处返回的 `HandlerAdapter` 实现类为：`RequestMappingHandlerAdapter` 。

### 3.调用拦截器的 `preHandle` 方法

调用处理执行链对象的 `applyPreHandle()` 方法，代码如下:

![applyPreHandle()](http://images.gitbook.cn/238efb40-f0fb-11e7-ac61-75f3525e47e1)

可以看出框架会遍历在 **第1步：获取处理链对象** 中获取的拦截器对象，依次调用`preHandle()` 方法，如果某次调用返回 `false`，则会调用 `triggerAfterCompletion()` 方法，并返回 `false`。`triggerAfterCompletion()` 实现如下:

![triggerAfterCompletion()](http://images.gitbook.cn/4019b1a0-f0fc-11e7-aaa1-4505fd6074a0)

即依次调用拦截器对象的 `afterCompletion()` 方法。

### 4.调用具体的接口方法

调用 **第2步：获取的处理适配器** 对象的 `handle()` 方法处理请求。而 `RequestMappingHandlerAdapter` 类中处理请求的是 `handleInternal()` 方法。方法执行时序图如下：

![handlerInternal()](http://images.gitbook.cn/f3314df0-f482-11e7-aa0c-95761436cde1)

从上图中可以看出，框架先创建 `ServletInvocableHandlerMethod` 对象，然后调用该对象的 `invokeAndHandle(webRequest, mavContainer)` 方法，最后获取 `ModelAndView` 对象。具体实现如下：

![invokeHandleMethod()](http://images.gitbook.cn/a545a400-f438-11e7-b903-c1c5b74d2c76)

而 `invokeAndHandle()` 方法中，主要操作为:

1. 根据 `request` 解析参数，并调用具体的方法，获取返回值（即调用 `invokeForRequest()` 方法）；
2. 调用返回值处理器处理返回值。

#### 4.1 调用具体的方法

该部分操作步骤为:

1. 解析参数；
2. 根据参数调用具体的方法，并获取返回值；
3. 返回返回值。

解析参数时，会先获取方法定义的所有参数列表，然后根据每个具体的 `MethodParameter`类型，调用具体的参数解析器类 `HandlerMethodArgumentResolver` 的解析参数方法 `resolveArgument()` 。源码如下：

![getMethodArgumentValues()](http://images.gitbook.cn/c4dee2d0-f44d-11e7-aa0c-95761436cde1)

此处的 `argumentResolvers` 对象为 `HandlerMethodArgumentResolverComposite` 类，该类封装了一系列的参数解析器，属性如下：

![HandlerMethodArgumentResolverComposite](http://images.gitbook.cn/08239480-f450-11e7-b903-c1c5b74d2c76)

解析时，根据 `HandlerMethodArgumentResolver.supportsParameter(MethodParameter methodParameter)` 判断该参数解析器是否支持该参数。前面参数由于使用的是 `@RequestBody` 注解，因此调用 `RequestResponseBodyMethodProcessor` 类，典型的还有 `@RequestParam, @PathVariable, @RequestHeader` 等：

![HandlerMethodArgumentResolver](http://images.gitbook.cn/2b598740-f453-11e7-aa0c-95761436cde1)

而 `RequestResponseBodyMethodProcessor` 类处理参数时，使用了 `HttpMessageConverter`消息转换机制。

##### 4.1.1 `HttpMessageConverter` 消息转换机制

该机制的流程为:

![HttpMessageConverter](http://images.gitbook.cn/197e7c80-f46a-11e7-9976-43dd6592a81a)

由于请求的 `content-type` 为 `application/json` 类型，所以此处调用的消息转换类为 `MappingJackson2HttpMessageConverter` ，默认的字符集为 `UTF-8`:

![default-charset](http://images.gitbook.cn/b0420150-f456-11e7-9976-43dd6592a81a)

而 `read()` 的实现为调用 `Jackson` 的类 `ObjectMapper.readValue()` 方法:

![read()](http://images.gitbook.cn/cbb21560-f456-11e7-b903-c1c5b74d2c76)

**注:** 在调用 `read()` 方法之前，如果无匹配的 `HttpMessageConverter` 类，则会抛出 `HttpMediaTypeNotSupportedException` 异常；

在调用 `write()` 方法之前，如果无匹配的 `HttpMessageConverter` 类，则会抛出 `HttpMediaTypeNotAcceptableException` 异常。

**解析完参数后**，则通过反射机制调用具体的方法并获取返回值:

![invoke()](http://images.gitbook.cn/1c60d690-f457-11e7-b903-c1c5b74d2c76)

#### 4.2 处理返回值

该部分的逻辑和前面解析参数逻辑类似。主要为调用具体的 `HandlerMethodReturnValueHandler` (返回值处理器) 处理返回值。框架中默认的返回值处理器有:

![HandlerMethodReturnValueHandler](http://images.gitbook.cn/a8759620-f457-11e7-9a02-c1395eaddd88)

此处由于使用 `@ResponseBody` 注解，因此调用 `RequestResponseBodyMethodProcessor` 处理器。该处理器中，使用 `HttpMessageConverter` 消息转换机制（见 4.1.1），调用 `write()`方法处理返回值：

![write()](http://images.gitbook.cn/d5b926f0-f458-11e7-9a02-c1395eaddd88)

而在调用 `write()` 方法之前，会调用 `getProducibleMediaTypes()` 方法获取可生产的媒体类型，如果用户自定义 `RequestMapping 的 produces` 属性，则此处会返回该值；如果用户未定义，则根据返回值 `Class` 类型，遍历系统中的消息转换器列表，获取支持的媒体类型列表：

![getProducibleMediaTypes()](http://images.gitbook.cn/5fccba20-f46b-11e7-9976-43dd6592a81a)

### 5.调用拦截器的 `postHandle` 方法

调用拦截器对象的 `applyPostHandle()` 方法，代码如下：

![applyPostHandle()](http://images.gitbook.cn/906459d0-f0fc-11e7-97ee-f59a3681bc6a)

从上图可以看出框架会遍历拦截器对象列表，以处理链对象中拦截器对象列表**相反的顺序**调用拦截器对象的 `postHandle()` 方法。

### 6.处理结果

调用 `processDispatchResult()` 方法，实现代码如下:

![processDispatchResult()](http://images.gitbook.cn/8405bc50-f0fd-11e7-aaa1-4505fd6074a0)

这部分处理主要分为如下几部分:

1. 处理异常；
2. 渲染 `ModelAndView` 对象；
3. 调用处理执行链的 `triggerAfterCompletion()` 方法。

#### 6.1 处理异常

如果在前面的处理中抛出了异常，则会获取相应的模型视图对象。有两种处理方式：如果异常对象为 `ModelAndViewDefiningException` 类型，则直接获取模型视图对象；否则的话调用当前系统内的处理异常解析器 ( `HandlerExceptionResolver` ) 处理:

![handlerExceptionResolvers](http://images.gitbook.cn/e3ca1a70-f47a-11e7-b903-c1c5b74d2c76)

如果某个异常解析器返回了有效的模型视图对象，则跳出循环。

此处的 `ExceptionHandlerExceptionResolver` 类通过用户自定义的 `@ExceptionHandler` 方法解析异常，如果用户未定义，则跳出该解析器。此处示例代码为：

```
@ExceptionHandler(Exception.class)
public String handlerException(HttpServletRequest request, HttpServletResponse response, Exception ex) {
    System.out.println("handlerException...");
    return "redirect:/error.do";
}

```

在该类的 `doResolveHandlerMethodException()` 方法中，会创建一个 `ServletInvocableHandlerMethod` 对象，然后调用该对象的 `invokeAndHandle()` 方法:

![doResolveHandlerMethodException()](http://images.gitbook.cn/efd30bf0-f47b-11e7-9a02-c1395eaddd88)

与前面处理正常 `HandlerMethod` 流程类似，就不深入探讨了。

#### 6.2 渲染 `ModelAndView` 对象

在渲染方法 `render()` 中，如果传入的 mv 对象是 `View` 引用类型，即为 `String` 字符串时，则调用当前的视图解析器 `ViewResolver` 解析该字符串，如当前配置了视图解析器为：



```
<!-- view path -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/"/>
    <property name="suffix" value=".html"/>
</bean>

```

则该实现会在视图解析器列表 `viewResolvers` 中：

![viewResolvers](http://images.gitbook.cn/936a43f0-f477-11e7-9976-43dd6592a81a)

在解析时，将会添加上对应的 `prefix, suffix`, 处理后的 `View` 对象为:

![View](http://images.gitbook.cn/b38653e0-f477-11e7-9a02-c1395eaddd88)

接下来则调用 `View` 对象的 `render()` 方法，根据提供的 `Model` 对象渲染该视图对象。

#### 6.3 调用处理执行链的 `triggerAfterCompletion()` 方法

该方法只调用在 `preHandle()` 方法中成功调用且返回为 `true` 的拦截器，且从列表后往前调用：

![triggerAfterCompletion()](http://images.gitbook.cn/92b7a050-f478-11e7-aa0c-95761436cde1)

## 三、解决方案

根据前面的原理介绍可知，文章开头的两种场景错误都是由于在 第4步：调用具体的接口方法 出错：场景1为解析参数时出错，场景2为返回值处理时出错。

### 1. 场景1出错原因

见 4.1.1 消息转换机制。场景1中，ajax 请求如果未设置 `Content-Type` 则会使用默认的类型: `application/x-www-form-urlencoded; charset=UTF-8`，而由于接收参数定义为 `@RequestBody`，对应的参数解析器为 `RequestResponseBodyMethodProcessor` 类，调用 `HttpMessagConverter` 消息转换机制。而 SpringMVC 中并无支持该类型的 `HttpMessageConverter` 类，因此抛出异常。

### 2. 场景1解决方案

指定具体的 Content-Type, 如:

```
$.ajax({
    url: url,
    method: 'post',
    contentType: 'application/json', // 解决 415 错误
    data: JSON.stringify(obj), 
    success: function(data) {
        console.log(data);
    },
    error: function(data) {
        console.log("error...");
        console.log(data);
    }
});

```

### 3. 场景2出错原因

见 4.2 处理返回值。场景2中，定义了 `@ResponseBody` 注解，调用的返回值处理器为 `RequestResponseBodyMethodProcessor`，调用 `HttpMessagConverter` 消息转换机制。返回的 `Class` 类型是 `String`，而在 SpringMVC 的消息转换器列表中支持该返回值类型的消息转换器有 `StringHttpMessageConverter, MappingJackson2HttpMessageConverter` 支持的媒体列表有：

![MediaTypes](http://images.gitbook.cn/f31ad730-f46b-11e7-9a02-c1395eaddd88)

取得媒体列表后，会选取其中的一个：

![selectMediaType](http://images.gitbook.cn/26811ad0-f46c-11e7-b903-c1c5b74d2c76)

而 `String` 返回值类型取得的媒体类型为 `text/plain;charset=ISO-8859-1` , 将该媒体类型设置为 `response` 的 `contentType`, 因此返回中文乱码：

![Content-Type](http://images.gitbook.cn/b27ab140-f46c-11e7-b903-c1c5b74d2c76)

而 `Category` 类支持的媒体类型只有两种，因此返回值正常：

![MediaTypes](http://images.gitbook.cn/599ebcb0-f46c-11e7-aa0c-95761436cde1)

### 4. 场景2解决方案

在接口上方定义 `produces` ，如:

```
@RequestMapping(value = "/save-by-map", method = RequestMethod.POST, java produces = "text/plain;charset=utf-8")
@ResponseBody
public String saveByMap(@RequestBody Map<String, Object> valMap) {
    categoryService.save(valMap);
    return valMap.toString();
}

```

或者在 springmvc 的配置文件中配置消息转换器的媒体类型，如:

```
<mvc:annotation-driven >
    <!-- 消息转换器 -->
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="supportedMediaTypes" value="text/plain;charset=UTF-8"/>
       </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

```

## 四、设计模式

本章内容不包括对具体的设计模式原理介绍，只介绍前面框架源码中出现的设计模式的应用场景，如读者对相关的设计模式原理有兴趣，可查看每小节的链接或者自行学习。

前面研究的原理中，涉及的设计模式如下：

### 1.模板方法模式

> [模板方法模式原理](http://blog.csdn.net/u012099869/article/details/78689257)

该模式在框架中使用的地方很多，主要作用为: 定义算法的骨架。

如获取处理执行链方法中，`getHandler()` 方法定义了算法的骨架如下：

![getHandler 方法](http://images.gitbook.cn/40ebd320-f056-11e7-99ec-1991674b03a0)

而 `getHandlerInternal()` 则由具体的子类来实现:

![getHandlerInternal 方法](http://images.gitbook.cn/4819bef0-f056-11e7-85cc-1fa4202332c7)

### 2.适配器模式

该模式本质为: (接口不兼容时，通过)转换匹配，(从而)复用功能。

如 `HandlerAdapter` 的实现，可适配所有的处理器类型。是常见的适配器模式实现方式：

![HandlerAdapter](http://images.gitbook.cn/01775540-f423-11e7-b903-c1c5b74d2c76)

可以看出 `handle()` 方法传入 `handler` 对象，在实现类中操作传入的 `handler` 对象。有新的处理器类型时，实现该接口即可。如 `@EnableMvc` 方式使用的 `RequestMappingHandlerAdapter`，以及已经弃用的 `AnnotationMethodHandlerAdapter`。

还有种缺省适配方式，由子类选择覆盖需要的方法。如 `WebMvcConfigurerAdapter` 类：

![WebMvcConfigurerAdapter](http://images.gitbook.cn/e06869b0-f4fa-11e7-af68-d97458d241f7)

### 3.责任链模式

该模式本质为：分离对象的职责，动态的组合在一起。

常见的有两种实现方式:

1. 责任链：请求在链上传递，有一个对象处理过则跳出；
2. 功能链：请求在链上传递，链上的每个责任对象负责处理某一方面的功能，处理过不跳出，向下传递。

如 `HandlerExecutionChain` 的实现，为功能链的实现：![HandlerExecutionChain](http://images.gitbook.cn/d298e3b0-f422-11e7-9976-43dd6592a81a)定义的拦截器列表 `interceptorList`，循环调用 `preHandle()` 方法 ，处理用户请求之前，均会经过该拦截器列表中所有的对象处理。

### 4.策略模式

该模式本质为：分离算法，选择实现。即对应同一操作，提供不同的策略实现。

如参数解析器 `HandlerMethodArgumentResolver`:

![HandlerMethodArgumentResolver](http://images.gitbook.cn/43a72460-f4fd-11e7-af6f-edb57aa9bfa8)

返回值处理器 `HandlerMethodReturnValueHandler`:![HandlerMethodReturnValueHandler](http://images.gitbook.cn/4f7cc650-f4fd-11e7-af6f-edb57aa9bfa8)

### 5.工厂方法模式

该模式形式为：提供一个创建对象实例的接口，让子类决定实例化哪一个类。

如在 `RequestMappingHandlerAdapter.invokeHandleMethod()` 方法中，创建 `ServletInvocableHandlerMethod` 时，会传入 `WebDataBinderFactory` 对象。而该类即为典型的工厂方法模式的实现:

![WebDataBinderFactory](http://images.gitbook.cn/4d2bf120-f500-11e7-af68-d97458d241f7)

在其默认的实现里，通过 `createBinderInstance()` 方法创建 `WebDataBinder` 对象，该实现可由子类自行扩展:

![createBinderInstance](http://images.gitbook.cn/5edbeae0-f502-11e7-9515-d9c8916190ef)

### 6.组合模式

该模式定义为：将对象组合成树型结构，以统一叶子对象和组合对象。

一般有以下两种场景：

1. 统一整体和部分的操作；
2. 统一地使用组合结构中的所有对象。

SpringMVC 中参数解析时调用的 `HandlerMethodArgumentResolverComposite` 类, 组合一系列的参数解析器，即为第2种用法:

![HandlerMethodArgumentResolverComposite](http://images.gitbook.cn/9a65f160-f515-11e7-8ed1-b72a2cac04d1)

还有 `HandlerMethodReturnValueHandlerComposite, WebMvcConfigurerComposite` 等。

### 7.单例模式

模式定义为：保证一个类仅有一个实例，并提供一个全局访问点。该模式在日常的开发中应用也非常广泛。

两种典型的实现方式：

1. 懒汉式：一般为延迟加载方式，在使用的对象实例不存在时，再创建对象实例；
2. 饿汉式：一般在初始化时，创建对象实例。

SpringMVC 中，获取 `WebAsyncManager` 类时，即通过缓存的方式实现的懒汉式单例模式。将创建的 `WebAsyncManager` 对象缓存在 `ServletRequest` 中：

![WebAsyncManager](http://images.gitbook.cn/d3a95150-f52a-11e7-af6f-edb57aa9bfa8)

注：`WebAsyncManager` 类为管理异步请求处理的主要类。

------

SpringMVC 框架中还有很多使用的设计模式未被列出，以上列举的只是前面深入源码过程中比较典型的几个。

## 五、写在最后

以上就是我所理解的 SpringMVC 处理 HTTP 请求的过程。而前面所列出的也只是整个 SpringMVC 框架的冰山一角，还有很多等待着我们去探索。在深入学习的过程中，感触最深的也是自己的无知，此处引用一句名言: 越学习，越发现自己的无知。