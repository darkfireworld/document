# main函数

本文将介绍一下Java的main函数，C语言main函数的实现机制。

## C语言main函数

### 多种样式的main

我们知道，C语言的入口函数为main，但是通常会出现如下几种不同格式的main：

1. int main(int argc,char** argv)

2. void main() **不推荐**

3. int main()

但是，因为标准C语言不允许函数重载，所以在在运行阶段，程序如何识别该调用哪个入口函数？首先，我们需要知道Link的过程是将 main.o 和 ctr0.o（GCC默认提供的运行时） 进行链接，生成exe文件的过程。

程序载入后，首先会运行PE文件（Windows）指定的入口地址，而ctr0.o 包含了具体的入口地址_start(不同的编译器实现不同)。查看_start的汇编代码如下：

    push argv
    push argc
    call _main
    push %ebp
    mov  %esp,%ebp
    sub  12,%esp

从而构建了一个调用栈结构（_cdecl调用约定，其实_stdcall也差不多）:


    argv
    argc
    ret addr
    %ebp
    临时栈变量空间
    %esp

而在C语言中，读取一个参数的汇编代码为：

**mov -4(%ebp),%eax**

无论是哪种格式的main函数，因为在栈中都存在了具体的参数**argc，argv**，所以只要在Link的时候，出现一个_main符号，既可以完成链接，并且正确运行。

### 具体的参数内容

在C语言中，main函数的参数是可以通过 (int argc,char** argv) 来获取的，如运行命令

> main.exe a b c

其给定的main参数为：

    argc = 4
    argv = {"main.exe","a","b","c"}

## Java Main

我们都知道Java的入口是main函数，也就是

	public static void main(String args[]){
		  
	}

那么从输入命令行 

> java Main arg1 arg2 arg3 

是如何运行到 main 函数的呢？

1. java 程序（Windows下就是java.exe） 接受到参数 Main arg1 arg2 arg3

2. java 分析参数， 获取到 arg1 arg2 arg3，以及指定的Main程序

3. java 初始化Lacuner.java，配置好ClassLoader

4. java 使用AppClassLoader加载指定的Main.class.

5. java 调用Main中指定的public static void main(String args[])，其中参数为 args = ["arg1","arg2","arg3"]

上述就是Java的main函数启动过程。

# 总结

C语言和Java语言的main args 参数最大的区别在于，C语言argv[0] = main.exe, 而Java为真正的参数。

# 参考

[C语言栈与调用惯例](http://www.cnblogs.com/Anker/p/3244857.html)

[PE文件格式详解](http://www.cppblog.com/oosky/archive/2006/11/24/15614.html)