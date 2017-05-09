---
title: Android Weekly Notes Issue 256
date: 2017-05-09 17:34:44
tags: [Android, Android Weekly, ViewPager, Animation, Tail Recursion, Kotlin, MVI, Dagger2, Floating Menu, RxJava, Loader, BottomNavigationView]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #256
May 7th, 2017
[Android Weekly Issue #256](http://androidweekly.net/issues/issue-256)
本期内容包括: 一个给ViewPager切换时加动画的库; Tail Recursion和它在Kotlin中的实现; MVI模式中的状态恢复; Dagger2的新API使用; 一个新的框架库Flax介绍.
代码部分包括: ViewPager加动画的库; 悬浮菜单; RxLoader结合RxJava和Loader实现数据加载; 一个封装BottomNavigationView的库, 使得状态切换类似于ViewPager.

<!-- more -->

# ARTICLES & TUTORIALS
## [ViewPagerAnimator – The Basics](https://blog.stylingandroid.com/viewpageranimator-the-basics/)
一个轻量级的库: [ViewPagerAnimator](https://github.com/StylingAndroid/ViewPagerAnimator). 本文介绍它的基本用法, 举了一个例子, 可以在切换pager的时候改变背景颜色.


## [Tail recursion and how to use it in Kotlin](https://medium.com/@JorgeCastilloPr/tail-recursion-and-how-to-use-it-in-kotlin-97353993e17f)
尾部递归和它在Kotlin中的实现.

官方文档见: [tail-recursive-functions](https://kotlinlang.org/docs/reference/functions.html#tail-recursive-functions). 关键字: `tailrec`.


## [Reactive with MVI Part 6 - Restoring State](http://hannesdorfmann.com/android/mosby3-mvi-6)
使用MVI(Model-View-Intent)模式, 保持数据流的单向性, 会很大程度上简化状态恢复. 本篇就介绍怎么做和为什么.

这篇文章关注的状态分两种: 一种是memory中的状态(比如屏幕旋转时); 一种是persistent的状态, 即存在Bundle中的状态.

### In Memory
对于Memory中的状态, 很简单, 我们只需要保证我们的RxJava流在Android组件生命周期之外仍然发送新的状态.

对于MVP来说, 这就是让Presenter在View的生命周期之外存活, 每当view重新attach到presenter上之后, 就按照前一个状态重新渲染. 只有当view完全被销毁了之后presenter才释放.

使用情形: 屏幕旋转, back stack回退.

### Persistent State
在Android中通常用`Activity.onSaveInstanceState(Bundle)`来保存状态. 在MVI中View有一个`render(state)`方法, 所以一个显而易见的方法是让state实现`Parcelable`然后保存在bundle中.


### 结论
使用单向数据流和一个表达状态的Model以后, 很多和状态相关的事情变得很容易实现了.

但是通常处于两个理由, 不会把状态放在bundle里: 第一, Bundle有大小限制; 第二, 我们仅仅讨论了如何序列化和反序列化状态, 但恢复状态可能是另一回事.


## [Dagger 2: Android Modules](https://medium.com/proandroiddev/dagger-2-android-modules-e168821cfc57)
Dagger 2发布了新版本2.11-rc2.
在2.11中有新的API: `@ContributesAndroidInjector`.

[dagger2的changelog](https://github.com/google/dagger/releases).

本文介绍了新API相关的用法, 和之前的实现做了比较.

新的用法总结起来有以下三点:
- 继承`DaggerApplication`来注入相关的dispatchers.
- 在component中包含`AndroidSupportInjectionModule.class`.
- 创建一个bind方法, 用`@ContributesAndroidInjector`标注.


## [Hello Flax — A Reactive Architecture For Android](https://hackernoon.com/hello-flax-a-reactive-architecture-for-android-8e56af9c575a)
当前Android中的一个趋势是创建reactive的app.
作者最初听说这个概念是从[Flux](https://facebook.github.io/flux/). 后来看了一系列[MVI模式的文章](http://hannesdorfmann.com/android/mosby3-mvi-1), 作者自己也做了一个MVI的[尝试](https://medium.com/@CodyEngel/lets-try-model-view-intent-with-android-3190a899c3a1). 之后他就创建了[Flax](https://github.com/CodyEngel/Flax), 是一个轻量级的框架库(还在进一步开发中).


本文介绍了Flax库的使用, 基本可以总结为以下几点:
- Model作为唯一的状态真相.
- View只做无脑的渲染.
- Renderer接收Model变化的通知, 调用View的渲染方法.
- Responder接收用户交互事件, 调用Model的更新方法.


# LIBRARIES & CODE
## [ViewPagerAnimator](https://github.com/StylingAndroid/ViewPagerAnimator)
一个轻量级的ViewPager动画库.

## [floatingMenu](https://github.com/rjsvieira/floatingMenu)
一个悬浮的action menu, 点开后展开多个菜单选项.


## [RxLoader](https://github.com/kmdupr33/RxLoader)
一个轻量级的加载数据的库, 结合Loader和RxJava, 避免了内存泄露.
基本使用方法: 和你的`Observable`或者`Single` `compose`一下就好.

作者还有一篇文章详细介绍为什么他觉得他这个库很有必要: [RxLoader: Lightweight, Boilerplate-Free Data loading with Loaders and RxJava](https://www.philosophicalhacker.com/post/rxloader-boilerplate-free-data-loading-with-loaders-and-rxjava/).


## [AdaptableBottomNavigation](https://github.com/bufferapp/AdaptableBottomNavigation)
使用support库的`BottomNavigationView`的时候, 需要自己处理tab间的切换. 作者他们受到`TabLayout`的启发, 创建了一个`ViewSwapper`类, 可以简化`BottomNavigationView`的View管理, 有点像`ViewPager`的变种, 但去掉了滑动切换的功能.



