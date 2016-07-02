# android 启动模式简介

## 启动模式

* **standard模式（默认）**:只要创建了Activity实例，一旦激活该Activity，则会向任务栈中加入新创建的实例，退出Activity则会在任务栈中销毁该实例。
* **singleTop模式**: 如果激活的Activity实例处于栈顶则无需重新创建新的实例，会重用已存在的实例，否则会在任务栈中创建新的实例。
* **singleInstance模式**: Activity单独在一个taskStack中。如果已经存在则调用newIntent(),否则新创建一个放入单独的taskStack中。
* **singleTask模式**：
    1. 寻找相同taskaffinity的Task，如果没有寻找到，则新启动一个Task，并且把该Activity放置到底部
    2. 如果存在相同的taskaffinity，那么再查询TaskStack中是否存在已经实例化的Activity，如果存在，则清空该Activity上面的所有Activity，最后调用newIntent(), 否则新创建一个Activity放入Task中

## onActivityResult

在开发中经常会使用onActivityResult,来处理Activity之间的数据返回，使用该方法，有几个关键点:

1. 传递数据的时候，两个Activity必须在同一个Task Stack中，所以最好保证两个Activity都使用standard的启动模式。
2. 调用startActivityForResult的时候，requestCode必须大于0。
3. 返回的时候，注意使用setResult来设置返回值，且统一使用RESULT_OK等标准代码点来设置执行状态  

## INTENT FLAG

* FLAG_ACTIVITY_NEW_TASK ： 如果taskaffinity指向的TaskStack不存在，则新创建一个。
* FLAG_ACTIVITY_CLEAR_TOP :  清空要跳转Activity上面所有的Activity实例。
* FLAG_ACTIVITY_SINGLE_TOP：和singleTop启动模式类似。


## Activity Stack

### 问题描述

在项目中，遇到了需要像微信一样管理Activity，即实现效果如下：

假设App中有Main Activity（Root），Chat Activity，以及 Other Activity 三个Activity，其中Main Activity 在位于栈底，且为启动Activity

1. App内，无论从哪里进入ChatActivity，Back回去都进入Main Activity
2. 从HOME进入App，都为最后进入的Activity
3. 从通知栏进入App，都默认进入Chat Activity

### 实现效果1

为了实现效果1，通常有两种方法：

1. 修改Chat Activity 的Back方法，生成一个 Intent FLAG : SINGLE_TOP | CLEAR_TOP 标记的Intent，这样子，通过这个Intent启动MainActivity，会清空MainActivity上面所有的Activity。
2. 启动Chat Activity的的Intent中，直接添加 Intent FLAG : SINGLE_TOP | CLEAR_TOP ，从Main Activity启动，这样子在启动Chat Activity的时候，直接清空Main Activity上面所有的Activity，然后从Main startActivity(Chat Activity);

### 实现效果2

实现这个的效果，最重要的是，不使用特殊的启动模式，singleTask，singleInstance（未测试），可以使用singleTop，standard模式

### 实现效果3

这个效果实现的方法，和实现效果1类似，不过需要添加NEW_TASK 这种Intent Flag。然而，
如果我们把三种效果混合在一起，可能就遇到如下的问题。

### 混合问题

从通知进入ChatActivity，然后点击HOME按钮，进入程序列表页面，然后再点击App的ICON，进入App，这时候，我们发现**效果2**无法实现，而是新建了MainActivity，点击Back按钮，进入的页面是之前的ChatActivity，相当于AC栈为**MainActivity->ChatActivity->MainActivity.**

为什么会遇到这个问题？ 进过搜索资料以及实验，得出如下的结论：

1. 当Main Activity 的launchMode为singleTask, singleInstance（未测试） 的时候，通常不创建新的MainActivity
2. 当Main Activity的launchMode为standard,或者为signleTop的时候，则按照 栈顶的Activity（也就是Main Activity）的action 和category 是否和androidManifest.xml中的 `<action android:name="android.intent.action.MAIN"/>　和  <category android:name="android.intent.category.LAUNCHER"/>`一致，如果非一致，则创建新的Main Activity，否则就是简单的AC栈移动到前台。

为了解决这个问题，我们只需要在启动MainActivity的时候，添加

```java

intent.setAction(Intent.ACTION_MAIN);
intent.addCategory(Intent.CATEGORY_LAUNCHER);
　　　　 
当然为了能直接使用Context.startActivity 使用Intent，可以统一添加
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

```

### 最终解决方案

其实，上述遇到的问题解决起来比较棘手，其实最主要的原因是：**我们无法控制Activity Stack**。所以，
只要我们可以在应用内管理`Activity Stack`，上述的问题就可以迎刃而解。所以，隆重推荐一个解决方案：

1. 每次新建/销毁Activity的时候，将该Activity 从`自建的ActivityManager`中添加/删除。
2. 通过`ActivityManager`我们就可以轻松管理`ActivityStack的问题`。


