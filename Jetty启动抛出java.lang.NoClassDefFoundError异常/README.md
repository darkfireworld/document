# Jetty启动抛出java.lang.NoClassDefFoundError异常

启动的时候，使用`java -cp jetty.jar;WebContent\WEB-INF\classes Launcher`启动项目。结果
抛出`java.lang.NoClassDefFoundError`异常。代码如下：

```

/**
 * 启动器
 */
public class Launcher {
    public static void main(String args[]) throws Exception {
        new Launcher().start();
    }

    void start() throws Exception {
        // 服务器的监听端口
        Server server = new Server(80);
        // 关联一个已经存在的上下文
        WebAppContext context = new WebAppContext();
        // 设置描述符位置
        String path = Launcher.class.getResource("/").getPath();
        context.setDescriptor(path + "../web.xml");
        // 设置Web内容上下文路径
        context.setResourceBase(path + "/../../");
        // 设置上下文路径
        context.setContextPath("/admin/");
        context.setParentLoaderPriority(true);
        //开启HTML，CSS，JS热部署
        context.setInitParameter("org.eclipse.jetty.servlet.Default.useFileMappedBuffer", "false");
        server.setHandler(context);
        // 启动
        server.start();
        server.join();
    }
}

```


报错日志：

```
2016-06-05 17:01:13.080 [main] ERROR o.s.web.servlet.DispatcherServlet - Context initialization failed
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'addressConsigneeIo' defined in file [D:\Project\CORP\EP\netease-ep\trunk\WebContent\WEB-INF\classes\org\darkgem\io\address\AddressConsigneeIo.class]: Post-processing failed of bean type [class org.darkgem.io.address.AddressConsigneeIo] failed; nested exception is java.lang.IllegalStateException: Failed to introspect bean class [org.darkgem.io.address.AddressConsigneeIo] for resource metadata: could not find class that it depends on
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyMergedBeanDefinitionPostProcessors(AbstractAutowireCapableBeanFactory.java:940)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:518)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:482)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:772)
    at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:839)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:538)
    at org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:667)
    at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:633)
    at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:681)
    at org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:552)
    at org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:493)
    at org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)
    at javax.servlet.GenericServlet.init(GenericServlet.java:244)
    at org.eclipse.jetty.servlet.ServletHolder.initServlet(ServletHolder.java:640)
    at org.eclipse.jetty.servlet.ServletHolder.initialize(ServletHolder.java:419)
    at org.eclipse.jetty.servlet.ServletHandler.initialize(ServletHandler.java:875)
    at org.eclipse.jetty.servlet.ServletContextHandler.startContext(ServletContextHandler.java:348)
    at org.eclipse.jetty.webapp.WebAppContext.startWebapp(WebAppContext.java:1379)
    at org.eclipse.jetty.webapp.WebAppContext.startContext(WebAppContext.java:1341)
    at org.eclipse.jetty.server.handler.ContextHandler.doStart(ContextHandler.java:774)
    at org.eclipse.jetty.servlet.ServletContextHandler.doStart(ServletContextHandler.java:261)
    at org.eclipse.jetty.webapp.WebAppContext.doStart(WebAppContext.java:517)
    at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:68)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.start(ContainerLifeCycle.java:132)
    at org.eclipse.jetty.server.Server.start(Server.java:405)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart(ContainerLifeCycle.java:106)
    at org.eclipse.jetty.server.handler.AbstractHandler.doStart(AbstractHandler.java:61)
    at org.eclipse.jetty.server.Server.doStart(Server.java:372)
    at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:68)
    at org.darkgem.jetty.Launcher.start(Launcher.java:31)
    at org.darkgem.jetty.Launcher.main(Launcher.java:11)
Caused by: java.lang.IllegalStateException: Failed to introspect bean class [org.darkgem.io.address.AddressConsigneeIo] for resource metadata: could not find class that it depends on
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.findResourceMetadata(CommonAnnotationBeanPostProcessor.java:334)
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.postProcessMergedBeanDefinition(CommonAnnotationBeanPostProcessor.java:287)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyMergedBeanDefinitionPostProcessors(AbstractAutowireCapableBeanFactory.java:935)
    ... 34 common frames omitted
Caused by: java.lang.NoClassDefFoundError: Lorg/springframework/jdbc/core/JdbcTemplate;
    at java.lang.Class.getDeclaredFields0(Native Method)
    at java.lang.Class.privateGetDeclaredFields(Class.java:2583)
    at java.lang.Class.getDeclaredFields(Class.java:1916)
    at org.springframework.util.ReflectionUtils.getDeclaredFields(ReflectionUtils.java:710)
    at org.springframework.util.ReflectionUtils.doWithLocalFields(ReflectionUtils.java:652)
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.buildResourceMetadata(CommonAnnotationBeanPostProcessor.java:351)
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.findResourceMetadata(CommonAnnotationBeanPostProcessor.java:330)
    ... 36 common frames omitted
Caused by: java.lang.ClassNotFoundException: org.springframework.jdbc.core.JdbcTemplate
    at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:331)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
    ... 43 common frames omitted
    
```

从错误日志可以看出是因为找不到 `org.springframework.core.JdbcTemplate` 这个类。而发生的问题的源码位于` classes `下面的业务代码中。

从类加载器的角度来分析，所以形成如下结构：

```

AppClassloader(业务代码，依赖spring.jar)
      |
WebAppClssLoader(WEB-INF\lib\spring.jar)

```

这会造成一个问题：Spring的WebAppClssLoader 能找到具体的业务代码类，但是，`业务代码的AppClassLoader却无法查
询到Spring类(ClassLoader双亲委托机制)`。所以，造成了`java.lang.ClassNotFoundException`错误的抛出

解决方法：

1. 启动的时候指定所有的lib和jar
2. 指定WebAppClassLoader不使用AppClassLoader：`context.setParentLoaderPriority(false)`



