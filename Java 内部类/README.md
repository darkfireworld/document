# Java 内部类

在Java中，类是一个非常基本的概念，它贯穿了整个Java体系。在实际开发中，我们会经常创建不同的类。今天我们将来了解一下**Java内部类**。

## 成员内部类

这种内部类是最常见的，如：

	public class Main {

		public static void main(String[] args) {
			new Main().new Inner().println();
		}

		class Inner {
			private void println() {
				System.out.println("AAA");
			}
		}
	}

可以发现Inner就是一个简单的内部类，值得注意的是它的初始化，**需要优先初始化Main对象。**

## 匿名类

在Java中，特别是Android编程或者SWT编程，我们经常使用匿名类：

	public class Main {
		interface Callback{
			void ok();
		}
		public static void main(String[] args) {
			Callback callback = new Callback() {
				//构造代码区域
				{
					System.out.println("Build");
				}
				@Override
				public void ok() {
					System.out.println("OK");
				}
			};
			callback.ok();
		}
	}


通过匿名类，我们可以构造一个回调，而不用编写一个具体的类，好处就是内聚性强，不会出现莫名其妙的类被全局可见。而缺点就是缺少了构造函数。当然我们可以通过一定的手段来伪造一个构造函数，**如 {} 代码块**。

## 局部内部类

在函数中，我们也可以创建一个类：

	public class Main {

		public static void main(String[] args) {
			class Inner {
				private void println() {
					System.out.println("AAA");
				}
			}
			new Inner().println();
		}
	}

是不是感觉非常的不可思议，其实这种方式和我们经常使用的匿名类其实是一个道理，仅仅是多了构造方法和类名。因为该类的作用域为函数块，所以能避免污染全局空间。

## 静态内部类

在Java中，还有一种类，它们被称为静态内部类：

	public class Main {

		public static void main(String[] args) {
			new StaticClass().println();
		}

		static class StaticClass {
			public void println() {
				System.out.println("Static Class");
			}
		}
	}

可以发现，静态内部类和成员内部类非常的类似，唯一的区别就是添加了**static**关键字。通过声明static关键字，**使得该类能被独立的实例化，而不需要依赖Main。**

值得注意的是，Java文件中顶级类：

	public class Main {

		public static void main(String[] args) {
			System.out.println("Main");
		}
	}
	class TopCls{
		
	}


**如Main和TopCls其实都是默认的静态类。**