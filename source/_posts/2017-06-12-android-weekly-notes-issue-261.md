---
title: Android Weekly Notes Issue 261
date: 2017-06-12 18:41:02
tags: [Android, Android Weekly, Android O, Adaptive Icons, Design Patterns, Kotlin, Instant App, RecyclerView, ItemDecoration, Functional Programming, Gradle, RxJava 2, Camera2, Animation]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #261
June 11th, 2017
[Android Weekly Issue #261](http://androidweekly.net/issues/issue-261)
本期内容包括: Adaptive Icons; Kotlin实现的几种常用的设计模式; Android Instant App; Kotlin中的Ranges; 一个叫Graywater的库, 可以改善RecyclerView的性能; ItemDecoration的使用; 函数式编程; 提高Gradle的build的速度; 用RxJava 2包装Camera2 API. 

代码部分有一个Kotlin实现的RSS阅读器值得一看.

<!-- more -->

# ARTICLES & TUTORIALS
## [Adaptive Icons and more](https://blog.stylingandroid.com/adaptive-icons/)
关于Android O的Adaptive Icons, 这里是官方的文档: [Adaptive Icons](https://developer.android.com/preview/features/adaptive-icons.html).

这篇文章介绍了如何用Android Studio制作这种icon.



## [Gang of Four Patterns in Kotlin](https://dev.to/lovis/gang-of-four-patterns-in-kotlin)
用Kotlin实现的各种设计模式:
- Decorator -> 用extension functions.
- Builder -> 用`apply`.
- Prototype -> 用`data`类`copy`.
- Singleton -> 用`object`关键字.
- Template Method -> 用extension functions.
- Strategy -> 方法参数, `typealias`, 高阶函数.
- Iterator -> `interator()`.


这是作者实现的repo: [gof-in-kotlin](https://github.com/lmller/gof-in-kotlin).



## [From Westinghouse to Android Instant apps](https://tech.buzzfeed.com/from-westinghouse-to-android-instant-apps-60fbfaca4ebe)
作者讲了他和Instant App的故事.


## [Musings on Kotlin Ranges](http://blog.danlew.net/2017/06/05/musings-on-kotlin-ranges/amp/)
作者讲了他发现的一些关于Kotlin的[Ranges](https://kotlinlang.org/docs/reference/ranges.html)的有趣的事情:

- IntRange如果初始值比结束值大, 会被认为是空的. 想要逆序的话就得用`downTo`或者`reversed()`.
- in range会被编译器优化为两个<=条件判断, 所以我们可以利用这点来简化我们本来的判断.
- 几种range的for循环效率比较.


## [Introducing Graywater for Android](https://engineering.tumblr.com/post/161546559631/introducing-graywater-for-android)
介绍一个叫[Graywater](https://github.com/tumblr/Graywater)的库, 处理RecyclerView中的复杂项目, 据说可以改善滚动性能, 减少内存使用, 而且提供了一种组件化的构建方式.


## [Making the Domain Android App Instant](http://tech.domain.com.au/2017/06/making-the-domain-android-app-instant-%E2%9A%A1/)
Domain的Instant App实现, 概要介绍, 不涉及太多细节.
文章后面说了一些Instant App的限制, 比如: 4MB大小; 只有有限的权限, intent和库.


## [ItemDecoration - Avoid adding dividers to the view layout](https://medium.com/proandroiddev/itemdecoration-in-android-e18a0692d848)
`ItemDecoration`的介绍.

首先, 不要用在布局里加View的方法来加divider, 这对性能不好. 增加了多余的View, 还可能需要增加层级.

其次, 加View的方式也有一些副作用, 比如左右滑动item动画的时候, divider会和View一起移动, 这显然不好看.

最后, 加View的方式也不如`ItemDecoration`那样具有灵活性. 比如你想加不同长度的divider给不同位置的item.

所以推荐使用`ItemDecoration`. 自动25.0.0开始, support库还添加了`DividerItemDecoration`类.


注意: 
- 一个RecyclerView可以添加多个`ItemDecoration`.
- `onDraw()`是在绘制item之前, `onDrawOver()`是在绘制item之后.


## [Functional Programming for Android Developers — Part 3](https://medium.com/@anupcowkur/functional-programming-for-android-developers-part-3-f9e521e96788)
函数式编程教学第三部分, 主要讲高阶函数和Closures.

前两部分见:
- [Functional Programming for Android Developers — Part 1](https://medium.freecodecamp.com/functional-programming-for-android-developers-part-1-a58d40d6e742)
- [Functional Programming for Android Developers — Part 2](https://medium.freecodecamp.com/functional-programming-for-android-developers-part-2-5c0834669d1a)


## [How to speed up your slow Gradle builds](https://android.jlelse.eu/how-to-speed-up-your-slow-gradle-builds-5d9a9545f91a)
Google I/O 2017关于如何提高gradle build速度的10个建议:
- 1.使用最新的Gradle plugin.
- 2.避免使用老的multidex, 在API 21以前会有性能影响.
- 3.在开发时disable multi-APK.
- 4.最小化包含的资源.
- 5.在开发时关闭PNG优化.
- 6.使用Instant Run.
- 7.避免非故意的改动. 如把vesionCode和当前时间相关, 这样每次build就等于manifest会被改变. 还有`Crashlytics`会为每次build生成id.  可以在develop的时候关闭这些.
- 8.不要使用动态的依赖版本.
- 9.注意memory的设置. 在`gradle.properties`中, 如`org.gradle.jvmargs=-Xmx2048m`.
- 10.使能缓存. 在`gradle.properties`中, `org.gradle.caching=true`.


## [Reactive selfies with Camera2 API on Android - Part 1](https://techblog.badoo.com/blog/2017/06/07/reactive-selfies-with-camera2-api-on-android-part-1/)
作者的一个教程, 用RxJava2包装Camera2的API.
文章讲得很仔细, 项目代码见: [Camera2API_rxJava2](https://github.com/ArkadyGamza/Camera2API_rxJava2).


## [Re-animation](https://medium.com/google-developers/re-animation-7869722af206)
作者更新了他关于向量动画的文章, 因为support库25.4中加入了对老版本的兼容.


# LIBRARIES & CODE
## [Karchitec](https://github.com/msesma/Karchitec)
Kotlin的RSS阅读器, 使用了Google的android architecture components库.


## [SwiftKotlin](https://github.com/angelolloqui/SwiftKotlin)
一个工具, 可以把Swift代码转换为Kotlin代码.

## [Graywater](https://github.com/tumblr/Graywater)
一个改善RecyclerView滚动性能的库.

## [Fontify](https://github.com/mehdok/Fontify)
提供不同语言自定义字体和style的TextView, EditText和Button.
