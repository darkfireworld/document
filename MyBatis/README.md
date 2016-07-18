# MyBatis

MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

## 入门

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

##