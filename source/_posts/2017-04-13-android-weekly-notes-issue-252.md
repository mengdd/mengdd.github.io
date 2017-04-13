---
title: Android Weekly Notes Issue 252
date: 2017-04-13 16:07:42
tags: [Android, Android Weekly, Animation, AnimationList, Kotlin, Parcelable, RecyclerView, MVI, JUnit 5]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #252
April 9th, 2017
[Android Weekly Issue #252](http://androidweekly.net/issues/issue-252).
本期内容: 变化的渐变背景实现; Kotlin 1.1特性; Parcelable数据处理; RecyclerView动画实现; MVI模式的实现; 远程team的合作; 面向对象的原则: Law of Demeter; 用JUnit 5和Kotlin结合写测试.
(本期内容有点水, 不知道是我的状态不好还是Weekly的状态不好).
<!-- more -->


# ARTICLES & TUTORIALS
## [Make a moving Gradient Background in Android](http://thetechnocafe.com/make-a-moving-gradient-background-in-android/)
用AnimationList做一个不断改变的渐变色背景. 

用根节点是<<animation-list>的drawable作为View的背景:
```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
 
    <item
        android:drawable="@drawable/gradient_blue"
        android:duration="5000"/>
 
    <item
        android:drawable="@drawable/gradient_red"
        android:duration="5000"/>
 
    <item
        android:drawable="@drawable/gradient_teal"
        android:duration="5000"/>
 
    <item
        android:drawable="@drawable/gradient_purple"
        android:duration="5000"/>
 
    <item
        android:drawable="@drawable/gradient_indigo"
        android:duration="5000"/>
 
</animation-list>
```
其中的item是不同的渐变色drawable, 比如:
```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android">
 
    <gradient
        android:angle="135"
        android:endColor="#34e89e"
        android:startColor="#0f3443" />
 
</shape>
```
在Java代码中获取, 然后start它即可:
```java
LinearLayout linearLayout = (LinearLayout) findViewById(R.id.linear_layout);
 
AnimationDrawable animationDrawable = (AnimationDrawable) linearLayout.getBackground();
 
animationDrawable.setEnterFadeDuration(2500);
animationDrawable.setExitFadeDuration(5000);
 
animationDrawable.start();
```

另外, 作者推荐一个发现渐变色组合的网站: [UiGradients](https://uigradients.com/#Pinky).


## [Kotlin 1.1 is also for Android Developers](https://blog.jetbrains.com/kotlin/2017/04/kotlin-1-1-is-also-for-android-developers/)
Kotlin 1.1带来的一些很酷的features.


## [Proper Parcelable Testing](http://blog.danlew.net/2017/04/03/proper-parcelable-testing/amp/)
作者写了一个继承`Parcelable`接口的类, 然后把它存在savedInstanceState的Bundle里, 本文是他使用时的一些小建议.


## [RecyclerView interaction with Animated Markers - Part 3](https://www.thedroidsonroids.com/blog/workcation-app-part-3-recyclerview-interaction-with-animated-markers/)
动画实现: 当滚动列表的时候, 对应的Marker在地图上突出显示.

## [My take on Model View Intent (MVI) — Part 1: State Renderer](https://hackernoon.com/model-view-intent-mvi-part-1-state-renderer-187e270db15c)
作者很推荐MVI模式, 讲了自己的应用是如何实现这种模式的.

简要说了用Espresso对本程序进行自动化UI测试的方法.

## [Effective Remote Teams](http://blog.viacom.tech/2017/04/07/effective-remote-teams/)
remote team如何工作.

## [Object Oriented Tricks: #2 Law of Demeter](https://hackernoon.com/object-oriented-tricks-2-law-of-demeter-4ecc9becad85)
面向对象编程Tricks系列之2: Law of Demeter, 德米特法则. 

每一个单元应该尽量少地知道其他单元的业务, 仅知道和自己紧密联系的单元业务. **Tell Don't Ask.**

## [JUnit 5: Kotlin](https://blog.stylingandroid.com/junit-5-kotlin/)
如何结合Kotlin和JUnit 5来写测试.


# LIBRARIES & CODE
## [spruce-android](https://github.com/willowtreeapps/spruce-android)
一个轻量级的动画库.

## [Traceur](https://github.com/T-Spoon/Traceur)
一个RxJava2的debug工具, 可以打出异步调用的最初崩溃信息.


## [scratch](https://github.com/willowtreeapps/scratch)
清除用户数据然后重启应用.

## [what_the_thing](https://github.com/vigzmv/what_the_thing)
拿着相机对着东西, 然后学习如何用不同的语言来说它们的一个app, 用React Native实现.