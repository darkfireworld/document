# Java乱码

在实际开发中会遇到乱码问题，**通过采用统一的编码UTF-8**，我们可以规避大部分乱码问题。

> UTF-8 格式编码格式兼容性最好，可以解决网络字节序大小端问题，并且可以表示大部分字符。

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

在使用`JDBC`连接MySQL的时候，也需要注意连接的字符集为`UTF-8`，这样子，`MySQL的JDBC驱动`能正常的识别编码：

```

jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8

```

## Java代码

在代码中，会出现中文乱码的问题，建议统一采用 UTF-8的编码格式，然后编译的时候，指定编码格式 `javac -encoding utf8`

且注意项目的默认字符集也设置为`utf-8`格式

## JSP

JSP的乱码问题，也和Java代码乱码一样，需要统一配置为`UTF-8`。

## web.xml

在容器中，可以尽可能的使用UTF-8来解析`request/response`报文：

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

## TOMCAT 

TOMCAT如果没有配置好，那么会出现乱码问题，因为TOMCAT默认采用ISO-XXX的编码，即接受到HTTP请求后，会默认的认为该编码为IOS-XXX进行解码，
SO，到Java业务代码中的时候，中文就乱码了。我们需要修改conf/server.xml这个文件，让它知道发送过来的字符集为UTF-8:

```
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />
```

## HTTP Content-Type

HTTP 的`HEAD`中存在一个`Content-Type`属性，通过这个属性，我们可以规定`HTTP`的编码格式：

```

request:
    Content-Type:application/x-www-form-urlencoded;charset=utf-8
    
response:
    Content-Type: text/html;charset=utf-8
    
```

默认的request HTTP请求为UTF-8编码，而response为当前`ISO-XXXX`编码，需要指定为`UTF-8`。


## 总结

乱码问题的基本解决思路就是统一编码为`UTF-8`。如果真的遇到编码问题，则从`数据库->Application->HTTP报文->Client`分步调试，看看究竟是`哪个转换步骤`乱码了。

