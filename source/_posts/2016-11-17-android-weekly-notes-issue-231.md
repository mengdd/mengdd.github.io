---
title: Android Weekly Notes Issue 231
date: 2016-11-17 18:25:37
tags: [Android, Android Weekly, MVP, Passive View, RxJava, RxBinding, Android Studio, Tools, BottomNavigationView, Tracking, Analytics, Kotlin, Dagger2, Scope, Subcomponent, Espresso, Reference, Memory Leak, MVVM, TV, Audio, Media]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #231
November 13th, 2016
[Android Weekly Issue #231](http://androidweekly.net/issues/issue-231)

Android Weekly阅读笔记, Issue #231, 本期内容包括: MVP中的View做成passive响应式的, 返回Observable; Android Studio使用技巧; `BottomNavigationView`的使用; App tracking; Kotlin; 用Kotlin实现的Filter Animation效果; Dagger2的`Scope`和`Subcomponent`使用; Espresso测试中mock dagger注入; Android和Java中的Reference和内存泄露; MVVM + RxJava构架实际使用的经验分享; 还有TV以及Audio相关的内容等.

<!-- more -->

# ARTICLES & TUTORIALS
## [Reactive Views: retrying errors](https://medium.com/xing-engineering/reactive-views-retrying-errors-a59fffbd827f#.m2n2c6v6i)
作者他们的app近来重构采用了RxJava和Clean Architecture, 进而想要使用[passive view](http://martinfowler.com/eaaDev/PassiveScreen.html), 然后他们就发现了关于Reactive Views的一系列文章, 尤其是这一篇: [RxUi: Talking to Android View layer in a Reactive way](https://artemzin.com/blog/rxui-talking-to-android-view-layer-in-a-reactive-way/).

他们的主要工作就是把View也改成响应式的, 即View返回Observable. 在Presenter初始化的时候和View的Observable绑定, 所以事件发生的时候会trigger到presenter.

这项工作主要需要依赖于[RxBinding](https://github.com/JakeWharton/RxBinding), 由于RxBinding没有提供长按RecyclerView item的bind, 所以他们自己写了[一个](https://gist.github.com/Shyish/92257b6348312b541aa4f6b205eb14e4).
Snackbar也是: [SnackbarActionOnSubscribe](https://gist.github.com/Shyish/8af4cd774320c57ced0ec21f8840797e).


作者采用这种方式重构了他们的代码, 使得view变成完全passive的.

并且其中还有一个`retryWhen()`使用的解释: [RxJava's repeatWhen and retryWhen, explained](http://blog.danlew.net/2016/01/25/rxjavas-repeatwhen-and-retrywhen-explained/).

## [50 Android Studio Tips, Tricks & Resources](https://medium.com/@mmbialas/50-android-studio-tips-tricks-resources-you-should-be-familiar-with-as-an-android-developer-af86e7cf56d2#.gzyghprf0)
设置Logcat的颜色; 使用[Live Templates](https://medium.com/google-developers/writing-more-code-by-writing-less-code-with-android-studio-live-templates-244f648d17c7#.2p54ef8jr); 快捷键使用; Android Studio的插件; 还有一些资源分享.

## [BottomNavigationView](https://blog.stylingandroid.com/bottomnavigationview/)
Design support library 25.0.0推出了BottomNavigationView, 本文介绍其使用.

## [The key concepts of app tracking for developers](https://medium.com/@sergii/the-key-concepts-of-app-tracking-for-developers-a11bebf1e65e#.mhdpwt9x9)
这篇文章主要讲移动应用数据追踪和分析的几个原则:

- 为什么你需要tracking;  
- 什么时候需要收集数据; 
- 用什么Analytics tool; 
- 用户隐私相关; 
- 代码设计模式以及挑战; 
- 如何debug和测试输出;
- 如何分析数据;

## [Why You Must Try Kotlin For Android Development?](https://medium.com/@amitshekhar/why-you-must-try-kotlin-for-android-development-e14d00c8084b#.z0xt70upu)
为什么要使用Kotlin来做Android开发? 简洁, 安全, 灵活, 和Java可互相操作.

文中介绍了Null Safety, Smart Casting, Default Arguments, Named Arguments, Functional Programming, Concise Code.

## [Implementing Filter Animation in Kotlin](https://yalantis.com/blog/develop-filter-animation-kotlin-android/)
作者他们搞了一个应用FIT, 为女性IT工作者提供社区和交流平台, 想要成为Quora加上Linkedin.

为了让用户选择分类和过滤器tag进行搜索, 他们开发了一个组件: [SearchFilter](https://github.com/Yalantis/SearchFilter). 

文中讨论了这种设计的动画实现, 库是用Kotlin写的.

## [DI 101 — Part 3](https://medium.com/di-101/di-101-part-3-f0136e67db8#.rdp4e4fwc)
本文讲什么是Scope, 如何定义Scope, 如何使用Scope和@Subcomponent.

Subcomponent会继承父类的所有bindings.

作者定义了一个Activity的Scope, 然后定义了一个Subcomponent专门给这个Activity用, 这个Subcomponent只在这个Activity的生命周期里存活. 代码例子比较简洁易懂.

## [How Dagger 2 Helps In Android Espresso Tests](http://www.ottodroid.net/?p=514)
这篇文章介绍了如何在写Espresso测试的时候, 使用一个测试用的Dagger Component.

## [Finally understanding how references work in Android and Java](https://medium.com/google-developer-experts/finally-understanding-how-references-work-in-android-and-java-26a0d9c92f83#.95piwft68)
这篇文章讲了Java中的引用类型和Android中的内存泄露.

Java中的引用类型:
- Strong reference
- WeakReference
- SoftReference
- PhantomReference

作者详细介绍了每一种引用并用例子说明了使用场景. 

## [MVVM + RxJava: Learnings](https://medium.com/upday-devs/mvvm-rxjava-learnings-1819423f9592#.3rat89dq5)
作者他们的新闻应用采用MVVM + RxJava架构, 本文总结了他们遇到的问题和学到的两点:
1. 暴露状态, 而不是事件;
2. 所有的事情都应该通过ViewModel.

 
## [Adding TV Channels to Your App with the TIF Companion Library](http://android-developers.blogspot.com.au/2016/11/adding-tv-channels-to-your-app-with-the-tif-companion-library.html)
TV Input Framework(TIF)和Android TV让第三方应用开发者可以很容易地创建自己的电视频道. 

## [Background Audio in Android With MediaSessionCompat](https://code.tutsplus.com/tutorials/background-audio-in-android-with-mediasessioncompat--cms-27030)
Android support library中的`MediaSessionCompat`使用, 以及如何用它来做一个背景音乐.

# LIBRARIES & CODE
## [FirebaseUI-Android](https://github.com/firebase/FirebaseUI-Android)
FirebaseUI for Android — UI Bindings for Firebase.

## [ChipsLayoutManager](https://github.com/BelooS/ChipsLayoutManager)
一个自定义的RecyclerView的layout manager, 流式地显示很多小块的TextView.