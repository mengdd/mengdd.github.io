---
title: Android Weekly Notes Issue 219
date: 2016-08-24 17:32:23
tags: [Android, Android Weekly, Bottom Sheet, Dagger2, Security, Kotlin, JobScheduler, Trello, TextInputLayout, Code Style, Java 8, Lambda, Wear, Firebase, AutoValue]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #219
August 21st, 2016
[Android Weekly Issue #219](http://androidweekly.net/issues/issue-219)

<!-- more -->

## ARTICLES & TUTORIALS
### [Android: Bottom Sheet](https://medium.com/@emrullahluleci/android-bottom-sheet-30284293f066#.i3i4ggv13)
Bottom Sheet是一个从底部滑上来的组件, 关于这个[Google Material Design](https://material.google.com/components/bottom-sheets.html)有相关的guidelines.
这篇文章主要讲了基本使用, 比较简单.

这里私心推荐一下我自己的repo和另一个我觉得很好的教程:
[AndroidDesignWidgetsSample](https://github.com/mengdd/AndroidDesignWidgetsSample)
[CodePath-handling-scrolls-with-CoordinatorLayout](https://guides.codepath.com/android/Handling-Scrolls-with-CoordinatorLayout#bottom-sheets)

### [DI 101 - Part 1](https://medium.com/di-101/di-101-part-1-81896c2858a0#.sdgvcu8v3)
Android平台的依赖注入.
文章开始讲了下依赖注入的概念, 当前在Android上的依赖注入最著名的是[Dagger2](http://google.github.io/dagger/).
然后文章讲了如何set up dagger 2, 举了个例子, 写Module, Component, 然后使用.

这里再私心推荐一下我自己的一篇教程:
[Using Dagger2 in Android](http://www.cnblogs.com/mengdd/p/5613889.html)

### [Android Security: Welcome to Shell (Permissions)](http://doridori.github.io/Android-Security-welcome-to-shell/#sthash.2BSXwRAF.dpbs)
作者在Reddit上看到了这么一句话:
`ADB is a shell that you get on a PC with the same permissions as if you were to run a shell/terminal app on the phone itself.`
于是就写了这篇文章来讨论一下shell命令的权限问题, 关于系统底层的权限如何工作.
作者在里面提到了这本书[Android Security Internals: An In-Depth Guide to Android's Security Architecture](https://www.amazon.co.uk/Android-Security-Internals-Depth-Architecture/dp/1593275811).
关于Android安全方面的研究, 作者还建了一个repo: [Android-Security-Reference](https://github.com/doridori/Android-Security-Reference), 里面是关于安全问题的各种notes, still WIP.

### [Lessons from converting an app to 100% Kotlin](https://medium.com/keepsafe-engineering/lessons-from-converting-an-app-to-100-kotlin-68984a05dcb6#.mrqsqq9ap)
这是关于Kotlin的系列文章之part 1, part 2的文章在这里[Kotlin: The Good, The Bad, and The Ugly](https://medium.com/keepsafe-engineering/kotlin-the-good-the-bad-and-the-ugly-bf5f09b87e6f#.eyvm3gp5t), 讨论Kotlin的语言设计.
本篇文章讲什么呢?
作者是一个应用的leader engineer, 学习了几天Kotlin之后, 觉得可以解决Java存在的一些痛点, 于是把应用改为用Kotlin了, 这篇文章是在此过程中的一些想法.

**方法数问题**: 因为dex对方法数有要求, 不能超过64k, 见这里:[multidex](https://developer.android.com/studio/build/multidex.html), 作者用了这个工具来统计方法数[dexcount-gradle-plugin](https://github.com/KeepSafe/dexcount-gradle-plugin). 最后证明迁移到Kotlin之后, 代码行数减少了30%, 方法数减少了10%.

**Retrolambda**: 本来Retrolamda会生成匿名类, 并加上一些方法. Kotlin有内置的方法(apply), lamda可以直接传入, 不用生成匿名类, 不用添加额外的方法.

**Guava**: Guava的功能已经被Kotlin的标准库覆盖, 作者举例了Guava中的`ComparisonChains`, `Optional`, lazy fields和`Preconditions`等, 均有对应的Kotlin方法.

**ButterKnife**: ButterKnife仍然可以使用, 但是[Kotlin Android Extensions](https://kotlinlang.org/docs/tutorials/android-plugin.html)提供了更加自然的方式来访问绑定的views. 还有其他的方案比如[Kotterknife](https://github.com/JakeWharton/kotterknife)和[Anko](https://github.com/Kotlin/anko), 但是这俩都各自有些缺点, 不如Kotlin Android Extensions好用.

**RxJava**: RxJava仍然是很好的, 但是由于对集合并没有函数式的方法, 所以有时候会用Kotlin替代一下.

Kotlin的一个优势就是它和Java可以互相调用, 所以可以逐步改动. 
Intellij有自动把Java转化为Kotlin的功能, 但是有时候会有错.

作者推荐了学习Kotlin的资源:[Reference](http://kotlinlang.org/docs/reference/).
最后鼓励大家使用Kotlin, 因为它现在已经足够成熟了.

### [Rewriting Android Priority JobQueue - Lessons Learned](http://www.birbit.com/rewriting-android-priority-jobqueue-lessons-learned/)
作者有一个repo: [android-priority-jobqueue](https://github.com/yigit/android-priority-jobqueue), 是为Android写的任务队列管理framework, 用于调度管理后台任务.
后来Android自己也加了这个类[JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html).
最近作者重写了这个库, 改善了稳定性并加了new features, 发了V2版, 然后写个文章分享一下心得:
- 不要通过share memory来通信, 应该通过通信来share memory.
以前是多个线程访问加锁的共享资源, 线程里的一些字段标记为volatile. 新版JobManager改为单线程, 只有它可以访问共享资源, 其他线程都和JobManager通信. 这里有个文章在说这种方法[Share Memory By Communicating](https://blog.golang.org/share-memory-by-communicating)
- 如果你的代码需要做时钟相关的事情, 抽象出来.
这主要是为测试和CI考虑.
- 加新API之前多想想.

### [Trello Android Schema Upgrades](https://tech.trello.com/android-schemas/)
Trello Android之前的数据库升级方式相当简单粗暴, 他们drop整个数据库, 重新创建, 然后用server上的数据填进来.
这样在以前是没有问题的, 因为Trello的每一个操作都会立即发送给server, 不支持离线操作, 所以server上的数据永远是最新的.
但是最近他们想支持离线工作了, 这就说明不能简单地删数据库了, 因为其中可能含有没有发给服务器的离线数据.
他们要升级数据库, 这篇文章讲了他们的升级策略和他们为数据库升级而写的测试.

### [Animating the text <-> dots translation on password field](https://twitter.com/crafty/status/766967057921417216?s=03)
这个链接点进去是Twitter.
在新的support库升级(August 2016, v24.2.0)[Support Library Revision History](https://developer.android.com/topic/libraries/support-library/revisions.html)中, TextInputLayout增加了密码可见的toggle. Nick Butcher决定给按钮和文字的改变都加上动画.
这是他的repo: [plaid](https://github.com/nickbutcher/plaid).

### [Introducing Android code style guidelines at Buffer](https://overflow.buffer.com/2016/08/18/introducing-android-code-style-guidelines-buffer/)
团队工作中, 有统一的代码风格很重要, 代码风格主要是代码的可读性和一致性相关问题.
作者他们为自己的Android App归档了新的code style: [project style guidelines](https://github.com/bufferapp/android-guidelines/blob/master/project_style_guidelines.md), 当然啦, 文档是在使用中不断成长的.
做这种事主要目标是以下几个点: 一致性, 可读性, 可维护性, 易于浏览查询, 有意义.
文档写得很详细, 涉及到各个方面, 值得一看.

### [Building UserScope with Dagger 2](http://frogermcs.github.io/building-userscope-with-dagger2/)
关于Dagger 2里面自定义scope, 作者之前有一篇文章[Dependency injection with Dagger 2 - Custom scopes](http://frogermcs.github.io/dependency-injection-with-dagger-2-custom-scopes/), 本篇文章继续了这个话题.
所谓scope呢, 就是限制了单例的生存周期, 有些单例可能在整个应用生命周期都存在, 另一些单例可能只需要存在一定的时间. Dagger 2默认只提供了一个scope @Singleton, 所以我们要根据需要自定义自己的scope.
作者的例子中定义了@UserScope, 以实例说明了他的实现, 还讨论了UserScope的状态恢复问题.
例子代码: [Dagger2Recipes-UserScope](https://github.com/frogermcs/Dagger2Recipes-UserScope)

### [Using Java 8 Lambda expressions in Android](http://mayojava.github.io/android/java/using-java8-lambda-expressions-in-android/)
Java 8的一个重要特性是加入了Lambda表达式.

**Lambda表达式的语法**:
- 括号里是用逗号分隔的参数列表, 类型可以省略, 如果只有一个参数, 连括号也可以省略.
举例:
```java
TextView textView = (TextView) findViewById(R.id.text_view);
textView.setOnLongClickListener(v -> System.out.println("Long Click"));
```
- 箭头符号 `->`
- 箭头后面的body是单个表达式或者一个语句块.
如果是单个表达式, java runtime会返回它的值;
如果是语句块, 用大括号`{}`包起来.

**在Android中使用Lambda表达式**:
需要改build.gradle:
```
android {
  ...
  defaultConfig {
    ...
    jackOptions {
      enabled true
    }
  }
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

还有另一种方式是使用RetroLambda plugin: [gradle-retrolambda](https://github.com/evant/gradle-retrolambda)

### [Developing for Android Wear - A Noob’s perspective](https://medium.com/@moyinoluwa/developing-for-android-wear-a-noob-s-perspective-de47c4686ffb#.xjawtq69e)
这篇文章讲了作者作为一个新手, 第一次开发Wear应用的时候遇到的种种问题.
比如, 手表和手机是需要配对的, 在手机上安装Google的这个[软件](https://play.google.com/store/apps/details?id=com.google.android.wearable.app), 才能和手表配对, 配对了之后, 给手机安装release版apk的时候就会自动给手表安装应用.
也可以在Android Studio中选择wear来单跑Wear应用安装到手表, 但是只有配对了才能和手机有通信.
使用Wear的模拟器, 还需要运行这个命令adb -d forward tcp:5601 tcp:5601来和连接到电脑的手机连接.
还有在传输Assets时, 作者按照官方文档的例子, 却遇到了一些方法不能在UI线程调用的问题, 后来也解决了.

### [Remote config with Firebase](http://segunfamisa.com/posts/firebase-remote-config)
作者讲了如何实现Firebase的Remote config.
Remote config可以使我们控制应用的更新, 而不用重新发布一个版本.
为什么要使用远程配置呢? 主要的原因是测试新的功能, 然后可以根据用户的反馈快速地做出响应, 把更好的行为呈现给用户. 简单来说就是做A/B Test.
Firebase的remote config很强大, 指定参数后可以指定应用条件, 包括国家, 系统, 应用版本, 随机等等.

## DESIGN
### [Updates in Material design guidelines](https://material.google.com/material-design/whats-new.html)
2016年8月新发布的Material design包括了以下更新:
Notifications, Widgets, 确认操作和操作后的提示.
比较重要的更新是:
Navigation现在包括了如何使用Up和Back button.
还有使用全屏模式的三种模式:Lean back, Immersive, Lights Out以及它们相应的交互行为.

## LIBRARIES & CODE
### [Auto-value-firebase](https://github.com/mattlogan/auto-value-firebase)
AutoValue的扩展, 用来创建Firebase的数据库对象.
[AutoValue](https://github.com/google/auto/tree/master/value)是google的一个库. 用来创建interchangeable的对象, 即如果两个对象的所有fields是相等的, 我们认为这两个对象相等.

### [Icicle](https://github.com/segunfamisa/Icicle)
基于注解的一个工具, 用来保存和恢复实例的状态.
感觉跟[Icepick](https://github.com/frankiesardo/icepick)一样.

### [ReadMoreTextView](https://github.com/borjabravo10/ReadMoreTextView)
一个自定义的TextView, 可以指定按照文字长度或者行数截取显示, 带展开和关闭按钮操作.

### [Android-priority-jobqueue](https://github.com/yigit/android-priority-jobqueue)
一个为Android写的后台任务队列管理程序.

## NEWS
### [Support Library Revision History](https://developer.android.com/topic/libraries/support-library/revisions.html)
Android Support Library 24.2.0发布啦(August 2016).

v4被分成了很多小模块.

**API更新**:
Custom Tabs可以控制instant app是否打开.
TextInputLayout加了密码可见的toggle.
Transition兼容到API 14及其以上.
Custom Tabs support library支持给secondary toolbar用RemoteViews.
AppCompatResources加了可以通过getDrawable()方法, 以resource id加载<vector>和<animated-vector>的功能.
CoordinatorLayout现在支持定义inset views, 然后指定其他Views给其让路. 就是当Snackbar出现的时候, FloatingActionButton躲开的那种行为, 只不过现在给任意的child view都可以设置了.
DiffUtil类可以计算出两个集合的不同, 然后得出一个更新操作的list, 可以交给RecyclerView.Adapter.
新增了RecyclerView.OnFlingListener. 有SnapHelper和LinearSnapHelper可供选择使用.

**行为改变**:
day/night模式改变的时候, activity将会自动重启.
如果status bar是透明的, Snacker现在会在navigation bar后面绘制.

其他还有一些deprecations和bug fixes.


