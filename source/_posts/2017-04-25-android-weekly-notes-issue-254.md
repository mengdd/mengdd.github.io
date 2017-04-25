---
title: Android Weekly Notes Issue 254
date: 2017-04-25 14:37:06
tags: [Android, Android Weekly, Gradle Plugin, Kotlin, Interview, Mirror, React Native, Fragment, CI, Groovy]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #254
April 23rd, 2017
[Android Weekly Issue #254](http://androidweekly.net/issues/issue-254)
本期内容包括: 如何用Kotlin写一个Gradle Plugin; 使用Kotlin的语法和最佳实践; 如何面试一个Android developer; 如何准备一次演讲; 一个反射的库: Mirror; React Native中的导航实现; Fragments的使用讨论, 使用选择, 以及各种相关的库; Yelp的Android CI系统搭建; 一个Groovy脚本, 用于保存apk和mapping文件.

<!-- more -->


# ARTICLES & TUTORIALS
## [How to Create Gradle Plugin in Kotlin](https://www.thedroidsonroids.com/blog/how-to-create-gradle-plugin-in-kotlin/)
关于如何写自定义的gradle插件, 网上有很多教程, 比如[官方教程](https://docs.gradle.org/current/userguide/custom_plugins.html). 但是大多数的教程都是用Groovy写的, 并且只有Hello World.

在这篇文章中, 我们要解决一个真实的问题(把JaCoCo集成到Gradle TestKit中), 并且用Kotlin来写.

开发完成后可以把插件发布到这里: [Gradle Plugins](https://plugins.gradle.org/), 本文的插件介绍在[这里](https://plugins.gradle.org/plugin/pl.droidsonroids.jacoco.testkit). 具体细节见原文.


## [Idiomatic Kotlin. Best Practices](https://blog.philipphauer.de/idiomatic-kotlin-best-practices/)
为了充分利用Kotlin, 我们需要修改一些在Java中用的最佳实践, 其中很多都可以用Kotlin中提供的更好的特性来取代.

文中的例子很详细, 举了很多个cases, 我们应该把Kotlin当做Kotlin来用, 而不是当做Java来用. 

推荐阅读原文.

## [Interviewing Android Developers](https://medium.com/@brendan_fahy/interviewing-android-developers-435ce69b06fa)
如何面试一个Android开发? 作者分享了他的一些经验和喜欢提的问题.


## [All You Need is Just a Little Patience](http://zdominguez.com/2017/04/17/all-it-takes-patience.html)
作者分享了他准备演讲的过程.
其中包括他花了多久准备代码, 如何做的Slides, 以及最后的演讲演练.


## [Mirror: Easy Reflection for Java and Android](https://medium.com/genymobile/mirror-easy-reflection-for-java-and-android-923f54b1f165)
作者他们开发了一个叫[mirror](https://github.com/Genymobile/mirror)的库, 用注解的形式来简化反射调用.


## [Navigation and Styling with React Native](https://developerlife.com/2017/04/15/navigation-and-styling-with-react-native/)
作者之前有一个文章: [getting started with React Native](https://developerlife.com/2017/03/31/getting-started-with-react-native/), 导航用了`Navigator`类, 实际上这是一种旧的做法, 新的做法叫做: [React Navigation](https://reactnavigation.org/).

关于基础的部分请查看上一篇教程, 本篇介绍Drawer, Stack和Tab三种导航. Demo在这里: [react-native-weather](https://github.com/r3bl-alliance/react-native-weather/tree/f877f53f8038295401d4934fa8e2d7db79a4625c).


另外还介绍了库[react-native-elements](https://react-native-training.github.io/react-native-elements/)和[react-native-vector-icons](https://github.com/oblador/react-native-vector-icons)的使用.


[React Native Elements](https://react-native-training.github.io/react-native-elements/)这个库包含了一系列有用的UI components, 比如按钮, 图标等.

[react-native-vector-icons](https://github.com/oblador/react-native-vector-icons)提供了一系列的矢量图, 相比bitmap的优势: 可以指定tint颜色, 拉伸无损, 占用空间小.

关于各类导航的实现见原文的详细介绍.


## [Fragments: The Solution to (and Cause of) All of Android’s Problems](https://news.realm.io/news/michael-yotive-state-of-fragments-2017/)
### Context
在Google I/O 2016的这个演讲中: [efficiently use fragments](https://www.youtube.com/watch?v=k3IT-IJ0J98), Adam讨论了使用Fragments常见的问题:
- 复杂的生命周期.
- Fragment Manager Transactions异步带来的问题.
- 自定义View还是Fragment?


Adam说如果我们以不同的方式来考虑问题, 上面这些挑战大多都可以被克服:
- 首先, Fragment只是应用中可组合的入口点. 从抽象层面来看, 和Activity一样.
- 其次, Fragment不是什么神奇的View. Fragment可以使用View来实现UI, 也可以不使用. 在实践中, Fragment应该响应View中的事件, 但View不应该知道Fragment.

但是在常年的使用中, 开发者也开始寻找不用Fragment的解决方案.

### The Rise of Fragment-less Architecture
在2014年, Square发表了这么一篇文章[advocating against using Fragments](https://medium.com/square-corner-blog/advocating-against-android-fragments-81fd0b462c97), 讨论了使用Fragment时遇到的种种问题.

除了指出Fragment存在的缺陷, Square还创建了不用Fragment创建应用的工具库: [Flow和Mortar](https://medium.com/square-corner-blog/simpler-android-apps-with-flow-and-mortar-5beafcd83761).


[Flow](https://github.com/square/flow)这个库允许你把你的UI screen用POJO类定义. 一个screen只需要知道和自己关联的View和更新View需要的方法. Flow也提供了基本的导航栈.

如果Flow已经包装了UI, 提供了导航栈, 
那么我们的业务逻辑放在哪里呢? 这时候[Mortar](https://github.com/square/mortar)出现了.

Mortar使用了MVP模式, 把业务逻辑包装在Presenter里, 和View交互. 每一个Presenter都是有scope的, (Thanks to Dagger), 所以presenter销毁的时候资源会被清理. Mortar也提供了对presenter生命周期的管理.

使用这两个库的优点:
- 没有Fragments.
- 强制模块化和代码的可测试性. (MVP, DI).
- 有效利用内存. (Dagger scopes).

但是也有一些缺点:
- 学习曲线较陡.
- 很多boilerplate code.
- 这两个库的release周期都非常长.


Square的blog和库激发了Android社区, 形成了两个团体: 一个是想用设计模式让Fragment更好用; 另一个是再也不用Fragment. 让我们先看一下反对派.


### Down With Fragments
用Fragments开发有一个流行的模式是[Fragment Navigation Pattern](https://www.toptal.com/android/android-fragment-navigation-pattern), 即一个Activity多个Fragments的架构. 如果你的应用采取这种架构, 你可以转换回去, 使用多个Activities. 如果你没有采用这种架构, 想摆脱Fragment, 可以考虑使用自定义View来代替.

但是这两种方法都有各自的挑战:
对于转换用Activity来说, 需要额外考虑如何支持平板, 很可能你需要写一些特殊的逻辑或者干脆创建不同的Activity. 使用自定义View, 在UI状态的维护方面又要做更多的努力, 需要自己保存和恢复View的状态, 以及管理backstack. 

有一些库是为了解决这些问题而产生的: 比如[simple-stack](https://github.com/Zhuinden/simple-stack), 一个backstack管理库;
[Conductor](https://github.com/bluelinelabs/Conductor)是一个用来搭建基于View的应用的库.


### Long Live Fragments
Square提到过Fragment很难独立测试, 所以后来他们展示了如何从Android生命周期中解耦业务逻辑, 让他们的presenter可测试. 后来Android社区终于开始利用各种设计模式, 写出更整洁和可测试的代码.

关于这些模式的一些资源:
- Eric Maxwell的[MV*模式](https://news.realm.io/news/eric-maxwell-mvc-mvp-and-mvvm-on-android/).
- Google官方的[Android Architecture Blueprints](https://github.com/googlesamples/android-architecture).
- Google在他们的Android Testing Codelab中使用[MVP模式](https://codelabs.developers.google.com/codelabs/android-testing/index.html?index=..%2F..%2Findex#3).


### Smarter Fragments
如果因为种种原因, 你还要继续使用Fragments, 我们当然可以做一些事情让它更容易工作, 同时增加可测试性和测试覆盖率. 基本的思想就是通过移动业务逻辑代码, 使它们从android代码中解耦出来, 然后就可以对业务逻辑代码进行单元测试.


此处作者举了一个MVP + Dagger的例子, 代码略.


## [Continuous Integration on Android](https://engineeringblog.yelp.com/2017/04/continuous-integration-on-android.html)
作者分享在Yelp他们是如何构建Continuous Integration (CI)系统的.


## [A Groovy Script to Save Them All](https://medium.com/pressure-labs/a-new-devs-guide-to-google-play-sanity-4-a-groovy-script-to-save-them-all-456a12672886)
一个Groovy脚本: [deployApks.groovy](https://gist.github.com/robertsimoes/23e4ae728ef3fd3f924440c059a4b6e1), 拷贝apk和mapping文件到目标目录, 可以帮助节约不少时间.


# LIBRARIES & CODE
## [Litho](http://fblitho.com/)
Facebook发布的一个库. Litho使用声明式的API来定义UI组件. 你只需要通过一组输入来描述你的UI布局, framework就会帮你做其他事, 利用代码生成, Litho会执行优化, 同时保持代码简单易用.


# TOOLS
## [How to navigate through your java projects on Github like a boss?](https://medium.com/@droidchef/how-to-navigate-through-your-java-projects-on-github-like-a-boss-488a37e16310)
一个Chrome插件, 自动为Github上代码中的类生成超链接, 点击可前往.

