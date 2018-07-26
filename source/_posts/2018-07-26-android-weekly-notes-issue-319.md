---
title: Android Weekly Notes Issue 319
date: 2018-07-26 11:14:29
tags: [Android, Android Weekly, MotionLayout, Animation, Kotlin, Android Studio, IDE, CI, Gradle, LiveData, Transformations, MediatorLiveData]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #319
July 22nd, 2018.
[Android Weekly Issue #319](https://androidweekly.net/issues/issue-319)
本期内容包括: `MotionLayout`加动画; Kotlin中的when; Android Studio的快捷键; Google的开发者认证; 一个天气应用的的数据层构建; 一个Android项目的CI搭建; 用Kotlin写gradle build脚本; 
测试库KotlinTest的使用; `LiveData`, `Transformations`和`MediatorLiveData`的使用; Kotlin中property delegation的使用; Kotlin的一个事件分发库: KDispatcher.

<!-- more -->

# ARTICLES & TUTORIALS
## [Creating Animations With MotionLayout for Android](https://code.tutsplus.com/tutorials/creating-animations-with-motionlayout-for-android--cms-31497)
`ConstraintLayout`非常有用, 但是想要给它的内容加复杂的动画却有点费时间. 所以Google在I/O 2018推出了`MotionLayout`.

`MotionLayout`在support库中, 继承了`ConstraintLayout`, 是目前唯一一个只用xml就可以为它的内容声明动画的控件, 并对于它所有的动画都提供了细粒度的控制.

本篇文章讲述如何使用`MotionLayout`.

### 创建布局
首先`MotionLayout`的使用和`ConstraintLayout`一样, 在其中放入View然后设置约束.

### 创建Motion Scene
然后, 创建一个motion scene:
```xml
<?xml version="1.0" encoding="utf-8"?>
<MotionScene
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
 
    <!-- More code here -->
     
</MotionScene>
```

在其中加入`ConstraintSet`, 通常是两个, 一个为动画开始, 一个为结束:
```xml
<ConstraintSet android:id="@+id/starting_set">
    <Constraint android:id="@+id/actor"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        android:layout_width="60dp"
        android:layout_height="60dp"
        />
</ConstraintSet>
 
<ConstraintSet android:id="@+id/ending_set">
    <Constraint android:id="@+id/actor"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        android:layout_width="60dp"
        android:layout_height="60dp"
        />
</ConstraintSet>
```

注意每一个`ConstraintSet`都必须指明大小和位置, 因为它们将会覆盖layout中的设置.

还需要一个`Transition`元素:
```xml
<Transition
    android:id="@+id/my_transition"
    app:constraintSetStart="@+id/starting_set"
    app:constraintSetEnd="@+id/ending_set"
    app:duration="2000">
     
</Transition>
```

最后在`MotionLayout`中关联motion scene文件:
```
app:layoutDescription="@xml/my_scene"
```

### 运行动画
配置完成后, 当你运行程序的时候, `MotionLayout`会自动应用`Transition`中`constraintSetStart`属性指定的constraint set.
所以为了开启动画, 你只需调用控件的`transitionToEnd()`方法.


### 事件监听
可以给`MotionLayout`附加`TransitionListener`来监听动画事件.

### 创建关键帧
上面的例子只规定了起始点和重点, 所以是一个直线, 如果想要更改路径, 可以通过定义几个关键点.

在motion scene的`Transition`中增加`KeyFrameSet`.

`MotionLayout`支持多种关键帧的类型, 本文示范了`KeyPosition`和`KeyCycle`的用法.
前者规定一个位置, 后者规定振荡模式.

### 和动画的元素交互
可以直接和动画的元素交互, 目前支持点击和拖拽事件.

分别是通过在`Transition`中增加:
```xml
<OnClick
    app:target="@+id/actor"
    app:mode="transitionToEnd"/>
```
和:
```xml
<OnSwipe
    app:touchAnchorId="@+id/actor"
    app:touchAnchorSide="top"
    app:dragDirection="dragUp" />
```


## [When is “when” exhaustive?](https://medium.com/@ataulm/til-when-is-when-exhaustive-31d69f630a8b)
Kotlin中的`when`的使用.
要么提供所有的分支, 要么提供一个`else`.

本文提出写一个扩展, 让编译器强制我们实现一个完备的`when`.


## [Android Studio - Taming the interface](https://jeroenmols.com/blog/2018/07/16/androidstudioshortcuts3/)
Android Studio的一些快捷键:
* ⌥ + number: open/close views
* ⇧ + ⌘ + ↑: enlarge view
* ⇧ + ⌘ + ↓: shrink view
* ⇧ + ⌘ + →: enlarge side view
* ⇧ + ⌘ + ←: shrink side view
* ⇧ + ⌘ + F12: close all views

* ⇧ + ⌘ + ]: next tab
* ⇧ + ⌘ + [: previous tab
* ^ + ⇧ + →: text view (xml layout editing)
* ^ + ⇧ + ←: design view (visual layout editing)
* ⌥ + letter: invoke button

* ⌘ + ⇧ + A: action lookup


## [Becoming Google Certified Associate Android Developer](https://medium.com/@sodiqOladeni/becoming-google-certified-associate-android-developer-907bdb61d79f)
一个谷歌认证的考试: Associate Android Developer Exam介绍.

包括一个需要在24h内完成的项目和一个10分钟的在线视频面试. 费用$149, 完成之后可以获得Associate Android Developer称号.


## [Maintainable Architecture – Daily Forecast](https://blog.stylingandroid.com/maintainable-architecture-daily-forecast/)
一个天气应用的数据层设计.
(Kotlin, dagger. 上一次介绍的是
UI层.)

## [Cloud Continuous Integration on Android with Kotlin Project](https://proandroiddev.com/cloud-continuous-integration-on-android-with-kotlin-project-8d6f12cbf0c4)
作者为自己的项目:
- 搭建CI, 用[Travis](https://travis-ci.org/).
- 集成覆盖率测试报告, 用[Codecov.io](https://codecov.io/) + `JaCoCo`.
- 可以把CI和覆盖率状态加在readme里.


## [Moving Your Gradle Build Scripts to Kotlin](https://pspdfkit.com/blog/2018/moving-your-gradle-build-scripts-to-kotlin/)
gradle是用Groovy写的.

首先讨论了Groovy写脚本的几个缺点:
* 动态类型.
* IDE的代码提示慢, 不完整.
* Groovy写的Gradle不能调试.

本文介绍解决这个问题的方法: 增加一个`buildSrc` module.
`buildSrc`是一个特殊的module, 在gradle构建中是第一个被编译的模块, 在里面你可以做这些事情:
* 写自定义的gradle插件.
* 提取工具函数和配置, 让它们全局可见.
* 写自定义的task类.
* 整理自定义的task graph.
* 单步调试.

文中介绍了用kotlin写build脚本的详细做法.


## [Data Driven Testing with KotlinTest](https://proandroiddev.com/data-driven-testing-with-kotlintest-a07ac60e70fc)
[Data driven testing](http://spockframework.org/spock/docs/1.1/data_driven_testing.html)是指同样的测试代码, 但是要用很多不同的输入数据和期待结果来进行测试.

[KotlinTest](https://github.com/kotlintest/kotlintest)是一个Kotlin写测试的库.

本文介绍了如何使用它来写一些测试.


## [Reactive patterns using Transformations and MediatorLiveData](https://medium.com/google-developers/livedata-beyond-the-viewmodel-reactive-patterns-using-transformations-and-mediatorlivedata-fda520ba00b7)
利用[LiveData](https://developer.android.com/topic/libraries/architecture/livedata)来构建流式的构架.

`LiveData`的目的: 解决View controller(Activity, Fragment)和数据源(ViewModel)之间的通信, 让View只在生命周期允许的时候接收数据. 也即, 你不用手动地去取消View和ViewModel之间的订阅.

你可以利用这种方式去观察应用的其他组件, 比如:
* SharedPreference的改变.
* Firestore的document或collection.
* Authentication SDK(比如FirebaseAuth)中的当前用户.
* [Room](https://developer.android.com/topic/libraries/architecture/room)中的一个query.

这种机制的好处是以为一切都绑定在一起, 一旦数据改变了, UI就会自动更新.

缺点就是`LiveData`没有想Rx那样, 自带绑定流式数据或管理线程的工具集.


为了在组件中传递数据, 我们需要map和combine的方法, 用了MediatorLiveData, 结合Transformations中的方法:
* 一对一静态转换: [Transformations.map](https://developer.android.com/reference/android/arch/lifecycle/Transformations#map%28android.arch.lifecycle.LiveData%3CX%3E,%20android.arch.core.util.Function%3CX,%20Y%3E%29): 在ViewModel中把从repository取来的数据转化成UI model.
* 一对一动态转换: [Transformations.switchMap](https://developer.android.com/reference/android/arch/lifecycle/Transformations#switchMap%28android.arch.lifecycle.LiveData%3CX%3E,%20android.arch.core.util.Function%3CX,%20android.arch.lifecycle.LiveData%3CY%3E%3E%29): 在观察repository之前需要观察user manager得到一个用户id.
* 一对多依赖: [MediatorLiveData](https://developer.android.com/reference/android/arch/lifecycle/MediatorLiveData)让你可以给单个observable加一到多个数据源.

**什么时候不需要用LiveData?**
当你的组件和UI没有联系的时候, 就很可能用不上`LiveData`.


**Antipattern:** 共享`LiveData`的实例, 可能会有数据更新错误或显示错误的情况. 所以尽量不要共享`LiveData`的实例.

文章后面说了一些`MediatorLiveData`, `map`和`switchMap`使用时的注意事项, 以及用Kotlin来实现时的简洁版.


## [Delegate your Lifecycle to Kotlin](https://blog.blueapron.io/delegate-your-lifecycle-to-kotlin-17c1d0d876c9)
[Property delegation](https://kotlinlang.org/docs/reference/delegated-properties.html)是Kotlin中非常有用的一个特性.

通过这个模式, 我们可以把get和set一个property的任务交给一个类来处理. 但是表面上用起来仍然像是在直接使用这个property.

本文讲如何用Kotlin的property delegation来处理Fragment和Activity之间的通信. 

(笔者注: 但是我觉得文中所讲的例子并没有内存泄露的情况, 所以问题不存在, 解决方案也没有必要了.)

## [KDispatcher simple and light-weight event bus for Kotlin](https://medium.com/@sphc/kdispatcher-simple-and-light-weight-event-bus-for-kotlin-e0fa4aaea1c7)
一个Kotlin的事件分发库: [KDispatcher](https://github.com/Rasalexman/KDispatcher).

类似于EventBus.


# LIBRARIES & CODE
## [wearfaceutils](https://github.com/purposebakery/wearfaceutils)
写Android手表表面的工具类.

## [pickle](https://github.com/fourlastor/pickle)
一个Android cucumber测试的代码生成库.

## [kotlintest](https://github.com/kotlintest/kotlintest)
Kotlin测试框架.

## [SdkSearch](https://github.com/JakeWharton/SDKSearch)
Android应用和Chrome extension用来搜索Android Sdk的文档.

## [ketro](https://smilecs.github.io/ketro/)
Kotlin包装的Retrofit和LiveData.

## [KDispatcher](https://github.com/Rasalexman/KDispatcher)
Kotlin的事件分发库, 功能类似于EventBus.