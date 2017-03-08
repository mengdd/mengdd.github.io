---
title: Android Weekly Notes Issue 247
date: 2017-03-08 14:49:43
tags: [Android, Android Weekly, Offline, RxJava, RxJava2, Testing, MVI, FlexboxLayout, Version Name, Git, Fragment, Transition, MVP, JUnit 5]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #247
March 5th, 2017
[Android Weekly Issue #247](http://androidweekly.net/issues/issue-247).

本期内容包括: 离线模式的实现; RxJava2的测试支持; MVI模式中的单向数据流; FlexboxLayout的使用; 用脚本来配置项目的版本名和版本号; Fragment的转场动画; MVP模式的几点原则; 
RxJava中需要注意的一些点; RxJava在Android中的实现例子; JUnit 5使用.

<!-- more -->


# ARTICLES & TUTORIALS
## [Offline support: “Try again, later”, no more](https://medium.com/@yonatanvlevin/offline-support-try-again-later-no-more-afc33eba79dc#.p90haop40)
作者他们的应用很好地处理了离线模式, 他们的基本原则是, 应用中不需要显示任何的loading控件.

在离线装填下, 在用户看来仍是可以提交请求的, 只不过出现了一个sync的小图标, 一旦当用户再次连上网络, 他的请求就会被发送出去.

然后作者讲了他们的程序设计:
首先是MVP结构, 使用了Content Provider包装的SQLite数据库. (此处列举了使用Content Provider的若干优点).

后台的同步工作, 他们选择了`GCMNetworkManager`.

基本流程是这样: 但用户提交请求, 首先存储在数据库中, 状态为pending, 然后后台service发送请求, 如果成功, 则更新数据库中的状态为synced; 如果失败, 则用`GcmNetworkManager`schedule一个task, 在网络连接恢复时再做一次尝试, 成功和失败的处理同上一步.

## [Story Code](https://publicobject.com/2017/02/06/story-code/)
作者讲了一种方法, 以一种叙事的方式来写一个测试故事, 然后把它分成很多个小的测试cases. 这样可以用来驱动API的设计和其实现等.

## [Testing RxJava2](https://www.infoq.com/articles/Testing-RxJava2)
本文介绍RxJava2中内置的关于测试的支持.

测试一个`Observable`可以用`TestObserver`; 测试`Flowable`可以用`TestSubscriber`.

如何测试在不同线程上的工作? 
有几种选择:
- 把Observable变为blocking的. -> `blockingIterable()`, 缺点: 测试慢.
- 强制测试等待, 直到某个条件达成. -> `awaitTerminalEvent()`. 此处还推荐一个库: [awaitility](https://github.com/awaitility/awaitility).
- 把schedular换为一个immediate的. `RxJavaPlugins.setComputationSchedulerHandler(scheduler -> Schedulers.trampoline());`. 需要最后reset一下, 可以用JUnit的TestRule来进行简化.

用`TestScheduler`可以操纵时间, 进行白盒测试.
利用它在测试中可以精确控制时间过去了多少, 我们可以测试在中间的时间点的状态.
值得注意的是它控制的并不是真实的时间, 真实的时间还是立即就度过了的, 所以不会降低测试的速度.

我们也可以利用`TestRule`和`RxJavaPlugins`来把这个scheduler设置为测试时候要切换成的scheduler.

## [Syncing Changes](http://tech.trello.com/syncing-changes/)
Trello的离线模式实现文章系列之二. 基本的原则是在离线的时候把改动(deltas)存在数据库里, 之后有机会再同步给server.

本文介绍了他们如何计算delta和将它们按时间上传到服务器.

## [Reactive Apps With MVI - Part 4](http://hannesdorfmann.com/android/mosby3-mvi-4)
MVI系列文章的第四篇. 本篇讲如何构建独立的UI单元.

作者认为Presenter之间的Parent-Child关系是一种code smell, 因为这样引入了一种强耦合的关系, 不好读, 不好维护.

你也许要问那Presenter之间如何通信呢? 答案是, 它们根本就不需要通信, 它们只需要更新和观测同一个Model(可以说业务逻辑), 让底层来通知它们事件的发生就可以了.


## [Resources for Learning how to Test Android Apps](https://www.philosophicalhacker.com/post/some-resources-for-learning-how-to-test-android-apps/)
关于Android测试的相关资源分享.

## [Unboxing the FlexboxLayout](https://blog.devcenter.co/unboxing-the-flexboxlayout-a7cfd125f023#.ulop7q1jz)
作者想实现一个动态关键字的流式布局, 可以根据parent的宽度自动换行.

他想了几种方法, 都不太合适, 所以最后选择了`FlexboxLayout`.

作者尝试了单独使用`FlexboxLayout`和 将`FlexboxLayoutManager()`设置为`RecyclerView`的Layout Manager两种办法来实现他想要的效果.


## [Configuring Android Project - Version Name & Code](https://medium.com/@dmytrodanylyk/configuring-android-project-version-name-code-b168952f3323#.v20pogayh)
首先介绍了[git-describe](https://git-scm.com/docs/git-describe)命令.

`git describe -tags`可以输出当前最近的tag和它之后有几个提交, 还有最新提交的hash.

作者建议使用这个库: [grgit](https://github.com/ajoberstar/grgit), 写一个script-git-version.gradle:

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.ajoberstar:grgit:1.5.0'
    }
}

import org.ajoberstar.grgit.Grgit

ext {
    git = Grgit.open(currentDir: projectDir)
    gitVersionName = git.describe()
    gitVersionCode = git.tag.list().size()
    gitVersionCodeTime = git.head().time
}

task printVersion() {
    println("Version Name: $gitVersionName")
    println("Version Code: $gitVersionCode")
    println("Version Code Time: $gitVersionCodeTime")
}
```
在主文件中apply这个文件.

执行printVersion task后会输出类似这样的信息:

```
Version Name: 1.0-2-gdca226a
Version Code: 2
Version Code Time: 1484407970
```

这样使用:
```
productFlavors {
    dev {
        versionCode gitVersionCodeTime
        versionName gitVersionName
    }

    prod {
        versionCode gitVersionCode
        versionName gitVersionName
    }
}
```
这样就自动生成了版本号和版本名.
你还可以进一步设计, 在版本名中包括分支名, 时间戳之类的.


## [Workcation App - Part 1. Fragment custom transition](https://www.thedroidsonroids.com/blog/android/workcation-app-part-1-fragments-custom-transition/)
作者的系列文章, 讨论他的项目中的动画的实现.
本文是第一篇, 介绍进入map的转场动画.

首先, 在map加载完毕后, 存一个截图放在缓存里, 然后用一个自定义的Transition来做缩放和渐变的动画, 最后把它设置为fragment的转场动画.


## [Model-View-Presenter: Android guidelines](https://medium.com/@cervonefrancesco/model-view-presenter-android-guidelines-94970b430ddf#.s1d1l3dkt)
MVP实现的一些guidelines和最佳实践.
- 1.View要无脑和被动.
- 2.Presenter要和framework无关, 不依赖任何Android的类.
- 3.写一个协议描述View和Presenter的交互.
- 4.定义命名规则来区分职责.
- 5.不要在Presenter里创建生命周期的回调方法.
- 6.Presenter和View是一对一的关系. 可以定义`attach()`和`detach()`或`start()`和`stop()`来关联和解除关联.
- 7.不要在Presenter里用Bundle保存状态. 因为不能包含Android的类.
- 8.不要保存Presenter. 因为Presenter并不是一个数据类.
- 9.在Model中提供cache来恢复View的状态.


## [5 Not So Obvious Things About RxJava](https://medium.com/@jagsaund/5-not-so-obvious-things-about-rxjava-c388bd19efbc#.dl9lo390z)
RxJava使用学习中的五点(RxJava1.2.6).
- 什么时候用`map()`或者`flatMap()`.
- 不使用`Observable.create()`来创建observables. 使用其他更方便的方法, 比如`syncOnSubscribe`, `fromCallable`, `fromEmitter`.
- 如何处理Backpressure.
- 如和能让流不因为errors而停下来.
- 如何分享Observable到多个订阅者 -> `share()`或`publish()`.


## [Simplify Concurrency with Reactive Modelling on Android](https://www.toptal.com/android/simplify-concurrency-reactive-modelling-android)
用RxJava来处理Android上的并发和异步.
作者的文章中举了很详尽的各种例子.


## [JUnit 5: Getting Started](https://blog.stylingandroid.com/junit-5-getting-started/)
使用JUnit 5做测试.
本文讲了一些在Android上setup可能会遇到的问题及怎么解决.


# LIBRARIES & CODE
## [FastHub](https://github.com/k0shk0sh/FastHub)
一个Android的Github客户端.

## [gradle-android-javafmt-plugin](https://github.com/f2prateek/gradle-android-javafmt-plugin)
一个gradle plugin, 自动format代码.

## [HtmlCompat](https://github.com/Pixplicity/HtmlCompat)
Android中Html类的兼容库.