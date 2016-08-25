# Spring

## Mvc

### 例子

**依赖：**

![](F2FC.tmp.jpg)

首先，我们需要引入上述的JAR包，以及一些相关联的JAR，如：

* log4j
* jetty

这样子，就完成了Spring Jar的引入。

**web.xml**

```xml

   <!-- UTF-8 编码-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--Spring MVC-->
    <servlet>
        <servlet-name>springMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <!--使用注解方式的WebApplicationContext容器-->
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <!--指定Spring注解配置类-->
            <param-name>contextConfigLocation</param-name>
            <param-value>org.darkgem.SpringConf</param-value>
        </init-param>
        <!--项目不支持直接的ASYNC-->
        <async-supported>false</async-supported>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <!--拦截所有请求-->
    <servlet-mapping>
        <servlet-name>springMVC</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

    <!--index 配置-->
    <welcome-file-list>
        <welcome-file>/index.html</welcome-file>
    </welcome-file-list>

    <!--error-page 配置-->
    <error-page>
        <exception-type>java.lang.Exception</exception-type>
        <location>/exception.html</location>
    </error-page>
    <error-page>
        <error-code>404</error-code>
        <location>/404.html</location>
    </error-page>
    <error-page>
        <error-code>500</error-code>
        <location>/500.html</location>
    </error-page>

```

在`web.xml`中，我们配置了：

1. 添加支持UTF-8的字符过滤器
2. 添加`DispatcherServlet`
3. 异常/404/500 等页面

容器在启动后，会加载和初始化`DispatcherServlet`这个Servlet类。而**Spring容器**的初始化就是通过`DispatcherServlet`实现的。

注意：UTF-8字符过滤器会**智能**的处理`charset=utf-8`的设置。

**Spring配置：**

```java

@Configuration
@PropertySource("classpath:project.properties")
@EnableAspectJAutoProxy(proxyTargetClass = true)
@ComponentScan("org.darkgem")
public class SpringConf {
    static Logger logger = LoggerFactory.getLogger(SpringConf.class);


    /**
     * Spring MVC 配置
     */
    @Component
    // 开启Spring Mvc
    @EnableWebMvc
    // Spring Mvc 配置器，当然，也可以不通过这个Adapter配置
    static class SpringMvcConf extends WebMvcConfigurerAdapter {

        // 配置默认ServletHandler
        @Override
        public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
            configurer.enable();
        }

        // 配置视图渲染
        @Override
        public void configureViewResolvers(ViewResolverRegistry registry) {
            registry.order(1);

            FastJsonJsonView fastJsonJsonView = new FastJsonJsonView();
            fastJsonJsonView.setExtractValueFromSingleKeyModel(true);

            registry.enableContentNegotiation(fastJsonJsonView);

        }
    }
}
```

在上述注解式配置中，通过`SpringMvcConf`配置了SpringMvc组件：

1. 默认ServlerHandler处理
2. JSON渲染器

**控制器：**


```java

@Controller
@RequestMapping("/todo/TodoCtrl")
public class TodoCtrl {

    @RequestMapping("/hello")
    public Message hello() {
        return Message.okMessage("Hello");
    }
}


```

上述，是一个简单的Ctrl控制器，用于返回Hello字符串。

**测试：**

我们通过Jetty启动容器，然后通过浏览器测试：

![](6D95.tmp.jpg)

可以发现，我们的Spring Mvc已经正常工作了。

### 接口和类

#### DispatcherServlet

`DispatcherServlet`是核心HTTP请求的调度器。

通过它，将SpringMvc接入到J2EE容器中，从而可以处理HTTP请求：

```xml

  <!--Spring MVC-->
    <servlet>
        <servlet-name>springMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>org.darkgem.SpringConf</param-value>
        </init-param>
        <!--项目不支持直接的ASYNC-->
        <async-supported>false</async-supported>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springMVC</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

```

并且通过它的父类`FrameworkServlet`完成了对**Spring容器**的整合工作。

#### HandlerMapping

通过`HandlerMapping`的子类，我们可以通过request获取一个handler对象：

```java

/**
 * Interface to be implemented by objects that define a mapping between
 * requests and handler objects.
 *
 * 定义request <-> handler的mapping
 */
public interface HandlerMapping {
    
    ...

	/**
	 * Return a handler and any interceptors for this request. The choice may be made
	 * on request URL, session state, or any factor the implementing class chooses.
	 * <p>The returned HandlerExecutionChain contains a handler Object, rather than
	 * even a tag interface, so that handlers are not constrained in any way.
	 * For example, a HandlerAdapter could be written to allow another framework's
	 * handler objects to be used.
	 * <p>Returns {@code null} if no match was found. This is not an error.
	 * The DispatcherServlet will query all registered HandlerMapping beans to find
	 * a match, and only decide there is an error if none can find a handler.
	 *
     * 针对给定的request对象，返回一个handler以及拦截器。如果，返回null，则表示无法查
     * 询到request对应的handler。
     * 
	 */
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;

}

```

`HandlerMapping`存在如下几个重要的子类：

* RequestMappingHandlerMapping: 最新的HandlerMapping子类，支持@RequestMapping等注解。
* SimpleUrlHandlerMapping: 简单的URL->Handler映射器。`<mvc:default-servlet-handler />`就是通过它实现mapping的。


给定一个request对象，然后通过`HandlerMapping#getHandler`映射器，将获取到一个待执行`HandlerExecutionChain`对象：

```java

public class HandlerExecutionChain {

	
    // handler
	private final Object handler;
    
    // 拦截器
	private HandlerInterceptor[] interceptors;
    
    ....
}

```

之后，handler对象将通过`HandlerAdapter`处理器进行适配，然后进行具体的执行流程。

注意：如果`getHandler`返回`null`，则表示该映射器无法处理该request对象。

#### HandlerAdapter


```java


/**
 *
 * Handler 执行器
 */
public interface HandlerAdapter {

	/**
	 * Given a handler instance, return whether or not this {@code HandlerAdapter}
	 * can support it. 
     *
     * 给定一个handler实例，判断是否被这个HandlerAdapter支持。
     * 通常是通过检验对象的类型信息。
	 */
	boolean supports(Object handler);

	/**
	 * Use the given handler to handle this request.
	 *
     * 使用给定的handler对象去处理本次request。
     * 注意：如果返回null，则表示本次请求已经处理完成。
	 */
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	/**
	 * Same contract as for HttpServlet's {@code getLastModified} method.
	 * Can simply return -1 if there's no support in the handler class.
	 *
     * 类似HttpServlet的getLastModified方法。
     * 可以简单的返回-1。
	 */
	long getLastModified(HttpServletRequest request, Object handler);

}


```

`HandlerAdapter`的重要子类：

* RequestMappingHandlerAdapter: 最新的`HandlerAdapter`子类，支持`参数解析器`，`返回对象处理器`等特性。
* HttpRequestHandlerAdapter: handler类型为`HttpRequestHandler`（如：`DefaultServletHttpRequestHandler`）的适配器。

通过`HandlerAdapter#handle`方法，可以**修饰/处理**来自于`HandlerMapping#getHandler`获取的handler对象。

注意：如果`handle`返回null，则表示该请求已经被处理完成。

#### ViewResolver

```java

/**
 * Interface to be implemented by objects that can resolve views by name.
 *
 */
public interface ViewResolver {

	/**
	 * Resolve the given view by name.
	 *
     * 通过给定name解析View对象
	 */
	View resolveViewName(String viewName, Locale locale) throws Exception;

}

```

`ViewResolver`重要子类：

1. ViewResolverComposite: ViewResolver 聚合类。
2. ContentNegotiatingViewResolver: 根据**MIME**匹配View。

**关于MIME**：Request的MIME来自于的URL后缀[`*.json` | `*.html`] 和 `Accept`头属性，而View的MIME来自于`View#getContentType`方法。

* HandlerExceptionResolver: **Spring容器内**异常处理器


#### 异常处理

### 处理流程

![](9BB6.tmp.jpg)

### 总结

## 参考

* [HTTP 基础与变迁](https://segmentfault.com/a/1190000006689489)
* [HTTP 缓存的四种风味与缓存策略](https://segmentfault.com/a/1190000006689795)
* [Spring MVC 入门示例讲解](http://www.importnew.com/15141.html)
* [SpringMVC系列之主要组件](http://www.cnblogs.com/xujian2014/p/5435471.html)
