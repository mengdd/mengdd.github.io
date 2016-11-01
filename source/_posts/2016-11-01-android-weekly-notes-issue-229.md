---
title: Android Weekly Notes Issue 229
date: 2016-11-01 18:01:50
tags: [Android, Android Weekly, Pury, Multiselection, Audio, Test, Dagger, Dagger2, ConstraintLayout, Mobile Vision, Barcode Detection, RxJava, SOLID]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #229
October 30th, 2016
[Android Weekly Issue #229](http://androidweekly.net/issues/issue-229)
Android Weekly笔记, 本期内容包括: 性能库Pury的插件化; 一种新的多选设计和实现; 音频播放; Dagger的测试mock方案; ConstraintLayout的链式约束; Mobile Vision API的二维码扫描功能; RxJava的使用缺陷讨论; SOLID原则图解.

<!-- more -->

# ARTICLES & TUTORIALS
## [Get access to raw profiling results with plugins for Pury](https://medium.com/@nikita.kozlov/get-access-to-raw-profiling-results-with-plugins-for-pury-f9a7cc5e8345#.y26lx22wo)
Pury是一个做profile的工具, 前面有过一篇文章介绍: [Pury](https://medium.com/@nikita.kozlov/pury-new-way-to-profile-your-android-application-7e248b5f615e#.57ggfep5p).

本文是作者的另一篇文章, 讲Pury的插件化和扩展.

另外, 作者最近正在集成Google Analytics到Pury中.


## [Building a Multiselection Solution for Android in Kotlin](https://yalantis.com/blog/how-we-created-a-multiselection-solution-for-android/)
在移动应用上的多选设计很难, 通常不是很灵活, 用起来也不舒服. 

本文推荐了一种全新的多选设计: 把屏幕分为两部分: 包括主要的列表和选中列表. 选中的项目自动移动到选中列表中去.

这个设计概念的实现: `ViewPager` + 两个`RecyclerView`.

作者选用了kotlin来实现. 列举了几个kotlin的features: Extension functions, Null safety, Collections, Better syntax.

作者的库: [Multi-Selection](https://github.com/Yalantis/Multi-Selection).

本文中还介绍了如何使用这个库.

## [Audio (not) playing in Android](https://medium.com/uptech-team/audio-not-playing-in-android-cde9a0fdfafd#.kp7qsjuha)
关于Android上的音频播放, 作者的总结文章. 

音频播放的方式有:
- [MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer.html)
- [SoundPool](https://developer.android.com/reference/android/media/SoundPool.html)
- [AudioTrack](https://developer.android.com/reference/android/media/AudioTrack.html)
- [ExoPlayer](https://github.com/google/ExoPlayer)

关于`MediaPlayer`的使用, 官方文档: [Media Playback](https://developer.android.com/guide/topics/media/mediaplayer.html), 本文中有一张图是`MediaPlayer`的生命周期图.

作者逐个列举了实际使用这些API时可能会遇到的一些issues. 并且最终选择的最佳解决方案是Google的[ExoPlayer](https://github.com/google/ExoPlayer), 2.0版本已经解决了她之前遇到的所有issues.

## [Providing test doubles with Dagger 1 and Dagger 2](https://blog.egorand.me/providing-test-doubles-with-dagger-1-and-dagger-2/)
这篇文章讲在使用Dagger1和Dagger2的项目中, 如何为测试mock依赖.

## [ConstraintLayout Chains – Part 1](https://blog.stylingandroid.com/constraintlayout-chains-spread-chains/)
作者讲了`ConstraintLayout`的一个重要特性: chains, 链.
chains是一个机制, 把一些独立的Views链起来, 然后我们可以对这一个集合来采取一些行为.

比如选中一个parent下的两个TextView(这两个本来是分别对齐parent的左右), 然后选择"Center Horizontally", 就是建立了一个链.
在xml中实际上给这两个view都各自加上了一条限制条件, 限制它们在对方的(左/右)边, 这两条对称性的限制条件就构成了一个链.

这种链叫spread chains, 是默认的style.


## [Machine Learning with the Mobile Vision API — Part 2 ](https://hackernoon.com/machine-learning-for-android-developers-with-the-mobile-vision-api-part-2-barcode-detection-61e84c858518#.3dy9fgj56)
使用[Mobile Vision的Barcode API](https://developers.google.com/vision/barcodes-overview)来进行二维码检测.
Code: [barcode-detector](https://github.com/moyheen/barcode-detector).

## [Reactive Frustrations](https://upday.github.io/blog/reactive_frustrations_1/)
大多数Rx相关的文章都说优点, 本篇不同, 作者分享了在使用RxJava过程中碰到的一些烦人的事情.
不过尽管有这些挫折, RxJava仍然是一个很棒的工具.

### 文档
RxJava的文档有时候对初学者来说会很具迷惑性.
推荐看: [RxMarbles](http://rxmarbles.com/), 有操作符图解.
### 匿名类
RxJava的使用中会构建很多匿名类.
推荐使用: [Retrolambda](https://github.com/orfjackal/retrolambda), [Kotlin](https://kotlinlang.org/), 或[Jack](https://source.android.com/source/jack.html).
### 忘记subscribe
这是一个常见的错误, 如果只写好了Observable但没有触发, 通常是没有subscribe, 因为Observable是被动的, 只有当被订阅的时候才会触发.
### 代码的推理
有时候很难看见一块代码就知道执行结果, 必须往上游排查.

所以作者在他们的项目中规定了一项对于Observable的命名规范:
`...Once`表示只发射一次; `...Stream`表示会发射值, 或者不发射, 但是不会completes; `...OnceAndStream`订阅时会发射值, 之后可能会继续发射, 但是不会停止.

### `...map`操作符
有一些比较容易混淆的操作符:
- `flatMap`: 并行;
- `switchMap`: 中断前一个, 串行;
- `concatMap`: 等待前一个结束, 串行;


## [Designing something S.O.L.I.D](https://www.novoda.com/blog/designing-something-solid/)
**SOLID**是软件开发的五项原则:

SOLID (single responsibility, open-closed, Liskov substitution, interface segregation and dependency inversion).

这里是[Wiki](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))的解释.

这篇文章图形化地解释了SOLID, 配图和例子都很有趣.

# DESIGN
## [Design Is Never Done](https://design.google.com/articles/design-is-never-done/)
Material Design的新工具套件和开源项目.

# LIBRARIES & CODE
## [EasyMVP](http://6thsolution.github.io/EasyMVP/)
一个MVP库. 比较特别的几点:
- 使用注解来注入Presenter(可以和Dagger2结合使用, 否则只能注入无参构造), 绑定Presenter和View的生命周期;
- 使用Loaders来字啊configurations changes时保存Presenter;
- 加上`easymvp-rx`插件后, 遵循Clean Architecture原则, 加入了domain层, 提供了UseCase的基类;


## [Input Mask](https://github.com/RedMadRobot/input-mask-android)
一个小的工具库, 可以按格式显示用户的输入. 比如在输入上加括号, 每三位数字空一格之类的.

## [sdk-artifact-sync](https://github.com/JakeWharton/sdk-artifact-sync)
一个脚本, 同步你local Android SDK中的所有artifacts到一个remote的Maven artifact host上.

## [material-remixer](https://github.com/material-foundation/material-remixer)
material-remixer是一个工具, 利用它可以实时调整产品的UI参数. 目标平台: Android, iOS和Web都能用的工具.

# News
## [ConstraintLayout beta 2 is now available](https://sites.google.com/a/android.com/tools/recent/constraintlayoutbeta2isnowavailable)
ConstraintLayout beta 2发布啦, 修改了一些issues并改善了性能.

## [Google Play Services Release Notes](https://developers.google.com/android/guides/releases#october_2016_-_v98)
Google Play Service 9.8发布了.

## [Some new Firebase libraries](https://firebase.googleblog.com/2016/10/start-your-week-off-with-some-new-firebase-libraries.html)
Firebase也发了新版.