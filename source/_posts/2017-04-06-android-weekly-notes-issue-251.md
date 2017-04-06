---
title: Android Weekly Notes Issue 251
date: 2017-04-06 17:04:00
tags: [Android, Android Weekly, Android O, Topography, Kotlin, Toolbar, JUnit 5, Lambda, Testing, Espresso, Mockito, Dependency Injection, Plugin]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #251
April 2nd, 2017
[Android Weekly Issue #251](http://androidweekly.net/issues/issue-251).
本期内容: Android O新增的API: View的tooltips; Android中的字体设置; 该不该将Kotlin用于产品代码; 实现一个带自定义动画的搜索Toolbar; JUnit 5中用Lambda表达式; 用Mockito和Espresso写测试; 
native的mobile开发应该扩展一下自己的知识; Kotlin中的依赖注入实现; Kotlin中lambda表达式的简化; 一个Intellij IDEA的插件, 帮助你改善Java代码的可读性.

<!-- more -->

# ARTICLES & TUTORIALS
## [Preliminary look at View tooltips](https://medium.com/@bherbst/preliminary-look-at-view-tooltips-b127583c5691)
Android O新推出了一个API, 是给View加tooltips.

如何使用:
可以在xml里面用属性[android:tooltipText](https://developer.android.com/reference/android/view/View.html#attr_android:tooltipText), 或者使用Java方法[View.setTooltipText()](https://developer.android.com/reference/android/view/View.html#setTooltipText%28java.lang.CharSequence%29)来指定提示文字.

它们的外观看起来就像一个toast(半透明的灰色方框, 有圆角). 它支持多行, 有最大宽度, 超过98个字符结尾会以...省略.
目前还不能被定制.

它在什么时候出现呢? 长按和悬停.
当然你的长按如果已经被处理过了(`OnLongClickListener`返回了true), 它就不会出现了.


## [Perfecting Custom Typography in Android](https://www.bignerdranch.com/blog/perfecting-custom-typography-in-android/)
关于字体设置的微调, 作者他们弄了一个小工具: [Typesetter](https://github.com/bignerdranch/Typesetter)来提高设计师和开发者沟通字体时候的效率.

### 字体尺寸
对于字体, 通常我们建议用`sp`(scaleable pixel), 1pt=1sp. (pt是point).

`sp`考虑到了用户设备上的字体设置, 所以通常是建议用sp来设置字体大小.

但是作者他们最近在应用中一些字很大的地方, 选择使用了`dp` (density-independent pixel), 这是因为这些字本来已经很大了, 所以他们不想让它们被调节以后变得更大.

### Leading
在字体排版中, Leading是指字体行之间的竖直间距. 和`line spacing`和`line height`是同义词, 同样也由`pt/sp`作为单位.

字体文件中会有一个基本的leading值, 根据字体不同可能会不同.

在Android中TextView的leading可以通过`lineSpacingExtra`和`lineSpacingMultiplier`属性来定义. 在代码中可以通过方法[setLineSpacing()](https://developer.android.com/reference/android/widget/TextView.html#setLineSpacing(float,%20float))来定义. 注意用这个方法时, 单位是像素.


### Tracking
Tracking指字间距(letterspacing).
在Android中可以通过属性`letterSpacing`来设置(API 21及以上), 以em为单位的分数测量.


## [Kotlin in Production: Should you stay or should you go?](https://medium.com/@dpreussler/kotlin-in-production-should-you-stay-or-should-you-go-a3428b44b236)
关于是否应该使用Kotlin, 作者发表了一些他的想法. 总体来说作者是支持Kotlin的, 对于各个可能存疑的点, 他都做出了解释.


## [How We Made the ToolBar on Android Move Like Jelly (in Kotlin)](https://yalantis.com/blog/toolbar-jelly-animation-kotlin-android/)
在Toolbar上点击搜索按钮, 展开关键词输入框的时候, 加一个动画, 让它有弹性地震动一下, 如何实现呢?
本文给出了详细代码.


## [JUnit 5: Lambdas](https://blog.stylingandroid.com/junit-5-lambdas/)
如何在测试中使用lambda表达式, 这篇文章里作者讨论了如何在项目中使用lambda表达式的一些方法.

有两个比较好的方法:
- 用[retrolambda](https://github.com/orfjackal/retrolambda).
- 用Kotlin.


## [Testing MVP using Espresso and Mockito](https://josiassena.com/testing-mvp-using-espresso-and-mockito/)
如何用Mockito和Espresso给一个MVP架构的程序写单元测试和UI测试.


## [The rise of the full-stack native mobile app developer](https://medium.com/@erikhellman/the-rise-of-the-full-stack-native-mobile-app-developer-a0757388bc1b)
这篇文章就说native的客户端开发应该扩展自己, 学一点后端知识, 来应对行业发展和以后的趋势.

## [Kotlin Dependency Injection with the Reader Monad](https://medium.com/@JorgeCastilloPr/kotlin-dependency-injection-with-the-reader-monad-7d52f94a482e)
Dependency Injection (DI)依赖注入是一种概念, 和具体使用的工具无关, 所以有各种不同的方法可以实现它.
本文只是提供一种思路, 用Kotlin中函数式的一些特性来做依赖注入.

首先介绍了什么是`Functors`, `Applicatives`和`Monads`, 作者推荐看这系列文章: [Kotlin Functors, Applicatives, And Monads in Pictures. Part 1/3](https://medium.com/@aballano/kotlin-functors-applicatives-and-monads-in-pictures-part-1-3-c47a1b1ce251).

后来作者举了实际的例子, 详情见原文.


## [How lambdas work in Kotlin & setOnClickListener transformation](https://antonioleiva.com/lambdas-kotlin-android/)
关于Kotlin中的lambda是如何简化的, 以`setOnClickListener()`为例:
它在Java中是这样定义的:
```java
public void setOnClickListener(OnClickListener l) {
   ...
}
```
在Kotlin中是这样的:
```
fun setOnClickListener(l: (View) -> Unit)
```

最原始的写法:
```
view.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        toast("Hello")
    }
})
```
然后IDE会提示你改为用lambda表达式:
```
view.setOnClickListener({ v -> toast("Hello") })
```
然而这个还可以进一步被简化:

如果一个方法的**最后一个参数**是一个函数, 那么它可以写在括号外面:
```
view.setOnClickListener() { v -> toast("Hello") }
```

如果一个方法只有一个参数, 并且是一个函数, 括号可以被删掉:
```
view.setOnClickListener { v -> toast("Hello") }
```

如果你并没有用到lambda表达式的参数, 你可以省略左边的部分:
```
view.setOnClickListener { toast("Hello") }
```

如果你的表达式只有一个参数, 而你要用它, 你仍然可以不写左边的部分, 用`it`来代替它:
```
view.setOnClickListener { doSomething(it) }
```


## [Making Java Code Easier to Read (Without Changing it)](https://medium.com/@andrey_cheptsov/making-java-code-easier-to-read-without-changing-it-adeebd5c36de)
如何在不改变代码的情况下, 增加Java代码的可读性?

IntelliJ IDEA为Java 8以下的用户提供了代码折叠功能, 来模拟lambda的语法.

作者自己又开发了一个新的插件[Advanced Java Folding](https://plugins.jetbrains.com/plugin/9320-advanced-java-folding), 进一步扩展了这个代码折叠的功能. 本文介绍其中一些features. 这些特性在插件中都是可选的, 可以根据需要和喜好配置.

(经过折叠以后的Java代码确实看起来很像Kotlin).

本期还有两篇Android Things的文章就不介绍啦.


# LIBRARIES & CODE
## [JellyToolbar](https://github.com/Yalantis/JellyToolbar)
一个带弹性抖动动画的搜索Toolbar实现, 本期有一篇文章介绍.


## [Android Router](https://github.com/TangXiaoLv/Android-Router)
一个轻量级的组件化协议框架, 用来解耦复杂项目.


## [Typesetter](https://github.com/bignerdranch/Typesetter)
一个小工具, 用来调整和显示字体, 本期有相关文章.


## [Telegram](https://github.com/DrKLO/Telegram)
Telegram是一个通信应用, 关注速度和安全. 这是该应用的官方开源代码.


## [Badger](https://github.com/volders/Badger)
一个给图片加数字小标(badges)的库.


## [classyshark-calculate-size](https://github.com/borisf/classyshark-calculate-size)
这个工具可以计算出你依赖的库在apk的classes.dex中到底占多少大小.


## [SlidingRootNav](https://github.com/yarolegovich/SlidingRootNav)
一个类似于`DrawerLayout`的ViewGroup, 抽屉部分隐藏在内容的下面. 内容可以向右滑动缩小以露出抽屉.
