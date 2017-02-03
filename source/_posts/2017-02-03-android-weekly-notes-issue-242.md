---
title: Android Weekly Notes Issue 242
date: 2017-02-03 17:27:49
tags: [Android, Android Weekly, Design Patterns, ObjectBox, MVC, MVP, MVVM, Google Actions, Animation, FAB, Kotlin, Firebase, Model-View-Intent, Gradient, Color]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #242
January 29th, 2017
[Android Weekly Issue #242](http://androidweekly.net/issues/issue-242)
本期内容包括: Android中常用的设计模式; 基于NoSQL的移动对象数据库--ObjectBox; MVC, MVP和MVVM模式的讨论; 一个Google Actions的Java SDK; 一个带黏性动画的FAB的实现; Kotlin 1.1的新功能; Firebase的实时数据库讨论; Model-View-Intent模式应用的实现; 关于实现gradient时透明颜色的使用.

<!-- more -->

# ARTICLES & TUTORIALS
## [Common Design Patterns for Android](https://www.raywenderlich.com/109843/common-design-patterns-for-android)
### Creational Patterns
- Builder
- 依赖注入: 举例: Dagger 
- Singleton

### Structural Patterns
- Adapter
- Facade: 举例: Retrofit

### Behavioral Patterns
- Command: 举例: EventBus
- Observer: 举例: RxAndroid
- Model View Controller
- Model View ViewModel

## [ObjectBox - The new Mobile Database](http://greenrobot.org/announcement/introducing-objectbox-beta/)
[ObjectBox](http://greenrobot.org/objectbox/)是greenrobot发布的一个新的mobile对象数据库, 主要关注于性能, 据说superfast.

在ObjectBox中, 主要是把NoSQL技术迁移到mobile端使用. 之前他们创建的greenDAO, 据说是最快的Object/Relational Mapper (ORM) for Android and SQLite.

ObjectBox的5大特性:
- Superfast.
- Object API.
- Instant unit testing.
- Simple threading.
- No manual schema migrations.

[文档](http://greenrobot.org/objectbox/documentation/)
[Demo](https://github.com/greenrobot/ObjectBoxExamples)

## [MVC vs. MVP vs. MVVM on Android](https://realm.io/news/eric-maxwell-mvc-mvp-and-mvvm-on-android/)
MVC, MVP, MVVM模式的介绍.

## [Building Google Actions with Java](https://medium.com/@froger_mcs/building-google-actions-with-java-696cffedbd01#.d6uuck1ho)
非官方的[Google Actions Java SDK](https://github.com/frogermcs/Google-Actions-Java-SDK), 本文为开发者介绍其如何使用.

## [Android Gooey FAB is EASY](http://myhexaville.com/2017/01/18/android-gooey-fab-easy/)
实现一个胶黏的FAB.
首先作者展示了效果, 点击FAB, 从中逐渐分离中一个新的小按钮. 作者讨论了这种效果可能的实现方法:
- 用bitmap的mesh transformation, 这是能高度自定义的.
- 创建自定义View, 自己绘制Path.
- 最简单的办法: 用Animated Vector Drawable, 即本文所介绍的方法.

源码在这里: [Android-Animations](https://github.com/IhorKlimov/Android-Animations)

## [What Comes in Kotlin 1.1 for Android Developers?](https://blog.elpassion.com/what-comes-in-kotlin-1-1-for-android-developers-831d559f780f#.wlmujxwi3)
Kotlin 1.1的新features.
- Coroutines. 改善Kotlin中的异步编程.
- Type Aliases. 可以为类型起别名.
- Inlining Property Accessors.
- Less Restrictive Inheritance. sealed类的子类不用再放在同一个类中; 非final的类现在也可以继承data类了.
- Destructuring and Underscores.
- Methods Count. 作者对比了一个sample程序, 用kotlin的不同版本, 发现用最新版kotlin确实会增加一些方法数, 但它仍然算是一个很轻量的库.

## [Understanding the Power of Firebase Security Rules](https://medium.com/@dftaiwo/understanding-the-power-of-firebase-security-rules-part-1-f46aae773a24#.cw34j1v2z)
作者要写关于Firebase的一系列文章: 第一和第二篇主要介绍实时数据库的规则, 第三篇介绍Storage的规则.

关于实时数据库规则的主要内容包括:
- 识别你的用户.
- 控制数据访问权限.
- 验证创建, 更新和删除操作.

## [Reactive apps with Model-View-Intent - Part 3](http://hannesdorfmann.com/android/mosby3-mvi-3)
上一篇中介绍了用Model-View-Intent模式来构建一个单相数据流的简单屏. 这篇文章接着讲如何用MVI和state reducer来构建一个复杂屏.

(感觉太复杂了我没仔细看).

## [Android Dev Tip #3: A gotcha with color/transparent](https://android.jlelse.eu/android-dev-tip-3-99da754151ad#.rarx8jafm)
如果你要在xml中用gradient写一个渐变色, 对于透明色`@android:color/transparent.
`的使用一定要注意.

透明色`@android:color/transparent.
`的色值是`#00000000`, 所以它实际上代表的是一个透明的黑色.

在gradient进行插值的时候, 会对ARGB每一个通道的色值都分别进行插值然后叠加.

所以如果你想要保持颜色不变, 只改变透明度, 也即Alpha通道的值, 你就应该把透明色中RGB颜色设置为和原来的颜色一样. 

# LIBRARIES & CODE
## [PreviewSeekBar](https://github.com/rubensousa/PreviewSeekBar)
一个带Preview的SeekBar.

## [AndroidTestingBox](https://roroche.github.io/AndroidTestingBox/)
一个Android项目, 用于实验各种测试工具.

## [FunctionalRx2](https://github.com/pakoito/FunctionalRx2)
a collection of constructs to simplify a functional programming approach to Java and Android.

## [gradle-completion](https://github.com/eriwen/gradle-completion)
gradle的tab补全, for bash and zsh.

## [ObjectBox](https://github.com/greenrobot/ObjectBox)
超快的移动平台对象数据库.

## [superlightstack](https://github.com/nextdimension/superlightstack)
一个轻量级的库, 用于创建View的stack, 并处理转换和状态维持.

## [PicassoFaceDetectionTransformation](https://github.com/aryarohit07/PicassoFaceDetectionTransformation)
一个配合picasso使用的图像转换库, 可以根据人脸检测自动确定范围而切图.

(Readme中附有配合Glide和Fresco使用的版本.)

## [cwac-netsecurity](https://github.com/commonsguy/cwac-netsecurity)
This library contains a backport of the Android 7.0 network security configuration subsystem.