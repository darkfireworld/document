# Jetty热修改HTML，CSS，Javascript

发开的时候，通常需要热修改HTML资源文件，但是Jetty会锁住资源，导致修改失败。

## webdefault.xml 方式

通过修改`webdefault.xml`文件里的useFileMappedBuffer参数，把值设成false。
可以实现Jetty热加载HTML，CSS。

## code 方式

```java

context.setInitParameter("org.eclipse.jetty.servlet.Default.useFileMappedBuffer", "false");

```