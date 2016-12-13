---
title: Android Weekly Notes Issue 234
date: 2016-12-13 11:08:45
tags: [Android, Android Weekly, ConstraintLayout, React Native, fastlane, Effective Java, Sticker, Flavor, Animation, SQLDelight, OkLog]
categories: [Android, Android Weekly, React Native, Java]
---

# Android Weekly Issue #234
December 4th, 2016
[Android Weekly Issue #234](http://androidweekly.net/issues/issue-234)
本期内容包括: ConstraintLayout的使用; React Native教程; fastlane管理模拟器; Android中的任务调度; 文字sticker的实现; 给Android library加flavor; 更好的关键帧动画; SQLDelight的使用; icon Animation; OkLog的使用等等.

<!-- more -->

# ARTICLES & TUTORIALS
## [Guide to ConstraintLayout](https://medium.com/@loutry/guide-to-constraintlayout-407cd87bc013#.pdg54u72z)
这篇文章教你如何使用`ConstraintLayout`, 有很多实际的例子.

## [React Native Express](http://www.reactnativeexpress.com/)
一步一步地教你跨平台的Reactive Native, 比官方的文档要深入, 并且提供例子.

## [Managing Android Virtual Devices during test session](https://medium.com/azimolabs/managing-android-virtual-devices-during-test-session-98a403acffc2#.cu4nfhl6u)
作者他们用[fastlane](https://github.com/fastlane/fastlane)管理模拟器, 并且开发了一个插件.

## [You don’t have to use WeakReference to avoid memory leaks](https://medium.com/google-developer-experts/weakreference-in-android-dd1e66b9be9d#.vmxu20g30)
并不是到处都要用`WeakReference`来避免内存泄漏.

## [Effective Java for Android](https://medium.com/rocknnull/effective-java-for-android-cheatsheet-bf4e3433889a#.8t44xdb4t)
一个cheat-sheet, Effective Java中提到的内容, 作者列出了他认为在Android开发中最重要的几项:

- 用private来限制不可实例化.
- 使用静态工厂方法.
- 使用Builders.
- 避免互换性.
- 静态内部类.
- 使用泛型.
- 返回空的集合而不是null.
- 字符串连接用StringBuilder, 不要用+.
- 可恢复的异常.

## [Scheduling tasks in Android made easy](https://blog.hypertrack.io/2016/12/01/scheduling-tasks-in-android-made-easy/)
分发异步任务的时候, 用很多选择: `AlarmManager`, `Handler`, `JobSheduler`, `GcmNetworkManager`. 作者他们的库: [smart-scheduler-android](https://github.com/hypertrack/smart-scheduler-android)就是用来有效地处理异步任务调度问题.

## [How to create beautiful text stickers for Android](https://medium.com/uptech-team/how-to-create-beautiful-text-stickers-for-android-10eeea0cee09#.11x8ar94q)
之前作者有一篇文章讲了如何创建Snapchat一样的图片stickers.

本篇讲如何创建文字的stickers, 代码: [MotionViews-Android](https://github.com/uptechteam/MotionViews-Android).

## [Elite Worship](http://blog.sqisland.com/2016/12/elite-worship.html)
Chiu-Ki Chan分享了一些她的看法, 关于精英崇拜, 和如何让社区更加平等, 鼓励每一个人都参与进来.

## [Product Flavors for Android Libraries](https://medium.com/@sahildave/product-flavors-for-android-library-d3b2d240fca2#.ravhhk30a)
如何给Android Library加上不同的flavor使用.

## [Keyframes: Delivering scalable, high-quality animations](https://code.facebook.com/posts/354469174916519)
Facebook分享了一个库[Keyframes](https://github.com/facebookincubator/Keyframes)用来导出AE的动画, 并且在移动设备上播放它.

## [SQLDelight: Getting Started](https://medium.com/@tonyowen/sqldelight-getting-started-67054fe51306#.rske25ore)
[sqldelight](https://github.com/square/sqldelight)是一个库, 可以用SQL语句来生成Java Model类.
SQLDelight也是一个Intellij插件.

作者介绍了如何使用SQLDelight, 注意生成models需要结合AutoValue.

## [Your ViewHolders are Dumb. Make ’em Not Dumb](https://medium.com/@jonfhancock/your-viewholders-are-dumb-make-em-not-dumb-82e6f73f630c#.auaur0y3r)
作者举例说明ViewHolder应该如何优化代码, 解放Adapter.


## [An Introduction to Icon Animation Techniques](http://www.androiddesignpatterns.com/2016/11/introduction-to-icon-animation-techniques.html)
如何创建漂亮的icon动画.

## [OkLog 2.0 — improved Android network logging](https://medium.com/@simonpercic/oklog-2-0-improved-android-network-logging-a72b2ffe4c66#.6h8w44eh8)
[OkLog](https://github.com/simonpercic/OkLog)是一个库, 可以在logcat中打印网络请求和响应, 点击进入页面查看, 本文介绍2.0版本的改进.

## [How to Build an Android App for Fire TV (Part 4)](https://medium.com/amazon-appstore/developing-for-the-living-room-how-to-build-an-android-app-for-fire-tv-part-4-cbe572a6f1e6#.zfg39casg)
本文是为Fire TV搭建一个Android App系列文章的第四篇.

# LIBRARIES & CODE
## [android-PageFlip](https://github.com/eschao/android-PageFlip)
3D的翻页效果.

## [smart-scheduler-android](https://github.com/hypertrack/smart-scheduler-android)
用于周期性和非周期性任务分发的工具类.

## [PageLoader](https://github.com/arieridwan8/pageloader)
一个简单的可定制化的loading页面库.

## [fastlane-plugin-automated-test-emulator-run](https://github.com/AzimoLabs/fastlane-plugin-automated-test-emulator-run)
fastlane插件, 用于启动模拟器进行自动化测试.

## [Keyframes](https://github.com/facebookincubator/Keyframes)
导出AE动画并在移动设备上播放的库.