# 弹出键盘布局闪动原理和解决

在开发中，遇到一个问题：做一个微信一样，表情输入和软键盘在切换的时候，聊天界面
不闪动的问题。为了解决这个问题，需要知道一下Android的软键盘弹出的时候发生的几个变化。

当AndroidMainfest.xml 中配置**android:windowSoftInputMode="adjustResize|stateHidden"** 属性后，
如果弹出软键盘，那么会重绘界面。基本流程如下(API 10)：

1. Android 收到打开软键盘命令
2. Android 打开软键盘后，调用App 注册在AWM 中的接口，告知它，界面需要进行变化.

    2.1 调用ViewRoot.java#W#resized(w,h)
    
    2.2 调用viewRoot.dispatchResized()
    
    2.3 构造Message msg = obtainMessage(reportDraw ? RESIZED_REPORT :RESIZED)，然后post过去
    
3. 在RootView的handleMessage的case RESIZED_REPORT: 收到具体的大小，配置App Window的大小，特别是 bottom 的大小，
最后调用requestLayout进行重绘

> 所以只要在父布局onMeasure之前，隐藏/显示 合适高度的VIEW，既可以使得其他子VIEW高度不
> 变化，从而避免界面闪动。
> from [Android键盘面板冲突 布局闪动处理方案](http://blog.dreamtobe.cn/2015/09/01/keyboard-panel-switch/)

# 参考

* [KeyboardAwareDemo](https://github.com/angeldevil/KeyboardAwareDemo)
* [JKeyboardPanelSwitch](https://github.com/Jacksgong/JKeyboardPanelSwitch)
