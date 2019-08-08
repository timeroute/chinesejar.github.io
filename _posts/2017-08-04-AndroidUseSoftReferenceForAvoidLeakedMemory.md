---
layout: page
title: Android 使用 SoftReference 解决 Activity 存栈的内存泄漏问题
date: 2017-08-04 12:09:32
tags:
- Android
- SoftReference
---

当 Android 想要退出应用时，我们总是希望完全退出。但是 Android 并没有提供一个完全退出 App 的函数。

Google 上搜索了一下，方法有很多，但是基本都是只退出了当前的 Activity, 并没有完全 finish 所有的 Activity。

利用存 Activity 栈的方式来 finish 所有这个方法目前来看是对我直接有效的，但是实际运行过程中， leakcanary 会报内存泄漏的异常，原因是我的 List 会对每一个 Activity 启动时进行添加，而当我 finish 某个 activity 时，List 里对应的 Activity 无法被 GC，这样导致我的内存开销增加了。如果我对每一次 Activity 的 finish 之后再清除对应的 List 里的 Activity， 这样我觉得会很麻烦，一点都不优雅。下面介绍一下 **SoftReference** 对象。

> **SoftReference**，即“软引用”，由垃圾收集器根据内存需求自行清除。假设垃圾收集器在某个时间点确定对象是可以轻松访问的。那时候，它可能会选择原子地清除对该对象的所有软引用，以及对任何其他可轻松访问的对象的所有软引用，通过一个强引用链可以从该对象到达该对象。在同一时间或稍后的时间，它将排入在引用队列中注册的新清除的软引用。在虚拟机抛出OutOfMemoryError之前，所有对软到达对象的软引用都保证被清除。否则，在清除软引用的时间或者对一组对不同对象的引用将被清除的顺序没有约束。但是，鼓励虚拟机实现偏离清除最近创建或最近使用的软引用。

**softreference** 可以在 Activity 完成生命周期并且没有其他被引用的情况下被 GC 释放。所以 List 存 SoftReference 可以解决问题。

### 继承一个 Application 类

```
public class MyApp extends Application {
    private List<SoftReference<Activity>> activityList = new LinkedList<>();
    private static MyApp instance;
    public static Context context;

    @Override
    public void onCreate(){
        super.onCreate();
        context = getApplicationContext();
        LeakCanary.install(this);
    }

    public static MyApp getInstance() {
        if(null == instance) {
            instance = new MyApp();
        }
        return instance;
    }
    //添加 Activity 的软引用到容器中
    public void addActivity(SoftReference<Activity> softReference)  {
        activityList.add(softReference);
    }
    //遍历所有Activity并finish
    public void exit(){
        for(int i=0;i<activityList.size();i++){
            Activity activity = activityList.get(i).get();
            if(activity != null){
                activity.finish();
            }
        }
        System.exit(0);
    }
}
```

### 添加 Activity 的软引用
在 BaseActivity 的 onCreate() 方法对继承的 Activity 添加软引用到 MyApp 的 List 里。
```
MyApp.getInstance().addActivity(new SoftReference<>(this));
```

### 调用 exit() 方法
当需要退出 App 时，只需调用 MyApp 的 exit() 方法即可。
```
MyApp.getInstance().exit();
```
