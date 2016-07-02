# Java乱码

在开发JavaEE的过程中，新手们会经常遇到乱七八糟的乱码问题。**其根本问题，就是字符编码未统一。**在实际开发中，**建议采用统一的编码UTF-8**。因为该格式的兼容性最好，可以解决网络字节序大小端问题，并且可以表示大部分字符。

## 代码中文乱码

在代码中，会出现中文乱码的问题，建议统一采用 UTF-8的编码格式，然后编译的时候，指定编码格式 javac -encoding utf8

且注意项目的默认字符集也设置为utf-8格式

## JSP乱码

JSP的乱码问题，也和Java代码乱码一样，需要统一配置为UTF-8

## TOMCAT 乱码

TOMCAT如果没有配置好，那么会出现乱码问题，因为TOMCAT默认采用ISO-XXX的编码，即接受到HTTP请求后，会默认的认为该编码为IOS-XXX进行解码，SO，到Java业务代码中的时候，中文就乱码了。

我们需要修改conf/server.xml这个文件，让它知道发送过来的字符集为UTF-8

    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />

## Client端乱码

因为和TOMCAT约定的字符集编码为UTF-8，所以，我们也需要在Client端发送的请求和接受的响应采用UTF-8进行解码。

## MySQL 乱码

统一MySQL服务器编码为UTF-8：

```
[mysqld]#mysqld
character-set-server=utf8 
[mysql] #mysql
default-character-set=utf8
[client] #client
default-character-set=utf8
```

以及设置JDBC连接：

```
jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8

```