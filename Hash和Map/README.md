# Hash和Map

在Java中，我们经常使用HashMap，HashSet，HashTable等类型，但是它们背后到底是什么呢？

## Hash

什么是Hash？来看一下百度的解释：

>Hash，一般翻译做“散列”，也有直接音译为“哈希”的，就是把任意长度的输入（又叫做预映射， pre-image），通过散列算法，变换成固定长度的输出，该输出就是散列值。

**Hash其实就是一种算法，通过该算法，可以把巨量长度的内容，均匀地计算到固定空间大小的值域中。**

**Hash的内容一致，则计算出来的Hash值也必定一致；但是Hash值一致，内容不一定一致。**

## Map

在编程语言中，Map是一种数据结构类型，简单的来说就是一种KV数据模型，通过Key可以快速的获取读取到Value。相同的Key指向同一个Value，而不同的Key指向不同的Value。

## HashMap

在Java中，我们经常使用HashMap，通过HashMap我们能在O(1)的时间内读取到Value，极大的加快了数据的查询。

但是HashMap的构造原理是什么？其实从名称上，我们不难得出和Hash相关。我们首先要了解一下Object#hashCode() 以及 Object#equals() 两个和Hash息息相关的API。

### Object#hashCode

我们先看一下关于该API的Javadoc描述中的一段：

>If two objects are equal according to the {@code equals(Object)}
method, then calling the {@code hashCode} method on each of
the two objects must produce the same integer result.

** 如果两个对象相等，则返回的hashCode一定相等。**

其实这个API就是对象经过hash运算后，返回的hash值。**通常返回对象的内存地址，但是Java中一些原生类型已经被重写了这个方法，如 String,Integer,Double等。**

注意：**System.identityHashCode(obj) 永远返回对象的内存地址，即使对象的hashCode被重写了。**

### Object#equals

该API的Javadoc描述：

>Indicates whether some other object is "equal to" this one.

这个API其实就是对比两个对象是否相等，通常：

	public boolean equals(Object obj) {
		return (this == obj);
	}

**注意：在Java中一个对象的引用可以认为是该对象的内存地址。这个和C语言的指针类似。**

默认是直接对比内存地址是否相等来判断是否为同一个对象。但是 String,Integer,Double 的该方法已经被重写。

### 内部机制

HashMap的内部机制其实非常的简单。其构造图为：

![HashMap](3f61e235-4213-4680-a966-4bad62c0fbc1.jpg)

通过如下步骤，就可以通过KEY快速的定位到VALUE：

1. 通过计算，获取KEY的hashCode
2. 通过hashCode % size 定位到具体的HashTable的槽中
3. 如果存在元素，则需要遍历在上面的 List，通过equals来确定KEY相等的Entry，然后返回它

**所以，可以知道hashCode其实就是将KEY映射到某个槽中，避免遍历整个TABLE来查询KEY相等的Entry，最后通过equals判断List上KEY相等的Entry。**

## 总结

通过Hash算法，可以计算出某个数据的**特征值**。通过特征值，可以把数据存放在不同的地址加快查询，也可以检测数据是否被篡改过。