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

### 重要类

#### @Transactional


### 源码分析

### 扩展知识

## 参考

* [JTA 深度历险 - 原理与实现](http://www.ibm.com/developerworks/cn/java/j-lo-jta/)