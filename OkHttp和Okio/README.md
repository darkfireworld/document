# OkHttp和Okio

文本将介绍OkHttp和Okio基本使用

## OkHttp

HTTP 是现在APP访问网络最流行的方式。通过它我们可以交换数据和媒体信息。而高效的使用HTTP可以让你的加载数据更快并且节省带宽。

OkHttp就是一种HTTP客户端连接，它有如下特性：

1. HTTP/2多路复用Socket到同一个主机，共享链接。
2. **采用连接池技术，可以有效的减少Http链接数量。**
3. 无缝集成GZIP压缩技术。
4. 支持Response Cache，避免重复请求。
5. 域名多IP支持。

OkHttp可以处理常见的网络问题：

1. 如果OkHttp连接一个域名失败后，它会尝试连接下一个该域名的IP地址。**需要DNS支持** 
2. OkHttp在初始化链接的时候，会采用最新的TLS特性（SNI，ALPN），如果失败会采用TLS1.0进行链接

使用OkHttp是非常简单的。它的request/response API采用非常流畅的Builder模式构建。
并且它支持同步阻塞调用以及异步调用。

OkHttp支持Android2.3+。对于Java最低支持1.7+

OkHttp会**自动管理HTTP连接的生命周期**：

1. 操作Response.body().string()等类型的API，OkHttp会自动将该HTTP连接加入到ConnectionPool中或者直接释放连接
2. 如果采用stream方式操作流，则需要自己手动关闭，否则会发生HTTP连接泄漏（OkHttp通过WeakReference机制，尽最大努力管理这些泄漏的HTTP连接）
3. OkHttp不读取Resonse\#Head\#Keep-Alive属性来决定该HTTP连接是否能复用，而是直接加入到ConnectionPool进行复用
4. 当从ConnectionPool中获取HTTP连接的时候，OkHttp发现该HTTP连接已经失效，则关闭该连接，并且重新选择一个HTTP连接进行复用
5. OkHttp会定时清空ConnnectionPool中超时或者超量的HTTP连接

### GET 请求

This program downloads a URL and print its contents as a string. Full source.

	package com.company;

	import okhttp3.*;

	public class Main {

		public static void main(String[] args) throws Exception {
			OkHttpClient client = new OkHttpClient();
			//请求
			Request request = new Request.Builder()
					.url("http://www.baidu.com/")
					.get()
					.build();
			//发起请求
			Response response = client.newCall(request).execute();
			//结果
			System.out.println(response.body().string());
		}
	}
		

### POST 请求

This program posts data to a service. Full source.

	package com.company;

	import okhttp3.*;

	public class Main {

		public static void main(String[] args) throws Exception {
			OkHttpClient client = new OkHttpClient();
			//参数
			RequestBody requestBody = new FormBody.Builder()
					.add("DGM", "DGM")
					.build();
			//请求
			Request request = new Request.Builder()
					.url("http://www.baidu.com/")
					.post(requestBody)
					.build();
			//发起请求
			Response response = client.newCall(request).execute();
			//结果
			System.out.println(response.body().string());
		}
	}

### 集成

**OkHttp需要依赖Okio采用使用。**

**Maven**

	<dependency>
	  <groupId>com.squareup.okhttp3</groupId>
	  <artifactId>okhttp</artifactId>
	  <version>(insert latest version)</version>
	</dependency>

**Gradle**

    compile 'com.squareup.okhttp3:okhttp:(insert latest version)'

## Okio

Okio是一款新的类库，它可以使得 java.io.\* 和 java.nio.\* 更加方便的被使用以及处理数据。

### Copy例子

	package com.company;

	import okio.BufferedSink;
	import okio.BufferedSource;
	import okio.Okio;

	import java.io.File;

	public class Main {

		public static void main(String[] args) throws Exception {
			//创建buffer
			BufferedSource source = Okio.buffer(Okio.source(new File("data/file1")));
			BufferedSink sink = Okio.buffer(Okio.sink(new File("data/file" + System.currentTimeMillis())));
			//copy数据
			sink.writeAll(source);
			//关闭资源
			sink.close();
			source.close();
		}
	}


可以发现，通过Okio可以非常方便的处理io数据。

### ByteString 和 Buffer

在Okio中通过ByteString和Buffer这两种类型，提供了高性能和简单的API：

1. ByteString是一种不可改变的byte序列。提供了一种基于String，采用char访问的二进制模式。通过ByteString可以像一般value一样处理二进制数据。并且提供了对encode/decode中的HEX，Base64以及UTF-8支持。

2. Buffer是一种可变的byte序列。就像ArrayList一样，你不需要知道Buffer的大小。在处理buffer的read/write的时候，就像queue一样。

通过这两个类，可以极大的增强io访问的数据处理。

### Source 和 Sink

这两个类是在 InputStream 以及 OutputStream 上进行抽象而成的。 它还具有如下特性：

1. Timeout： 可以提供超时处理机制。

2. Easy to implement： Source 仅仅声明了read，close，timeout方法。实现起来非常的方便。

3. Easy to use：通过实现/使用BufferedSource和BufferedSink接口，可以更加方便的操作二进制数据。

4. No artificial distinction between byte streams and char streams：可以非常方便的将二进制数据处理为UTF-8字符串，int等类型数据。

Source 和 Sink 实现了InputStream 以及 OutputStream。你可以将Source看成InputStream，将Sink看成OutputStream。**而通过BufferedSource和BufferedSink可以非常方便的进行数据处理。**

## 总结

通过开源的square工具，我们可以非常方便的处理io以及http数据。在最新的Android6.0+中，已经剔除了Apache URLConnection类，而采用OkHttp。所以可见OkHttp的代码质量还有有保证的。

## 参考

* [Okio](https://github.com/square/okio)
* [OkHttp](http://square.github.io/okhttp/)
* [OkHttp3源码分析-复用连接池](http://www.jianshu.com/p/92a61357164b)