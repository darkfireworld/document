# maven 入门指南

Maven项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。

**一般来说，我们使用maven来构建（编译，单元测试，打包，发布等）一个项目。**

## 下载和安装

### 下载 

首先，去官网下载最新(v3.x)的[maven](http://maven.apache.org/download.cgi)。

### 解压

![maven-unzip](897B.tmp.jpg)

### 配置环境

```shell

set M3_HOME=%~DP0maven
set JAVA_HOME=%~DP0java

set PATH=%PATH%;%M3_HOME%\bin;%JAVA_HOME%\bin;

```

### test

![mvn--test](C380.tmp.jpg)


## 简单的java项目


### 项目结构

![mvn-project-struct](C5F.tmp.jpg)


### 代码

**pom.xml：**

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--项目的组名-->
    <groupId>org.darkfireworld</groupId>
    <!--项目的工程名-->
    <artifactId>maven</artifactId>
    <!--项目的版本信息，添加*-SNAPSHOT表示快照版本-->
    <version>1.0-SNAPSHOT</version>
    <!--打包方式-->
    <packaging>jar</packaging>
    
    <!--依赖组-->
    <dependencies>
        <!--依赖-->
        <dependency>
            <!--依赖的组名-->
            <groupId>junit</groupId>
            <!--依赖的工程名-->
            <artifactId>junit</artifactId>
            <!--依赖版本号-->
            <version>4.11</version>
        </dependency>
    </dependencies>
</project>

```


**App.java:**

```java

public class App {
    public String say() {
        return "maven";
    }
}

```

**AppTest.java**

```java

public class AppTest {

    @Test
    public void testMain() {
        Assert.assertEquals("maven", new App().say());
    }
}

```

### run

输入命令`mvn test`，我们就可以进行编译，测试了：

```log

[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building maven 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven ---
[WARNING] Using platform encoding (GBK actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven ---
[WARNING] Using platform encoding (GBK actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory C:\Users\Administrator\Desktop\mvn\src\test\resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ maven ---
[INFO] Surefire report directory: C:\Users\Administrator\Desktop\mvn\target\surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.038 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.241 s
[INFO] Finished at: 2016-07-06T10:29:39+08:00
[INFO] Final Memory: 9M/155M
[INFO] ------------------------------------------------------------------------


```

运行`mvn package`，我们就可以获取一个jar包.

![mvn-package](FA44.tmp.jpg)


## 生命周期(clean、default、site)

Maven定义了三套生命周期：`clean、default、site`，每个生命周期都包含了一些`阶段（phase）`。
三套生命周期相互独立，但各个生命周期中的phase却是有顺序的，且后面的phase依赖于前面的phase。
执行某个phase时，其前面的phase会依顺序执行，但不会触发另外两套生命周期中的任何phase。

### clean

* clean：清除上次构建的文件

### site

* site：生成site文档

### default(重要)

* compile:编译
* test：测试
* package：打包
* install：安装到本地
* deploy：部署到服务器


## 插件

Maven的核心文件很小，主要的任务都是由插件来完成。定位到：`%本地仓库%\org\apache\maven\plugins`，可以看到一些下载好的插件：

![mvn-plugins](CD81.tmp.jpg)

### Plugin Goals

一个插件通常可以完成多个任务，每一个任务就叫做插件的一个目标(goals)。

![mvn-plugin-goals](5345.tmp.jpg)


### 配置插件

Maven插件高度易扩展，可以方便的进行自定义配置。如：配置maven-compiler-plugin插件编译源代码的JDK版本为1.6：

```xml

<project>
    ...
    <build>
        <plugins>
            <!--编译器设置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <!-- 指定编码格式，否则在DOS下运行mvn compile命令时会出现莫名的错误，因为系统默认使用GBK编码 -->
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
<project>

```


## 插件和生命周期

### 运行mvn

使用mvn的过程，就是调用一个插件的目标（goals）或者某个阶段（phase）

![mvn--help](F550.tmp.jpg)


### bind过程

Maven的生命周期是抽象的，实际需要插件来完成任务。这一过程是通过将插件的目标（goal）绑定到生命周期的具体阶段（phase）来完成的。
如：将`maven-compiler-plugin`插件的`compile目标`绑定到default生命周期的`compile阶段`，完成项目的源代码编译：

![maven-plugin-bind](A0FB.tmp.jpg)

### 默认bind

在maven中，已经存在了一些预定义的插件以及相应的生命周期绑定。详细可见`%M3_HOME%\maven-core\src\main\resources\META-INF\plexus\default-bindings.xml`。


## 依赖和坐标

通过maven我们可以管理项目的构建过程。而依赖管理是项目构建最重要的一个点。现在，我们来说说`dependency`这个属性。

通过`dependency`来描述一个依赖构件的`坐标`属性：

* groupId（必选）：依赖构件的组名
* artifactId（必选）：依赖构件的项目名
* version(必选)：依赖构件的版本号
* type（可选）：依赖构件的类型，默认是jar。
* scope(可选)：依赖构件范围

### 依赖范围(scope)

我们知道，一个项目在测试的时候需要`junit`，然而在具体发布的时候却不需要。maven通过`scope`属性来指定`依赖范围`。常见的依赖范围：

* compile：编译，测试，打包，运行都有效。如`spring系列`
* test: 编译，测试阶段有效。如`junit系列`
* provided：编译，测试有效。如`servler-api系列`。
* runtime：测试，运行有效。如`具体jdbc实现类`。

### 依赖传递

使用maven很方便的地方是：maven的依赖具有传递性。

比如说，我们引入`junit:junit:4.11`的时候，也会引入它的依赖`org.hamcrest:hamcrest-core:1.3`：

![junit-dependency](23F3.tmp.jpg)

当项目中指定依赖`junit:junit:4.11`的时候，maven就回去download对应的依赖到本地仓库：

![maven-junit-download](1D9E.tmp.jpg)

然后，读取该依赖构件的`POM.xml`，发现`junit:junit:4.11`依赖`org.hamcrest:hamcrest-core:1.3`：

![junit-pom](9426.tmp.jpg)

然后，maven就回去下载`org.hamcrest:hamcrest-core:1.3`，重复这个过程，直到项目的依赖全部整合完成。

当然这个过程中，会遇到**依赖冲突**这个问题，这里不进行讨论，因为这个概率比较小。

### 统一版本号

当我们引入`spring-frameworkd`的时候，每次都要填写`version`。而升级`spring-frameworkd`的时候，又要一个一个的修改`version`，非常的麻烦。

这时候，我们可以通过`<properties>`标签来实现统一依赖版本号：

```xml

<project>
    <!--定义一个属性，通过${}来引用-->
    <properties>
        <spring-version>3.2.0.RELEASE</spring-version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring-version}</version>
        </dependency>
    </dependencies>
</project>

```

## 仓库和镜像


在maven中，通过仓库来管理所有的`构件(jar,aar,war...)`。而通过`坐标`来引用这些构件。

### 仓库类型

在maven中存在如下几种类型的仓库：

1. 本地仓库
2. 远程仓库
    3. 中央仓库
    4. 私服仓库
    5. 公共仓库

maven通过`坐标`查询依赖的时候，优先级别为：

1. 本地仓库
2. 中央仓库
3. 其他仓库

### 本地仓库

maven在解析一个坐标的时候，会优先使用`本地仓库`。

通过修改`%M3_HOME%/conf/setting.xml`，我们可以指定本地仓库地址(默认~/.m2)：

```xml

<settings>
    <localRepository>D:\Link\maven\repo\</localRepository>
</settings>

```

我们可以看看一个本地仓库的内容是啥，以`junit:junit:4.11`为例：

![maven-repo-files](67C.tmp.jpg)

**注意，如果坐标仓库中的`POM.xml#packaging`为aar（android类库），则maven会下载`groupId:artifactId:version.aar`，并非依赖其他插件实现。
而其它文件(*-sources.jar , *-javadoc.jar ...)的文件名都是比较固定的。**

### 远程仓库

远程仓库指的是所有非本地仓库。通过在pom.xml中添加如下描述，就可以添加一个远程仓库了：

```xml
<project>
    <!--仓库管理-->
    <repositories>
        <!--远程仓库-->
        <repository>
            <!--仓库ID-->
            <id>Sonatype</id>
            <!--仓库名称-->
            <name>Sonatype Repository</name>
            <!--仓库URL-->
            <url>http://repository.sonatype.org/content/groups/public/</url>
            <!--仓库布局模式-->
            <layout>default</layout>
            <!--是否使用该仓库中的release依赖-->
            <releases>
                <enabled>true</enabled>
            </releases>
            <!--是否使用该仓库中的snapshots依赖-->
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>

```

注意：**其中 id 必须唯一，若不唯一，如设置为 central 将覆盖中央仓库的配置。**


#### 中央仓库

中央仓库其实就是一个**超级大的默认的远程仓库**。

其预定义在依赖**SUPER_POM**(`%M3_HOME%\maven-model-builder\src\main\resources\org\apache\maven\model\pom-4.0.0.xml`)：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- START SNIPPET: superpom -->
<project>
  <modelVersion>4.0.0</modelVersion>
  <!--依赖的中央仓库-->
  <repositories>
    <repository>
      <!-- ID central 特指中央仓库-->
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
  <!--插件的中央仓库-->
  <pluginRepositories>
    <pluginRepository>
      <!-- ID central 特指中央仓库-->
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
  <!--默认构建参数-->
  <build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.8</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.3.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
  ...
</project>
<!-- END SNIPPET: superpom -->

```

可见super pom 提供了一些默认的项目属性。而我们**项目中的pom.xml其实都继承于super pom**。


### 镜像

镜像就相当于仓库代理。通过`M3_HOME`/conf/settings.xml，我们就可以配置镜像：

```xml

<settings>
    ...
    <mirrors>
        <mirror>
            <!--镜像ID-->
            <id>jcenter</id>
            <!--镜像名称-->
            <name>jcenter</name>
            <url>https://jcenter.bintray.com/</url>
            <!--需要代理的仓库ID，如果为 * 则代理所有的仓库-->
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
    ...
</settings>

```

镜像的工作过程大致如下：

```

        | --> mirror server [mirrorOf:central] --> real repo server(eg. central)
request |  
        | --> jcenter repo
        
```

相当于，在查询远程仓库的时候流程如下：

1. 先判断要查询的远程仓库是否被`mirror`代理
2. 如果被代理，则查询`mirror`
3. 如果没有被代理，则查询real repo.

### 私服

私服，其实就是一个自己搭建的仓库。用来存放一些公司内部的依赖构件。当然了，我们也可以通过私服缓存中央仓库中的构建，加快内网访问。

一个比较经典的架构：

```

                                                        |---> central repo (eg. jcenter)
                                                        |
user [mirrorOf:*] -> sf(私有构件，中央仓库缓存构件) --->
                                                        |---> other repo

```

常用的sf工具[sonatype](http://www.sonatype.com/download-oss-sonatype)，通过它，我们就可以快速的搭建一个sf了。


## 快照

在开发中，会遇到一个依赖构件每天都在更新且版本号不变的情况（持续构建）。而maven特有的本地仓库机制，不能适应这种情况。
所以，我们引入了快照(**SNAPSHOT**)的概念：

> 如果依赖的构建是一个快照版本**（version中使用-SNAPSHOT结尾）**，那么maven在构建的时候，首先会检测该本地仓库缓
> 存是否失效（一般24小时），如果已经失效，则去远程仓库获取最新的版本。注：`mvn clean package -U` 可以刷新快照。

如何引用/构建一个快照版本的构建呢？其实非常的简单，只要`pom.xml`中设置`version`后面添加**`-SNAPSHOT`**即可。比如说：

```xml

    <version>1.3.0-SNAPSHOT</version>
    
```

## 多环境配置

在实际项目中，通常会出现多个运行环境，比如说：本地，测试，正式。不同的环境的参数配置都是不一样的。所以我们需要编写不同的参数配置。通过maven
的`<profiles>`标签，就可以配置不同的环境属性了。

**项目结构：**

![profiles-project-struct](A313.tmp.jpg)

**POM.xml**

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--项目的组名-->
    <groupId>org.darkfireworld</groupId>
    <!--项目的工程名-->
    <artifactId>maven</artifactId>
    <!--项目的版本信息，添加*-SNAPSHOT表示快照版本-->
    <version>1.0-SNAPSHOT</version>
    <!--打包方式-->
    <packaging>jar</packaging>

    <!--依赖组-->
    <dependencies>
        <!--依赖-->
        <dependency>
            <!--依赖的组名-->
            <groupId>junit</groupId>
            <!--依赖的工程名-->
            <artifactId>junit</artifactId>
            <!--依赖版本号-->
            <version>4.11</version>
        </dependency>
    </dependencies>
    <!--环境配置-->
    <profiles>
        <!--debug环境-->
        <profile>
            <!--ID-->
            <id>debug</id>
            <!--属性配置，通过${}引用-->
            <properties>
                <!--替换的token，使用${username}引用-->
                <username>debug</username>
            </properties>
            <!--激活属性-->
            <activation>
                <!--默认激活-->
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <!--release环境-->
        <profile>
            <!--ID-->
            <id>release</id>
            <!--属性配置，通过${}引用-->
            <properties>
                <!--替换的token，使用${username}引用-->
                <username>release</username>
            </properties>
        </profile>
    </profiles>
    <!--编译脚本-->
    <build>
        <!--处理资源文件-->
        <resources>
            <!-- src/main/resources 下所有 xml 文件：需要变量替换 -->
            <resource>
                <!--资源目录-->
                <directory>src/main/resources</directory>
                <!--开启Token替换-->
                <filtering>true</filtering>
                <!--仅仅支持xml替换，所以配置文件统一为*.xml-->
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <!-- src/main/resources 下所有非 xml 文件：原样拷贝，不进行变量替换 -->
            <resource>
                <!--资源目录-->
                <directory>src/main/resources</directory>
                <!--不开启Token替换-->
                <filtering>false</filtering>
                <!--所有非xml-->
                <excludes>
                    <exclude>**/*.xml</exclude>
                </excludes>
            </resource>
        </resources>
        <!--插件-->
        <plugins>
            <!--编译器设置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <!-- 指定编码格式，否则在DOS下运行mvn compile命令时会出现莫名的错误，因为系统默认使用GBK编码 -->
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!--资源处理设置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <!-- 指定编码格式，否则在DOS下运行mvn命令时当发生文件资源copy时将使用系统默认使用GBK编码 -->
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

**resources/config.xml**

```xml

<config>
    ${username}
</config>

```


**resources/META-INF/hello.cc**

```

don't replase token

${username}

```

然后我们运行命令`mvn compile`，就可以发现`target/classes/config.xml`已经被替换为默认debug的${username}->debug。
而`META-INF/hello.cc`保持${username}。

**注意： 我们可以通过 `mvn compile -P release ` 来指定使用release配置。**

通过`<profiles>`属性，我们可以非常方便的提供不同环境的配置信息。当然了`<profiles>`不仅仅是是这种功能，还有其他功能。


## JavaEE 项目

maven也支持JavaEE项目，只需要修改`packaging`为war，并且添加/src/main/webapp即可。

![javaee-struct](101A.tmp.jpg)

项目地址：[java-fast-framework](https://github.com/darkfireworld/java-fast-framework.git)

## 多模块构建

在实际开发中，获取会遇到多模块构建的问题，通过maven多模块功能，我们可以实现这种多模块构建。

比如说，我们要实现如下的项目结构：

![multi-module-struct](63D5.tmp.jpg)

### root module

root 相当于一个容器，并且记录一些通用的属性信息，比如说：仓库，通用依赖，build模式等，pom.xml为：

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.darkfireworld</groupId>
    <artifactId>maven-multi-project</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--打包模式为pom-->
    <packaging>pom</packaging>
    <!--子模块-->
    <modules>
        <module>io</module>
        <module>biz</module>
        <module>ctrl</module>
    </modules>

    <!--编写一些通用的属性，比如说，依赖，仓库，build-->
    <!--仓库-->
    <repositories>
        <repository>
            <id>central</id>
            <name>jcenter</name>
            <url>https://jcenter.bintray.com/</url>
        </repository>
    </repositories>
    <!--通用依赖-->
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>19.0</version>
        </dependency>
    </dependencies>

    <!--通用构建模式-->
    <build>
        <!--插件-->
        <plugins>
            <!--编译器设置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <!-- 指定编码格式，否则在DOS下运行mvn compile命令时会出现莫名的错误，因为系统默认使用GBK编码 -->
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!--资源处理设置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <!-- 指定编码格式，否则在DOS下运行mvn命令时当发生文件资源copy时将使用系统默认使用GBK编码 -->
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

**注意配置`packaging`为pom类型。**

### io module

io module 相当于数据接入层，用于和数据库，restful api 交互的层面，pom.xml为:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>maven-multi-project</artifactId>
        <groupId>org.darkfireworld</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>io</artifactId>
    <packaging>jar</packaging>

</project>

```

### biz module

biz module相当于业务处理模块，比如说：发送邮件，定时任务等，pom.xml为：

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>maven-multi-project</artifactId>
        <groupId>org.darkfireworld</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>biz</artifactId>
    <packaging>jar</packaging>
    <dependencies>
        <!--依赖IO模块-->
        <dependency>
            <groupId>org.darkfireworld</groupId>
            <artifactId>io</artifactId>
            <!--当前项目的版本号-->
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>

```

**注意，因为biz模块需要依赖io模块，所以，我们使用`dependency`来引用io模块，且`<version>${project.version}</version>`。**

### ctrl module

ctrl module 相当于mvc模块，用于接入http请求等，pom.xml为：

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>maven-multi-project</artifactId>
        <groupId>org.darkfireworld</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>ctrl</artifactId>
    <!--war结构-->
    <packaging>war</packaging>

    <!--依赖-->
    <dependencies>
        <!--biz 模块依赖-->
        <dependency>
            <groupId>org.darkfireworld</groupId>
            <artifactId>biz</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--io模块依赖-->
        <dependency>
            <groupId>org.darkfireworld</groupId>
            <artifactId>io</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
    
</project>

```

**注意，因为ctrl 模块是接入mvc的，所以打包模式为`<packaging>war</packaging>`。**

### 打包多模块

运行命令`mvn package`，我们就可以获取到最终的war包：

![target-war](AF29.tmp.jpg)

解压war查看内容，可以发现是一个标准的war项目，并且已经打包了依赖的 `io，biz` jar包：

![war-lib](8DA7.tmp.jpg)

项目地址：[maven-multi-project](maven-multi-project.zip)

## 参考

* [Maven 快速入门及简单使用](http://www.cnblogs.com/luotaoyeah/p/3764533.html)
* [Maven 教程](https://ayayui.gitbooks.io/tutorialspoint-maven/content/)
* [maven中snapshot快照库和release发布库的区别和作用](http://www.mzone.cc/article/277.html)
* [Maven：mirror和repository 区别](http://my.oschina.net/sunchp/blog/100634)
* [Maven系列一pom.xml 配置详解](http://www.cnblogs.com/yangxia-test/p/4396159.html)
* [Maven系列二setting.xml 配置详解](http://www.cnblogs.com/yangxia-test/p/4409736.html)
* [maven学习（上）- 基本入门用法](http://www.cnblogs.com/yjmyzz/p/3495762.html)
* [maven学习（中）- 私服nexus搭建](http://www.cnblogs.com/yjmyzz/p/3519373.html)
* [maven学习（下）利用Profile构建不同环境的部署包](http://www.cnblogs.com/yjmyzz/p/3941043.html)
* [Maven学习 (六) 搭建多模块企业级项目](http://www.cnblogs.com/quanyongan/archive/2013/05/28/3103243.html)
* [如何使用Android Studio把自己的Android library分享到jCenter和Maven Central](http://www.open-open.com/lib/view/open1435109824278.html)
* [Maven和Gradle对比](http://www.huangbowen.net/blog/2016/02/23/gradle-vs-maven/?utm_source=tuicool&utm_medium=referral)
* [玩转迭代开发](https://github.com/darkfireworld/self-doc/tree/master/玩转迭代开发)






