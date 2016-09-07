# Spring Mvc

## 例子

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
    /**
     * Spring MVC 配置
     */
    @Component
    @EnableWebMvc
    static class SpringMvcConf extends WebMvcConfigurerAdapter {

        // MessageConverter Support for @RequestBody, etc
        @Override
        public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
            // 添加JSON支持
            converters.add(new FastJsonHttpMessageConverter());
        }

        // 默认Servlet支持
        @Override
        public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
            configurer.enable();
        }

    }
}

```

在上述注解式配置中，通过`SpringMvcConf`配置了SpringMvc组件：

1. 默认ServlerHandler处理
2. FastJson HttpMessageConverter 

**控制器：**


```java


@Controller
@RequestMapping("/todo/TodoCtrl")
public class TodoCtrl {

    @RequestMapping("/hello")
    @ResponseBody
    public Message hello() {
        return Message.okMessage("Hello");
    }
}

```

上述，是一个简单的Ctrl控制器，用于返回Hello Json 字符串。

**测试：**

我们通过Jetty启动容器，然后通过浏览器测试：

![](6D95.tmp.jpg)

可以发现，我们的Spring Mvc已经正常工作了。

## 接口和类

### DispatcherServlet

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

### HandlerMapping

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

给定一个request对象，然后通过`HandlerMapping#getHandler`映射器，将获取到一个待执行`HandlerExecutionChain`对象，
之后，handler对象将通过`HandlerAdapter`处理器进行适配，然后进行具体的执行流程。

注意：如果`HandlerMapping#getHandler`返回`null`，则表示该映射器无法处理该request对象。

### HandlerExecutionChain


```java

public class HandlerExecutionChain {

	
    // handler
	private final Object handler;
    
    // 拦截器
	private HandlerInterceptor[] interceptors;
    
    ....
}

```

通过`HandlerExecutionChain`对象，包裹handler以及相应的拦截器。

### HandlerInterceptor

```java

/**
 * 拦截器
 */
public interface HandlerInterceptor {

	/**
	 * Intercept the execution of a handler. Called after HandlerMapping determined
	 * an appropriate handler object, but before HandlerAdapter invokes the handler.
	 *
	 * handler的拦截器。
	 *
	 * 当HandlerMapping筛选出合适的handler对象后，但是在HandlerAdapter调
	 * 用这个handler之前，该拦截器方法将被调用。
	 *
	 * 注意：如果返回false，则表示拦截器已经完成对本次请求的处理。
	 */
	boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
	    throws Exception;

	/**
	 * Intercept the execution of a handler. Called after HandlerAdapter actually
	 * invoked the handler, but before the DispatcherServlet renders the view.
	 * Can expose additional model objects to the view via the given ModelAndView.
	 *
	 * handler的拦截器。
	 *
	 * 当HandlerAdapter已经调用完handler，但是在DispatcherServlet渲染view之前，
	 * 该拦截器方法将被调用。
	 *
	 * 注意：通过这个方法，可以向ModelAndView中添加额外的参数。
	 */
	void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
			throws Exception;

	/**
	 * Callback after completion of request processing, that is, after rendering
	 * the view. Will be called on any outcome of handler execution, thus allows
	 * for proper resource cleanup.
	 *
	 * 当请求被处理完成后（view渲染完毕，preHandle返回false，抛出异常），通过这个方法
	 * 可以清理资源。
	 *
	 * 注意：相对于preHandle的调用顺序，该方法是逆序调用的。
	 */
	void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception;

}


```

通过`HandlerInterceptor`可以实现J2EE中的Filter的功能，从而拦截handler的执行。

注意：SpringMVC的**CORS（跨域）**就是通过拦截器实现的。详细可见：`CorsInterceptor`。

### HandlerAdapter


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

* RequestMappingHandlerAdapter: 最新的`HandlerAdapter`子类，支持如下特性：
	1. **参数解析器(HandlerMethodArgumentResolver)**: 提供参数注入(`@RequestParam`,`@RequestBody`,`HttpSession`,etc )支持。
	2. **结果解析器(HandlerMethodReturnValueHandler)**: 提供结果处理(`@ResponseBody`,`StreamingResponseBody`,etc)支持。
	3. **转换器(HttpMessageConverter)**: 此特性被包含于`参数解析器`和`结果解析器`中。详情可见:`RequestResponseBodyMethodProcessor`。

* HttpRequestHandlerAdapter: handler类型为`HttpRequestHandler`（如：`DefaultServletHttpRequestHandler`）的适配器。

通过`HandlerAdapter#handle`方法，可以**修饰/处理**来自于`HandlerMapping#getHandler`获取的handler对象。

注意：如果`HandlerAdapter#handle`返回null，则表示该请求已经被处理完成。

### ViewResolver

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

### View

```java

/**
 * 视图渲染类
 */
public interface View {

	...

	/**
	 * Return the content type of the view, if predetermined.
	 * <p>Can be used to check the content type upfront,
	 * before the actual rendering process.
	 * @return the content type String (optionally including a character set),
	 * or {@code null} if not predetermined.
	 *
	 * 返回该View渲染出来的content的Content-Type属性（Response#Header）。
	 */
	String getContentType();

	/**
	 * Render the view given the specified model.
	 *
	 * 给定model然后渲染它
	 */
	void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;

}

```

通过`View`的子类（FastJsonJsonView，FreeMarkerView，...）可以完成对`model`的渲染。

### HandlerExceptionResolver

```java
/**
 * Interface to be implemented by objects that can resolve exceptions thrown during
 * handler mapping or execution, in the typical case to error views. Implementors are
 * typically registered as beans in the application context.
 *
 */
public interface HandlerExceptionResolver {

	/**
	 * Try to resolve the given exception that got thrown during handler execution,
	 * returning a {@link ModelAndView} that represents a specific error page if appropriate.
	 *
	 * 尝试解决在执行过程中抛出的异常，并且返回ModelAndView对象，用来渲染错误页面
	 * 
	 * 注意：如果返回null，则按照默认流程处理。
	 */
	ModelAndView resolveException(
			HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex);

}

```

通过`HandlerExceptionResolver`接口，可以处理在执行过程中发生的**异常**，比如说：未授权，系统错误。

## 注解配置

在最新的SpringMVC中，可以使用`@EnableWebMvc`注解开启SpringMVC特性：

```java

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}


```

可见`@EnableWebMvc`将会Import具体的MVC配置类`DelegatingWebMvcConfiguration`：

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();


	@Autowired(required = false)
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (configurers == null || configurers.isEmpty()) {
			return;
		}
		this.configurers.addWebMvcConfigurers(configurers);
	}

    // 具体的配置
    ....

}

```

可以发现`DelegatingWebMvcConfiguration`具有如下特性：

1. 被`@Configuration`标记了，这表明它是一个**配置类***。
2. 通过`setConfigurers`方法，注入了容器中所有的`WebMvcConfigurerAdapter`继承类[MVC 配置类]。
3. 在它的父类`WebMvcConfigurationSupport`中，存在@Bean标记的方法。

这样子，通过SpringIoc的**注解式配置**特性，就可以将SpringMVC相关的Bean导入到容器中来了。

## 初始化

首先，我们来看看`DispatcherServlet`的继承图为：

![](277C.tmp.jpg)

可以发现`DispatcherServlet`继承了`HttpSerlvet`接口。

通过J2EE容器(Jetty)调用`HttpSerlvet#init`方法，就实现了SpringMVC的初始化：

```java

HttpServletBean:

    @Override
	public final void init() throws ServletException {
		if (logger.isDebugEnabled()) {
			logger.debug("Initializing servlet '" + getServletName() + "'");
		}

		// Set bean properties from init parameters.
        // 设置基础的环境配置
		try {
			PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
			BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
			ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
			bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
			initBeanWrapper(bw);
			bw.setPropertyValues(pvs, true);
		}
		catch (BeansException ex) {
			logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
			throw ex;
		}

		// Let subclasses do whatever initialization they like.
        // 调用子类初始化方法
		initServletBean();

		if (logger.isDebugEnabled()) {
			logger.debug("Servlet '" + getServletName() + "' configured successfully");
		}
	}
    
FrameworkServlet：

    /**
	 * Overridden method of {@link HttpServletBean}, invoked after any bean properties
	 * have been set. Creates this servlet's WebApplicationContext.
	 */
	@Override
	protected final void initServletBean() throws ServletException {
		...
		try {
            // 初始化 WebApplicationContext 对象
			this.webApplicationContext = initWebApplicationContext();
			...
		}
		...
	}
    
FrameworkServlet：   

    protected WebApplicationContext initWebApplicationContext() {
		...
        
		WebApplicationContext wac = null;

		...
		if (wac == null) {
			// No context instance was injected at construction time -> see if one
			// has been registered in the servlet context. If one exists, it is assumed
			// that the parent context (if any) has already been set and that the
			// user has performed any initialization such as setting the context id
            // 查询是否已经存在一个WebApplicationContext对象
			wac = findWebApplicationContext();
		}
		if (wac == null) {
			// No context instance is defined for this servlet -> create a local one
            // 创建一个WebApplicationContext对象
			wac = createWebApplicationContext(rootContext);
		}

		if (!this.refreshEventReceived) {
			// Either the context is not a ConfigurableApplicationContext with refresh
			// support or the context injected at construction time had already been
			// refreshed -> trigger initial onRefresh manually here.
            // 如果无法通过刷新事件调用onRefresh方法，则主动触发该方法。
			onRefresh(wac);
		}

		...

		return wac;
	}
```

通过`initWebApplicationContext`方法，我们实现了Spring容器的初始化，大致流程为：

1. 调用`findWebApplicationContext`方法，查询是否存在`WebApplicationContext`实例。
2. 如果不存在wac实例，则调用`createWebApplicationContext`方法，创建新的wac实例。
3. 如果无法通过刷新事件调用onRefresh方法，则主动触发onRefresh方法，**初始化子类特性**。

一般来说，我们都会进入`createWebApplicationContext`方法，完成容器的初始化：

```java

FrameworkServlet：

	protected WebApplicationContext createWebApplicationContext(WebApplicationContext parent) {
        // 创建上下文
		return createWebApplicationContext((ApplicationContext) parent);
	}
    
FrameworkServlet：

    protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
        // 获取上下文Class类型，默认为XmlWebApplicationContext.class上下文。
        // 这里为web.xml中通过contextClass指定的AnnotationConfigWebApplicationContext上下文对象
		Class<?> contextClass = getContextClass();
		...
        // 实例化
		ConfigurableWebApplicationContext wac =
				(ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
        // 设置环境
		wac.setEnvironment(getEnvironment());
        // 设置父容器，如果有的话
		wac.setParent(parent);
        // 设置配置对象，这里为web.xml中通过contextConfigLocation指定的SpringConf配置类
		wac.setConfigLocation(getContextConfigLocation());
        // 刷新wac容器
		configureAndRefreshWebApplicationContext(wac);

		return wac;
	}

FrameworkServlet：

    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
		...
        
        // 设置ServletContext
		wac.setServletContext(getServletContext());
        // 设置ServletConfig
		wac.setServletConfig(getServletConfig());
        // 设置命名空间
		wac.setNamespace(getNamespace());
        // 设置容器初始化监听器，用来调用onRefresh()方法
		wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));

		...
        
        // 容器初始化
		wac.refresh();
	}
```

通过`createWebApplicationContext`方法，实现了如下的流程：

1. 设置ServletContent实例
2. 设置ServletConfig实例
3. 设置容器刷新监听器，通过它，将会调用`FrameworkServlet#onRefresh`方法。
4. 刷新容器

当容器刷新完成后，会发布**ApplicationEvent**事件，此时，会触发`DispatcherServlet#onRefresh`方法：

```java

DispatcherServlet：

    @Override
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}
    
DispatcherServlet：

	/**
	 * Initialize the strategy objects that this servlet uses.
	 * <p>May be overridden in subclasses in order to initialize further strategy objects.
	 */
	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
    
```

通过**onRefresh**方法，我们就完成了SpringMVC中核心调度器`DispatcherServlet`的初始化。

## 处理流程

![](9BB6.tmp.jpg)

可以发现`DispatcherServlet`在整个SpringMVC中起着**调度器**的作用，核心代码：

```java

    /**
	 * Process the actual dispatching to the handler.
	 */
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		...

		try {
            // 解析出来的mv
			ModelAndView mv = null;
            // 异常
			Exception dispatchException = null;

			try {
				...
				// Determine handler for the current request.
                // 通过request对象，获取一个handler
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    // 没有发现合适的handler，则报错404
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
                // 选择一个合适的HandlerAdapter
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
                // 处理last-modified这个header属性，如果handler支持它。
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
                // 调用拦截器的preHandle方法
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
                // 通过HandlerAdapter实际处理handler, mv可能为null
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                
                ...
                // 如果没有给定viewName，则赋予一个默认的viewName
				applyDefaultViewName(processedRequest, mv);
                // 调用拦截器的postHandle方法
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
                // 如果捕获异常，则记录这个异常
				dispatchException = ex;
			}
            // 1. 如果存在异常，则进行处理异常
            // 2. 如果存在mv，则进行render view
            // 3. 调用拦截器的afterCompletion方法
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
            
		}catch (Exception ex) {
            ...
		}
		...
	}

```

调度函数`doDispatch`的大致流程为：

1. 通过`getHandler`方法，获取一个`HandlerExecutionChain`对象（包含：handler，拦截器）。
2. 通过`getHandlerAdapter`方法，获取handler相对应的**适配器**。
3. 对**last-modified**属性支持，默认忽略。
4. 调用拦截器的`preHandle`方法
5. 通过适配器ha调用handler对象，进行真正的业务处理。
6. 调用拦截器的`postHandle`方法
7. 如果在上述过程中发生异常，则记录它。
8. 调用`processDispatchResult`方法处理获取到的`ModelAndView mv`和`Exception dispatchException`。
9. 调用拦截器的`afterCompletion`方法。

到此一个HTTP请求就被处理完成了。

### getHandler

```java

	/**
	 * Return the HandlerExecutionChain for this request.
	 * <p>Tries all handler mappings in order.
	 *
     * 针对当前的request返回一个HandlerExecutionChain对象。
     * 所有的handler mappings都是通过Order排序的。
     * 
     * 如果返回null则表示没有查询到合适的handler。
	 */
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		for (HandlerMapping hm : this.handlerMappings) {
			if (logger.isTraceEnabled()) {
				logger.trace(
						"Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
			}
			HandlerExecutionChain handler = hm.getHandler(request);
			if (handler != null) {
				return handler;
			}
		}
		return null;
	}
```

而具体被查询的`handlerMappings`为：

![](D659.tmp.jpg)

通过`HandlerMapping`将为当前request匹配一个合适的handler对象。

注意：提供**<mvc:default-servlet-handler/>**支持的`HandlerMapping`排序在最后。

### getHandlerAdapter

```java

	/**
	 * Return the HandlerAdapter for this handler object.
	 *
     * 返回一个支持当前handler的HandlerAdapter对象，如果没有，则抛出异常。
	 */
	protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		for (HandlerAdapter ha : this.handlerAdapters) {
			if (logger.isTraceEnabled()) {
				logger.trace("Testing handler adapter [" + ha + "]");
			}
			if (ha.supports(handler)) {
				return ha;
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}

```

而具体被查询的`handlerAdapters`为：

![](9A26.tmp.jpg)

通过调用`HandlerAdapter#supports`判断是否支持handler对象。

### processDispatchResult

```java

    /**
	 * Handle the result of handler selection and handler invocation, which is
	 * either a ModelAndView or an Exception to be resolved to a ModelAndView.
     *
     * 处理之前调用handler产生的 ModelAndView mv 以及 Exception ex 结果
	 */
	private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

		boolean errorView = false;

		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
                // ModelAndViewDefiningException 类型的异常，直接读取mv信息
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
                // 进行异常处理
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// Did the handler return a view to render?
        // 决定是否进行异常处理：mv != null
		if (mv != null && !mv.wasCleared()) {
            // 渲染视图
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
						"': assuming HandlerAdapter completed request handling");
			}
		}

		...

		if (mappedHandler != null) {
            // 调用拦截器的 afterCompletion 方法
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}


```

通过`processDispatchResult`方法，可以将之前调用的结果(Exception ex，ModelAndView mv)进行处理：

1. 如果`Exception ex != null`则调用`processHandlerException`进行处理异常。
2. 如果`ModelAndView mv != null`则调用`render`进行渲染视图。
3. 最后调用拦截器的`afterCompletion`方法。

注意：**当异常类型为`ModelAndViewDefiningException`的时候，会直接获取mv数值，不会使用`HandlerExceptionResolver`处理它。**

### processHandlerException

```java

    /**
	 * Determine an error ModelAndView via the registered HandlerExceptionResolvers.
	 *
	 */
	protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
			Object handler, Exception ex) throws Exception {

		// Check registered HandlerExceptionResolvers...
		ModelAndView exMv = null;
        // 遍历异常处理链
		for (HandlerExceptionResolver handlerExceptionResolver : this.handlerExceptionResolvers) {
            // 尝试处理异常，如果处理成功，则返回mv
			exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
			if (exMv != null) {
				break;
			}
		}
		if (exMv != null) {
			...
		}

		throw ex;
	}
    
```

通过`processHandlerException`方法，遍历了已注册的**异常处理链**。如果其中的`handlerExceptionResolver`解决了该异
常，则返回一个`ModelAndView exMv`。

### render

```java

    /**
	 * Render the given ModelAndView.
	 *
     * 渲染给定的ModelAndView对象
	 */
	protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
		// Determine locale for request and apply it to the response.
        // 选择一个本地化对象
		Locale locale = this.localeResolver.resolveLocale(request);
		response.setLocale(locale);

		View view;
        // 判断view是否为String类型，也就是viewName。
		if (mv.isReference()) {
			// We need to resolve the view name.
            // 我们将通过已注册的ViewResolver获取合适的View进行渲染
			view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
			if (view == null) {
				throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
						"' in servlet with name '" + getServletName() + "'");
			}
		}
		else {
			// No need to lookup: the ModelAndView object contains the actual View object.
            // 直接获取View对象
			view = mv.getView();
			if (view == null) {
				throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
						"View object in servlet with name '" + getServletName() + "'");
			}
		}

		...
		try {
            // 渲染
			view.render(mv.getModelInternal(), request, response);
		}
		catch (Exception ex) {
			....
		}
	}


```

通过`render`方法，可以将mv数据进行渲染，大致流程如下：

1. 获取一个合适的View对象。
2. 通过View对象渲染mv中的model数据。

注意：通过`ViewResolver#resolveViewName`方法获取View的过程，**并非一定依赖viewName**。如：`ContentNegotiatingViewResolver`。


## 扩展知识

### url-pattern

通过**url-pattern**可以配置Servlet/Filter的路径，大致分为如下几种格式：

1. 精确路径：/a/b/c
2. 最长路径：/a/*
3. 扩展路径：*.do
4. 默认路径：/

当一个HTTP请求过来的时候，会按照上述的优先级进行匹配。

注意：如果条目1-3都未能匹配URL，则会采用**条目4(默认路径)**进行匹配URL。

### DefaultServlet

在Tomcat和Jetty中存在`DefaultServlet`:

> This servlet, normally mapped to / , provides the handling for static content, 
> OPTION and TRACE methods for the context.


简单的来说，通过`DefaultServlet`可以处理**静态资源**。

### error-page

通过**error-page**标签，可以配置一些常见的错误页面，避免直接将错误信息暴露给用户：

```xml


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

当容器接受到**异常，404，500**等错误的时候，容器处理流程如下：

1. 检测`web.xml`是否存在相应的`error-page`条目。
2. 如果存在，则使用`location`地址，对请求进行**forward**处理，从而会再次发起**路由过程**
3. 如果不存在，则使用**默认错误**页面，写入到response中。

注意：SpringMVC的`HandlerExceptionResolver`并不能处理**render时期的异常**。

### flush

当**第一次**调用`ServletResponse#getOutputStream#flush`或者`ServletResponse#getWriter#flush`的时候，
**会优先向客户端传输 RESPONSE HEADERS**，并且标记RESPONSE为**COMMITTED状态**。

注意：调用`flush`方法之后，对**RESPONSE HEADERS**修改操作将无效。

### Response Ways 

1. 动态-模版: 标准MVC模式。
2. 动态-ANY: 遇到标准MVC模式无法处理的情况(JSON, XML, 动态文件下载, etc)时，可以直接在Ctrl中生成响应报文。
3. 静态页面(默认): 当请求静态文件的时候，则可以直接返回它。

良好的**响应方式**设计，有利于提高维护性。

## 参考

* [HTTP 基础与变迁](https://segmentfault.com/a/1190000006689489)
* [HTTP 缓存的四种风味与缓存策略](https://segmentfault.com/a/1190000006689795)
* [Spring MVC 入门示例讲解](http://www.importnew.com/15141.html)
* [SpringMVC系列之主要组件](http://www.cnblogs.com/xujian2014/p/5435471.html)
