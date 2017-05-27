---
title: Android Weekly Notes Issue 258
date: 2017-05-27 16:49:12
tags: [Android, Android Weekly, Architecture Components, Room, LiveData, Lifecycle, ViewModel, MVVM, ConstraintLayout, Kotlin, Architecture, Design]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #258
May 21st, 2017
[Android Weekly Issue #258](http://androidweekly.net/issues/issue-258)
本期内容: 围绕着Google I/O的热潮, 本周的posts除了几篇小工具和软件设计原则的讨论, 其他都是在说Android Architecture Components和Kotlin.

<!-- more -->


# ARTICLES & TUTORIALS
## [DebugPort 2.0](https://medium.com/@JasonWyatt/introducing-android-debugport-2-0-88ec4ed4db94)
REPL(Read Eval Print Loop)是在命令行界面中, 读取每一行输入, 执行, 然后输出结果, 提供即时的反馈. Java 9会提供这么一个REPL的环境, 叫JShell.

考虑到在Android中能用上可能要等很长时间, 所以作者一年前开始弄了一个这个项目: [Android-DebugPort](https://github.com/jasonwyatt/Android-DebugPort).

在你的应用中加入这个依赖, 然后运行, 在通知栏就会弹出通知, 然后在电脑上就可以使用telnet来连接到app.

可以调用方法, 执行脚本, 查询数据库等.


## [Understanding Law of Demeter](https://medium.com/@ankit.sinhal/design-principal-understand-law-of-demeter-4a44ac18e923)
软件设计要求松耦合.
Law of Demeter, 即迪米特原则, 亦称为最少知道原则.

文章介绍了在这种原则下应该如何设计类和它的方法.


## [Object Oriented Tricks: #6 SLAP your functions](https://hackernoon.com/object-oriented-tricks-6-slap-your-functions-a13d25a7d994)
SLAP: Single Level of Abstraction Priciple, 单层抽象原则.

一个太长的方法往往有各种缺点, 那么怎么判断一个方法过长了呢? 根据行数来判断不太科学, 所以SLAP可以用在这里:
一个代码块中的代码应该处于同一层抽象级别.

一个方法不应该有几个不同级别的抽象, 换言之, 一个方法只做一件事.

当你在方法代码中用注释来分割代码块的时候, 这些代码可以提取出来, 这样最初的方法就变成了一个隐藏实现细节, 只展示逻辑步骤的ComposedMethod.


## [Android and Architecture](https://android-developers.googleblog.com/2017/05/android-and-architecture.html)
Google推出Architecture Components的预览, 宣布了一种新的Android App构架指导.

### 意见不是处方
写Android应用有多种方式, 这里提供的只是一系列的指导意见, 来帮助你更好地构架Android应用. 

Android framework中有一些定义良好的API, 用于处理和底层系统的交互, 比如Activity, 但是这些是你应用的入口点, 并不是你的构架块. 这些framework的组件并不强制你分离数据和UI, 也不提供清晰的方式来处理独立于生命周期的数据保存.

### Building Blocks
Architecture Components帮助你实现一个更明智的架构, 它们帮助你: 自动管理activity和fragment的生命周期, 避免内存和资源泄露; 保存Java数据对象到SQLite数据库.


### Lifecycle Components
新的[lifecycle-aware components](https://developer.android.com/topic/libraries/architecture/lifecycle.html)提供了一种方式来把你应用的核心组件绑定到生命周期事件上, 删除了显式的依赖路径.

这里核心的类是[Lifecycle](https://developer.android.com/reference/android/arch/lifecycle/Lifecycle.html), 它使用了生命周期状态和事件(states and events)的枚举来追踪相关组件的生命周期.

`LifecycleOwner`是一个接口, 其中`getLifecycle()`方法返回一个`Lifecycle`对象. `LifecycleObserver`是一个观察者, 可以监控组件的生命周期事件(通过在方法上加注解).

### LiveData
[LiveData](https://developer.android.com/topic/libraries/architecture/livedata.html)
是一个数据持有类, 同时也是一个Obervable, 允许它的数据被监听, 同时它也知道生命周期.

当你的UI订阅了底层数据的改动, 并且绑定了一个`LifecycleOwner`, `LiveData`会确保这个observer:
- 在生命周期处于活跃状态时得到数据更新.
- 当生命周期结束时取消订阅.
- 在`LifecycleOwner`重新开始时(比如旋转变化或从back stack中跳出), 得到最新的数据.


### ViewModel
[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel.html)是一个辅助类, 用于盛放Activity或Fragment的UI数据, 从UI controller的逻辑中分离出view的数据.

只要Activity/Fragment存在, ViewModel的数据就被保持着. 包括Activity/Fragment因为状态变化而销毁重建的情况. 这样ViewModel就在重建时提供可用的数据.

结合`ViewModel`和`LiveData`, 就为数据提供了一个了解生命周期的可观察的容器. 其中`LiveData`负责通知, `ViewModel`负责数据保存.


### Data Persistence
提出了[Room library](https://developer.android.com/topic/libraries/architecture/room.html), 提供了一个在SQLite之上的object-mapping抽象层, 允许流式的数据库访问. 

Room中三个主要的组件:
- `Entity`: 代码数据库中一行的数据, 用带有注解的Java数据类构建. 每一个Entity在它自己的表中被保存.
- `DAO` (Data Access Object): 定义了访问数据库的方法. 使用注解来绑定SQL到每个方法.
- `Database`. 用注解来定义entities列表和数据库版本;  定义DAO的列表; 同时也是底层数据库连接的主要访问点.


### Guide to App Architecture
[Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide.html)显示如何用Architecture Components来构建一个稳健的, 模块化的, 可测试的app.

## [Why Kotlin?](http://blog.danlew.net/2017/05/17/why-kotlin/)
作者分享了为什么相比于Java, 他更喜欢Kotlin.


## [My take on“Architecture Components”](https://medium.com/@yonatanvlevin/weighing-in-on-the-holy-architecture-war-my-take-on-architecture-components-31f7025e9c66)
作者分享了他在项目中使用Architecture components的一些心得.

他们的项目原先是MVP + Content Provider的. 为了实验加入Architecture components, 他又新写了一个MVP的[Demo项目](https://github.com/parahall/star_wars_movies), 把它转化成MVVM, 又加入Architecture components的新组件来实现. (代码比较多, 我没有详细看).


## [ConstraintLayout.com](https://blog.stylingandroid.com/constraintlayout-com/)
这个网站[ConstraintLayout.com](https://constraintlayout.com/)成立了, 用于讨论和`ConstraintLayout`相关的一切.


## [Looking at Room and LiveData - Part 1](https://riggaroo.co.za/android-architecture-components-looking-room-livedata-part-1/)
Android之前一直没有提出过一个官方的构架模式, 所以大家总在讨论各种MVP, MVVM, MVI模式, 各种库等等. 终于在Google I/O 2017, Android团队发布了Architecture Components, 来提供一个构建Android App的最佳实践指导.

### 什么是Architecture Components?
Architecture Components是一系列的库和指导集合, 作为构建Android app的基础. 针对一些常见的使用场景, 目的是减少样板和重复代码, 让你能集中精力到你应用的核心功能代码上.

主要组件包括:
- Room: a SQLite object mapper.
- LiveData: a lifecycle aware observable.
- ViewModel: Activity/Fragment和应用其他部分的交流点.
- Lifecycle: 核心部分, 包含了组件的生命周期.
- LifecycleOwner: 核心接口, 用于有生命周期的组件.
- LifecycleObserver: 当某个生命周期事件发生的时候, 应该做些什么呢?  


之后作者写了一个小App作为例子, 用了MVVM + Android Architecture Components. 其中详细介绍了Room和LiveData的用法.



## [Room — Getting Started](https://medium.com/@tonyowen/a-room-with-a-view-getting-started-ec010f9f5448)
一个简单的例子, 介绍如何使用Room. (例子用Kotlin).


## [Generating Kotlin code with KotlinPoet](https://medium.com/square-corner-blog/generating-kotlin-code-with-kotlinpoet-119dc20f74d4)
Jake Wharton的文章, Square推出了[KotlinPoet](https://github.com/square/kotlinpoet), 用来生成kotlin代码.


## [Why you should totally switch to Kotlin](https://medium.com/@magnus.chatt/why-you-should-totally-switch-to-kotlin-c7bbde9e10d5)
要转换到Kotlin的若干个理由. (有17条).


## [30 New Android Libraries](https://medium.com/@mmbialas/30-new-android-libraries-released-in-the-spring-of-2017-which-deserve-your-attention-faea359a1915)
30个Android的新库, 都是自2017年3月份以后发布的. 
作者主观挑选的, 所以并没有排序.

列表比较长, 我就不一一列了, 不少库还是挺有趣的.


# LIBRARIES & CODE
## [android-architecture-counter-sample](https://github.com/dlew/android-architecture-counter-sample)
使用Kotlin和Android architecture components写的一个sample app.


## [KotlinPoet](https://github.com/square/kotlinpoet)
Kotlin代码生成库.


## [memechat](https://github.com/efortuna/memechat)
用[Flutter](https://flutter.io/)构建的一个聊天应用, 用了Firebase, Google Sign in和设备相机.


# News
几个大新闻: 
- Android O发布了第二个预览版.
- Android team发布了Architecture Components. - Android正式宣布了支持Kotlin.
- Instant App现在对所有开发者开放
- Android Studio发布了3.0 Canary, 推出了一系列的新工具.