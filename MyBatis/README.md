# MyBatis

MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

## 基础入门

### 项目结构

项目使用MVN构建，如下图：

![](5229.tmp.jpg)

### 安装

首先，我们先配置JAR依赖（MyBatis+MySQL+Log）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.test</groupId>
    <artifactId>test</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--通用属性配置-->
    <properties>
        <mybatis-version>3.4.1</mybatis-version>
        <mysql-version>5.1.39</mysql-version>
        <slf4j-version>1.7.18</slf4j-version>
        <logback-version>1.1.7</logback-version>

    </properties>
    <dependencies>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql-version}</version>
        </dependency>
        <!--logback系列-->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback-version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>${logback-version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-access</artifactId>
            <version>${logback-version}</version>
        </dependency>

        <!--slf4j系列-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j-version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>${slf4j-version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jul-to-slf4j</artifactId>
            <version>${slf4j-version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j-version}</version>
        </dependency>
    </dependencies>

</project>
```

这样子，就完成了JAR的依赖配置。

### configuration

使用MyBatis 的时候，需要创建一个`configuration.xml`文件，主要是用来配置MyBatis的基础属性：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--基础配置-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!--扫描mapper目录-->
    <mappers>
        <package name="mapper"/>
    </mappers>
</configuration>

```

### Bean + Mapper

使用MyBatis的时候，需要编写Bean和Mapper，而Mapper是将数据库和Bean映射的描述：

```java

/**
 * Article.java
 */
package mapper;
public class Article {
    String id;
    String content;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}


/**
 * ArticleMapper.java
 */
package mapper;

import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;

import java.util.List;


public interface ArticleMapper {

    @Select("SELECT * FROM t_article")
    List<Article> selectAll();
}

```

而数据库的表结构为：

![](71D8.tmp.jpg)

### 运行

运行如下代码：

```java

import mapper.Article;
import mapper.ArticleMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;

/**
 * DO
 */
public class App {
    public static void main(String[] args) throws Exception {
        //读取配置
        String resource = "configuration.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //构建SqlSessionFactory，全局唯一
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //打开一个会话，相当于JDBC连接，线程不安全
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //获取一个Mapper
        ArticleMapper articleMapper = sqlSession.getMapper(ArticleMapper.class);
        //SELECT
        for (Article article : articleMapper.selectAll()) {
            System.out.println(String.format("ID=%s，CONTENT=%s", article.getId(), article.getContent()));
        }
    }
}

```
运行结果：

```
11:17:58.931 [main] DEBUG org.apache.ibatis.logging.LogFactory - Logging initialized using 'class org.apache.ibatis.logging.slf4j.Slf4jImpl' adapter.
11:17:59.045 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
11:17:59.045 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
11:17:59.046 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
11:17:59.046 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - PooledDataSource forcefully closed/removed all connections.
11:17:59.053 [main] DEBUG org.apache.ibatis.io.VFS - Class not found: org.jboss.vfs.VFS
11:17:59.053 [main] DEBUG org.apache.ibatis.io.JBoss6VFS - JBoss 6 VFS API is not available in this environment.
11:17:59.054 [main] DEBUG org.apache.ibatis.io.VFS - Class not found: org.jboss.vfs.VirtualFile
11:17:59.055 [main] DEBUG org.apache.ibatis.io.VFS - VFS implementation org.apache.ibatis.io.JBoss6VFS is not valid in this environment.
11:17:59.055 [main] DEBUG org.apache.ibatis.io.VFS - Using VFS adapter org.apache.ibatis.io.DefaultVFS
11:17:59.056 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Find JAR URL: file:/C:/Users/Administrator/Desktop/untitled/target/classes/mapper
11:17:59.056 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Not a JAR: file:/C:/Users/Administrator/Desktop/untitled/target/classes/mapper
11:17:59.116 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Reader entry: Article.class
11:17:59.116 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Reader entry: ArticleMapper.class
11:17:59.116 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Listing file:/C:/Users/Administrator/Desktop/untitled/target/classes/mapper
11:17:59.116 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Find JAR URL: file:/C:/Users/Administrator/Desktop/untitled/target/classes/mapper/Article.class
11:17:59.116 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Not a JAR: file:/C:/Users/Administrator/Desktop/untitled/target/classes/mapper/Article.class
11:17:59.117 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Reader entry: ����   1 
11:17:59.117 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Find JAR URL: file:/C:/Users/Administrator/Desktop/untitled/target/classes/mapper/ArticleMapper.class
11:17:59.117 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Not a JAR: file:/C:/Users/Administrator/Desktop/untitled/target/classes/mapper/ArticleMapper.class
11:17:59.118 [main] DEBUG org.apache.ibatis.io.DefaultVFS - Reader entry: ����   1  
11:17:59.118 [main] DEBUG org.apache.ibatis.io.ResolverUtil - Checking to see if class mapper.Article matches criteria [is assignable to Object]
11:17:59.118 [main] DEBUG org.apache.ibatis.io.ResolverUtil - Checking to see if class mapper.ArticleMapper matches criteria [is assignable to Object]
11:17:59.178 [main] DEBUG org.apache.ibatis.transaction.jdbc.JdbcTransaction - Opening JDBC Connection
Mon Jul 18 11:17:59 CST 2016 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
11:17:59.457 [main] DEBUG org.apache.ibatis.datasource.pooled.PooledDataSource - Created connection 1061804750.
11:17:59.457 [main] DEBUG org.apache.ibatis.transaction.jdbc.JdbcTransaction - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@3f49dace]
11:17:59.460 [main] DEBUG mapper.ArticleMapper.selectAll - ==>  Preparing: SELECT * FROM t_article 
11:17:59.504 [main] DEBUG mapper.ArticleMapper.selectAll - ==> Parameters: 
11:17:59.531 [main] DEBUG mapper.ArticleMapper.selectAll - <==      Total: 2

ID=1，CONTENT=文章1
ID=2，CONTENT=文章2

```

可以发现，已经成功的读取到数据库中的数据到JavaBean中了。

项目地址：[simple-mybatis-project](simple-mybatis-project.zip)

## Spring集成

在使用MyBatis的时候，通常是和Spring框架一起使用的。MyBatis和Spring的集成是非常方便的，只需要几个步骤即可。

### 项目结构

![](AFA7.tmp.jpg)

### 安装

首先，我们需要引入最新的mybatis-spring的jar包：

```
<!--mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.0</version>
</dependency>
```

### 配置

mybatis和spring集成的时候，我们就不需要MyBatis的配置文件(configuration.xmk)了，只需要在Spring
的配置文件中，引入MyBatis#SqlSessionFactorty的Bean配置即可，如下是比较完成的配置：

```
    <!--注意，这里还需要添加dataSource-->
    <!--事务性-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--mybatis -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource" />
        <--扫描org.darkgem.io下面的所有TypeHandler-->
        <property name="typeHandlersPackage" value="org.darkgem.io"/>
    </bean>
    <!--mapper-scan-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--扫描所有org.darkgem.io下面的Mapper接口-->
        <property name="basePackage" value="org.darkgem.io" />
    </bean>
```

这样子，就配置了MyBatis。

### Bean + Mapper

如下是MyBatis的Bean以及相应的Mapper和SQL Mapper：

```


/**
 * Article.java
 */
package org.darkgem.io.article;

public class Article {
    String id;
    String content;
    Type type;

    public Article(String id, String content, Type type) {
        this.id = id;
        this.content = content;
        this.type = type;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public Type getType() {
        return type;
    }

    public void setType(Type type) {
        this.type = type;
    }

    public enum Type {
        IT,
        LIFT;
    }
}



/**
 * ArticleMapper.java
 */

package org.darkgem.io.article;

import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedTypes;
import org.apache.ibatis.type.TypeHandler;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;


public interface ArticleMapper {

    List<Article> selectList(@Param("content") String content, @Param("type") Article.Type type);

    /**
     * 类型转换
     */
    @MappedTypes(Article.Type.class)
    class ArticleTypeHandler implements TypeHandler<Article.Type> {
        @Override
        public void setParameter(PreparedStatement ps, int i, Article.Type parameter, JdbcType jdbcType) throws SQLException {
            ps.setString(i, parameter.toString());
        }

        @Override
        public Article.Type getResult(ResultSet rs, String columnName) throws SQLException {
            return Article.Type.valueOf(rs.getString(columnName));
        }

        @Override
        public Article.Type getResult(ResultSet rs, int columnIndex) throws SQLException {
            return Article.Type.valueOf(rs.getString(columnIndex));
        }

        @Override
        public Article.Type getResult(CallableStatement cs, int columnIndex) throws SQLException {
            return Article.Type.valueOf(cs.getString(columnIndex));
        }
    }
}


<!--ArticleMapper.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.darkgem.io.article.ArticleMapper">

    <sql id="columns">
        id,content,type
    </sql>

    <resultMap id="resultArticle" type="org.darkgem.io.article.Article">
        <constructor>
            <idArg column="id" javaType="String"/>
            <arg column="content" javaType="String"/>
            <arg column="type" javaType="org.darkgem.io.article.Article$Type"/>
        </constructor>
    </resultMap>
    <select id="selectList" resultMap="resultArticle">
        SELECT
            <include refid="columns"/>
        FROM
            t_article
        WHERE
            content like #{content} and type = #{type}
    </select>
</mapper>



/**
 * ArticleIo.java
 */
package org.darkgem.io.article;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class ArticleIo {

    //这里直接引入Mapper进行调用，它是线程安全的
    @Autowired
    ArticleMapper articleMapper;

    public List<Article> selectList(String key, Article.Type type) {
        return articleMapper.selectList('%' + key + '%', type);
    }
}



```

注意，在`ArticleMapper.java`中，我们配置了关于Article中Type这个`枚举`类型的处理器。
通过TypeHandler，我们可以完成Java和Jdbc类型的转换。

### Controller

现在，来配置一下Spring MVC 调用MyBatis的代码：


```java

package org.darkgem.web.mvc.main;

import org.darkgem.io.article.Article;
import org.darkgem.io.article.ArticleIo;
import org.darkgem.web.support.handler.Token;
import org.darkgem.web.support.msg.Message;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/main/MainCtrl")
public class MainCtrl {
    @Autowired
    ArticleIo articleIo;

    @Transactional
    @RequestMapping("/ask")
    public Object ask(@Token String token) {
        return Message.okMessage(articleIo.selectList(token, Article.Type.IT));
    }
}

```

### 运行


截图：

![](C616.tmp.jpg)

日志：

```

2016-07-18 12:08:23.606 [qtp6566818-15] DEBUG o.s.web.servlet.DispatcherServlet - DispatcherServlet with name 'springMVC' processing GET request for [/main/MainCtrl/ask.do]
2016-07-18 12:08:23.607 [qtp6566818-15] DEBUG o.s.w.s.m.m.a.RequestMappingHandlerMapping - Looking up handler method for path /main/MainCtrl/ask.do
2016-07-18 12:08:23.608 [qtp6566818-15] DEBUG o.s.w.s.m.m.a.RequestMappingHandlerMapping - Returning handler method [public java.lang.Object org.darkgem.web.mvc.main.MainCtrl.ask(java.lang.String)]
2016-07-18 12:08:23.608 [qtp6566818-15] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Returning cached instance of singleton bean 'mainCtrl'
2016-07-18 12:08:23.608 [qtp6566818-15] DEBUG o.s.web.servlet.DispatcherServlet - Last-Modified value for [/main/MainCtrl/ask.do] is: -1
2016-07-18 12:08:23.609 [qtp6566818-15] DEBUG o.s.j.d.DataSourceTransactionManager - Creating new transaction with name [org.darkgem.web.mvc.main.MainCtrl.ask]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT; ''
2016-07-18 12:08:23.609 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, stmt-20007} created
2016-07-18 12:08:23.611 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, stmt-20007, rs-50007} query executed. 0.692501 millis. 
SELECT 'x'
2016-07-18 12:08:23.611 [qtp6566818-15] DEBUG druid.sql.ResultSet - {conn-10005, stmt-20007, rs-50007} open
2016-07-18 12:08:23.611 [qtp6566818-15] DEBUG druid.sql.ResultSet - {conn-10005, stmt-20007, rs-50007} Header: [x]
2016-07-18 12:08:23.611 [qtp6566818-15] DEBUG druid.sql.ResultSet - {conn-10005, stmt-20007, rs-50007} closed
2016-07-18 12:08:23.612 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, stmt-20007} closed
2016-07-18 12:08:23.612 [qtp6566818-15] DEBUG druid.sql.Connection - {conn-10005} pool-connect
2016-07-18 12:08:23.612 [qtp6566818-15] DEBUG o.s.j.d.DataSourceTransactionManager - Acquired Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@213795f2] for JDBC transaction
2016-07-18 12:08:23.612 [qtp6566818-15] DEBUG o.s.j.d.DataSourceTransactionManager - Switching JDBC Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@213795f2] to manual commit
2016-07-18 12:08:23.612 [qtp6566818-15] DEBUG druid.sql.Connection - {conn-10005} setAutoCommit false
2016-07-18 12:08:23.614 [qtp6566818-15] DEBUG org.mybatis.spring.SqlSessionUtils - Creating a new SqlSession
2016-07-18 12:08:23.614 [qtp6566818-15] DEBUG org.mybatis.spring.SqlSessionUtils - Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@669462bf]
2016-07-18 12:08:23.614 [qtp6566818-15] DEBUG o.m.s.t.SpringManagedTransaction - JDBC Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@213795f2] will be managed by Spring
2016-07-18 12:08:23.614 [qtp6566818-15] DEBUG o.d.i.a.ArticleMapper.selectList - ==>  Preparing: SELECT id,content,type FROM t_article WHERE content like ? and type = ? 
2016-07-18 12:08:23.615 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, pstmt-20008} created. 
SELECT
             
        id,content,type
     
        FROM
            t_article
        WHERE
            content like ? and type = ?
2016-07-18 12:08:23.615 [qtp6566818-15] DEBUG o.d.i.a.ArticleMapper.selectList - ==> Parameters: %1%(String), IT(String)
2016-07-18 12:08:23.615 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, pstmt-20008} Parameters : [%1%, IT]
2016-07-18 12:08:23.615 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, pstmt-20008} Types : [VARCHAR, VARCHAR]
2016-07-18 12:08:23.619 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, pstmt-20008} executed. 3.417743 millis. 
SELECT
             
        id,content,type
     
        FROM
            t_article
        WHERE
            content like ? and type = ?
2016-07-18 12:08:23.619 [qtp6566818-15] DEBUG druid.sql.ResultSet - {conn-10005, pstmt-20008, rs-50008} open
2016-07-18 12:08:23.619 [qtp6566818-15] DEBUG druid.sql.ResultSet - {conn-10005, pstmt-20008, rs-50008} Header: [id, content, type]
2016-07-18 12:08:23.619 [qtp6566818-15] DEBUG druid.sql.ResultSet - {conn-10005, pstmt-20008, rs-50008} Result: [1, 文章1, IT]
2016-07-18 12:08:23.620 [qtp6566818-15] DEBUG o.d.i.a.ArticleMapper.selectList - <==      Total: 1
2016-07-18 12:08:23.621 [qtp6566818-15] DEBUG druid.sql.ResultSet - {conn-10005, pstmt-20008, rs-50008} closed
2016-07-18 12:08:23.622 [qtp6566818-15] DEBUG druid.sql.Statement - {conn-10005, pstmt-20008} closed
2016-07-18 12:08:23.622 [qtp6566818-15] DEBUG org.mybatis.spring.SqlSessionUtils - Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@669462bf]
2016-07-18 12:08:23.622 [qtp6566818-15] DEBUG org.mybatis.spring.SqlSessionUtils - Transaction synchronization committing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@669462bf]
2016-07-18 12:08:23.622 [qtp6566818-15] DEBUG org.mybatis.spring.SqlSessionUtils - Transaction synchronization deregistering SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@669462bf]
2016-07-18 12:08:23.622 [qtp6566818-15] DEBUG org.mybatis.spring.SqlSessionUtils - Transaction synchronization closing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@669462bf]
2016-07-18 12:08:23.622 [qtp6566818-15] DEBUG o.s.j.d.DataSourceTransactionManager - Initiating transaction commit
2016-07-18 12:08:23.622 [qtp6566818-15] DEBUG o.s.j.d.DataSourceTransactionManager - Committing JDBC transaction on Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@213795f2]
2016-07-18 12:08:23.624 [qtp6566818-15] DEBUG druid.sql.Connection - {conn-10005} commited
2016-07-18 12:08:23.624 [qtp6566818-15] DEBUG druid.sql.Connection - {conn-10005} setAutoCommit true
2016-07-18 12:08:23.625 [qtp6566818-15] DEBUG o.s.j.d.DataSourceTransactionManager - Releasing JDBC Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@213795f2] after transaction
2016-07-18 12:08:23.625 [qtp6566818-15] DEBUG o.s.jdbc.datasource.DataSourceUtils - Returning JDBC Connection to DataSource
2016-07-18 12:08:23.625 [qtp6566818-15] DEBUG druid.sql.Connection - {conn-10005} pool-recycle
2016-07-18 12:08:23.626 [qtp6566818-15] DEBUG o.s.w.s.v.ContentNegotiatingViewResolver - Requested media types are [text/html, application/xhtml+xml, image/webp, application/xml;q=0.9, */*;q=0.8] based on Accept header types and producible media types [*/*])
2016-07-18 12:08:23.627 [qtp6566818-15] DEBUG o.s.w.s.v.ContentNegotiatingViewResolver - Returning [com.alibaba.fastjson.support.spring.FastJsonJsonView: name 'com.alibaba.fastjson.support.spring.FastJsonJsonView#35aea049'] based on requested media type '*/*;q=0.8'
2016-07-18 12:08:23.627 [qtp6566818-15] DEBUG o.s.web.servlet.DispatcherServlet - Rendering view [com.alibaba.fastjson.support.spring.FastJsonJsonView: name 'com.alibaba.fastjson.support.spring.FastJsonJsonView#35aea049'] in DispatcherServlet with name 'springMVC'
2016-07-18 12:08:23.627 [qtp6566818-15] DEBUG o.s.web.servlet.DispatcherServlet - Successfully completed request


```

可以发现，MyBatis已经和Spring集成成功。

项目地址：[java-fast-framework](https://github.com/darkfireworld/java-fast-framework.git)


## 参考

* [mybatis](http://www.mybatis.org/mybatis-3/zh/index.html)
* [mybatis-spring](http://www.mybatis.org/spring/zh/index.html)
