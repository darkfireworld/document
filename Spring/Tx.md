# Spring

## Tx

通过 Spring 事务，我们可以非常方便地对应用进行事务管理。

### 例子

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


### @Transactional

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

#### 事务管理器

在一个应用中，可以存在多种类型的**事务管理器**：

1. Jms事务管理器
2. DataSource事务管理器
3. Jta事务管理器

即使同一类型的事务管理器，也可能存在多个不同的实例：

1. MySQL_1 的DataSource事务管理器。
2. MySQL_2 的DataSource事务管理器。
3. Oracle_1 的DataSource事务管理器。

所以，当存在**多个**事务管理器的时候，那么使用`@Transactional`注解需要指定事务管理器。

#### 事务传播

`事务传播`这个概念是基于应用层的，而**非数据库级别**。在Spring中，大致存在如下类型：

1. **REQUIRED**（默认）: 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。
2. **SUPPORTS**: 支持当前事务，如果当前没有事务，就以非事务方式执行。
3. **MANDATORY**: 使用当前的事务，如果当前没有事务，就抛出异常。
4. **REQUIRES_NEW**: 新建事务，如果当前存在事务，把当前事务挂起。
5. **NOT_SUPPORTED**: 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6. **NEVER**: 以非事务方式执行，如果当前存在事务，则抛出异常。

合理的定义`事务传播`特性，能优化应用的整体性能。

#### 隔离级别

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

### 重要类

在源码分析之前，我们先看一下Spring事务管理中，重要的类。



### 源码分析

### 扩展知识

## 参考

* [MySQL 中隔离级别 RC 与 RR 的区别](http://www.cnblogs.com/digdeep/archive/2015/11/16/4968453.html)
* [MySQL 加锁处理分析](http://hedengcheng.com/?p=771)
* [JTA 深度历险 - 原理与实现](http://www.ibm.com/developerworks/cn/java/j-lo-jta/)