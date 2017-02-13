---
title: Android Weekly Notes Issue 244
date: 2017-02-13 14:53:29
tags: [Android, Android Weekly, Fragment, ClassyShark, Firebase, Permission, Privacy Policy, Kotlin, Testing, FileProvider, Nougat, Android 7]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #244
February 12th, 2017
[Android Weekly Issue #244](http://androidweekly.net/issues/issue-244)
本期内容包括: Android Fragments使用教程; ClassyShark使用; Firebase的Personal App Indexing功能引出的一些权限问题; 关于应用内没有提供Privacy Policy的后续处理; Kotlin中的annotation processor讨论; Pull和Push模式的讲解; 为什么Android测试这么难; Android 7 Nougat不再支持用Intent发送`file:// URI`, 应用需要改用`FileProvider`实现原有功能.
<!-- more -->

# ARTICLES & TUTORIALS
## [Android Fragments Tutorial: An Introduction](https://www.raywenderlich.com/149112/android-fragments-tutorial-introduction)
一篇如何使用Fragments的讲解.

## [Exporting data from ClassyShark](https://medium.com/@BorisFarber/exporting-data-from-classyshark-e3cf3fe3fab8#.e2r76detr)
用[ClassyShark](https://github.com/google/android-classyshark)的APK dashboardA检查apk的问题(重复依赖, 过期依赖等).
本文介绍如何一次性导出全部的数据.

## [Post-mortem : Firebase vs permissions](http://jeremie-martinez.com/2017/02/08/firebase-permissions/)
两周前Firebase发布了一个新功能: [Personal App Indexing](https://firebase.google.com/docs/app-indexing/android/personal-content). 之后遇到了一些权限相关的问题, 本文讨论遇到的具体问题和解决方法, 然后他们发布了一个hotfix版本.

## [Did you get one of these Google Play Developer Policy Violation Emails?](https://medium.com/@ali.muzaffar/did-you-get-one-of-these-google-play-developer-policy-violation-emails-6c529ceb082d#.glctt861o)
如果你的应用使用了一些"dangerous permissions", 你需要在应用或者Google Play上附有privacy policy, 否则你就会收到Google Play的邮件.

作者他的Demo app也收到了这种邮件, 所以他提供了他的解决方法.

他找到了这个[网站](https://privacypolicytemplate.net/), 这是他最后写成的[Gist](https://gist.github.com/alphamu/c42f6c3fce530ca5e804e672fed70d78). 利用[RawGit](https://rawgit.com/)可以将github上的文件url转成用HTML显示的url. 之后在app中设置一个链接, 点击打开这个url就可以了.

## [Pushing the limits of Kotlin annotation processing](https://medium.com/@workingkills/pushing-the-limits-of-kotlin-annotation-processing-8611027b6711#.7crkk5m68)
关于Kotlin的annotation processor支持, 是一个很复杂的问题, 作者讨论了关于这个问题的历史进展和当前的局限性.

## [Pull vs Push & Imperative vs Reactive - Reactive Programming](http://www.uwanttolearn.com/android/pull-vs-push-imperative-vs-reactive-reactive-programming-android-rxjava2-hell-part2/)
作者用浅显的代码例子解释了Pull和Push模式的区别, 一个是自己不停地查询读取, 另一个是等改变发生的时候收到通知. 

## [Why Android Testing is so Hard: Historical Edition](https://www.philosophicalhacker.com/post/why-android-testing-is-so-hard-historical-edition/)
为什么Android项目这么难测试呢? 作者认为主要有三方面的历史原因: 
- Performance方面的考虑. 
- 对Android组件的误解.
- Android和Unit Testing出现的时机.

## [Sharing files though Intents: are you ready for Nougat?](https://medium.com/@quiro91/sharing-files-though-intents-are-you-ready-for-nougat-70f7e9294a0b#.h3f06hxg7A)
Android 7 Nougat引入了一些文件系统的权限变化, 来增强安全性.

如果你已经把`targetSdkVersion`升到了24+, 并且你用Intent发送一个`file:// URI`, 你将会得到一个`FileUriExposedException`.

解决办法是使用`FileProvider`.

# LIBRARIES & CODE
## [SlidingSquaresLoader](https://github.com/biodunalfet/SlidingSquaresLoader)
一个有趣的动画方块的loading图案.

## [ason](https://github.com/afollestad/ason)
一个JSON库, 简化了序列化, 更易使用.

## [Intro-To-RxJava](https://github.com/PareshMayani/Intro-To-RxJava)
上一期有一篇文章提过的RxJava练习项目.

## [chuck](https://github.com/jgilfelt/chuck)
An in-app HTTP inspector for Android OkHttp clients.
截取请求和响应, 点击通知可以查看UI显示.

## [android-parcelable-intellij-plugin-kotlin](https://github.com/nekocode/android-parcelable-intellij-plugin-kotlin)
为kotlin的类生成Parcelable代码的插件.
