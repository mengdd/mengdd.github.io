---
title: Android Weekly Notes Issue 238
date: 2017-01-03 18:02:01
tags: [Android, Android Weekly, Firebase, Notification, RecyclerView, Background work, Flutter, Gradle, Testing, Android Things, CI]
categories: [Android Weekly]
---

# Android Weekly Issue #238
January 1st, 2017
[Android Weekly Issue #238](http://androidweekly.net/issues/issue-238)
本期内容包括: Firebase发送Notification; RecyclerView的预取; 后台工作的实现方式讨论; RecyclerView分组数据; 跨平台应用工具Flutter介绍; Gradle依赖管理; 
写测试的一些注意事项; Android Things应用搭建及一些思考; 如何搭建CI等.
 
<!-- more -->

# ARTICLES & TUTORIALS
## [Mastering Firebase Notifications](https://medium.com/@Miqubel/mastering-firebase-notifications-36a3ffe57c41#.ykkpzrs4l)
用Firebase发通知:
- Console Notifications.
- 使用命令行, 发送curl命令.
- `FirebaseMessagingService`在应用前台的时候处理通知.
- 如果应用前后台的时候都需要处理, 则发送data而不是notification.
官方文档[Firebase cloud-messaging](https://firebase.google.com/docs/cloud-messaging/android/receive).

## [RecyclerView Prefetch](https://medium.com/google-developers/recyclerview-prefetch-c2f269075710#.21takened)
作者研究了RecyclerView的渲染时间, 发现在滚动的时候很多的时间会花在新item的创建和bind上, 这样会推迟UI线程的其他工作, 还有RenderThread的后续工作, 如果超出了frame boundary, 就有可能会造成明显的卡顿.

而同时前一帧, UI线程可能处于空闲状态.

那么我们有没有可能以一种预取的方式, 把即将出现的View在提前的空闲阶段准备好呢?

pre-fetch的优化已经在[Support Library v25](https://developer.android.com/topic/libraries/support-library/revisions.html#rev25-0-0)加入, [v25.1.0](https://developer.android.com/topic/libraries/support-library/revisions.html#25-1-0)有进一步的加强. 如果你没有自定义LayoutManager, 也没有嵌套`RecyclerView`, 那么你升级support library之后就自动获得了这项优化. 其他两种情况你还需要调用一些方法.

你可以设置`LayoutManager.setItemPrefetchEnabled()`来对比开启和关闭预取功能前后的不同. 性能测量用[Systrace](https://developer.android.com/studio/profile/systrace.html)和[GPU profiling](https://developer.android.com/studio/profile/dev-options-rendering.html).

## [Things to consider before running background tasks](https://blog.yipl.com.np/things-to-consider-before-running-background-tasks-e71f00d2ad3a#.baugcaodi)
完成后台任务的几种方式和各自的优缺点分析.
- Thread.
- AsyncTask.
- Service.
- IntentService.
- Loader.
- JobService and JobScheduler. GCM Network Manager.
- RxJava.

## [Android RecyclerView - Grouping Data](https://krtkush.github.io/2016/07/08/android-recyclerview-grouping-data.html)
作者展示了如何将RecyclerView中的数据分组展示, 在他的例子中是按照时间分组, 每一组开始是该组的时间占据一行.

其实主要是前期的数据处理, 首先创建一个HashMap, 分组依据作为key, 符合该依据的数据作为值存在对应key的value list里; 然后给日期和数据创建一个共同的基类, 把HashMap再重新展开成一个List, 里面穿插好数据. 最后用RecyclerView按照数据类型不同显示两种布局.

## [Flutter Intro](https://medium.com/@develodroid/flutter-i-intro-and-install-a8bf6dfcc7c8#.f9ktsu3r8)
[Flutter](https://flutter.io/)是一个Google推出的新工具, 用来构建跨平台的应用.

本文介绍了如何setup和创建一个Hello World.

## [How to add Gradle dependencies using ‘foreach’](https://hackernoon.com/android-how-to-add-gradle-dependencies-using-foreach-c4cbcc070458#.aplxhrmn3)
一种管理依赖的方式, 把所有的依赖定义在同一个文件的不同分组里, 然后在每个module各自添加自己的分组即可.

## [Best practices to improve app engagement](https://android-developers.googleblog.com/2016/12/important-best-practices-to-improve-app-engagement.html) 
如何提高app的用户参与度.

## [The Do’s and Don’ts of Writing Test cases in Android](https://blog.mindorks.com/the-dos-and-don-ts-of-writing-test-cases-in-android-70f1b5dab3e1#.7ol81s1wo)
作者分享了在写测试的时候需要注意的几点:
- 首先明确我们要测试的是什么, 预先条件是否满足, 如果是因为前置条件不满足, 那么并不是我们的测试本身失败了.
- 每个测试都是独立完成的, 测试的执行顺序不应该影响结果.
- 在测试中不要写条件语句. 条件语句是在实际代码中的, 每一个条件都应该对应一个单独的测试case.
- 测试应该不受外部因素影响, 比如server和网络. 因为如果因为这类原因测试失败了, 并不代表我们的代码有bug.

## [Making Rainbow HAT Work with the Android Things](https://blog.egorand.me/making-rainbow-hat-work-with-the-android-things-2/)
一个Android Things应用.

## [Christmas Voice – Part 2](https://blog.stylingandroid.com/christmas-voice-part-2/)
一个小应用, 录音, 做转换并播放.

## [Will Android do for the IoT what it did for mobile?](https://medium.com/@carl.whalley/will-android-do-for-iot-what-it-did-for-mobile-c9ac79d06c#.41phc9zbb)
关于Android Things的一些看法.

## [Set up a CI server for Android dev](https://medium.com/@pamartineza/how-to-set-up-a-continuous-integration-server-for-android-development-ubuntu-jenkins-sonarqube-43c1ed6b08d3#.lzs2m4zg8)
如何搭建Android项目的CI, (Ubuntu + Jenkins + SonarQube).

# DESIGN
## [Material: Growth & communications](https://material.io/guidelines/growth-communications/introduction.html#)
如何进行用户引导, feature发现和手势教育.

# LIBRARIES & CODE
## [PanoramaImageView](https://github.com/gjiazhe/PanoramaImageView)
一个ImageView, 在设备转动的时候可以自动滚动内容.

## [TextDecorator](https://github.com/nntuyen/text-decorator)
可以给文字分段加上各种装饰, 下划线, 点击事件等.

## [Delightful-SQLBrite](https://github.com/geralt-encore/Delightful-SQLBrite)
一个示例应用, 展示[SQLDelight](https://github.com/square/sqldelight)和[SQLBrite](https://github.com/square/sqlbrite)结合使用.

## [mainframer](https://github.com/gojuno/mainframer)
一个远程build的脚本.

## [material-about-library](https://github.com/daniel-stoneuk/material-about-library)
创建一个Material风格about页面的库.

## [android-snowfall](https://github.com/JetradarMobile/android-snowfall)
下雪View.

## [Android-ExpandIcon](https://github.com/zagum/Android-ExpandIcon)
展开/合上的上下箭头icon, 支持点击和手势滑动切换.

## [RxAnimations](https://github.com/0ximDigital/RxAnimations)
Rx形式的动画库.