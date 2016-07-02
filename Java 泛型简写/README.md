# Java 泛型简写

> 有许多原因促成了泛型的出现，而最引人注意的一个原因，就是为了创建容器类。

# 泛型类

```java

/**
 * 泛型类
 */
public class Store<T> {
    T value;

    public Store(T value) {
        this.value = value;
    }

    static public void main(String[] args) {
        Store<String> store = new Store<String>("DARKGEM");
        System.out.println(store.getValue());
    }

    public T getValue() {
        return value;
    }

    public void setValue(T value) {
        this.value = value;
    }
}

```

# 泛型接口

```java

public class App {
    public static void main(String[] args) {
        Store<String> store = new Store<String>() {
            String val;

            public String get() {
                return val;
            }

            public void set(String val) {
                this.val = val;
            }
        };
        store.set("DARKGEM");
        System.out.println(store.get());
    }
    /**
     * 泛型接口
     */
    public interface Store<T> {
        T get();

        void set(T val);
    }
}

```

# 泛型方法

```java

public class App {

    /**
     * 泛型方法
     */
    static <T> String transform(T val) {
        return String.valueOf(val);
    }

    static public void main(String[] args) {
        System.out.println("AAA");
    }
}

```
# <? extends Abs>

```java

import java.util.LinkedList;
import java.util.List;

public class App {

    public static void main(String[] args) {
        //compile pass
        Abs abs = new Impl();
        //compile fail
        List<Abs> fail = new LinkedList<Impl>();
        //compile pass
        List<? extends Abs> pass = new LinkedList<Impl>();
    }

    /**
     * 抽象
     */
    static abstract class Abs {

    }

    /**
     * 实际
     */
    static class Impl extends Abs {

    }
}

```

通过 `<? extends Abs>` 可以实现`父抽象类容器`引用`子实现类容器`这种情况。

# Tips

泛型的概念是针对`编译器`的，在编译处理后，`泛型信息都可以被去除，然后使用Object代替`，因为Object是所有对象的Root。

泛型在一定程度上，可以美化我们的code，在编译时就可以确定一些错误。但是也不要`滥用`泛型特性。

# 参考

[Java泛型：泛型类、泛型接口和泛型方法](https://segmentfault.com/a/1190000002646193)