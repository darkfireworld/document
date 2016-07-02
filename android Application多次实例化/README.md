# android Application多次实例化

android application 这个对象可能会被多次实例化, 这种情况在使用`android:process元素`的
时候就会出现。

下面给出一段获得当前进程的名称的代码, 通过它, 就可以`判断是否运行在主进程`了：

```

/**
    返回进程的名称, 如果是主进程, 则返回package name
*/
public static String getCurProcessName(Context context) {
    int pid = Process.myPid();
    ActivityManager mActivityManager = (ActivityManager)context.getSystemService("activity");
    Iterator iter = mActivityManager.getRunningAppProcesses().iterator();

    RunningAppProcessInfo appProcess;
    do {
        if(!iter.hasNext()) {
            return null;
        }

        appProcess = (RunningAppProcessInfo)iter.next();
    } while(appProcess.pid != pid);

    return appProcess.processName;
}


```