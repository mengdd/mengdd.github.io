---
title: Android Weekly Notes Issue 230
date: 2016-11-11 12:57:21
tags: [Android, Android Weekly, Mockito, ConstraintLayout, Kotlin, Async-Await, RxJava, RxJava2.0, Activity, Rotation, Retrofit, JDK, Espresso, Authentication, KeyStore, proguard, MVP, WifiManager, ConfigurationManager, Material Design]
categories: [Android, Android Weekly]
---
# Android Weekly Issue #230
November 6th, 2016
[Android Weekly Issue #230](http://androidweekly.net/issues/issue-230).

Android Weekly笔记, 本期内容包括: Mockito的扩展; ConstraintLayout的链式约束; Kotlin的Async-Await; RxJava2.0; 屏幕旋转导致的Activity重建; Throwable类的设计问题; Espresso测试中如何等待异步请求返回; Kotlin的扩展和运算符重载; Android KeyStore实现用户验证.

代码部分有proguard的库, mvp的库和WifiManager, ConfigurationManager的包装库.

<!-- more -->

# ARTICLES & TUTORIALS
## [Extending Mockito](http://jeroenmols.com/blog/2016/10/31/mockitomatchers/)
这篇文章讲了如何扩展Mockito, 简化对参数的验证.

首先作者举了之前验证参数的例子, 用的是ArgumentCaptor, 写起来很麻烦, 用了自定义的matcher之后简化了很多.

## [ConstraintLayout Chains – Part 2](https://blog.stylingandroid.com/constraintlayout-chains-part-2/)
上一篇文章讲过在ConstraintLayout中如何创建对称的链式约束, 本篇文章介绍chainStyle的不同设置和比较.

默认的spread chain: 均匀分布;
inside spread chain: 边缘元素顶边, 中间均匀分布.

如果指定了权重, 将会按照权重布局. 那么spread chain和inside spread chain就没有区别了.

packed chain: 默认会把所有元素都放在一起放在中间, 可以指定bias来定义偏移基准, 默认bias是0.5, bias设置为0.25的意思就是往左偏. 

## [A glimpse of Async-Await on Android](https://medium.com/@haarman.niek/async-await-in-android-f0202cf31088#.bdf3jarxd)
Kotlin 1.1推出了[coroutines](https://github.com/Kotlin/kotlin-coroutines), 这是一个让计算可以在某个点暂停然后之后又恢复的功能, 例子是几年前C#的[Async-Await](http://blog.stephencleary.com/2012/02/async-and-await.html).

作者先举例说明了异步操作的几种常见实现, 最后结合自己的库用Async-Await做了一个例子.

## [What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)
RxJava2.0.0已经发布了. 这是它的wiki page来介绍2.0有什么不同.

## [Activity Revival and the case of the Rotating Device](https://medium.com/google-developers/activity-revival-and-the-case-of-the-rotating-device-167e34f9a30d#.fwrqz8nit)
本篇文章讲configuration变化(比如屏幕旋转)导致的Activity重建.

为什么configuration变化的时候要重建Activity呢? 因为系统想要尽力地做一些helpful的事情, 希望在这种时候能重新加载正确的资源.

怎么处理呢?

方法一: 让系统自动处理. 在屏幕旋转时, `onSaveInstanceState()`会在Activity销毁前调用, 可以存储一些状态, 之后重建的时候从bundle中拿出来恢复.

方法二: 自己处理. 如果你想要获取更多控制, 那么你可以在manifest中声明`configChanges`类型, 然后在Activity中覆写`onConfigurationChanged()`方法, 来自己做处理.

另外文章中还讨论了网络请求, 屏幕方向设置, retained fragment的使用等.

## [RxJava and Retrofit Throwing a Tantrum](https://medium.com/square-corner-blog/no-cause-for-concern-rxjava-and-retrofit-throwing-a-tantrum-96c9e4ba8a6c#.p1ck4zijo)
作者讨论了他们在项目中遇到的一个问题.
他们用`RxJavaHooks.enableAssemblyTracking();`来收集RxJava崩溃栈信息, 可以显示出到底是哪一个Observable崩了. 
使用这个工具以后发现了一个问题, 进而研究了JDK的`Throwable`类.

原来cause不存在(this)和cause未知(null)是两种不同的情况, 但是`Throwable`的`getCause()`方法都会返回null.

## [Retrofitting Espresso](http://collectiveidea.com/blog/archives/2016/10/13/retrofitting-espresso/)
用Espresso做测试, 如何等待网络请求结束再验证UI.

## [Composing functions in Kotlin with extensions and operators](https://www.novoda.com/blog/composing-functions-in-kotlin-with-extensions-and-operators/)
结合Kotlin的extensions和operator overloading功能, 改善function, 让代码变得更优雅.

## [Authentication sucks. Bad security too](https://medium.com/@flschweiger/authentication-sucks-bad-security-too-345ed20463d4#.yl40vbtgd)
一个例子, 说明为什么老的验证方法用户体验不好, 我们如何在仍然考虑用户安全的情况下进行改善.

解决方案是用Android 6.0推出的Android Keystore.
作者展示了如何实现并提供了[Demo](https://github.com/flschweiger/SafeApp).

# Design
## [Eight don’ts for your Material Design app](https://blog.prototypr.io/common-material-design-bad-practices-to-avoid-b7995f251329#.ij9u38lu7)
Material Design app需要避免的8个点.

# LIBRARIES & CODE
## [Android-proguards](https://github.com/yongjhih/android-proguards)
使用一行就可以加上所有流行库的proguard.

## [Moxy](https://github.com/Arello-Mobile/Moxy)
一个MVP的库.

## [WiseFy](https://github.com/isuPatches/WiseFy)
包装了Android的WifiManager和ConnectivityManager的一个库.

# VIDEOS & PODCASTS
[Droidcon NYC 2016](https://www.youtube.com/playlist?list=PLnVy79PaFHMXJha06t6pWfkYcATV4oPvC)
所有Droidcon NYC 2016的视频资源.