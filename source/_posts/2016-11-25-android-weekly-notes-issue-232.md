---
title: Android Weekly Notes Issue 232
date: 2016-11-25 16:32:57
tags: [Android, Android Weekly, Kotlin, RxJava, RxJava2, MVVM, Retrofit, Sensor, Testing, Unit Test, Reductor, Redux, Process, State, Anko, VectorDrawable, PNG, Switch]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #232
November 20th, 2016
[Android Weekly Issue #232](http://androidweekly.net/issues/issue-232)
本期内容包括: Kotlin的优势讨论; MVVM模式结合RxJava和Retrofit的应用构架实现; Android中传感器使用; 如何给App写单元测试; Reductor的组合使用; Android应用进程被杀死的状态恢复和问题处理; Kotlin中的Anko; 后台任务处理库"Android Job"; VectorDrawable和PNG的使用问题等.

本期开源库: 给ImageView和RelativeLayout的底部加曲线; 长按弹框; Switch Button控件; 给View加深度/厚度的库.

<!-- more -->

# ARTICLES & TUTORIALS
## [How Kotlin became our primary language for Android](https://medium.com/uptech-team/how-kotlin-became-our-primary-language-for-android-3af7fd6a994c#.a50t4ple8)
作者他们team想要完全用kotlin开发一个应用.
本文是他们的心得体会.

关于函数式编程的学习, 作者推荐: [一个Scala的课程](https://www.coursera.org/specializations/scala).

Kotlin的优势: 和Java可以互相调用; 函数式语言; function purity; 高阶函数(函数可以作为参数或返回值); 不可变性(val); Null-safety; Anko;  Kotlin Android extensions(移除了ButterKnife); 还有对初学者很友好, 可以摆脱很多第三方的依赖, 函数扩展等等优势.

## [RxJava 2: Android MVVM Lifecycle App Structure with Retrofit 2](https://medium.com/@manuelvicnt/rxjava2-android-mvvm-lifecycle-app-structure-with-retrofit-2-cf903849f49e#.jx3vg232m)
作者一年多以前写过一个这个文章: [RxJava: Android MVVM App structure with Retrofit](https://medium.com/@manuelvicnt/rxjava-android-mvvm-app-structure-with-retrofit-a5605fa32c00#.44uq87s6w), 介绍MVVM结合Retrofit和RxJava的App架构模式. 此篇文章是一年后作者对此的改进.

主要内容有:
- 1.通过View和ViewModel之间的协议接口, 让ViewModel知道View的生命周期变化.
- 2.RxJava2的流式类型: Completable, Maybe, Flowable的使用.
- 3.用RxJava操作符组合网络请求: 让不同的网络请求一起发送, 并且都返回以后才得到通知 -> 用`.zip()`. 顺序连接不同的网络请求 -> `.flatMap()`, `.andThen()`.
- 4.后台网络请求和View更新的处理: 不取消网络请求, 等View再次resume的时候检查状态再更新. 这里提供了两种选择, 一种是用前面提到的协议接口中的生命周期方法, 另一种是用`AsyncProcessor`.
- 5.Mock Retrofit的网络请求.

## [Tech Talks - You Do Have Something To Say!](https://medium.com/upday-devs/tech-talks-you-do-have-something-to-say-a1a0ae23fa0#.61m7x6rj8)
这篇文章鼓励你分享你的知识, 经验, 问题及解决方法,  无论是通过演讲还是写出来的方式.

## [Da Real Fragmentation - Sensors](http://pguardiola.com/blog/darealfragmentation-sensors/)
介绍了Android中传感器的使用.

## [Simple unit tests for Android](https://stfalcon.com/en/blog/post/simple-unit-tests-for-android)
如何给你的App写简单的单元测试.

## [Reductor - Redux for Android. Part 2](https://yarikx.github.io/Reductor-composition/)
这是系列文章中的一篇, 继续讲[Reductor](https://github.com/Yarikx/reductor) library – Redux的Android版实现.

这篇文章结合例子将如何组合使用以及用@CombinedState来生成代码.

## [Android process death — and the (big) implications for your app](https://medium.com/inloop/android-process-kill-and-the-big-implications-for-your-app-1ecbed4921cb#.iipoq2fne)
本文探讨进程被杀死有可能导致的种种问题.

你的Android应用如果在paused或者stopped状态, 那么它任何时候都有可能会被系统杀死. 这时候你的Activity, Fragment和View状态将被保存, 当你回到应用的时候, 系统会重新启动进程, 重新创建Activity, 存储的状态会在bundle中返回.

**这个过程存在一个问题**: 整个进程都被杀死了, 所有单例(或application scope的对象), 临时数据, 还有retained Fragment中的数据, 这些所有都会处于一种全新创建的状态, 但唯有一个不同, 一些在bundle中存储的状态被恢复出来了.

这样有可能会导致一些异常, 比如你的界面想要恢复一种状态, 但是数据已经被清空了.

**如何测试这种情况呢?**
- 使用App, home键把它放进后台, 杀死app, 再恢复.
- 打开选项"Don't Keep Activities". 这种测试并不会杀死进程, 只会测试Activity的状态恢复.
- 设置Developer options中的Background Process Limit为"No background processes". 这样把应用放在后台, 打开另一个应用, 再回来自己的应用, 将会重启进程.

**相关问题信号**
- 单例
- 保存可变数据的共享的实例
- Application类中保存的数据和状态
- 可变的静态字段
- Retained fragments(状态恢复了, 但是数据却丢失了)
- 基本上任何没有在`onSaveInstanceState()`中保存但是你却依赖的状态

这些问题没有唯一的解决方案, 取决于你的应用.

## [400% faster layouts with Anko](https://medium.com/@vergauwen.simon/400-faster-layouts-with-anko-da17f32c45dd#.bz6a3y8ql)
作者把自己的一个布局改为用Kotlin的Anko, 然后测试性能.

好处是:
- 1.性能提升了, 避免了XML的运行时解析所花费的时间.
- 2.可以动态地加入逻辑, 比如版本判断, 屏幕尺寸, 方向判断等.

作者用的测试性能的工具是: [AndroidDevMetrics](https://github.com/frogermcs/AndroidDevMetrics)


## [Background Work with Android Job and Dagger](http://www.adavis.info/2016/11/background-work-with-android-job-and.html)
在Android上的后台工作, 你可以选择`Alarm Manager`, `Job Scheduler`或`GCM Network Manager`.

为了帮开发者从每种实现中抽象出来, Evernote开源了一个库: Android Job. 本文介绍了这个库如何使用.

## [VectorDrawable PNG](https://blog.stylingandroid.com/vectordrawable-png/)
作者他们的应用中有VectorDrawable的版本兼容问题, 用support library中的Compat版本也不好使, 于是他们在旧版本决定使用自动生成的png.

然后发现了生成png的颜色设置问题, 在旧版本生成的图片用的是fillColor而不是tintColor. 把fillColor设置成想要的颜色即可.

# DESIGN
## [Depth Library by Daniel Zeller](https://www.androidexperiments.com/experiment/depth-library)
一个应用, 展示了[Depth-LIB-Android-](https://github.com/danielzeller/Depth-LIB-Android-)的功能.

# LIBRARIES & CODE
## [Crescento](https://github.com/developer-shivam/crescento/)
在`ImageView`和`RelativeLayout`底部加上曲线的库.

## [LongPressPopup](https://github.com/RiccardoMoro/LongPressPopup)
长按出现弹框的库.

## [RMSwitch](https://github.com/RiccardoMoro/RMSwitch)
一个Switch Button的库, 带有更多自定义扩展功能.

## [Depth-LIB-Android](https://github.com/danielzeller/Depth-LIB-Android-)
这个库给View加上深度/厚度.

