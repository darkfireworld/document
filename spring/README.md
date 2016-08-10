# Spring

## Ioc

### refresh

```java
    
    AnnotationConfigApplicationContext:
        1. 定义一个DefaultListableBeanFactory作为beanFactory
        2. 加载默认的BeanFactoryPostProcessor处理@Autowired
        3. 注册给定的 SpringConf
        
    
    synchronized (this.startupShutdownMonitor) {
        // 初始化容器环境
        prepareRefresh();

        // 告诉子类，开始refresh容器，并且最后获取beanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // 预处理这个beanFactory，添加一些通用的特性
        // 1. 添加 ApplicationContextAwareProcessor 用来处理 ResourceLoaderAware,ApplicationContextAware,EnvironmentAware
        // 2. 添加 environment,systemProperties,systemEnvironment 这几种类型的bean对象到singleton对象池中
        // 3. beanFactory.ignoreDependencyInterface 添加一些被忽略依赖的接口 ???
        // 4. beanFactory.registerResolvableDependency 添加一些默认依赖解析对象
        
        prepareBeanFactory(beanFactory);

        try {
            // 允许子类进行处理beanFactory
            postProcessBeanFactory(beanFactory);

            
            // 调用 BeanFactoryPostProcessor 
            // 
            // part1: BeanDefinitionRegistry 类型处理
            //     如果 beanFactory instanceof BeanDefinitionRegistry 则进行如下流程（默认）
            //     1. 调用beanFactory中所有的 BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry方法，默认为：ConfigurationClassPostProcessor
            //          1. 调用getBean实例化这个 BeanDefinitionRegistryPostProcessor，存储到集合 processedBeans 中
            //          2. 通过 BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry加载更多的Bean
            //              1. ConfigurationClassPostProcessor 设置beanFactory#beanDefinitionMap为初始扫描集合S
            //              2. 通过 ConfigurationClassPostProcessor 通过 ConfigurationClassParser 解析集合S
            //              3. ConfigurationClassParser 会解析通过注解 @Component 指向的对象，然后分析注解 @ComponentScan,@PropertySources,@ImportResource,@Bean,@Import
            //              4. 将新添加的bean设置为集合S，如果集合S.size() != 0 ，则循环步骤2(默认 ConfigurationClassParser 会迭代处理步骤三扫描出来的bean)
            //     2. 检测是否存在新的 BeanDefinitionRegistryPostProcessor 对象，如果存在则继续步骤1。
            //     3. 调用processedBeans集合中的所有BeanFactoryPostProcessor#postProcessBeanFactory 方法
            //     
            // part2: !BeanDefinitionRegistry 类型处理
            //     1. 获取所有 BeanFactoryPostProcessor 类型，并且实例化，然后调用 BeanFactoryPostProcessor#postProcessBeanFactory 方法
                
            invokeBeanFactoryPostProcessors(beanFactory);

            // 实例化 beanFactory中的 BeanPostProcessor，并且注册它们到beanFactory中
            registerBeanPostProcessors(beanFactory);

            // 初始化国际化支持
            initMessageSource();

            // 初始化Spring事件总线
            initApplicationEventMulticaster();

            // 针对特殊的上下文初始化一些特定的bean
            onRefresh();

            // 注册ApplicationListener的bean都Spring事件总线上
            registerListeners();

            // 初始化所有非lazy的singleton类型的bena到容器中
            finishBeanFactoryInitialization(beanFactory);

            // 完成refresh操作，发布相应的事件
            finishRefresh();
        }

        catch (BeansException ex) {
           ...
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }

```


### BeanFactoryPostProcessor 生命周期

### BeanPostProcessor 生命周期

### Bean 生命周期


http://developer.51cto.com/art/201104/255961.htm

## Aop

## Tx

## Mvc

## 总结
