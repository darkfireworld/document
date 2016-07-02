# Android PowerManager

首先Android手机有两个处理器，一个叫Application Processor（AP），一个叫
Baseband Processor（BP）。AP是ARM架构的处理器，用于运行Linux+Android系统；
BP用于运行实时操作系统（RTOS），通讯协议栈运行于BP的RTOS之上。

非通话时间，BP的能耗基本上在5mA左右，而AP只要处于非休眠状态，能耗至少在50mA以上
，执行图形运算时会更高。另外LCD工作时功耗在100mA左右，WIFI也在100mA左右。
一般手机待机时，AP、LCD、WIFI均进入休眠状态，这时Android中应用程序的代码也会停
止执行。

**Android为了确保应用程序中关键代码的正确执行，提供了Wake Lock的API，使得应用程序
有权限通过代码阻止AP进入休眠状态。**但如果不领会Android设计者的意图而滥用Wake Lock API，
为了自身程序在后台的正常工作而长时间阻止AP进入休眠状态，就会成为待机电池杀手。
比如前段时间的某应用，比如现在仍然干着这事的某应用。

**首先，完全没必要担心AP休眠会导致收不到消息推送。通讯协议栈运行于BP，一旦收到数据包，
BP会将AP唤醒，唤醒的时间足够AP执行代码完成对收到的数据包的处理过程。**其它的如
Connectivity事件触发时AP同样会被唤醒。那么唯一的问题就是程序如何执行向服务器发送心
跳包的逻辑。你显然不能靠AP来做心跳计时。

**Android提供的Alarm Manager就是来解决心跳等周期性问题的。**Alarm应该是BP计时（或其它某个带石英钟的芯片，不太确定，但绝对不是AP），
触发时唤醒AP执行程序代码。那么Wake Lock API有啥用呢？**比如心跳包从请求到应答，
比如断线重连重新登陆这些关键逻辑的执行过程，就需要Wake Lock来保护。**而一旦一个
关键逻辑执行成功，就应该立即释放掉Wake Lock了。两次心跳请求间隔5到10分钟，基本不会
怎么耗电。除非网络不稳定，频繁断线重连，那种情况办法不多。

# 关于 PowerManager 

## 基本使用过程

1. PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);通过 Context.getSystemService().方法获取PowerManager实例。
2. 然后通过PowerManager的newWakeLock((int flags, String tag)来生成WakeLock实例。int Flags指示要获取哪种WakeLock，不同的Lock对cpu 、屏幕、键盘灯有不同影响。
3. 获取WakeLock实例后通过acquire()获取相应的锁，然后进行其他业务逻辑的操作，最后使用release()释放（释放是必须的）。

## 常用FLAG

1. PARTIAL_WAKE_LOCK:保持CPU 运转，屏幕和键盘灯有可能是关闭的。
2. SCREEN_DIM_WAKE_LOCK：保持CPU 运转，允许保持屏幕显示但有可能是灰的，允许关闭键盘灯
3. SCREEN_BRIGHT_WAKE_LOCK：保持CPU 运转，允许保持屏幕高亮显示，允许关闭键盘灯
4. FULL_WAKE_LOCK：保持CPU 运转，保持屏幕高亮显示，键盘灯也保持亮度

## 权限

```

<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.DEVICE_POWER" />

```
另外WakeLock的设置是 Activiy 级别的，不是针对整个Application应用的。
可以在activity的onResume方法里面操作WakeLock,  在onPause方法里面释放。


# android 休眠的坑

## 向服务器轮询的代码不执行

利用Timer和TimerTask，来设置对服务器进行定时的轮询，但是发现机器在某段时间后，
轮询就不再进行了。查了很久才发 现是休眠造成的。后来解决的办法是，
利用系统的AlarmService来执行轮询。因为虽然系统让机器休眠，节省电量，但并不是完全
的关机，系统有一部 分优先级很高的程序还是在执行的，比如闹钟，利用AlarmService可
以定时启动自己的程序，让cpu启动，执行完毕再休眠。

## 后台长连接断开

最近遇到的问题。利用Socket长连接实现QQ类似的聊天功能，发现在屏幕熄灭一段时间后，
Socket就被断开。屏幕开启的时候需进行重连，但 每次看Log的时候又发现网络是链接的
，后来才发现是cpu休眠导致链接被断开，当你插上数据线看log的时候，网络cpu恢复，一
看网络确实是链接的， 坑。最后使用了PARTIAL_WAKE_LOCK，保持CPU不休眠。

## 调试时是不会休眠的

在调试的时候，就发现，有时Socket会断开，有时不会断开，后来才搞明白，因为我有时是
插着数据线进行调试，有时拔掉数据线，这 时Android的休眠状态是不一样的。而且不
同的机器也有不同的表现，比如有的机器，插着数据线就会充电，有的不会，有的机器的
设置的充电时屏幕不变暗 等等，把自己都搞晕了。其实搞明白这个休眠机制，一切都好说了。


# 关于AlarmManager

AlarmManager是Android提供给用户，用于在休眠模式下也可以执行某些动作的机制，
比如说心跳包，断线重连等。值得注意的是，AlarmManager的生命周期，
在android 3.1 之前，AlarmManager中注册的时钟会一直有效，除非被手动的cancel，
但是在android3.1之后，或者某些定制版本，AlarmManager中注册的时钟会因为App
被**FORCE STOP而被cancel**。

# 参考

* [android中强行停止（forceStopPackage）对alarmManager、Receiver的影响](http://blog.csdn.net/yuanyl/article/details/44201695)


