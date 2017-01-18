# Markdown语法

## 标题

```

# 标题一

## 标题二

### 标题三

```

## 块注释

```

> 注释1
> 注释2

```

## 斜体

```

*我是斜体*

```

## 粗体

```

**我是粗体**

```

## 分割线

```

我下面是分割线

***

我上面是分割线

```

## 无序列表

```

* 列表1

* 列表2

* 列表3

* 列表4

```

## 有序列表

```

1. 列表1

2. 列表2

3. 列表3

4. 列表4

```

## 链接

```

[baidu](http://www.baidu.com)

```

## 图片

```

![图片](http://static.oschina.net/uploads/space/2016/0314/214128_DEBJ_1999248.jpg)

```

## 代码

### 方法一

所有代码前添加一个tab

```

    public class Main{
        public static void main(String args[]){
            System.out.println("Markdown");`
        }
    }

```

### 方法二

使用如下格式处理代码片段

    ```java

    code

    ```
    
### 方法三

```

使用 `code`的方式处理代码，一般使用这种方法处理单条代码。

```

## 表格

```

| Tables        | Are           | Cool  |
|---------------|---------------|-------|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

```

## 内嵌HTML

```

	<table>
		<tr>
			<th>表头1</th>
			<th>表头2</th>
		</tr>
		<tr>
			<td>TDC</td>
			<td>TDC</td>
		</tr>
	</table>

```

`markdown`也支持内嵌html代码，加强了对复杂页面的呈现能力。

## 总结

markdown是一种神奇的语言，多练练就会用了。因为Markdown是纯文本格式，所以在对SVN和GIT支持非常好，不像doc文件，
修改后也不知道具体修改了什么地方。