# Spring Aop

在软件业，AOP为Aspect Oriented Programming的缩写，意为：**面向切面编程**，通过**预编译方式**和**运行期动态代理**实现程序功能的统一维护的一种技术。

## Aop机制

Aop的实现主要是通过修改**Class字节码**的方式实现。大致实现的方式有：

<table>
    <tr>
        <th>类型</th>
        <th>机制</th>
        <th>优点</th>
        <th>缺点</th>
    </tr>
    
    <!--静态Aop-->
    <tr>
        <td>静态Aop</td>
        <td>通过在<strong>编译期</strong>，直接修改字节码的方式完成Aop</td>
        <td>性能较好</td>
        <td>需要第三方编译器支持</td>
    </tr>
    
    <!--javaagent-->
    <tr>
        <td>javaagent</td>
        <td>利用Java 5提供新特性`Instrumentation`，开发者可以对加载的字节码进行转换。</td>
        <td>性能比较好，通用性强</td>
        <td>需要使用特殊命令启动Java</td>
    </tr>
    
    <!--ClassLoader-->
    <tr>
        <td>ClassLoader</td>
        <td>使用自定义的Cl加载Class文件的时候，修改字节码。</td>
        <td>性能比较好，通用性强</td>
        <td>需要自定义ClassLoader</td>
    </tr>
    
    <!--接口代理-->
    <tr>
        <td>接口代理</td>
        <td>使用JVM提供的动态代理的方式，为接口生成代理类</td>
        <td>实现比较简单</td>
        <td>仅仅支持<strong>接口</strong>代理</td>
    </tr>
    
    <!--子类代理-->
    <tr>
        <td>子类代理</td>
        <td>通过生成子类的方式，对目标进行代理</td>
        <td>实现比较简单，能对非接口类进行代理</td>
        <td><strong>同类相互调用Aop失效</strong></td>
    </tr>
</table>

## Aop术语

在Aop中存在一些标准术语：

* **JoinPoint**：拦截点，比如说某个业务方法。
* **PointCut**：描述Joinpoint的表达式，表示需要拦截哪些方法。
* **Advice**：需要切入的逻辑代码。比如说：调用统计等。

Aop处理器通过Pointcut表达式，搜索需要被切入的Jointpoint，然后织入Advice。

## Aop类库

在Java中存在各种类型的Aop类库：`AspectJ`，`cglib`，`javassist`，`spring aop`...。

看到这么多Aop实现，相信大家都会有点懵逼。这里来梳理一下它们的关系。

### AspectJ

**AspectJ**是一套Java Aop的解决方案，它的前身是AspectWerkz（2005年停止更新），主要组成为：

1. aspectjweaver.jar: AspectJ关于Aop术语模型，如：Pointcut，Advice等。
2. aspectjrt.jar：AspectJ运行时支持类。
3. ajc：AspectJ的编译器。

AspectJ是通过ajc编译.class文件，**静态**织入advice的方式实现Aop功能的。

### cglib

cglib(Code Generation Library)是一个开源项目，它可以在运行期扩展Java类与实现Java接口。

**原理**：通过**动态生成字节码**的方式，创建一个目标类的子类，然后注册到JVM中。

这样子，cglib就可以动态代理目标对象了。

注意：`cglib`和`javassist`比较类似，但是cglib基本停止开发了。

### Spring Aop

最初的Spring仅仅就是一个Ioc容器，后来，因为需要支持企业级开发。引入了Aop的功能：

1. aopalliance.jar：Aop 术语声明包。
2. cglib.jar：Aop具体实现类库。

通过`aopalliance`和`cglib`实现了最初**xml配置**类型的Spring Aop功能。

而在后来，为了减轻xml配置的繁琐性，引入了**声明式**Spring Aop：

1. aspectjweaver.jar: AspectJ的Aop术语声明包。
2. cglib.jar：Aop具体实现。

相比较，完整的AspectJ解决方案，声明式Spring Aop更加简单。

## AspectJ In Spring

为了使用注解是AspectJ的功能，我们需要先开启AspectJ功能模块：

* 注解方式：`@EnableAspectJAutoProxy`
* XML方式：`<aop:aspectj-autoproxy />`

这时候，Spring容器中就会添加一个后处理器：`AnnotationAwareAspectJAutoProxyCreator`。

我们来看一下比较简单的例子：

```java

@Component
//标记为Spring Aop类
@Aspect
public class MyAop {
    Logger logger = LoggerFactory.getLogger(this.getClass());

    //指定 org.darkfireworld.bean..* 下面所有的类
    @Pointcut("execution(* org.darkfireworld.bean..*.*(..))")
    void pointCut() {
    }

    //声明环绕通知
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        logger.error("invoke " + pjp.getSignature().toString() + " before");
        Object o = pjp.proceed();
        logger.error("invoke " + pjp.getSignature().toString() + " after");
        return o;
    }
}


@Component
public class ManImpl implements Man {
    Logger logger = LoggerFactory.getLogger(this.getClass());

    public void say() {
        logger.error("Say Crazy");
        test();
    }

    public void test() {
        logger.error("test Aop");
    }

}

```

这里最关心的就是切点表达式了，这里来介绍一下**@Pointcut表达式**：

> execution( modifier-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)

<ul>
    <li> modifier-pattern: 指定方法的修饰符，支持通配符，该部分可以省略</li>
    <li>ret-type-pattern: 指定返回值类型，支持通配符，可以使用"*"来通配所有的返回值类型</li>
    <li>declaring-type-pattern: 指定方法所属的类，支持通配符，该部分可以省略。<strong>注意，最后面需要添加"."字符。</strong></li>
    <li>name-pattern: 指定匹配的方法名，支持通配符，可以使用"*"来通配所有的方法名</li>
    <li>param-pattern: 指定方法的形参列表，支持两个通配符，"*"和".."，其中"*"代表一个任意类型的参数，而".."代表0个或多个任意类型的参数。</li>
    <li>throw-pattern: 指定方法声明抛出的异常，支持通配符，该部分可以省略</li>
</ul>

执行日志：

```

o.d.post.MyAop - invoke void org.darkfireworld.bean.impl.ManImpl.say() before
o.d.b.i.ManImpl - Say Crazy
o.d.b.i.ManImpl - test Aop
o.d.post.MyAop - invoke void org.darkfireworld.bean.impl.ManImpl.say() after

```

可以发现，在调用`ManImpl#say`的时候，Aop已经正常工作了。

观察上面的日志，可以发现，如果是**同类相互调用(say->test)，Aop将会失效**，这个就是使用Spring Aop动态代理的缺点之一。

## 源码剖析

### 注册后处理

在Spring4.x中，一般都是通过`AnnotationAwareAspectJAutoProxyCreator`这个后处理器来实现Spring Aop的支持了。

通过`@EnableAspectJAutoProxy`注解，容器会加载这个后处理器。

```java

EnableAspectJAutoProxy:

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Import(AspectJAutoProxyRegistrar.class)
    public @interface EnableAspectJAutoProxy {

        /**
         * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed
         * to standard Java interface-based proxies. The default is {@code false}.
         */
        boolean proxyTargetClass() default false;

    }

AspectJAutoProxyRegistrar:

    class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

        /**
         * Register, escalate, and configure the AspectJ auto proxy creator based on the value
         * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
         * {@code @Configuration} class.
         */
        @Override
        public void registerBeanDefinitions(
                AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
                
            // 注册 AnnotationAwareAspectJAutoProxyCreator 后处理器到容器中
            AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
            
            // 判断开启Aop代理的类型（强制cglib还是自适应）
            AnnotationAttributes enableAJAutoProxy =
                    AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
            if (enableAJAutoProxy.getBoolean("proxyTargetClass")) {
                AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
            }
        }

    }
```

调用流程如下：

1. 容器调用`ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry`后处理器，添加新的BeanDefinition到容器中。
2. `ConfigurationClassPostProcessor`使用**ConfigurationClassParser**这个帮助类，分析配置类信息。
3. `ConfigurationClassParser`调用**processImports**方法处理@Import注解信息。
4. 分析@EnableAspectJAutoProxy这个注解，然后调用`AspectJAutoProxyRegistrar#registerBeanDefinitions`方法，最后注册Aop后处理器到容器中。

到此，就完成了Aop后处理器`AnnotationAwareAspectJAutoProxyCreator`的注册流程了。


### 代理对象

通过注册`AnnotationAwareAspectJAutoProxyCreator`后处理器，就可以介入到bean生命周期中，从而实现Aop代理：

```java

AbstractAutoProxyCreator:

	/**
	 * Create a proxy with the configured interceptors if the bean is
	 * identified as one to proxy by the subclass.
	 *
     * 针对被标识的bean进行代理。
	 */
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (!this.earlyProxyReferences.contains(cacheKey)) {
                // 尝试进行代理bean
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}
    
AbstractAutoProxyCreator:

    /**
	 * Wrap the given bean if necessary, i.e. if it is eligible for being proxied.
	 *
     * 尝试代理给定的bean
	 */
	protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		...

		// Create proxy if we have advice.
        // 判断给定的bean是否具有切入代码
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
            // 创建代理对象
			Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}

		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}
```

通过后处理`postProcessAfterInitialization`，实现了对aop的实现，具体过程：

1. 通过`getAdvicesAndAdvisorsForBean`方法，获取给定bean的**Advice**代码片段。
2. 如果存在`Advice`，则通过`createProxy`创建代理对象。

我们先看`getAdvicesAndAdvisorsForBean`实现：

```java

AbstractAdvisorAutoProxyCreator:

	/**
	 * Return whether the given bean is to be proxied, what additional
	 * advices (e.g. AOP Alliance interceptors) and advisors to apply.
	 *
	 * 返回给定bean的advices。
	 */
	@Override
	protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, TargetSource targetSource) {
		List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
		if (advisors.isEmpty()) {
			return DO_NOT_PROXY;
		}
		return advisors.toArray();
	}

AbstractAdvisorAutoProxyCreator:

	/**
	 * Find all eligible Advisors for auto-proxying this class.
	 *
	 * 找到所有给定beanClass的Advisor
	 */
	protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
		// 寻找到所有的Advisor集合
		List<Advisor> candidateAdvisors = findCandidateAdvisors();
		// 寻找能匹配当前beanClass的Advisor集合。
		List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
		// 扩展给定Advisor集合，这里仅仅添加一个AspectJ指示器。
		extendAdvisors(eligibleAdvisors);
		if (!eligibleAdvisors.isEmpty()) {
			// 根据优先级排序Advisor集合
			eligibleAdvisors = sortAdvisors(eligibleAdvisors);
		}
		//返回Advisor集合
		return eligibleAdvisors;
	}
	
AnnotationAwareAspectJAutoProxyCreator:

	@Override
	protected List<Advisor> findCandidateAdvisors() {
		// Add all the Spring advisors found according to superclass rules.
		// 添加所有实现Spring Advisor方法的bean，支持FactoryBean<Advisor>类型。
		List<Advisor> advisors = super.findCandidateAdvisors();
		// Build Advisors for all AspectJ aspects in the bean factory.
		// 构造AspectJ类型的Advice，这里我们主要关注这个方法
		advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
		return advisors;
	}
	
BeanFactoryAspectJAdvisorsBuilder:

	/**
	 * Look for AspectJ-annotated aspect beans in the current bean factory,
	 * and return to a list of Spring AOP Advisors representing them.
	 * <p>Creates a Spring Advisor for each AspectJ advice method.
	 *
	 * 查询所有AspectJ注解类型的bean。并且返回Spring Aop Advisors集合。
	 */
	public List<Advisor> buildAspectJAdvisors() {
		List<String> aspectNames = null;

		synchronized (this) {
			aspectNames = this.aspectBeanNames;
			// 判断是否已经缓存过@Aspect的beanName
			if (aspectNames == null) {
				List<Advisor> advisors = new LinkedList<Advisor>();
				aspectNames = new LinkedList<String>();
				// 遍历容器中所有的bean定义
				String[] beanNames =
						BeanFactoryUtils.beanNamesForTypeIncludingAncestors(this.beanFactory, Object.class, true, false);
				for (String beanName : beanNames) {
					if (!isEligibleBean(beanName)) {
						continue;
					}
					// We must be careful not to instantiate beans eagerly as in this
					// case they would be cached by the Spring container but would not
					// have been weaved
					// 我们必须小心的处理这些bean，尽量避免初始化他们
					Class<?> beanType = this.beanFactory.getType(beanName);
					if (beanType == null) {
						continue;
					}
					// 判断当前beanType是否被@Aspect注解
					if (this.advisorFactory.isAspect(beanType)) {
						// 添加到aspectNames集合中
						aspectNames.add(beanName);
						AspectMetadata amd = new AspectMetadata(beanType, beanName);
						// 判断amd类型信息
						if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
							// 构造AspectJ织入类的元信息
							MetadataAwareAspectInstanceFactory factory =
									new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
							// 获取具体的Advisor列表。注意，返回的Advisor也会包裹AspectJ织
							// 入对象的beanName，只有在执行Advisor的时候，才会通过context#getBean获取它
							List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
							// 加入到相应的缓存中
							if (this.beanFactory.isSingleton(beanName)) {
								this.advisorsCache.put(beanName, classAdvisors);
							}
							else {
								this.aspectFactoryCache.put(beanName, factory);
							}
							advisors.addAll(classAdvisors);
						}
						else {
							...
						}
					}
				}
				// 添加到缓存中
				this.aspectBeanNames = aspectNames;
				return advisors;
			}
		}

		if (aspectNames.isEmpty()) {
			return Collections.emptyList();
		}
		// 通过 advisorsCache 和 advisorFactory 构造Advisor集合
		List<Advisor> advisors = new LinkedList<Advisor>();
		for (String aspectName : aspectNames) {
			List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
			if (cachedAdvisors != null) {
				advisors.addAll(cachedAdvisors);
			}
			else {
				MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
				advisors.addAll(this.advisorFactory.getAdvisors(factory));
			}
		}
		return advisors;
	}
```

上述的过程就是`getAdvicesAndAdvisorsForBean`的大致流程：

1. 获取bean factory中所有的Advisor集合，包括了：**Advisor和@Aspect**类型。
2. 获取Advisor集合中能**匹配**beanClass的Advisor。
3. **排序**Advisor集合

注意：**AspectJ织入对象只有在执行相应的Advisor的时候，才会通过context#getBean() 获取它。**
详细可见(断点)：`LazySingletonAspectInstanceFactoryDecorator#getAspectInstance`。

现在来看一下最后获取到的Advisor信息：

![](9513.tmp.jpg)

可以发现，自定义的`MyAop`已经被识别到了。

现在，我们在看看`createProxy`的过程：

```java

AbstractAutoProxyCreator:

	/**
	 * Create an AOP proxy for the given bean.
     * 
	 * 创建给定bean的Aop代理
	 */
	protected Object createProxy(
			Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {

		...
		//创建一个代理工厂
		ProxyFactory proxyFactory = new ProxyFactory();
		...
		
		// 通过拦截器构造Advisor集合
		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
		// 加入到代理工厂中
		for (Advisor advisor : advisors) {
			proxyFactory.addAdvisor(advisor);
		}
		// 设置代理目标
		proxyFactory.setTargetSource(targetSource);
		...
		// 通过代理工厂，获取具体bean的代理对象
		return proxyFactory.getProxy(getProxyClassLoader());
	}
	
ProxyFactory:

	/**
	 * Create a new proxy according to the settings in this factory.
	 * <p>Can be called repeatedly. Effect will vary if we've added
	 * or removed interfaces. Can add and remove interceptors.
	 * <p>Uses the given class loader (if necessary for proxy creation).
	 *
	 * 创建一个被代理的对象
	 */
	public Object getProxy(ClassLoader classLoader) {
		//1. 获取jdk或者cglib的代理支持
		//2. 构造具体的代理对象
		return createAopProxy().getProxy(classLoader);
	}
```

上述，就是`createProxy`的大致流程了。

## 小结

通过**AspectJ In Spring Aop**这种Aop方式已经可以应付大部分的场景了。当然，Spring Aop还有其他分支:

1. @EnableLoadTimeWeaving
2. Spring Advisor

这两种方式，其实和`AspectJ In Spring Aop`类似，也都是通过后处理器完成的。

## 扩展

`Aop`编程模式不仅仅在Java语言中有广泛的使用，在其他方面也有广泛的使用，比如说：INLINE HOOK（C/C++）等。
通过`Aop`编程模式，可以极大的减少重复代码量，实现`无侵入式`的监控，日志，事务等功能。

## 参考

* [AOP的实现机制](http://www.iteye.com/topic/1116696)
* [Spring AOP 实现原理与 CGLIB 应用](http://blog.jobbole.com/28791/)
* [Android平台inline hook实现](http://gslab.qq.com/article-168-1.html)