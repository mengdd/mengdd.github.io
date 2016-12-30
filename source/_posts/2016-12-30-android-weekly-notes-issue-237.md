---
title: Android Weekly Notes Issue 237
date: 2016-12-30 15:30:25
tags: [Android, Android Weekly, ConstraintLayout, Android Things, ExoPlayer, Java, Kotlin, Firebase, ContentProvider, Architecture, Concurrency]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #237
December 25th, 2016
[Android Weekly Issue #237](http://androidweekly.net/issues/issue-237)
这是本年的最后一篇issue, 感谢大家.
本期内容包括: ConstraintLayout的使用; Android Things的应用; 如何利用第三方库使得Java具有Kotlin的一些新特性; Firebase是如何利用`ContentProvider`进行初始化的; Kotlin上的并发处理; 其他还有一些关于程序架构, 代码优化相关的讨论.

<!-- more -->

# ARTICLES & TUTORIALS
## [Building interfaces with ConstraintLayout](https://medium.com/google-developers/building-interfaces-with-constraintlayout-3958fa38a9f7#.al6p1anu7)
本文介绍`ConstraintLayout`的chains和ratios. 另外还提到很多使用`ConstraintLayout`的实现细节.

所谓chains就是几个View之间建立的双向约束.

ratios是帮助你设置View的宽高比, 它所做的事情和[PercentFrameLayout](https://developer.android.com/reference/android/support/percent/PercentFrameLayout.html)差不多, 但是不用添加额外的ViewGroup.

## [Electronic Candle using Android Things](https://plus.google.com/+DaveSmithDev/posts/4JN7ZaSKxaM)
用ObjectAnimator和Android Things搭建的一个电子蜡烛.

## [ExoPlayer 2.1 - What’s new](https://medium.com/google-exoplayer/exoplayer-2-1-whats-new-2832c09fedab#.po64o4uha)
ExoPlayer 2.1有什么新功能.
这是他们的[release notes](https://github.com/google/ExoPlayer/blob/release-v2/RELEASENOTES.md).

## [Living (Android) without Kotlin](https://hackernoon.com/living-android-without-kotlin-db7391a2b170#.7fm956ryk)
如果你因为种种原因不能在项目中使用kotlin, 这篇文章告诉你如何借助于一些工具和库用Java实现Kotlin的一些features.

## [Christmas Voice – Part 1](https://blog.stylingandroid.com/christmas-voice-part-1/)
作者发布了一个改变声音的应用, 并且将其开源了: [ChristmasVoice](https://github.com/StylingAndroid/ChristmasVoice).

## [How does Firebase initialize on Android?](https://firebase.googleblog.com/2016/12/how-does-firebase-initialize-on-android.html)
Firebase在Android上是如何初始化的?

很多SDK在初始化的时候会要求应用传入`Context`. Firebase简化了这一步骤. 解决方案就是用了`ContentProvider`, 既解决了时间问题, 也得到了sdk需要的`Context`. 并且不需要应用的开发者添加任何额外的初始化代码.

选择`ContentProvider`主要有两点原因:
- `ContentProvider`初始化早.
当一个Android进程启动的时候, 首先会初始化每一个ContentProvider, 然后是Application, 最后是被Intent启动的组件. 

在ContentProvider初始化的时候, 就可以拿到Context了.

- `ContentProvider`可以merge到最终的manifest里.
[Manifest merge](https://developer.android.com/studio/build/manifest-merge.html)是在build的时候来定义你的应用最终的manifest. 最终的manifest会包含所有依赖的库的manifest中声明的组件.

如果你也想选择用`ContentProvider`来做应用或库的初始化, 请注意authority的唯一性问题和`ContentProvider`只在主进程运行的问题.

## [Seductive Code](https://publicobject.com/2016/12/19/seductive-code/)
当我们在改善代码可读性的时候, 很有可能会影响到性能和可维护性. 

作者举例说明了他在实际编程中遇到的几个问题.

## [Testing Android Things – Unit & Vendor tests](http://blog.blundellapps.co.uk/testing-android-things-iot-meets-java/)
如何开发Android Things应用, 才能让测试更加容易. 本文以一个很小的LED灯闪烁程序为例.

## [Engineering the Architecture Behind Uber’s New Rider App](https://eng.uber.com/new-rider-app/)
Uber团队重新打造了他们的ride app, 提出了一个新的构架模式: Riblets.

关于架构的选型, 已有的类型可以查看这个[iOS Architecture Patterns](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52#.tmcojtwgg).

## [Rebuilding the Buffer Android Composer](https://overflow.buffer.com/2016/12/22/rebuild-android-composer/)
作者重构了自己应用的代码, 应用了clean architecture, 本文讲述了其过程.

## [Papercut](http://stu.ie/?page_id=3133)
[Papercut](https://github.com/Stuie/papercut)是一个库, 用来标记那些我们觉得需要删除或者需要重构的代码.

## [Concurrency Primitives in Kotlin](https://blog.egorand.me/concurrency-primitives-in-kotlin/)
作者最近看了一本书, 讲Android的并发, 觉得很好, 想要用Kotlin来重写书中的例子, 结果发现:
- Kotlin中没有`synchronized`关键字.
- Kotlin中没有`volatile`关键字.
- Kotlin中的`Any`, 类比于Java中的`Object`, 但是却没有`wait()`, `notify()`和`notifyAll()`方法.

所以Kotlin中的并发是怎么处理呢? 这里有个问题: [Kotlin forum](https://discuss.kotlinlang.org/t/concurrency-in-kotlin/858), Kotlin语言的开发人员表示这些应该由库来处理, 而不是语言本身.

尽管Kotlin不支持, 但是它还是提供了一些底层的并发工具.
- 创建线程. 因为Kotlin可以调用Java代码, 所以仍然可以通过两种方法来创建线程.
- `@Synchronized`注解和`synchronized()`方法.
- `@Volatile`注解.
- 没有`wait()`, `notify()`和`notifyAll()`方法, 但是可以把`Object`对象作为锁, 然后调用锁的这些方法.

[stackoverflow](http://stackoverflow.com/questions/35520583/why-there-are-no-concurrency-keywords-in-kotlin)上有一个相关问题, 答案很不错, 列了处理并发的一些有用的库.

# LIBRARIES & CODE
## [KataScreenshotAndroid](https://github.com/Karumi/KataScreenshotAndroid)
一个Screen Kata应用, 用来练习做UI测试的.

## [Papercut](https://github.com/Stuie/papercut)
一个用来标记需要重构或者移除代码的工具库.

## [Squint](https://github.com/IntruderShanky/Squint)
一个可以自定义的对角线切割View.

## [Colorful](https://github.com/garretyoder/Colorful)
一个动态的主题库, 让你可以方便地修改应用的颜色.

## [scytale](https://github.com/yakivmospan/scytale)
包装了JCA API和AndroidKeyStore API, 让创建, 加密和管理任何Android API的keys变得更容易.