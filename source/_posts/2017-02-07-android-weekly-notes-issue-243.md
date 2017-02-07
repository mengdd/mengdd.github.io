---
title: Android Weekly Notes Issue 243
date: 2017-02-07 12:52:37
tags: [Android, Android Weekly, ConstraintLayout, Kotlin, RxJava, Testing, Service, Lottie, Animation, Mutability, Android 7, Nougat, Locales, StrictMode, RecyclerView]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #243
February 5th, 2017
[Android Weekly Issue #243](http://androidweekly.net/issues/issue-243)
本期内容包括: ConstraintLayout的动画; 用Kotlin写测试; RxJava的练习项目; 一个库: Coordinators的介绍; 一个自动报告Google Play反馈的工具; Service的测试; 动画工具Lottie的介绍; Mutability的讨论;
Nougat的多语言支持和相关的一个有趣的case; 使用StrictMode来发现问题.

<!-- more -->

# ARTICLES & TUTORIALS
## [Constraint Layout Animations](http://www.uwanttolearn.com/android/constraint-layout-animations-dynamic-constraints-ui-java-hell/)
作者举例说明了如何在Java代码中动态地改变约束条件, 从而使`ConstraintLayout`中的View动起来.

## [Android Testing with Kotlin](http://fernandocejas.com/2017/02/03/android-testing-with-kotlin/)
如果你想逐渐地迁移代码到Kotlin, 你可以从测试开始, 这样你也不用更改产品环境的代码, 就先熟悉了Kotlin.

本篇文章详细讲了如何setup, 然后写各种测试:

### JUnit测试
需要JUnit, [mockito-kotlin](https://github.com/nhaarman/mockito-kotlin)和[Kluent](https://github.com/MarkusAmshove/Kluent).

对于在`setUp()`方法中初始化的变量, 需要标记为`lateinit`.

### Robolectric测试
作者封装了一个基类, 把所有Mockito相关的东东包装在里面. 这样在Mockito升级的时候不用更改每一个测试文件.

### Espresso测试
同样, 这里作者也创建了几个基类, 将所有Espresso相关的东东包装起来.

## [Practical challenges for RxJava learners](https://medium.com/@sergii/practical-challenges-for-rxjava-learners-1821c454de9#.9icb22hrr)
作者建议通过实践来检验和学习RxJava技能, 之前他用过这个Repo: [Intro-To-RxJava](https://github.com/Froussios/Intro-To-RxJava), 现在他又新推出了这个[Repo](https://github.com/sergiiz/RxBasicsKata), 针对RxJava2的.

## [Coordinators: solving a problem you didn’t even know you had](https://hackernoon.com/coordinators-solving-a-problem-you-didnt-even-know-you-had-e86623f15ebf#.mcx15cssl)
Square发布了一个库叫[coordinators](https://github.com/square/coordinators), 这个库是用来分离View中的一些控制逻辑.

## [Review-Reporter: Part 1 ](https://medium.com/azimolabs/review-reporter-part-1-connecting-to-google-play-8abd37edc49f#.a0s5gx66j)
作者他们做了一个小项目: [Review-Reporter](https://github.com/AzimoLabs/Review-Reporter), 可以自动把Google Play上新的用户回复发到slack, firebase, Jira上. 本篇文章讲了他们是怎么做的.

## [How to test a Service](https://medium.com/@josiassena/android-how-to-unit-test-a-service-67e5340544a5#.qg3751nxg)
Android官方文档介绍了如何测试Service: [Testing your Service](https://developer.android.com/training/testing/integration-testing/service-testing.html). 本文作者介绍他是如何做的.

## [Introducing Lottie](https://medium.com/airbnb-engineering/introducing-lottie-4ff4a0afac0e#.e7wojthmp)
[Lottie](http://airbnb.design/lottie/)是一个iOS, Android和React Native的库, 可以实时渲染After Effects的动画, 让native的应用像使用静态文件一样简单地使用复杂的动画.

## [Learning to use and abuse Mutability](https://medium.com/google-developer-experts/learning-to-use-and-abuse-mutability-b4c71576299#.diungnuw6)
> An immutable class is a class whose state cannot be changed once it has been created.

这篇文章分享了作者关于Java中的mutability & immutability的一些想法.

## [A Curious Case of Multiple Locales](https://blog.egorand.me/a-curious-case-of-multiple-locales/)
Android N的一个新feature就是可以在设置中选择多种语言.

比如一个用户, 她会说意大利语和德语, 她使用的是一个低于Android 7的手机, 她把手机语言设置为意大利语.

有一个app, 支持两种语言, 默认是英语, 然后还支持德语.

但是这个应用在这个用户的手机上打开时, 发现自己并不支持意大利语, 于是会显示英语(默认)而不是德语, 因为应用又不知道这个用户还会德语.

后来用户把手机升级了, 用了Android 7的系统, 她发现可以设置支持多种语言, 于是, 于是她设置了两种语言, 意大利语和德语. 在新手机上装之前那个app的时候发现现在显示的是德语.

因为应用现在知道了用户还会讲德语.

现在, 假设我们需要进行向下兼容以前的旧版本设备, 我们加入了`appcompat-v7`, 用户更新后, 英语又出现了. 

这是因为`appcompat-v7`中含有一些意大利语的资源, 因为所有的资源在build的时候都会merge到一起, 所以现在app也包含了这些资源. 系统认为现在app能够支持用户的第一语言了, 然后就查找对应的资源, 当然没查找到, 于是就使用了默认资源, 也就是英语.

我们有什么办法可以解决这个问题呢? 答案是这样:
```
defaultConfig {  
  ...

  resConfigs "en", "de"
}
```
这样就告诉了Gradle我们只支持这两种语言, 所有其他的资源都不会被打包进来.

验证的方法是使用Android Studio的`Analyze APK`来查看string有多少种configurations.

## [Use StrictMode To Find Things You Did By Accident](https://blog.mindorks.com/use-strictmode-to-find-things-you-did-by-accident-in-android-development-4cf0e7c8d997#.l5tbilx16)
`StrictMode`是一个开发工具, 用于发现一些问题, 好让你来修复它们. 

一个常用的情景是用来捕捉主线程的IO操作, 避免ANR弹框.

如何使用呢? 很简单, 只需要在应用启动时初始化一下, 可以是你的Application, Activity或其他组件的`onCreate()`方法:
```java
public void onCreate () {
    if (DEVELOPER_MODE) {
        StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                .detectDiskReads()
                .detectDiskWrites()
                .detectNetwork()   // or .detectAll() for all detectable problems
                .penaltyLog()
                .build());
        StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                .detectLeakedSqlLiteObjects()
                .detectLeakedClosableObjects()
                .penaltyLog()
                .penaltyDeath()
                .build());
    }
    super.onCreate();
}
```

你可以决定检测到问题时要发生什么, 比如:
- `penaltyDeath()`: 整个进程崩溃.
- `penaltyDialog()`: 显示Dialog.
- `penaltyLog()`: 显示log.

更多的处理见: [StrictMode.ThreadPolicy.Builder](https://developer.android.com/reference/android/os/StrictMode.ThreadPolicy.Builder.html).

StrictMode文档: [StrictMode](https://developer.android.com/reference/android/os/StrictMode.html).

# LIBRARIES & CODE
## [android-material-stepper](https://github.com/stepstone-tech/android-material-stepper)
一个Material steppers的库, 类似于配合ViewPager使用的indicators.

## [AOSP Support Library Contribution Guide](https://android.googlesource.com/platform/frameworks/support/) 
Google开放了对support library的bug修改和文档更新.

## [sqlite-android](https://github.com/requery/sqlite-android)
一个Android的SQLite库, 包含了最新的SQLite版本.

## [Review-Reporter](https://github.com/AzimoLabs/Review-Reporter)
Google Play反馈的自动提示, 支持提示到Slack和Jira.

## [SimpleRecyclerView](https://github.com/jaychang0917/SimpleRecyclerView)
更简单好用的RecyclerView.