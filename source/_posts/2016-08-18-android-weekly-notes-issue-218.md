---
title: Android Weekly Notes Issue #218
date: 2016-08-18 13:20:56
tags: [Android, Android Weekly]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #218
August 14th, 2016
http://androidweekly.net/issues/issue-218

<!-- more -->

## ARTICLES & TUTORIALS

### PathMorphing with AnimatedVectorDrawables
[PathMorphing with AnimatedVectorDrawables](https://lewismcgeary.github.io/posts/animated-vector-drawable-pathMorphing/)
Android 5.0 推出了[VectorDrawable](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable.html), 矢量图为处理多种屏幕尺寸的带来了很多好处. 这篇文章先介绍了VectorDrawable的使用, 然后主要讲如何实时操纵图像的改变, 用[AnimatedVectorDrawable](https://developer.android.com/reference/android/graphics/drawable/AnimatedVectorDrawable.html)实现一个图像变形的效果.
文中的例子是Android和Apple的log在互相变化.
[source code available](https://github.com/lewismcgeary/AndroidtoAppleVectorLogo)

### Android UI Instrumentation test with Espresso
[Android UI Instrumentation test with Espresso](http://mayojava.github.io/android/android-ui-instrumentation-test-with-espresso/)
用Espresso写UI功能测试, 通常是: 定位UI元素, 然后与其交互, 检查UI元素的状态.
三种主要的组件是: ViewMatchers, ViewActions 和ViewAssertions.
一个简短的例子:
```java
onView(withId(R.id.my_view))            // withId(R.id.my_view) - ViewMatcher
    .perform(click())                  // click() - ViewAction
    .check(matches(isDisplayed()));   //matches(isDisplayed()) - ViewAssertion
```

为了测试不受animation的影响, 有时候可能需要把设备上的Developer Options里的下面几个animation全关掉:
Window animation scale
Transition animation scale
Animator duration scale
然后这个文章里有具体的例子介绍如何写并且运行测试, 还附有相关源码.

### How to Build an Android App for Fire TV - Part 1
[How to Build an Android App for Fire TV - Part 1](https://medium.com/amazon-appstore/developing-for-the-living-room-how-to-build-an-android-app-for-fire-tv-part-1-6ae108106fd2#.n39tl15pa)
创建在亚马逊的Fire TV上跑的Android应用.
文后可以点进part 2.

### Android Wear Development for beginners
[Android Wear Development for beginners](https://medium.com/android-news/android-wear-development-for-beginners-82c2b06ff13a#.15v0ar2g3)
Complication是指手表上显示的除了小时和分钟之外的东西, 比如, 一个电池指示标志.
使用了Complication API之后, 用户就可以自己选一个地方, 然后从应用的列表中选一个东东来显示.
Wear应用是嵌入到一个主应用里面的, 当google play上主应用的apk被安装到手机上的时候, Wearable应用会自动安装在配对的设备上.
这篇文章详细介绍了如何创建一个Wear应用, 代码在[github](https://github.com/moyheen/radar-watch-face)

官方文档: [Watch Face Complications](https://developer.android.com/wear/preview/features/complications.html)
官方sample: [android-WatchFace](https://github.com/googlesamples/android-WatchFace)

### Router — Everything in its Right Place
[Router — Everything in its Right Place](https://medium.com/stories-from-eyeem/router-everything-in-its-right-place-4ca437871052#.cvou4493z)
之前有一篇文章介绍了用装饰者模式来构建高度模块化的Android应用: [Creating Highly Modular Android Apps](https://medium.com/stories-from-eyeem/creating-highly-modular-android-apps-933271fbdb7d#.4gtrccg9n)
那篇文章里也有一个例子[Decorator](https://github.com/eyeem/decorator).
这篇文章讲同样采用装饰者思想的一个库: [Router](https://github.com/eyeem/router).

Router首先基于一个将URL映射到程序界面的库: [routable-android](https://github.com/clayallsopp/routable-android)
Router在此基础上做出了一些扩展和改进, 可以用一个map文件(YAML/JSON/XML)来定义基本的构架.
输入是URL(可以带参数), 根据map进行解析, 然后传到plugins, 然后每个plugin创造一部分的输出, 当左右plugins的工作结束后, 输出就可用了(输出是由多个plugins组装而成的).
文末附有[sample](https://github.com/eyeem/router/tree/master/app).

### Boosting app performance with reflectionless (de)serialization
[Boosting app performance with reflectionless (de)serialization](http://makingvimeo.com/post/148808044404/boosting-app-performance-with-reflectionless)
这篇文章研究了在解析JSON响应的时候如何提高效率.
作者他们的应用Vimeo Android用了Retrofit来做网络请求, 用Gson来反序列化, 不好的一点就是有点慢, 因为Gson用反射来解析JSON. 为了改进,他们想要去除反射.
他们创建了自己的Gson TypeAdapters, 并且利用程序中各个不同大小的model来测量对比了了反序列化的时间.
他们的实验测试了不同的机器对于不同大小model的处理, 在多数情况下, 不用反射会提高性能, 但是也有例外, 在解析很大的model时, 在高性能的机器上, 反而是使用反射的情况比较快.
他们的库: [stag-java](https://github.com/vimeo/stag-java)
STAG: Speedy Type Adapter Generation.

### Introduction to Automated Android Testing - Part 4
[Introduction to Automated Android Testing - Part 4](https://riggaroo.co.za/introduction-android-testing-part-4/)
讲如何写测试的系列文章, 有一个案例sample: [GithubUsersSearchApp](https://github.com/riggaroo/GithubUsersSearchApp).
举例了一个MVP的真实例子, 然后给P写单元测试.
Presenter里有一个CompositeSubscription, 用来管理RxJava的subscriptions, detach的时候会注销所有的订阅, 防止了内存泄露和可能存在危险的view操作.
还创建了一个Contract接口, 把View和Presenter的接口定义写在里面.
这里面还有很机智的一点是把RxJava要用到的Scheduler也从presenter的构造函数传入, 这样在测试的时候就可以使用`Schedulers.immediate()`, 而在View里面我们就按实际情况使用其他.

### Introduction to Android Testing - Part 3
[Introduction to Android Testing - Part 3](https://riggaroo.co.za/introduction-android-testing-part3/)
这应该是跟上面那条一个系列文章的第三篇.
介绍了如何用Retrofit和RxJava请求Github API然后解析到models.
后面是写单元测试, 步骤很清楚, given, when, then. 
可以从中学习一下怎么给这种Retrofit + RxJava的程序写单元测试.

### Git as a secure private Maven repository
[Git as a secure private Maven repository](http://jeroenmols.com/blog/2016/02/05/wagongit/)
[Bitbucket](https://bitbucket.org/) is a web-based hosting service for projects using Git.
讲了如何使用BitBucket或者Github作为一个private的Maven repository.
例子: [WagonGitExample](https://github.com/JeroenMols/WagonGitExample)
Gradle 脚本: [GitAsMaven](https://github.com/JeroenMols/GitAsMaven)

### Crash reporting in Firebase
[Crash reporting in Firebase](http://segunfamisa.com/posts/firebase-crash-reporting)
比较简单的一个文章, 如何set up Firebase的crash reporting.
其实Firebase Crash Reporting一旦构建好之后, 不需要加java代码, 所有uncaught的异常都是自动报告的.
[Firebase Report Crashes](https://firebase.google.com/docs/crash/android).

### Isometric AnimatedVectorDrawable – Part 1
[Isometric AnimatedVectorDrawable – Part 1](https://blog.stylingandroid.com/isometric-animatedvectordrawable-part-1/)
神奇的[AnimatedVectorDrawable](https://developer.android.com/reference/android/graphics/drawable/AnimatedVectorDrawable.html), 这篇文章讲了用它来实现栅格地形图, 游戏里可以升降的草地的类似的东东. (看文章里的图吧, 我也形容不好.)
遗憾的是pathData animation当前在VectorDrawableCompat library暂不支持, 所以文中所讲的技术只能在api 21及其之上使用.
文中的例子有9个方块, 4个三角形, 把SVG作为矢量图VectorDrawable导入Android Studio, 然后给每一个path起名字. 
本文只是part 1, 预告了下一篇文章将讲如何动画.
Source code available here: [IsometricAnimatedVector](https://github.com/StylingAndroid/IsometricAnimatedVector/tree/Part1)

## Design
### Don't just port an iOS navigation model to Android
[Don't just port an iOS navigation model to Android](http://www.androidpolice.com/2016/08/13/opinion-bottom-navigation-material-guidelines-platform-awareness/)
关于bottom nav bars的讨论.
Hamburger menu有时候感觉不是很理想, 是因为抽屉里的items总是隐藏状态, 用户不易发现和使用.
Bottom bar则把每一个item都时刻展现给用户, 在iOS上使用得很成功.
Google在2016年3月提供了Guides: [Bottom navigation](https://material.google.com/components/bottom-navigation.html).
- 什么时候该用bottom navigation呢?
应用有3到5个top级别的目的地, 且它们需要直接被访问, 从各个目的地之间转换, 并且它们应该是具有同等重要性的.
- 什么时候bottom navigation不适用呢?
不能因为怕用户看不见一个东东, 就把它放在bottom nav bar, 作为推广手段.
bottom nav bar也不是用来放menu的, 比如放不下了加个more tab, 展开以后是更多的二级页面入口; 也不要用来放一些弹出窗口, 它们同样也不是top level的目的地.
bottom nav bar不能放低级别的目的地.
最后文章强调了每个平台应该遵从自己的特性, 对Android来说, 如果完全拷贝iOS的设计可能不是一个好主意.

## LIBRARIES & CODE
### Stag-java
[stag-java](https://github.com/vimeo/stag-java)
Stag通过减少反射改善了Gson的性能, 为你的model对象自动生成TypeAdapters.

### Router
[Router](https://github.com/eyeem/router)