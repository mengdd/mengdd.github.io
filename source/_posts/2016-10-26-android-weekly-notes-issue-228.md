---
title: Android Weekly Notes Issue 228
date: 2016-10-26 12:00:24
tags: [Android, Android Weekly, Android 7, Nougat, Shortcuts, Alarm, BottomNavigationView, MVVM, Dagger, Dagger2, TensorFlow, Espresso, Test]
categories: [Android, Android Weekly, Nougat, Design Support Library]
---

# Android Weekly Issue #228
October 23rd, 2016
[Android Weekly Issue #228](http://androidweekly.net/issues/issue-228)

本期内容包括: 
Android 7.1的App Shortcuts; Searchbar的设计讨论; Nougat的Direct Reply; Alarms API讨论; Support Library的BottomNavigationView; MVVM模式; Dagger2的subcomponent实现; Test Rules介绍等.

<!-- more -->

# ARTICLES & TUTORIALS
## [Android 7.1 Static Shortcut](https://medium.com/@tonyowen/android-7-1-static-shortcut-6c42d81ba11b#.8emk7ssh1)

## [Exploring Android Nougat 7.1 App Shortcuts](https://catinean.com/2016/10/20/exploring-android-nougat-7-1-app-shortcuts/)
这两篇文章都在介绍Android 7.1的App Shortcuts.

本博客相关文章: [Android 7.1 App Shortcuts使用
](http://www.cnblogs.com/mengdd/p/5996665.html) .

## [Exposing the Searchbar](https://medium.com/@alexstyl/https-medium-com-alexstyl-animating-the-toolbar-7a8f1aab39dd#.283nz252o)
比起点击一个search icon, 然后进入搜索屏, 用户更喜欢一个search bar, 然后直接就可以在主屏上进行搜索.

作者对于他们的应用想到的解决方式就是, 在主屏上放一个search bar,然后 用一个transition, 把主屏和搜索屏(两个Activity)衔接起来, 这样用户在点击search bar之后, 不会感觉到他们打开了一个新屏.

另一个效果就是, 在点击search bar之后, 当前屏fade away, search bar展开, 在第二屏直接打开键盘, 用户可以进行搜索.

Code: [Material-SearchTransition](https://github.com/alexstyl/Material-SearchTransition).

## [Nougat - Direct Reply](https://blog.stylingandroid.com/nougat-direct-reply/)
Direct Reply是指用户可以直接回复Notification, 而不用打开app.
这篇文章作者示例了如何实现在message app中用Direct Reply清除消息和直接回复.

## [Da Real Fragmentation - Alarms](http://pguardiola.com/blog/darealfragmentation-alarms/)
作者这篇文章先是详细介绍了Alarm的各个选项和使用情形, 以及它的API版本变化.

## [Bottom Navigation View in the Design Support Library](https://blog.autsoft.hu/now-you-can-use-the-bottom-navigation-view-in-the-design-support-library/)
在Design Support Library 25.0.0中, Google发布了Bottom Navigation的官方实现: [BottomNavigationView](https://developer.android.com/reference/android/support/design/widget/BottomNavigationView.html).
这篇文章写了如何使用这个View, 并且最后列出了一些第三方库.

## [Shades of MVVM](https://www.bignerdranch.com/blog/shades-of-mvvm/)
作者讨论了MVVM模式及它的几种变形.

## [Activities Subcomponents Multibinding in Dagger 2](https://medium.com/azimolabs/activities-subcomponents-multibinding-in-dagger-2-85d6053d6a95#.p9bh8bjoc)
[dagger-2.7](https://github.com/google/dagger/releases/tag/dagger-2.7) 添加了`@Modules.subcomponents`.
本文演示了如何用这个更好地添加子ActivityComponent. 而不用每次都借助AppComponent. 这样做除了解耦之外, 对于测试时很有帮助.

例子代码: [Dagger2Recipes-ActivitiesMultibinding](https://github.com/frogermcs/Dagger2Recipes-ActivitiesMultibinding)

## [Experimenting with TensorFlow on Android Part 1 ](https://medium.com/@mgazar/experimenting-with-tensorflow-on-android-pt-1-362683b31838#.ylwet4d3p)
[TensorFlow](https://www.tensorflow.org/)是一个Machine Intelligence开源库, 主要的用途是数据计算, deep learning等.

[bazel](https://www.bazel.io/)是一个build tool, 功能类似于gradle.

本文讲了如何setup.

## [Understanding Test Rules](https://blog.egorand.me/understanding-test-rules/)
Espresso中的Rule是如何工作的呢?
之前有一个文章: [Using Rules To Influence JUnit Test Execution](http://cwd.dhemery.com/2010/12/junit-rules/)说明JUnit中的Rule是如何工作的.

然后作者讲了如何自定义一个TestRule.


# DESIGN
## [Sketch template for app shortcuts](https://plus.google.com/+RomanNurik/posts/3HMBgjn546j)
作者分享了为Android 7.1的app shortcuts功能而准备的sketch模板.

# LIBRARIES & CODE
## [PageIndicatorView](https://github.com/romandanylyk/PageIndicatorView)
Page Indicator, 结合Android ViewPager使用的, 转换时有点点连接的功能.

## [PermissionUtil](https://github.com/kayvannj/PermissionUtil)
一个Android 6.0 permission请求的库.

## [DeviceAnimationTestRule](https://github.com/VictorAlbertos/DeviceAnimationTestRule)
一个JUnit rule, 用来disable和enable设备动画.

## [DiagonalLayout](https://github.com/florent37/DiagonalLayout)
对角线布局, 感觉怪怪的.

# NEWS
## [Android 7.1 Developer Preview](http://android-developers.blogspot.com.au/2016/10/android71-dev-preview-available.html)
Android 7.1发了Developer Preview啦.
官网Overview在这里: [Android 7.1 for Developers](https://developer.android.com/preview/api-overview.html)

## [ConstraintLayout beta 1 is now available](https://sites.google.com/a/android.com/tools/recent/constraintlayoutbeta1isnowavailable)
ConstraintLayout beta 1发布啦.

# TOOLS
## [Learn You a Git](https://karumi.github.io/learnyougit/)
教你学习Git的工具.