# Android data package 目录介绍

在Android中，为APP提供了**私有的目录空间（卸载后，自动清除）**，分别在 

1. /data/data/{package}/

2. /sdcard/android/data/{package}/

一般比较私密的数据放置在/data/data/{package}中，而比较大的数据放置在/sdcard/android/data/{package}中。而卸载后还想保留的数据，比如说照片，是需要手动在/sdcard/创建自己的文件夹。

## /data/data/{package}/

该目录通常存在：

1. file: 存放文件

2. cache:存放缓存，如图片

3. db:数据库

4. lib:存放SO文件

5. shared_prefs:存放SharedPreferences文件目录