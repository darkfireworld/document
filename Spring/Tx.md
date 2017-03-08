# Spring Tx

通过 Spring 事务，我们可以非常方便地对应用进行事务管理。

## 例子

现在，我们来介绍一下Spring事务的简单使用。

首先，我们需要简单的配置一下Spring：

```java

	@Component
	//@EnableTransactionManagement 相当于 <tx:annotation-driven/>
    @EnableTransactionManagement
    static class IoConf {
        @Autowired
        Environment environment;

        // 配置数据源
        @Bean(initMethod = "init", destroyMethod = "close")
        public DataSource dataSource() throws Exception {
            DruidDataSource druidDataSource = new DruidDataSource();
            druidDataSource.setDriverClassName(environment.getProperty("jdbc-class"));
            druidDataSource.setUrl(environment.getProperty("jdbc-url"));
            druidDataSource.setUsername(environment.getProperty("jdbc-username"));
            druidDataSource.setPassword(environment.getProperty("jdbc-password"));
            druidDataSource.setFilters("log4j");
            return druidDataSource;
        }

        // 配置事务管理器
        @Bean
        public PlatformTransactionManager transactionManager(DataSource dataSource) {
            return new DataSourceTransactionManager(dataSource);
        }

        // 配置JdbcTemplate
        @Bean
        public JdbcTemplate template(DataSource dataSource) {
            return new JdbcTemplate(dataSource);
        }
    }
	
```

上述就是一个非常简单的配置，它包括了：

1. 开启**注解式**事务特性。
2. 配置MySQL数据源
3. 配置DataSource事务管理器
4. 配置JdbcTemplate这个Jdbc访问类

然后，我们添加一个业务代码：

```java


@Component
public class ArticleIo {
    @Autowired
    JdbcTemplate template;

    /**
     * 获取指定ID的文本
     *
     * @param id ID
     */
    public String select(String id) {
        return template.queryForObject("SELECT content FROM t_article WHERE id=?", new Object[]{id}, String.class);
    }

    /**
     * 更新一条记录
     *
     * @param id      ID
     * @param content 更新的内容
     */
    // 开启事务
    @Transactional
    public void update(String id, String content) {
        template.update("UPDATE t_article SET content=? WHERE id=?", content, id);
        //构造一次异常，测试事务特性
        throw new RuntimeException("Broke It");
    }
}


```

然后，我们编写测试用例：

```java

// 使用Spring Junit4 支持
@RunWith(SpringJUnit4ClassRunner.class)
// 加载Spring配置类
@ContextConfiguration(classes = {SpringConf.class})
public class ArticleIoTest {

    
    @Autowired
    ArticleIo articleIo;

    @Test
    public void testNotBroke() {
        System.out.println();
        System.out.println();
        System.out.println("Test Not Broke");
        System.out.println("BEFORE：" + articleIo.select("1"));
        try {
            // 随机生成crc32文本
            String randomText = Hashing.crc32().hashString(String.valueOf(System.currentTimeMillis()), Charset.defaultCharset()).toString();
            // 更新id=1的记录
            articleIo.update("1", randomText, false);
        } catch (Exception e) {
        }

        System.out.println("AFTER：" + articleIo.select("1"));
    }

    @Test
    public void testBroke() {
        System.out.println();
        System.out.println();
        System.out.println("Test Broke");
        System.out.println("BEFORE：" + articleIo.select("1"));
        try {
            // 随机生成crc32文本
            String randomText = Hashing.crc32().hashString(String.valueOf(System.currentTimeMillis()), Charset.defaultCharset()).toString();
            // 更新id=1的记录
            articleIo.update("1", randomText, true);
        } catch (Exception e) {
        }

        System.out.println("AFTER：" + articleIo.select("1"));
    }
}

```

执行结果：

```
Test Not Broke
BEFORE：初始化数据
AFTER：7d6dc0b2

Test Broke
BEFORE：7d6dc0b2
AFTER：7d6dc0b2

```

可以发现，通过**@Transactional**标记的方法，具有了事务特性了。


## @Transactional

```java

public @interface Transactional {

	/**
	 * Alias for {@link #transactionManager}.
     * 
	 * 属性transactionManager的别名
	 */
	@AliasFor("transactionManager")
	String value() default "";

	/**
	 * A <em>qualifier</em> value for the specified transaction.
	 * <p>May be used to determine the target transaction manager,
	 * matching the qualifier value (or the bean name) of a specific
	 * {@link org.springframework.transaction.PlatformTransactionManager}
	 * bean definition.
	 *
	 * 指定使用的事务管理器(PlatformTransactionManager)的名字。
	 */
	@AliasFor("value")
	String transactionManager() default "";

	/**
	 * The transaction propagation type.
	 * <p>Defaults to {@link Propagation#REQUIRED}.
	 * 
	 * 定义事务的传播属性
	 */
	Propagation propagation() default Propagation.REQUIRED;

	/**
	 * The transaction isolation level.
	 * <p>Defaults to {@link Isolation#DEFAULT}.
	 *
	 * 定义事务的隔离级别
	 */
	Isolation isolation() default Isolation.DEFAULT;

	/**
	 * The timeout for this transaction.
	 * <p>Defaults to the default timeout of the underlying transaction system.
	 *
	 * 定义超时时间
	 */
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	/**
	 * {@code true} if the transaction is read-only.
	 *
	 * 定义事务是否为只读类型事务。默认为false。
	 */
	boolean readOnly() default false;

	/**
	 * Defines zero (0) or more exception {@link Class classes}, which must be
	 * subclasses of {@link Throwable}, indicating which exception types must cause
	 * a transaction rollback.
	 *
	 * 定义需要回滚的异常类型。默认为Throwable。
	 */
	Class<? extends Throwable>[] rollbackFor() default {};

	...

}

```

在Spring注解式事务中，**@Transactional**是非常关键的注解，通过这个注解，规定了执行事务的特性：

1. 声明使用的**事务管理器**。默认为：名字为`transactionManager`指向的bean。
2. 定义**事务传播属性**。默认为：REQUIRED 属性。
3. 定义**事务隔离级别**。默认为：数据库默认事务隔离级别，MySQL为RR级别。
4. 定义执行超时时间。默认为：无限时。

### 事务管理器

在一个应用中，可以存在多种类型的**事务管理器**：

1. Jms事务管理器
2. DataSource事务管理器
3. Jta事务管理器

即使同一类型的事务管理器，也可能存在多个不同的实例：

1. MySQL_1 的DataSource事务管理器。
2. MySQL_2 的DataSource事务管理器。
3. Oracle_1 的DataSource事务管理器。

所以，当存在**多个**事务管理器的时候，那么使用`@Transactional`注解需要指定事务管理器。

### 事务传播

`事务传播`这个概念是基于应用层的，而**非数据库级别**。在Spring中，大致存在如下类型：

1. **REQUIRED**（默认）: 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。
2. **SUPPORTS**: 支持当前事务，如果当前没有事务，就以非事务方式执行。
3. **MANDATORY**: 使用当前的事务，如果当前没有事务，就抛出异常。
4. **REQUIRES_NEW**: 新建事务，如果当前存在事务，把当前事务挂起。
5. **NOT_SUPPORTED**: 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6. **NEVER**: 以非事务方式执行，如果当前存在事务，则抛出异常。

合理的定义`事务传播`特性，能优化应用的整体性能。

### 隔离级别

`隔离级别`是数据库中的概念，它是为了解决如下问题所提出的：

1. **脏读(Dirty Read)**: 可以读取未提交的数据。
2. **不可重复读(Unrepeatable Read)**: 同一个事务中多次执行同一个select, 读取到的**数据**发生修改。
3. **幻读(Phantom Read)**: 同一个事务中多次执行同一个select, 读取到的**数据行**发生改变。

而`ANSI SQL`定义了如下几个隔离级别：

<table>
	<tr>
		<th>Isolation Level</th>
		<th>Dirty Read</th>
		<th>Unrepeatable Read</th>
		<th>Phantom Read</th>
	</tr>
	<tr>
		<td>Read UNCOMMITTED</td>
		<td>YES</td>
		<td>YES</td>
		<td>YES</td>
	</tr>
	<tr>
		<td>READ COMMITTED</td>
		<td>NO</td>
		<td>YES</td>
		<td>YES</td>
	</tr>
	<tr>
		<td>READ REPEATABLE</td>
		<td>NO</td>
		<td>NO</td>
		<td>YES</td>
	</tr>
	<tr>
		<td>SERIALIZABLE</td>
		<td>NO</td>
		<td>NO</td>
		<td>NO</td>
	</tr>
</table>

注意：在MySQL中默认采用`READ REPEATABLE`默认。


## 接口和类

这里，我们先介绍一些重要的类和接口。

**基础模块：**

* PlatformTransactionManager: 事务管理器接口。
* TransactionDefinition: 事务定义接口。
* TransactionStatus: 事务状态接口。

上述三个都是接口类型，是Spring事务的核心接口类。

**通用模块：**

* AbstractPlatformTransactionManager: 默认事务管理器。
* DefaultTransactionDefinition: 默认事务定义类。
* DefaultTransactionStatus: 默认事务状态类。
* TransactionSynchronization: 事务同步-回调。
* TransactionSynchronizationManager: 事务同步管理器-当前事务资源管理。

上述的类和接口都是比较通用的模块，一般都是继承于它们实现具体的功能。

![](A676.tmp.jpg)

可以发现`AbstractPlatformTransactionManager`的实现类就是各种数据源的事务管理器。

**拦截模块：**

* TransactionProxyFactoryBean: 通过手动配置被拦截的方法。
* BeanFactoryTransactionAttributeSourceAdvisor: `Advisor`接口的实现类，事务注入管理器。
* TransactionInterceptor: 事务拦截器具体业务代码。
* DefaultTransactionAttribute: `TransactionAttribute`的实现类。
* TransactionAttribute: `TransactionDefinition`的子接口，通过定义`getQualifier`方法，指定使用的事务管理器。
* TransactionAttributeSource: 给定一个`Method`，然后读取`TransactionAttribute`信息。

通过`BeanFactoryTransactionAttributeSourceAdvisor`或者`TransactionProxyFactoryBean`对目标方法
使用`TransactionInterceptor`进行拦截，从而实现事务特性。

**注解模块：**

* @EnableTransactionManagement: 注解式配置的事务注解。
* AnnotationTransactionAttributeSource:`TransactionAttributeSource`的实现类，用于管理`TransactionAnnotationParser`解析器。
* TransactionAnnotationParser: 事务注解分析器。
* SpringTransactionAnnotationParser: Spring @Transactional事务注解扫描
* JtaTransactionAnnotationParser: Jta事务注解扫描。
* @Transactional: Spring 事务注解
* Isolation: 事务隔离级别
* Propagation: 事务传播属性

通过上述的注解，就可以通过注解的方式，完成Spring事务管理了。

## 源码分析

接下来，我们来分析一下**Spring注解式事务**的实现过程，大致过程如下：

1. 加载阶段：加载事务相关Bean（数据源，事务管理器，事务Aop处理器，JdbcTemplate）到容器中。
2. 注入阶段：利用Spring Aop特性，注入事务代码到**目标方法**（如：@Transactional标记方法）
3. 执行阶段: 调用具有事务管理功能的目标方法。

注意：在本文中，默认采用**注解式事务**处理。

### 加载阶段

在上述的DEMO中，我们采用`@EnableTransactionManagement`的方式开启Spring事务特性：

```java

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
// 导入 TransactionManagementConfigurationSelector 选择器
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {

	/**
	 * Indicate whether subclass-based (CGLIB) proxies are to be created ({@code true}) as
	 * opposed to standard Java interface-based proxies ({@code false}). The default is
	 *
     * AOP模式选择 
	 */
	boolean proxyTargetClass() default false;

	/**
	 * Indicate how transactional advice should be applied. The default is
	 * {@link AdviceMode#PROXY}.
	 * @see AdviceMode
	 */
	AdviceMode mode() default AdviceMode.PROXY;

	/**
	 * Indicate the ordering of the execution of the transaction advisor
	 * when multiple advices are applied at a specific joinpoint.
	 *
     * 优先级
	 */
	int order() default Ordered.LOWEST_PRECEDENCE;

}

```

然后通过**@Import**注解，(解析代码：`ConfigurationClassParser#collectImports`)，引入了`@TransactionManagementConfigurationSelector`类：

```java

public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

	/**
	 * {@inheritDoc}
	 * @return {@link ProxyTransactionManagementConfiguration} or
	 * {@code AspectJTransactionManagementConfiguration} for {@code PROXY} and
	 * {@code ASPECTJ} values of {@link EnableTransactionManagement#mode()}, respectively
	 */
	@Override
	protected String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] {AutoProxyRegistrar.class.getName(), ProxyTransactionManagementConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME};
			default:
				return null;
		}
	}

}


```

而在`TransactionManagementConfigurationSelector`中，通过`AdviceMode`做出选择，到底使用什么模式（AspectJ,Proxy）处理Aop，默认为**PROXY模式**：

```java


@Configuration
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {

	@Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
		BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
		advisor.setTransactionAttributeSource(transactionAttributeSource());
		advisor.setAdvice(transactionInterceptor());
		advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
		return advisor;
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionAttributeSource transactionAttributeSource() {
		return new AnnotationTransactionAttributeSource();
	}

	@Bean
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public TransactionInterceptor transactionInterceptor() {
		TransactionInterceptor interceptor = new TransactionInterceptor();
		interceptor.setTransactionAttributeSource(transactionAttributeSource());
		if (this.txManager != null) {
			interceptor.setTransactionManager(this.txManager);
		}
		return interceptor;
	}

}


```

通过`ProxyTransactionManagementConfiguration`配置类，加载了几个Bean定义：

1. BeanFactoryTransactionAttributeSourceAdvisor: `Advisor`接口的实现类，事务注入管理器。
2. TransactionAttributeSource: 从Method中读取事务注解信息。
3. TransactionInterceptor: 拦截器，也就是具体的事务Aop代码。

可以发现，`BeanFactoryTransactionAttributeSourceAdvisor`相当于事务注入的管理器，将`TransactionAttributeSource`
和`TransactionInterceptor`整合在一起。

### 注入阶段

通过利用Spring Aop的特性，可以非常方便的对一个Bean注入事务代码，流程如下：

1. 在Bean完成创建后，会调用`BeanPostProcessor#postProcessAfterInitialization`后处理，此时**Spring Aop后处理**将会被调用到。
2. Spring Aop读取容器中所有**Advisor**的实现类，其中就包括了`BeanFactoryTransactionAttributeSourceAdvisor`这个事务Aop管理器。
3. `BeanFactoryTransactionAttributeSourceAdvisor`委托`TransactionAttributeSource`接口，寻找待注入的**目标方法**。
4. 如果发现**目标方法**，则将注入`TransactionInterceptor`这个事务拦截器到目标方法。

这样子，就完成了对**目标方法**事务管理。

### 执行阶段

在调用**目标方法**的时候，都会调用`TransactionInterceptor#invoke`方法进行拦截:

```java

TransactionInterceptor:

	@Override
	public Object invoke(final MethodInvocation invocation) throws Throwable {
		// Work out the target class: may be {@code null}.
		// The TransactionAttributeSource should be passed the target class
		// as well as the method, which may be from an interface.
		Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);

		// Adapt to TransactionAspectSupport's invokeWithinTransaction...
        // 调用父类的invokeWithinTransaction方法，实现事务管理.
		return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {
			@Override
			public Object proceedWithInvocation() throws Throwable {
				return invocation.proceed();
			}
		});
	}
    
TransactionAspectSupport:

    /**
	 * General delegate for around-advice-based subclasses, delegating to several other template
	 * methods on this class. Able to handle {@link CallbackPreferringPlatformTransactionManager}
	 * as well as regular {@link PlatformTransactionManager} implementations.
	 *
     * 通用事务代理处理方法。
	 */
	protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation)
			throws Throwable {

		// If the transaction attribute is null, the method is non-transactional.
        // 读取事务定义
		final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
        // 通过事务定义，获取事务管理器，如果txAttr没有指定事务管理器的名字，则采用默认事务管理器
		final PlatformTransactionManager tm = determineTransactionManager(txAttr);
        // 读取切入点
		final String joinpointIdentification = methodIdentification(method, targetClass);

		if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
			// Standard transaction demarcation with getTransaction and commit/rollback calls.
            // 如果有必要，则开启一个事务
			TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
			Object retVal = null;
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
                // 调用业务代码
				retVal = invocation.proceedWithInvocation();
			}
			catch (Throwable ex) {
				// target invocation exception
                // 发生异常处理
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			}
			finally {
                // 清理本次事务记录信息
				cleanupTransactionInfo(txInfo);
			}
            // 尝试提交本次事务
			commitTransactionAfterReturning(txInfo);
			return retVal;
		}
		else {
			...
		}
	}
```

此时调用栈为：

![](B5FC.tmp.jpg)

执行**业务逻辑test方法**之前，调用了`invokeWithinTransaction`方法，其大致流程为：

1. 读取目标方法的事务定义信息：`TransactionAttribute`。
2. 通过`TransactionAttribute#getQualifier`获取一个**事务管理器**。如果`getQualifier`返回null，则使用**默认**的事务管理器。
3. 调用`createTransactionIfNecessary`方法，开启一个事务（如果有必要）。
4. 调用业务代码，如果发生异常，则调用`completeTransactionAfterThrowing`方法处理（尝试事务回滚）它。
5. 调用`cleanupTransactionInfo`清理本次事务记录信息。
6. 调用`commitTransactionAfterReturning`尝试提交本次事务（如果存在事务，且已经完成该事务）。

可见`invokeWithinTransaction`方法是事务管理的**核心过程**。接下来，将依次分析各个自过程。


**createTransactionIfNecessary:**

```java

	/**
	 * Create a transaction if necessary based on the given TransactionAttribute.
	 *
     * 根据TransactionAttribute定义，如果有必要，则创建一个事务。
	 */
	@SuppressWarnings("serial")
	protected TransactionInfo createTransactionIfNecessary(
			PlatformTransactionManager tm, TransactionAttribute txAttr, final String joinpointIdentification) {

        ....
        
		TransactionStatus status = null;
		if (txAttr != null) {
			if (tm != null) {
                // 给定事务注解信息，通过事务管理器获取当前事务状态信息。注意：可能并未开启事务
				status = tm.getTransaction(txAttr);
			}
			...
		}
        // 保存本次事务上下文到ThreadLocal中，最后形成 TransactionInfo Stack。
		return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
	}

```

通过`createTransactionIfNecessary`方法，主要完成两个功能：

1. **status = tm.getTransaction(txAttr)**: 事务的创建，并且获取当前事务状态信息。
2. **prepareTransactionInfo**: 保存本次事务上下文信息。

这样子，就开启了**目标方法**的事务管理，接下来就可以调用具体的业务逻辑了。

**completeTransactionAfterThrowing:**

```java

	/**
	 * Handle a throwable, completing the transaction.
	 * We may commit or roll back, depending on the configuration.
	 *
	 * 处理异常。
	 * 回滚还是提交，依赖于具体配置。
	 */
	protected void completeTransactionAfterThrowing(TransactionInfo txInfo, Throwable ex) {
		if (txInfo != null && txInfo.hasTransaction()) {
			// 之前开启过事务
			...
			if (txInfo.transactionAttribute.rollbackOn(ex)) {
				// 需要回滚
				try {
					// 回滚
					txInfo.getTransactionManager().rollback(txInfo.getTransactionStatus());
				}
				....
			}
			else {
				// We don't roll back on this exception.
				// Will still roll back if TransactionStatus.isRollbackOnly() is true.
				// 继续提交
				try {
					txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
				}
				....
			}
		}
	}

```

当出现异常的时候，调用`completeTransactionAfterThrowing`来处理。**根据事务定义决定回滚还是提交当前事务。**

注意：默认情况下仅仅只有`RuntimeException`和`Error`的异常会进行回滚操作。(见：`DefaultTransactionAttribute.java`)

**cleanupTransactionInfo:**

```java
	/**
	 * Reset the TransactionInfo ThreadLocal.
	 * <p>Call this in all cases: exception or normal return!
	 * 
	 * 还原之前保存的事务上下文。
	 */
	protected void cleanupTransactionInfo(TransactionInfo txInfo) {
		if (txInfo != null) {
			// 还原之前保存的事务上下文。
			txInfo.restoreThreadLocalStatus();
		}
	}
```

当业务逻辑调用结束后（包括异常结束），会调用`cleanupTransactionInfo`方法，还原之前通
过`prepareTransactionInfo`保存的事务上下文。

**commitTransactionAfterReturning:**

```java

	/**
	 * Execute after successful completion of call, but not after an exception was handled.
	 * Do nothing if we didn't create a transaction.
	 * @param txInfo information about the current transaction
	 */
	protected void commitTransactionAfterReturning(TransactionInfo txInfo) {
		if (txInfo != null && txInfo.hasTransaction()) {
			if (logger.isTraceEnabled()) {
				logger.trace("Completing transaction for [" + txInfo.getJoinpointIdentification() + "]");
			}
			txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
		}
	}
```

最后，如果事务是正常的结束的，就会调用`commitTransactionAfterReturning`方法，尝试进行提交当前事务。

到此，一个拥有事务管理功能的目标方法，被调用的处理流程基本结束了。

## 扩展知识

### AbstractPlatformTransactionManager

在Spring容器中，基本上所有的事务管理器都继承于`AbstractPlatformTransactionManager`抽象类：

1. DataSourceTransactionManager: JDBC 事务管理器。
2. JmsTransactionManager: JMS 事务管理器。
3. JtaTransactionManager: JTA 事务管理器。

通过`AbstractPlatformTransactionManager`封装了`PlatformTransactionManager`的基础逻辑：

1. getTransaction: 根据事务定义，开启一个事务，并且返回该事务状态。
2. commit: 根据事务状态，进行提交（可能为空提交）。
3. rollback: 根据事务状态，进行回滚（可能为空回滚）。

通过继承实现`AbstractPlatformTransactionManager`的抽象方法：

1. doGetTransaction: 返回当前事务对象。
2. isExistingTransaction: 判断当前是否已经开启了事务。
3. doBegin: 新的事务
4. doSuspend: 挂起当前事务。
5. doResume: 恢复指定事务。
6. doCommit: 提交事务
7. doRollback: 回滚事务
8. doCleanupAfterCompletion: 事务结束，资源清理回调。

就可以实现一个具体的事务管理器（DataSource，Jms，Jta...）了。

注意：**`PlatformTransactionManager`定义了事物管理器的规范(BEGIN,COMMIT,ROLLBACK,SUSPEND,RESUME...)**。

### JTA

在实际开发的过程中，我们会遇到多数据源(Jms,DataSource ...)的分布式事务问题，这时候，我们就需要借助JTA来快速实现。

**依赖引入：**

```xml
	<dependency>
		<groupId>com.atomikos</groupId>
		<artifactId>transactions-jms</artifactId>
		<version>3.9.3</version>
	</dependency>

	<dependency>
		<groupId>javax.transaction</groupId>
		<artifactId>jta</artifactId>
		<version>1.1</version>
	</dependency>
```

通过Maven引入依赖`atomikos`和`jta`.

**JTA：**

![](23C7.tmp.jpg)

上述就是`JTA`的jar包接口。

**配置：**

```java
	@Component
    @EnableTransactionManagement
    static class IoConf {
		// 数据源1
        @Bean
        public DataSource dataSource1() {
            AtomikosDataSourceBean dataSourceBean = new AtomikosDataSourceBean();
            dataSourceBean.setUniqueResourceName("mysql1");
            dataSourceBean.setXaDataSourceClassName(MysqlXADataSource.class.getName());
            Properties properties = new Properties();
            properties.put("user", "root");
            properties.put("password", "root");
            properties.put("url", "jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8");
            dataSourceBean.setXaProperties(properties);
            dataSourceBean.setTestQuery("SELECT 'x'");
            return dataSourceBean;
        }
		// 数据源2
        @Bean
        public DataSource dataSource2() {
            AtomikosDataSourceBean dataSourceBean = new AtomikosDataSourceBean();
            dataSourceBean.setUniqueResourceName("mysql2");
            dataSourceBean.setXaDataSourceClassName(MysqlXADataSource.class.getName());
            Properties properties = new Properties();
            properties.put("user", "root");
            properties.put("password", "root");
            properties.put("url", "jdbc:mysql://localhost:3307/test?useUnicode=true&characterEncoding=UTF-8");
            dataSourceBean.setXaProperties(properties);
            dataSourceBean.setTestQuery("SELECT 'x'");
            return dataSourceBean;
        }

        @Bean
        public PlatformTransactionManager transactionManager() throws Exception {
			// JTA用户事务管理器
            UserTransactionManager transactionManager = new UserTransactionManager();
            transactionManager.init();
			// JTA 用户事务实现类
            UserTransactionImp atomikosUserTransaction = new UserTransactionImp();
            atomikosUserTransaction.setTransactionTimeout(30);
            return new JtaTransactionManager(atomikosUserTransaction, transactionManager);
        }
		// 数据源1的JDBC模版
        @Bean
        public JdbcTemplate template1() {
            return new JdbcTemplate(dataSource1());
        }
		// 数据源2的JDBC模版
        @Bean
        public JdbcTemplate template2() {
            return new JdbcTemplate(dataSource2());
        }
    }

```

这样子就完成了对JTA事务的配置了。

**业务逻辑：**

```java
@Component
public class RIo {

    @Autowired
    @Qualifier("template1")
    JdbcTemplate template1;

    @Autowired
    @Qualifier("template2")
    JdbcTemplate template2;

    @Transactional()
    public void test() {
        template1.update("UPDATE t_article SET content=? WHERE id=?", String.valueOf(System.currentTimeMillis()), "1");
        template2.update("UPDATE t_address SET name=? WHERE id=?", String.valueOf(System.currentTimeMillis()), "2");
    }
}

```

可以发现，在同一个事务过程中，同时对两个数据源进行更新操作。

注意：通过`atomikos`连接池获取的**XAResource**资源，会被atomikos代理，从而实现了**自动**加入当前JTA事务
的功能。详情可见:`AtomikosConnectionProxy#newInstance`。

**测试用例：**

```java

	@Test
	public void test() {
		rIo.test();
	}
	
```

**运行截图：**

![](258D.tmp.jpg)

**小结：**

虽然通过JTA可以快速的实现**分布式事务管理**，但是JTA在性能上存在缺陷。一般来说，我们可以采用:

1. 弱XA ( MyCat, ChainedTransactionManager )
2. TCC ( Try-Confirm-Cancel )

来避免性能上的问题。

## 参考

* [MySQL 中隔离级别 RC 与 RR 的区别](http://www.cnblogs.com/digdeep/archive/2015/11/16/4968453.html)
* [MySQL 加锁处理分析](http://hedengcheng.com/?p=771)
* [JTA 深度历险 - 原理与实现](http://www.ibm.com/developerworks/cn/java/j-lo-jta/)
* [XA事务处理](http://www.infoq.com/cn/articles/xa-transactions-handle)