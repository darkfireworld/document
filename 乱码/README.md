# 乱码

乱码的根本问题在于：**使用错误的编码映射文本的二进制数据。**

如果整个项目采用`UTF-8`进行编码，则能很好的规避乱码问题。

> `UTF-8` 编码格式兼容性最好，可以解决网络字节序大小端问题，并且可以表示大部分字符。

## 数据库

通常，我们使用`MySQL`作为数据库，如下是`UTF-8`配置：

```
[mysqld]#mysqld
character-set-server=utf8 
[mysql] #mysql
default-character-set=utf8
[client] #client
default-character-set=utf8
```

## JDBC

在使用`JDBC`连接MySQL的时候，也需要指定连接的字符集为`UTF-8`。这样子，`MySQL JDBC Driver`才能正常
地识别编码：

```

jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8

```

## Java

在代码中，会出现中文乱码的问题，建议统一采用 UTF-8的编码格式，然后编译的
时候，指定编码格式 `javac -encoding utf8`。

且注意项目的默认字符集也设置为`utf-8`格式。

## JSP

JSP的乱码问题，也和Java代码乱码一样，需要统一配置为`UTF-8`。


## HTTP Content-Type

HTTP 的`协议头`中存在`Content-Type`属性，通过这个属性，我们可以规定`HTTP`的编码格式：

```

request:
    Content-Type:application/x-www-form-urlencoded;charset=utf-8
    
response:
    Content-Type: text/html;charset=utf-8
    
```

而在JavaEE的容器中，可以通过`web.xml`配置过滤器，来避免大多数乱码问题：

```

    <!-- 编码-->
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
    
```

通过这个过滤器，造成了两个影响：

1. `request`的时候，读取`multipart/formdata`和`application/x-www-form-urlencoded`格式的HTTP请求的时候，会
使用`UTF-8`解析请求参数。

2. `response`的时候，如果该`response`之前**没有被提交且不是WRITE模式**，则可以指定Client使用UTF-8解析HTTP报文。

注：`UTF-8`编码仅仅针对`文本内容`的HTTP报文。

## TOMCAT 

如果我们在URL里面添加`中文参数`，那么TOMCAT会默认采用ISO-XXX解析，这就会造成乱码的情况。

我们可以通过修改`conf/server.xml`这个文件，让TOMCAT知道解析的URL参数为`UTF-8`格式:

```
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />
```

注意：URL中一般不传递参数。

## 总结

乱码问题的基本解决思路就是统一编码为`UTF-8`。如果真的遇到编码问题，则检查**各个转换点**编码是否正确。

