---
title: Android Weekly Notes Issue 318
date: 2018-07-19 09:45:56
tags: [Android, Android Weekly, Jetpack, Navigation, NavigationView, BottomNavigationView, Kotlin, Realm, MVI, Facebook, Twitter]
categories: [Android, Android Weekly, Kotlin]
---

# Android Weekly Issue #318
July 15th, 2018
[Android Weekly Issue #318](https://androidweekly.net/issues/issue-318)
本期内容包括: Android Navigation Component结合`NavigationView`和`BottomNavigationView`; 建议build多个modules到一个大aar; 应用例子说明UI层构建; Realm代码迁移到Kotlin; MVI模式的用法; Facebook和Twitter的sdk集成实现;
Kotlin中的scope functions; Google的机器学习例子网站: Seedbank; 用Kotlin SDL包装一个Java Builder.

<!-- more -->

# ARTICLES & TUTORIALS
## [Android Jetpack - NavigationUI](https://proandroiddev.com/android-jetpack-navigationui-a7c9f17c510e)
[Jetpack](https://developer.android.com/jetpack/)中的Navigation Component可以帮助我们做Fragment间的导航转换, 从而减少一些样板代码.

本文介绍`NavigationView`和` BottomNavigationView`结合navigation graph的用法.


## [Why We Need “fat” AARs for Android Libraries](https://handstandsam.com/2018/07/13/why-we-need-fat-aars-for-android-libraries/)
作者希望能够用多个modules创建一个aar.
这样既有利于开发时候的业务分层, 也能够提供一个整体的第三方库.

这是作者提出的issue: https://issuetracker.google.com/issues/62121508

本文主要说明他需要这样做的理由.


## [Maintainable Architecture – UI Layer](https://blog.stylingandroid.com/maintainable-architecture-five-day-forecast-ui-layer/)
一个天气应用的UI层设计.
(Kotlin, dagger).

## [Migrating your Realm to Kotlin](https://blog.blueapron.io/migrating-your-realm-to-kotlin-ee0fa5fc29b)
作者他们要把自己的Android应用迁移到Kotlin, 本文讨论了其中数据层迁移中(Realm相关)发现的一些问题.


## [Model-View-Intent & Data Binding](https://proandroiddev.com/model-view-intent-data-binding-39c7a6a6512f)
作者以一个登录界面为例, 讲述`Model-View-Intent`模式的用法.

使用了这个`MVI`的library: [mosby](https://github.com/sockeqwe/mosby).

文中例子用Kotlin实现, 结合MVI和Data Binding.

## [Social Network Integration on Android](https://www.raywenderlich.com/191933/social-network-integration-on-android)
Facebook和Twitter的SDK集成, 实现登录和分享功能.


## [Kotlin Demystified: What are 'scope functions' and why are they special?](https://medium.com/google-developers/kotlin-demystified-scope-functions-57ca522895b1)
Kotlin的"scope functions"是允许改变变量scope的函数.
Kotlin的标准库中有五个: `apply`, `run`, `with`, `let`和`also`.

`run`可以创建一个scope:

```
fun myFun() {
    val outside = 6.2831853071
    run {
        val inside = 1.61803398875
        // Both outside and inside are usable and in scope
    }
    // inside is out of scope, and only outside is available
}
```


`apply`, `run`和`with`都有一个有用的特性: 可以用this表示这个调用用到的变量:
```
class Foo {
    //...
    myView.run {
        // this refers to myView rather than Foo inside the block.
        alpha = 0.5f
        background = ContextCompat.getDrawable(context, R.drawable.my_drawable)
    }
}
```

如果想用外面的变量, 可以像我们在内部类中做的那样: 用`this@Foo`.


scope functions也是函数, 需要返回值.
一种是返回接受者, 比如`apply`.
另一种是返回最后一个语句, 比如`run`和`with`.


`let`工作起来像`run`, 可以用来做一些不为null的时候的工作:
```
myIntent?.let {
    it.data = data
    startActivity(it)
}
```
其中`let`的引用对象不是用`this`, 而是用`it`.

也可以这样写:
```
myIntent?.let { intent ->
    intent.data = data
    startActivity(intent)
}
```

`also`工作起来像`apply`, 也是用`it`.
可以做一些额外的工作, 比如:
```
val myListener = Listener().also {
    addListener(it)
}
```
和:
```
val key: String get() = keystore.getKey(KEY_ID).also {
    Log.v(TAG, "Read key at ${System.currentTimeMillis()}")
}
```

还有一些操作符: `forEach`, `map`, `filter`, 它们实际上也创建了scope, 但是它们也有一些其他的工作, 比如迭代, 映射, 过滤等. Scope functions特殊的地方就在于, 只创建scope, 没有任何其他工作.

关于如何选择, 这里有一个[流程图](https://cdn-images-1.medium.com/max/1600/1*pLNnrvgvmG6Mdi0Yw3mdPQ.png).
如果你想返回对象本身, 那么用`apply`或`also`, 如果想返回一个其他结果, 用`let`, `run`和`with`.


## [Seedbank — discover machine learning examples](https://medium.com/tensorflow/seedbank-discover-machine-learning-examples-2ff894542b57)
Google启动的[Seedback](http://tools.google.com/seedbank/)是一个机器学习的例子网站, 每一个例子都可以用浏览器查看, 并且可以编辑扩展.

## [From Java Builders to Kotlin DSLs](https://kotlinexpertise.com/java-builders-kotlin-dsls/)
DSLs – Domain Specific Languages.

本文讲一个具体的DSL实现: 把一个Java的Builder用Kotlin包装.

作者把这个库[MaterialDrawer](https://github.com/mikepenz/MaterialDrawer)用Kotlin包装了: [MaterialDrawerKt](https://github.com/zsmb13/MaterialDrawerKt).


# LIBRARIES & CODE
## [android-face-detector](https://github.com/husaynhakeem/android-face-detector)
Android实时人脸检测的库, 基于Firebase的ML kit.

## [UnderlinePageIndicator](https://github.com/dcampogiani/UnderlinePageIndicator)
配合ViewPager使用的一个indicator, 给tab文字加上下划线, 有滑动动画.

## [Seedbank](https://tools.google.com/seedbank/)
机器学习例子库.
