---
title: Android Weekly Notes Issue 320
date: 2018-08-01 08:13:20
tags: [Android, Android Weekly, Firebase, Testing, Emoji, Gradle, Kotlin, Retrofit, LiveData, Jitpack, SSH, Git]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #320
July 29th, 2018
[Android Weekly Issue #320](https://androidweekly.net/issues/issue-320)
本期内容包括: Firebase的MLKit; 关于写Android单元测试的几个原则; 用JS代码写Emoji; 用Gradle执行Kotlin脚本; 一个用Kotlin结合Retrofit, LiveData进行网络请求的例子; 
用Jitpack发布库来解决想使用库的未发行版本的问题; 在电脑上为不同的项目配置不同的SSH和Git configs; Kotlin的Observable, 代理属性的使用; 谈谈在团队中的远程交流.

<!-- more -->


# ARTICLES & TUTORIALS
## [Exploring Firebase MLKit: Landmark Detection (Part Four)](https://medium.com/@hitherejoe/exploring-firebase-mlkit-on-android-landmark-detection-part-four-5e86b8deac3a)
使用Firebase MLKit进行地标检测.


## [Seven Principles of Great Android Unit Tests](https://www.activecampaign.com/blog/inside-activecampaign/seven-principles-great-unit-tests-android/)
好的单元测试的七个原则:
* 快. (不要用Robolectric, 少用PowerMock, 真正需要的时候才用Mockito.)
* 独立. (只做一件事, 彼此不依赖.)
* 彻底. (每种情况都有.)
* 可重复. (不能具有不确定性, 不能依赖真实IO, 不共享单例.)
* 专业. (设计, 重构.)
* 可读.
* 自动化.


## [Making Emojis with Code](https://medium.com/@daniellevass/making-emojis-with-code-8da03e87a27f)
这个网站可以用来写前端代码: [JSFiddle.net](https://jsfiddle.net/).

本文示范了如何用svg tag和js代码写一个有简单交互的emoji表情.

## [Execute Kotlin Scripts with Gradle](https://kotlinexpertise.com/execute-kotlin-scripts-with-gradle/)
如何用Gradle执行Kotlin脚本.


## [Simple network calls using Retrofit, LiveData, Kotlin Coroutines and DSL](https://proandroiddev.com/oversimplified-network-call-using-retrofit-livedata-kotlin-coroutines-and-dsl-512d08eadc16)
本文介绍了用Kotlin结合Retrofit进行网络请求, 结合LiveData返回数据和请求状态.

之后又用DSL进行抽象, 变成一个通用的请求方法, 以简化代码.


## [No Version? No Problem! .. Jitpack comes to the rescue](https://proandroiddev.com/no-version-no-problem-jitpack-comes-to-the-rescue-d8754b335c34)
如果想使用一个库中还未发布的部分(新特性, bug修复), 应该怎么办呢? 

有两种选择:
* 把库代码当做一个module引入.
* Fork后作为aar加入.

但是这两种方法都不是很好.

本文介绍的方法是利用[JitPack](https://jitpack.io/).

你可以fork一个库, 选择一个节点, 然后就可以在项目中使用它了.


## [There’s never been a better time to learn Android development](https://medium.com/@lehtimaeki/theres-never-been-a-better-time-to-learn-android-development-d1724409bdfc)
现在是一个学习Android的好机会, 因为开发的很多方面都被重置了.

首先, 开发语言: Kotlin.

其次, 布局: `ConstraintLayout`.

构架方式: Google推出的[Architecture Components](https://developer.android.com/topic/libraries/architecture/).

文中附有一些相关的资源链接.

## [Splitting SSH and git configs](https://iamjonfry.com/splitting-ssh-and-git-configs/)
如何在电脑上为工作和私人项目区分SSH Keys和Git configs.


## [Listeners with Observable, from Kotlin's Delegated Properties](https://www.kotlindevelopment.com/delegates-observable/)
要监听变量的变化有很多方法, 本文讲用Kotlin的`observable property`来实现.

例子:
```
class Book {
    var title: String by observable("untitled") { _, oldValue, newValue ->
        onTitleChanged?.invoke(oldValue, newValue)
    }

    var onTitleChanged: ((String, String) -> Unit)? = null
}
```

还可以用`vetoable`, 提供了筛选值的方法.
```
class Book {
    var title: String by vetoable("untitled") { _, oldValue, newValue ->
        !newValue.isEmpty()
    }
}
```

Kotlin还提供了一些有用的属性操作类, 见: [kotlin.properties](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/index.html).


## [Thoughts on Remote Communication](https://blog.danlew.net/2018/07/19/hear-me-talkin-to-ya-thoughts-on-remote-communication/)
关于工作中的远程交流问题讨论.


# LIBRARIES & CODE
## [Kin Ecosystem Android SDK](https://github.com/kinecosystem/kin-ecosystem-android-sdk)
Kin ecosystem sdk.

## [detox](https://github.com/wix/detox)
端到端的灰盒测试和自动化库. 支持iOS, Android和RN应用测试.

## [Philology](https://github.com/JcMinarro/Philology)
用这个库可以动态地更改Android应用中的字符串. 其实现是截取了View inflate的过程.

## [Awesome Kotlin Resources](https://www.kotlinresources.com/)
Kotlin的资源集合网站.