---
title: Android Weekly Notes Issue 250
date: 2017-03-31 16:50:01
tags: [Android, Android Weekly, Android O, JUnit 5, SpringAnimation, Dagger, Injection, BroadcastReceiver, Kotlin, RxJava]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #250
March 26th, 2017
[Android Weekly Issue #250](http://androidweekly.net/issues/issue-250).
本期内容: 好几篇关于Android O预览版的文章; JUnit 5的动态测试; 作为团队里唯一的Android开发如何学习和工作; Support库新推出的基于物理的动画API: SpringAnimation; Uber Rider项目重构中关于依赖注入的scope层级的改动; Kotlin和RxJava的简洁性.

<!-- more -->

# ARTICLES & TUTORIALS
## [O-h yeah! What we look forward to in Android O](https://www.novoda.com/blog/o-h-yeah-what-we-look-forward-to-in-android-o/)
Google宣布了最新Android O的预览程序. Novoda team查看了最新文档来看看什么新特性最让大家欣喜.

- 更宽广的色域和多种颜色空间支持: 
我们不再被限制在sRGB的颜色空间里, 文档见[ColorSpace](https://developer.android.com/reference/android/graphics/ColorSpace.html).
- 字体支持.
- Adaptive icons: Android N中提供了圆形的启动图标; 从Android O开始, 手机开发商和launcher开发者们可以指定一个mask到应用提供的背景图上.
- ACCESSIBILITY按钮: Accessibility services(比如TalkBack)可以在有软导航键的设备上添加一个按钮.
- 指纹手势.
- 自动大小的TextView.
- Autofill APIs. 自动填表.


## [JUnit 5: Dynamic Tests](https://blog.stylingandroid.com/junit-5-dynamic-tests/)
本篇讲如何简化一个测试cases都很类似的test suite.

首先, 找出不同的部分, 抽取一个方法, 把不相同的部分作为参数传进去.

然后用JUnit 5的动态测试(Dynamic Tests)特性.
两个关键组件: `TextFactory`和`DynamicTest`.
文中代码详细说明了它们的用法.

## [Flying Solo with Android Development](https://hackernoon.com/flying-solo-with-android-development-c52d911b62bf?gi=c13f45395440#.bdab0pxgy)
作者几经周转, 从4人Android团队到2人团队, 现在又到了一个新团队, 作为团队里唯一Android开发. 在这篇文章中, 她分享了一些觉得不错的学习资源和她平时的工作习惯以及建议.


## [Introduction to SpringAnimation with examples](https://www.thedroidsonroids.com/blog/android/springanimation-examples/)
本文讲弹簧效果动画的实现.

[Dynamic-animation](https://developer.android.com/reference/android/support/animation/package-summary.html)是Android Support Library 25.3.0最新引进的, 用于实现基于物理的动画.

作者这篇文章介绍了[SpringAnimation](https://developer.android.com/reference/android/support/animation/SpringAnimation.html)和[SpringForce](https://developer.android.com/reference/android/support/animation/SpringForce.html)的用法, 提供了几个例子, 动态改变View的位置, 旋转和大小属性: [android-springanimation-examples
](https://github.com/AlexKrupa/android-springanimation-examples).


## [Rewriting Uber Engineering’s Rider App with Deep Scope Hierarchies](https://eng.uber.com/deep-scope-hierarchies/)
Android Uber rider app的重构.
主要讨论了由于存在很多共用组件, 所以依赖注入的设计需要改进.

首先介绍了旧的设计: 两级Scope层次.
后来他们的新设计采用了深层次的scope层级, 减少了耦合.

最后又介绍了几种他们曾经考虑过的架构模式.


## [It’s time to kiss goodbye to your implicit BroadcastReceivers](https://medium.com/@iiro.krankka/its-time-to-kiss-goodbye-to-your-implicit-broadcastreceivers-eefafd9f4f8a#.67j4153n6)
Android O的preview已经出来了: [Android O Preview](https://developer.android.com/preview/index.html). 这是它列出来的[Behavior changes](https://developer.android.com/preview/behavior-changes.html).

如果你想要把app target到Android O, 而且你的manifest中注册了一些隐式的BroadcastReceiver. 那么这篇文章就是为你准备的.

Android做这一切的出发点都是为了节约电量.

Android 7.0的时候就[移除了三种隐式广播的支持](https://developer.android.com/about/versions/nougat/android-7.0-changes.html#bg-opt). 它们是`CONNECTIVITY_ACTION`,  `ACTION_NEW_PICTURE`和`ACTION_NEW_VIDEO`.

现在Android O中, 除了[background-broadcasts](https://developer.android.com/preview/features/background-broadcasts.html)中提到的, 其他所有在manifest中注册的隐式广播都不再工作了. (注意这里的关键字: manifest中注册, 隐式广播.)

那么你的manifest中如果有receiver, 现在应该怎么办呢?

首先看看你的广播是否是隐式的. 根据[文档](https://developer.android.com/preview/features/background.html#broadcasts), 所有跟你的应用没有直接关系的广播都是隐式的; 而直接相关的都是显式的.

然后检查你的应用是否真的受到了影响, 因为有一些隐式的广播是例外情况.

真的受到影响了怎么办呢? 使用`JobScheduler`来拯救. 但是它只在API 21以上有, 如果你的最低API小于21, 可以用官方推荐的[firebase-jobdispatcher-android](https://github.com/firebase/firebase-jobdispatcher-android). 作者他们团队用的是Evernote的[android-job](https://github.com/evernote/android-job).


如果上面的库仍然不能帮到你, 你可以考虑把广播换成动态注册的.
不管你的广播是隐式的还是显式的, `Context.registerReceiver()`是永远有效的. 但缺陷就是注销以后就不能再收到事件了.


## [Random Musings on the O Developer Preview 1](https://commonsware.com/blog/2017/03/22/random-musings-o-developer-preview-1.html)
作者对Android O预览的一些看法.

### 比较令人担心的几点
- 后台工作的处理: 许多隐式的广播可能不再起作用, 可能会改变某些应用的行为.
- 多显示支持. 允许用户把一个activity投射到外部显示器上. 这个行为还需要进一步测试, 如果我们投射到一个不可触摸的显示器上会怎样?
- 关于磁盘空间, 缓存目录.
- Support Libraries支持的最小API为14.

### 有启发性的几点
- 可以给Notification设置timeout.
- Picture-in-Picture (PIP)模式. 一种特殊的多窗口模式, 多数被用来播放视频, TV已经有了.
- 新字体.
- Storage Access Framework (SAF) -> Seekable streams.
- WebView将支持allow-cleartext设置.
- 应用安装其他应用需要用户授权.
- Content provider分页查询.
- `FragmentLifecycleCallbacks`.
- `SmsManager`可以创建tokens.
- `SharedPreferences`提供了接口, 可以更换底层存储实现.
- `findViewById()`不再需要强转.

### 其他你可能感兴趣的
- `View.setTooltipText()`.
- `TextView.setJustify()`.
- 提供了padding和margin的Vertical和Horizontal属性, 这样一下就可以设置同一方向上的两个值.
- `ProgressBar.setMin()`.
- `ANDROID_ID`现在是对每个应用来说, 而不是用户或设备. 见[Privacy](https://developer.android.com/preview/behavior-changes.html#privacy-all).


## [Writing Concise Code with Kotlin and RxJava](https://pspdfkit.com/blog/2017/writing-concise-code-with-kotlin-and-rxjava/)
用Github API举例子, 用RxJava和Kotlin实现一个功能, 说明了它们的简洁性.

# LIBRARIES & CODE
## [Fakeit](https://github.com/moove-it/fakeit)
Kotlin版的假数据生成器.

## [Cicerone](https://github.com/terrakok/Cicerone)
一个轻量级的Android导航库.

## [data-binding-validator](https://github.com/Ilhasoft/data-binding-validator)
表单数据验证器, 使用data binding framework实现.

## [LabCoat](https://github.com/Commit451/LabCoat)
GitLab client for Android.


# NEWS
## [O-MG, the Developer Preview of Android O is here!](https://android-developers.googleblog.com/2017/03/first-preview-of-android-o.html)
Google发布了下一个系统版本Android O的开发者预览版.


# TOOLS
## [Android Studio meets Slack](https://instapk.com/)
一个小工具, 可以直接把Android Studio打的包发到Slack去.
