# java中private、protected、public和default 区别

在java中我们使用private、protected、public和default来控制类中属性和函数的访问：

|           | 类内部| 本包    | 子类   |外部包 |
|-----------|-------|--------|-------|-------|
|public     |  √    | √	     |  √   |  √    |
|protected  |√	    |√	     |√	    | ×     |
|default    | √	    |√	     |×     |×      |
|private    | √	    |×	     |×	    |×      |


注意：Java的访问控制是停留在编译层的，也就是它不会在.class文件中留下任何的痕迹，只在编译的时候进行访问控制的检查。