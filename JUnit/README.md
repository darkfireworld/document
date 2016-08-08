# JUnit

JJUnit是用于编写和运行可重复的自动化测试的开源测试框架， 这样可以保证我们的代码按预期工作。
JUnit可广泛用于工业和作为支架(从命令行)或IDE(如Eclipse)内单独的Java程序。

## 入门

JUnit的安装和使用都非常的简单。这里使用**IDEA+Maven**演示。

**创建项目**

使用Idea和Maven创建一个最简单的Java项目：

![](A0F1.tmp.jpg)

**添加JUnit4.x依赖**

```
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
    </dependencies>
	
```

这样子，就算完成了JUnit的基本安装。

**例子**

```java

package app;

import org.junit.*;

import java.util.ArrayList;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 命名规则为：ClassNameTest
 */
public class AppTest {
    static final AtomicInteger count = new AtomicInteger(0);
    private ArrayList testList;

    /**
     * 每次运行@Test方法，都会实例化一个对象。
     */
    public AppTest() {
        System.out.println(String.format("CONSTRUCT CALL %d", count.incrementAndGet()));
    }

    /**
     * 指定一个静态方法，在所有@Test方法之前，执行一次。
     */
    @BeforeClass
    public static void onceExecutedBeforeAll() {
        System.out.println("@BeforeClass: onceExecutedBeforeAll");
    }

    /**
     * 指定一个静态方法，在所有@Test方法之后，执行一次。
     */
    @AfterClass
    public static void onceExecutedAfterAll() {
        System.out.println("@AfterClass: onceExecutedAfterAll");
    }

    /**
     * 在所有@Test方法之前执行
     */
    @Before
    public void executedBeforeEach() {
        testList = new ArrayList();
        System.out.println("@Before: executedBeforeEach");
    }

    /**
     * 在所有@Test方法之后执行
     */
    @After
    public void executedAfterEach() {
        testList.clear();
        System.out.println("@After: executedAfterEach");
    }

    /**
     * 命名规则：FunctionNameTest
     */
    @Test
    public void EmptyCollectionTest() {
        Assert.assertTrue(testList.isEmpty());
        System.out.println("@Test: EmptyArrayList");

    }

    /**
     * 命名规则：FunctionNameTest
     */
    @Test
    public void OneItemCollectionTest() {
        testList.add("oneItem");
        Assert.assertEquals(1, testList.size());
        System.out.println("@Test: OneItemArrayList");
    }

    /**
     * 忽略这个测试方法
     */
    @Ignore
    public void executionIgnoredTest() {
        System.out.println("@Ignore: This execution is ignored");
    }
}

```

上述是一个非常经典的例子，囊括了JUnit测试对象的**生命周期**。

**运行TestCase**

运行`TestCase`是非常方便的。现在几乎所有的主流IDE（Idea，Eclipse）都支持JUnit。以下是Idea的启动过程：

![](AEE7.tmp.jpg)

这样子就开启了调试模式运行`TestCase`。

**运行日志**

```
@BeforeClass: onceExecutedBeforeAll
CONSTRUCT CALL 1
@Before: executedBeforeEach
@Test: EmptyArrayList
@After: executedAfterEach
CONSTRUCT CALL 2
@Before: executedBeforeEach
@Test: OneItemArrayList
@After: executedAfterEach
@AfterClass: onceExecutedAfterAll
```

可以发现，JUnit的**生命周期**和注释保持一致。

## 提升

在上面小节，我们了解了JUnit的基本使用。我们现在来说一说JUnit的其他高级一点的内容。

### JUnitCore

在没有IDE的情况下，我们可以借助`main函数`，来运行我们的`TestCase`：

```
package runner;

import app.AppTest;
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure;

public class Main {
    public static void main(String[] args) {
        //通过JUnitCore指定，需要进行测试的TestCase
        Result result = JUnitCore.runClasses(AppTest.class);
        //搜集失败的测试用例信息
        for (Failure fail : result.getFailures()) {
            System.out.println(fail.toString());
        }
        //判断，单元测试是否全部通过
        if (result.wasSuccessful()) {
            System.out.println("All tests finished successfully...");
        }
    }
}

```

这样子，我们就可以通过命令行运行JUnit。

### Suite

在JUnit中，我们可以将几个`TestCase`合并在一起进行单元测试：

![](8F2A.tmp.jpg)

通过`@Suite.SuiteClasses()`将几个`TestCase`合并在一起，方便单元测试。

### 异常和超时

在某些情况下，我们需要测试`异常`和`超时`这两种情况。而这是通过`@Test.expected`和`@Test.timeout`来实现的。


```java

    /**
     * expected 期待获取的异常类型
     * timeout 测试用例超时时间
     * */
    @Test(expected = Exception.class, timeout = 1000)
    public void OneItemCollectionTest() throws Exception {
        Thread.sleep(500);
        System.out.println("@Test: OneItemArrayList");
    }
	
```

## Spring 整合

## 源码分析

每次调用@Test，都会新建对象

Spring ApplicationContext 对象复用

## 最佳实践

命名规范:类Test 函数Test

测试范围：函数级别，别跨@Test函数调用

## 参考

* [JUnit教程](http://www.yiibai.com/junit/)
* [利用junit对springMVC的Controller进行测试](http://www.tuicool.com/articles/7rMziy)
* [Maven单元测试报告及测试覆盖率](http://www.cnblogs.com/qinpengming/archive/2016/02/28/5225380.html)