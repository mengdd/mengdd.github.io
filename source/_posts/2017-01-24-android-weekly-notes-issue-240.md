---
title: Android Weekly Notes Issue 240
date: 2017-01-24 10:50:19
tags: [Android, Android Weekly, RxJava, Testing, Dagger2, Kotlin, SQLite]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #240
January 15th, 2017
[Android Weekly Issue #240](http://androidweekly.net/issues/issue-240)
Hello, 各位亲, 从本篇笔记开始, 以后并不包含Android Weekly的每一篇文章了, 只是选一些我感兴趣的做笔记. 想要看全部文章的还请点击上面的链接.

本期内容包括: 一个Android的RxJava教程; 关于测试中的注释讨论; Dagger2的实现细节讨论; Kotlin语言设计中和Effective Java相关的点和优化; Reactive app的构建模式, 一个好的model层的重要性; 怎样写数据库测试.

<!-- more -->

# ARTICLES & TUTORIALS
## [RxAndroid Tutorial](https://www.raywenderlich.com/141980/rxandroid-tutorial)
一个RxAndroid的Tutorial, 内容包括:
- Reactive Programming是什么. -> 把reactive programming比喻成excel里面的表达式.
- observable是什么.
- 如何把按钮点击和输入文字改变事件事件变为observables.
- 转换.
- 过滤.
- 指定线程.
- 把多个observables联合成一个.


## [Clean tests, Part 2: Comments](https://android.jlelse.eu/clean-tests-part-2-comments-4016ee82f186#.15u9kq7jz)
上一篇文章里作者讨论了测试代码的命名, 这篇讨论注释.
在测试里我们经常会见到这样的注释:
- // GIVEN
- // WHEN
- // THEN

注意每一次添加注释的时候都应该想清楚自己的代码是不是能够自解释, 而不是依赖于注释. 这条原则同样适用于产品代码和测试代码.

为每一个测试重复这三行其实没有什么意义, 因为这个顺序是显而易见的.

理想情况下, 简单的测试并不需要这些注释就显得很好看了, 如果是复杂的测试, 一般执行应该是一行, 验证也应该是一行, 如果需要太多验证我们应该考虑把它们抽取成多个测试方法. 而关于准备阶段, 如果我们真的需要很多准备的代码, 这是一种code smell, 可能说明我们要测试的这个方法做了太多事情, 可能我们应该先重构一下再进行测试.

我们也应该好好利用`setUp()`方法, 让我们的测试看起来更干净.

最后建议用一些比较好的assert库让最后的断言语句看起来更易懂.

## [Android Dagger2: Critical things to know before you implement](https://blog.mindorks.com/android-dagger2-critical-things-to-know-before-you-implement-275663aecc3e#.nxpqzmohn)
关于Dagger2的实现, 你应该搞清楚的几个关键点.
- 实现单例的时候, 如果提供了@Provides方法, 那么@Singleton也要在这个provides方法上声明, 声明在类上是没有用的. (类的单例声明只和构造@Inject配合使用).
- 在component中提供了get方法后, 如果这个get方法没有被调用, 则对象不会被实例化.
- Scope可以定义在该scope下的单例.

## [How “Effective Java” may have influenced the design of Kotlin](https://medium.com/@lukleDev/how-effective-java-may-have-influenced-the-design-of-kotlin-part-1-45fd64c2f974#.nqwl31wn6)
Kotlin的设计中考虑到的和Effective Java相关的几个点:
- Kotlin的构造默认参数值进一步简化了Builder模式.
- 更容易创建单例: 用object声明.
- 用了data声明后, 再也不用自己写`equals()`和`hashCode()`了.
- properties自带了默认的get/set, 使用更加简洁, 也支持后续扩展.
- Kotlin中的override关键字是强制的而不是可选的.

## [Reactive apps with Model-View-Intent - Part 1](http://hannesdorfmann.com/android/mosby3-mvi-1)
作者用RxJava + Model-View-Intent (MVI)构建的Reactive App, 也即UI响应状态变化的App.

首先作者列举了Android流行的模式MVC, MVP, MVVM, 这里面都会有一个Model. 但是作者发现大多数时候程序并没有Model这一层.

构建一个好的Model层可以解决很多问题:
- 状态.
- 屏幕方向旋转.
- 后退导航.
- 进程死亡.
- 不可变和单向的数据流.
- 可调试和重复的状态.
- 可测试性.

最基本的理念就是把这个Model层作为唯一的真实状态来源.

## [Testing SQLite on Android – Medium](https://medium.com/@MAFI8919/testing-sqlite-on-android-bfa0733e11e7#.a4hufzbc7)
如何写SQLite数据库测试.

# LIBRARIES & CODE
## [Desertplaceholder](https://github.com/JetradarMobile/desertplaceholder)
一个沙漠空白页面.

## [Android-SwitchIcon](https://github.com/zagum/Android-SwitchIcon)
Google launcher风格的Switch icon, enable时点亮, disable时灰去.

## [SlidesCodeHighlighter](https://github.com/romannurik/SlidesCodeHighlighter)
一个web应用, 让你可以把带有高亮的代码拷贝进slides.

## [GithubWidget](https://github.com/Nightonke/GithubWidget)
一个Github Widget, 显示Contributions, stars, followers, trending etc.