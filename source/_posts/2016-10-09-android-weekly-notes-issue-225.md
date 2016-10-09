---
title: Android Weekly Notes Issue 225
date: 2016-10-09 10:32:42
tags: [Android, Android Weekly, Android 7.0, Quick Settings, Firebase, Shared-Element Transtion, Wear, Espresso, Animation, ORM, ActiveAndroid, Crash Reporting, Google Cast, ExoPlayer, Packages, Task]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #225
October 2nd, 2016
[Android Weekly Issue #225](http://androidweekly.net/issues/issue-225)

本期内容包括: Android 7.0的Quick Settings; Firebase; 兼容旧版本的shared element transition; Wear; ORM: 用ActiveAndroid做数据库存储; 崩溃报告工具对比; Google Cast API介绍; Google的播放器库ExoPlayer 2.x发布; 项目的包结构整理; Task API的使用等等.

<!-- more -->
# ARTICLES & TUTORIALS
## Android 7.0的快速设置 [Quick Settings Tiles](https://medium.com/google-developers/quick-settings-tiles-e3c22daf93a8#.4q0cxslwd)
从Android 7.0 (API 24)开始, 任何app都可以创建一个quick settings tile, 快速访问关键功能.
它除了是一个展示最新信息的UI, 点击一个片还可以trigger后台任务, 打开dialog或activity.

一个好的quick settings tile:
决定是否要建立这样一个tile时, 主要考虑紧急性和频繁性两个方面.

每一个tile和一个[TileService](https://developer.android.com/reference/android/service/quicksettings/TileService.html)关联. 和其他service一样, 它需要在manifest中注册, 它的label和icon就是显示在quick settings上的文字和图片.

**TileService的生命周期**:
TileService是一个[bound service](https://developer.android.com/guide/components/bound-services.html), 它的生命周期主要由系统控制. 主要有三个阶段: being added, listening, being removed.
- `onTileAdded()`: 当用户添加这个tile到quick settings.
- `onStartListening()`: tile变为可见.
- `onStopListening()`: tile变为不可见.
- `onTileRemoved()`: 用户移除这个tile.

以上这是默认模式, 如果你准确地知道何时更新, 你可以使用[active mode](https://developer.android.com/reference/android/service/quicksettings/TileService.html#META_DATA_ACTIVE_TILE).
此时更新的回调`onStartListening()`是通过静态方法主动触发的.

**更新UI**:
UI是[Tile](https://developer.android.com/reference/android/service/quicksettings/Tile.html), 主要包含icon, label, description和state. 最后必须调用`updateTile()`方法.

**处理点击**:
在`onClick()`回调触发的时候, 我们可以启动一些后台工作, 或者`showDialog()`, 或者`startActivityAndCollapse()`.

对于锁屏的机器有一些限制, 不能打开dialog, 并且activity需要有一个特定的flag, 有一个`unlockAndRun()`方法可以让用户先解锁后做一些工作.

长按tile默认会打开app的app info屏, 当然这个行为也可以override. 只要给你想打开的activity加上`ACTION_QS_TILE_PREFERENCES`.

## Android开发最佳实践 [Android Development Best Practices](https://medium.freecodecamp.com/android-development-best-practices-83c94b027fd3)

关于性能:
[Best Practices for Performance](https://developer.android.com/training/best-performance.html); 
[Performance and Optimization](https://github.com/amitshekhariitbhu/awesome-android-complete-reference#performance-and-optimization)

关于架构:
[android-architecture](https://github.com/googlesamples/android-architecture)

写单元测试和UI测试.

使用Proguard, Stetho.
复用布局, 使用<merge>标签.
[reusing-layouts](https://developer.android.com/training/improving-layouts/reusing-layouts.html).

把launcher icons放在mipmap文件夹下.

多用shape和selector而不是图片.

避免深层次的布局.

向Intent或Bundler传数据时, 使用`Parcelable`而不是`Serializable`. 因为后者使用反射而比较慢.

不要在UI线程进行文件操作.

明白Bitmaps. 因为它们占用很多memory. [Displaying Bitmaps](https://developer.android.com/training/displaying-bitmaps/index.html)

使用style来避免重复的属性设置.

需要时使用Fragment.

明白Activity的生命周期.

使用得到公认的libraries而不是自己的实现.

在各种机器上测试.

## [Recap Of Google Launchpad Build Lagos : All About Firebase](http://chikemgbemena.com/2016/09/27/recap-lagos-launchpad-developers-conference-all-about-firebase/)

作者参加了一个叫Google Launchpad Build的会议, 这篇文章是总结, 全部是关于Firebase的.

## [Android Shared-Element Transitions for all](https://medium.com/@aitorvs/android-shared-element-transitions-for-all-b90e9361507d#.rlu4u7kmy)
在Lollipop+的设备上, shared element的transition动画很好实现, 但是在旧的版本上该怎么办呢? 作者展示了他的方法:
- Activity A捕捉origin view的初始值, 通过Intent把它们传给Activity B;
- Activity B完全透明地启动;
- Activity B读取bundle中的值, 准备场景;
- Acitivty B运行shared element动画.

几个实现细节:

需要知道View在B中的位置, 时机是layout之后, 但是draw之前, 即`onPreDraw()`.
返回时只需要把这个动画反向播放即可.


## [Writing Better Adapters](https://medium.com/@dpreussler/writing-better-adapters-1b09758407d2#.c5av797rd)
(这个上一期刚讲过, 不知道为什么重复了. )

就是关于RecyclerView的Adapter, 作者认为多种View类型时, Adapter中太多的instance of和强制类型转换不是一种好做法, 于是提出了他的做法. 

## [Android Wear: Accessing the Data Layer API](https://medium.com/@manuelvicnt/android-wear-accessing-the-data-layer-api-d64fd55982e3)

Data Layer API是Google Play services的一部分, 用于不同设备(手机和手表)间的数据交换.

作者先提供了代码, 发送和存储数据, 监听数据变化.

问题是, 如果Wear第二次向mobile请求数据, mobile发送了和上一次一样的数据, Wear并不会进入`onDataChanged()`, 因为数据并没有变化.

所以作者想知道如何从Data Layer API来获取数据, 并展示了他的方法在不同情形下的应用.

## [Espresso Tests For TextSwitcher](http://www.ottodroid.net/?p=493)

作者想给TextSwitcher写Espresso测试.

从Android Studio 2.2开始, 你可以录制你的操作, IDE将会自动为你生成Espresso测试代码. 但是作者录了一个有关TextSwitcher的测试之后, 跑失败了.

这是因为`TextSwitcher`继承了`ViewSwitcher`, 其实现其实是把两个TextView加到了布局里.
所以Espresso抛出了`AmbiguousViewMatcherException`.

所以作者根据可见性区分了它俩, 修复了测试.
还可以根据child view的index来区分.

## [Animating Android Activities and Views with Slide Animations](https://kylewbanks.com/blog/left-and-right-slide-animations-on-android-activity-or-view)

作者展示了如何给Activity和View加上左右滑动的动画.

## [Guide to ORM using ActiveAndroid: Part 1](http://www.rscottcarson.com/2016/09/22/the-ultimate-guide-to-orm-in-android-using-activeandroid-part-1/)

这是一个系列教程, 相关的代码在: [ActiveAndroid-Tutorial](https://github.com/rscottcarson/ActiveAndroid-Tutorial)

什么是ORM(Object-Relational Mapping)呢?
a technique to convert between incompatible type-systems in an object-oriented programming language.
在面向对象的语言中, 转换不兼容的类型的技术.

[ActiveAndroid](http://www.activeandroid.com/)是一个ORM(object relational mapper), 让你不用写SQL语句, 就可以读写数据库.

其他类似的工具还有[Realm](https://realm.io/docs/java/latest/)和[OrmLite](http://ormlite.com/).


## [A Comparison of Android Crash Reporting Tools](https://www.captechconsulting.com/blogs/a-comparison-of-android-crash-reporting-tools)

作者对比了几种崩溃报告工具, 并介绍了如何使用.
包括: Firebase, [Crashlytics](https://fabric.io/kits/android/crashlytics/), [Apteligent](https://www.apteligent.com/), [Bugsnag](https://bugsnag.com/).

## [Google Play Services: Google Cast v3 and Media](https://code.tutsplus.com/tutorials/google-play-services-google-cast-v3-and-media--cms-26893)
Google Cast是一个让用户把网上的内容发送到设备上的技术. 通常用来和TV交换内容.

作者详细地介绍了如何使用Google Cast SDK来创建应用.

注: 要建造客户端程序, 首先需要注册: https://cast.google.com/publish/.
这是收费的.

## [ExoPlayer 2.x - It’s here (plus FAQs)!](https://medium.com/google-exoplayer/exoplayer-2-x-its-here-plus-faqs-cce34b0d4c7b#.h6m9czs7y)
Google的库[google/ExoPlayer](https://github.com/google/ExoPlayer)升级到v2.x了. 
(它是一个Media Player, YouTube用的就是它.)
这次是个重大更新, 添加了很多新功能, 推荐大家以后用新版.

## [How We Rethought our Complete Package Structure for Buffer on Android](https://overflow.buffer.com/2016/09/26/android-rethinking-package-structure/)
作者他们重新整理了项目的包结构, 总结了整个过程还有从中学到的东东.

作者他们之前的包结构是按类型的, 有activities, fragments, adapters等包. 因为类名以类型终结, 所以索性就按整个分组.

当app变得越来越大, 这种组织方式发现就不太好, 感觉很难找东西, 并且感觉没什么结构.

经过改变之后, 作者他们采用了一种更加整洁并且易于导航的结构.

新结构中, 当添加一个新的feature, 就保持在同一个目录中, 这样就不用来回切换目录.

作者他们的新结构有四个总目录: 
- data
- ui
- injection
- util

**data**中包含网络请求及相关的models, preferences, database, data models, 还有其他和数据直接关联的东西.

其中和不同API关联的models又分别组织在子目录下.

**ui**目录中包含所有和UI相关的组件, 在这个包中按照功能又拆分了子目录. 其中有base包, 用来盛放Fragment, Activity和MVP的基类, 接口等; 还有common包, 用来盛放公共控件.

**injection**中包含所有依赖注入的类, 分component, module和scope的子目录.

**util**中含有Helper和Utility类.

## [Become a Firebase Taskmaster! (Part 3)](https://firebase.googleblog.com/2016/09/become-a-firebase-taskmaster-part-3_29.html)
这是系列文章的第三篇, 这个系列是关于Play services的[Task API](https://developers.google.com/android/guides/tasks). 

如果项目里已经依赖了Firebase, 变自动包含了Task API, 如果不想用Firebase, 可以单独添加依赖:
`compile 'com.google.android.gms:play-services-tasks:9.6.1'`

创建新的Task可以用下面这两个方法:
```java
Task<TResult> call(Callable<TResult> callable)
Task<TResult> call(Executor executor, Callable<TResult> callable)
```

第一个`call()`方法在主线程执行任务, 第二个`call()`方法可以把工作提交给一个`Executor`.


[Callable](https://developer.android.com/reference/java/util/concurrent/Callable.html)有点类似于Runnable:
```java
public class CarlyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "Call me maybe";
    }
}
```

参数制定了方法的返回值的类型, 进而也是创建出Task的类型.

```java
Task<String> task = Tasks.call(new CarlyCallable());
```


想要链式执行, 进行后续操作, 可以用[Continuation](https://developers.google.com/android/reference/com/google/android/gms/tasks/Continuation).
```java
public class SeparateWays implements Continuation<String, List<String>> {
    @Override
    public List<String> then(Task<String> task) throws Exception {
        return Arrays.asList(task.getResult().split(" +"));
    }
}
```
它继承接口时指定了输入和输出的类型, 它的输入来自于Task的输出.

可以多写几个Continuation类然后连起来:
```java
Task<String> playlist = Tasks.call(new CarlyCallable())
        .continueWith(new SeparateWays())
        .continueWith(new AllShookUp())
        .continueWith(new ComeTogether());
playlist.addOnSuccessListener(new OnSuccessListener<String>() {
    @Override
    public void onSuccess(String message) {
        // The final String with all the words randomized is here
    }
});

```

# LIBRIARIES & CODE
## [groupie](https://github.com/Genius/groupie)
显示和管理复杂的RecyclerView布局, 把你的items按照逻辑分组管理.

## [android-junit5](https://github.com/aurae/android-junit5)
Gradle插件, 用JUnit5做Android的单元测试.

## [epoxy](https://github.com/airbnb/epoxy)
用来构建复杂的RecyclerView屏.


