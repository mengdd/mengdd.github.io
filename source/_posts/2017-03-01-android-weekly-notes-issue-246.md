---
title: Android Weekly Notes Issue 246
date: 2017-03-01 17:45:13
tags: [Android, Android Weekly, Shared Element, FileProvider, Testing, FlexboxLayout, RxJava, RxJava2, In-App Billing, Kotlin, Music Player]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #246
February 26th, 2017
[Android Weekly Issue #246](http://androidweekly.net/issues/issue-246)
本期内容包括: RecyclerView上的Shared Element动画; 使用FileProvider分享文件有可能会碰到的权限问题; 测试和程序架构的一些讨论; FlexboxLayout的使用; RxJava中可以处理前后动作的两个方法; 
In-App Billing的实现; 如何用组合而非继承的方式来组织应用.

代码中有意思的项目: 一个开源的音乐播放器, 一个带状态的layout.

<!-- more -->

# ARTICLES & TUTORIALS
## [Shared Element Transitions with RecyclerView](http://mikescamell.com/shared-element-transitions-part-4-recyclerview/)
作者介绍了如何在RecyclerView中实现shared element动画.

## [Sharing files through Intents (part 2)](https://medium.com/@quiro91/sharing-files-through-intents-part-2-fixing-the-permissions-before-lollipop-ceb9bb0eec3a#.ci4hqoauq)
之前介绍过因为Android 7 Nougat对文件权限的限制, 不能再依靠Intent来发送`file://uri`数据了, 应该用`FileProvider`. 但是你采用了这些新方法之后, 在一些Android的旧版本上有可能会遇到问题.

你可能遇到这种异常: `java.lang.SecurityException: Permission Denial`.

在API 16及以上, 系统有一个方法`migrateExtraStreamToClipData()`会根据你的Intent的action帮你迁移数据到ClipData, 并自动帮你加上权限. 见代码: [Intent](http://androidxref.com/7.1.1_r6/xref/frameworks/base/core/java/android/content/Intent.java#9037). 但是之前的版本却没有.

所以解决办法是在原本的代码中加上这两句:
```java
if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.LOLLIPOP) {
    takePictureIntent.setClipData(ClipData.newRawUri("", photoURI));
    takePictureIntent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION|Intent.FLAG_GRANT_READ_URI_PERMISSION);
}
```

之所以要包括LOLLIPOP是因为`migrateExtraStreamToClipData()`这个方法是在preview版本之后才加上的, 所以不能保证所有的LOLLIPOP的设备都有这个方法.


## [What Unit Tests are Trying to Tell us about Activities: Pt. 1](https://www.philosophicalhacker.com/post/what-unit-tests-are-trying-to-tell-us-about-activities-pt1/)
"android-centric"的构架是指用Activity/Fragment作为屏幕基本构架单元的程序架构. 作者的系列文章要讨论为什么这种架构是对测试不友好的.

## [Build flexible layouts with FlexboxLayout](https://android-developers.googleblog.com/2017/02/build-flexible-layouts-with.html)
Google去年开源了[flexbox-layout](https://github.com/google/flexbox-layout), 目的是将CSS中的[Flexible Layout module](https://www.w3.org/TR/css-flexbox-1/)引入到Android中来. 本文介绍了FlexboxLayout十分有用的几种情况, 附有demos.

## [Making RxJava code tidier with doOnSubscribe and doFinally](https://medium.com/@ValCanBuild/making-rxjava-code-tidier-with-doonsubscribe-and-dofinally-3748f223d32d#.58wup7kxn)
使用`doOnSubscribe()`和`doFinally()`(RxJava 2)可以让RxJava的代码更加简洁.

- `doOnSubscribe()`中的代码在subscribe的时候被调用.

- `doFinally()`在`Observable`调用`onError()`或`onCompleted()`之后, 或者流被下游放弃的时候调用.

作者举的例子是用它们来show loading和hide loading, 这样它们也作为流的一部分, 而且subscriber可以只处理其他相关逻辑.


## [Implementing In-App Billing in Android](https://hackernoon.com/implementing-in-app-billing-in-android-4896232c7d6b?gi=575af60d0286#.scggjiasz)
关于Android In-App Billing的实现.

首先你会搜到[官方文档](https://developer.android.com/google/play/billing/index.html).

作者在本文中介绍了其他的一些可选方案.


## [Composite Views in Android: Composition over Inheritance](https://medium.com/@manuelvicnt/composite-views-in-android-composition-over-inheritance-4a7114609560#.n55x4611x)
作者介绍了这个库: [CompositeAndroid](https://github.com/passsy/CompositeAndroid), 它解决了一个什么问题呢? 

在App中, 如果多个Activity或者多个Fragment有一些共同的功能, 那么我们很可能就会创建一个基类Activity或者基类Fragment, 然后继承它. 当一些功能只被一些类共享时, 我们可能会继续不断创建基类, 产生一个无法维护的继承树.

解决的办法就是使用这个库, Activity只需要继承`CompositeActivity`, 所有共有的功能都会被当做插件加进来. 
这样我们遵守了一个原则: `组合优于继承`.

但是这个库也有一些缺点: 它还在alpha阶段; 如果你使用了一些不常用的生命周期, 可能会有问题; 它是基于support library的, 所以如果这个库不更新support库的版本, 你也无法更新.

所以作者提出了一个简单的解决方案, 不使用CompositeAndroid. 文中举例展示了他的实现.


# LIBRARIES & CODE
## [ShapeShifter](https://github.com/alexjlockwood/ShapeShifter)
创建路径变形动画的一个web-app, 支持导出到`AnimatedVectorDrawable`.

## [Shuttle](https://github.com/timusus/Shuttle)
一个开源的本地音乐播放器.

## [cortado](https://github.com/blipinsk/cortado)
在Espresso上提供了一个抽象层, 使用更流畅.

## [fragment-navigation-2.0](https://github.com/gyorgygabor/fragment-navigation-2.0)
Fragment导航库.

## [flexbox-layout](https://github.com/google/flexbox-layout)
Flexbox for Android.

## [kotlin-coroutines-retrofit](https://github.com/gildor/kotlin-coroutines-retrofit)
This is small library that provides Kotlin Coroutines suspending extension Call.await() for Retrofit 2.

## [StatefulLayout](https://github.com/gturedi/StatefulLayout)
一个内置包含loading, 错误, 空状态的布局.