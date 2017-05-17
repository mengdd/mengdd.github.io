---
title: Android Weekly Notes Issue 257
date: 2017-05-17 17:26:18
tags: [Android, Android Weekly, Gradle, Lint, PMD, FindBugs, Infer, Style, CheckStyle, JavaDoc, Jacoco, ID, ViewPager, Animation, Java 8, Kotlin, Nothing, Parcelable, Shortcuts, Android Studio]
categories: [Android, Android Weekly, Tools, IDE]
---


# Android Weekly Issue #257
May 14th, 2017
[Android Weekly Issue #257](http://androidweekly.net/issues/issue-257)
本期内容包括: Gradle中关于项目的一些设置; Android设备上的各种id讨论; ViewPagerAnimator这个库的进一步介绍; Kotlin中的`Nothing`类型介绍; 实现`Parcelable`的类和测试; Android开发中一些提高效率的快捷键. 

<!-- more -->

# ARTICLES & TUTORIALS
## [Make or break… with Gradle](https://medium.com/contentsquare-engineering-blog/make-or-break-with-gradle-dac2e858868d)
作者讲了他们的一些工作习惯:
- Git分支管理.
    - 所有向主分支的提交都必须通过Pull Request.
    - 仅在CI通过后才允许merge.
- Lint设置. 
    - `build.gradle`中的lintOptions设置.
    - `./gradlew check`会跑所有的单元测试, UI测试和Lint.
- 代码分析工具.
    - PMD.
    - FindBugs.
    - Infer.
    - 这里可以用Google提供的[Java Style Guide](http://checkstyle.sourceforge.net/reports/google-java-style-20170228.html).
- 生成文档.
    - 用CheckStyle要求所有的public方法都有JavaDoc注释(默认已经实现了).
    - 实现Gradle JavaDoc Plugin.
- 代码测试率报告.
    - Jacoco. 
    - 可以设置一些规则来检测意外打出的log和注释掉的代码(Code Smell).


## [Identifying an Android Device](http://handstandsam.com/2017/05/04/identifying-an-android-device/)
在Android设备上可以通过程序获取各种id来识别一个设备或者一次安装. 这篇文章就讨论各种id:
- 通过Settings.Secure获取到的Android ID.
- Android Build.SERIAL.
- Android Build.MODEL.
- Android Build.BRAND.
- Android Build.MANUFACTURER.
- Android Build.DEVICE.
- Android Build.PRODUCT.
- IMEI (International Mobile Equipment Identity).
- Phone Number.
- ICCID (Sim Serial Number).



## [ViewPagerAnimator – The Advanced Stuff](https://blog.stylingandroid.com/viewpageranimator-the-advanced-stuff/)
上次我们介绍了[ViewPagerAnimator](https://github.com/StylingAndroid/ViewPagerAnimator)这个库, 在ViewPager切换时进行动画, 但是上次只介绍了简单的颜色变化, 本文介绍一些关于API的高级设定: 变化的属性可以是自定义的类型; API的良好设计使得使用的代码在支持Java 8的环境下可以大幅度地得到简化.


## [Nothing (else) matters in Kotlin](https://medium.com/@quiro91/nothing-else-matters-in-kotlin-994a9ef106fc)
Kotlin中的一切都有一个类型, 甚至还有一个类型叫Nothing.

Kotlin中没有void类型, 当一个方法`fun`没有显示地声明返回值的时候, 它返回的其实是`Unit`类型.

`Unit`是一个真的类型, 继承`Any`(`Any`对应Java中的`Object`), 只接受单个的值, 是一个单例(为了避免每次方法返回Unit之后分配内存).

如果我们有一个方法, 方法中只抛出一个异常, 如果我们不特殊声明, 它的返回值仍是`Unit`, 但是也许我们应该返回`Nothing`:
```kotlin
fun fail(): Nothing {
    throw RuntimeException("Something went wrong")
}
```

`Nothing`是一个无人居住的类型, 在运行时没有值会是这个类型, 它也是其他类的子类.

当作为返回值时, `Unit`和`Nothing`到底有什么区别呢?

举例来说明:
```kotlin
val data: String = intent.getStringExtra("key") ?: fail()
textView.text = data
```
如果`fail()`方法返回`Nothing`, 我们要么得到`String`, 要么抛出异常; 如果返回`Unit`, 我们会得到一个error, 以为`Unit`不能转换为`String`.

如果不显式声明String类型呢?
```kotlin
val data = intent.getStringExtra("key") ?: fail()
textView.text = data
```
如果`fail()`返回`Nothing`, 类型是`String`;
如果返回`Unit`, 类型是`Any`, 但TextView期待的是一个`CharSequence`, 所以你仍然会得到一个error.


`Nothing`的用途就是用来显式地标记一个方法永远也不会成功地完成(它可能会抛出异常, 进入死循环或者导致一个控制流转变).

`Nothing?`有且仅有一个实例, 是`null`.
`Nothing?`是所有nullable类型的子类.


## [Android Parcelables Made Easy](https://medium.com/@calren24/android-parcelables-made-easy-acb742bcf96b)
可以在Android Studio中装一个插件, 来自动生成Parcelable的代码:
Android Studio > Preferences > Plugins > 搜索`Parcelable` > 安装`Android Parcelable Code Generator`.

安装之后, 在你的类中, 只需要声明字段, 然后Cmd + N, 选`Parcelable`就可以生成相关的代码了.

之后, 好的做法是为你的类写一个单元测试, 一旦有人加了新字段, 他们也需要保证Parcelable.
为了让你的测试fail的时候显示的信息更有效, 你还需要覆写`toString()`方法和`equals()`方法.(这些都是可以自动生成的).


## [Android shortcuts and tricks to boost up your productivity!](https://tech.fleka.me/android-shortcuts-and-tricks-to-boost-up-your-productivity-944548174582)
Android开发中一些提高效率的快捷键:
- 在Activity和它的布局间切换:
    - 在Activity声明的那一行用鼠标点icon.
    - Cmd + Shift + O 输入文件名.
    - Cmd + Shift + A 输入related symbol. (可以把这个存为一个自定义的shortcut).
- 在xml的文字和design之间切换: Ctrl + Shift + Left/Right.
- 扩展/缩减选中文字: Opt + Up/Down.
- 生成新类: Cmd + N; Opt + Enter.
- 去实现类: Cmd + Opt + B.
- 实现一个方法: Opt + Enter; Ctrl + I.
- 去基类方法: Cmd + U; 去实现类方法: Cmd + B.
- 覆写基类方法: Ctrl + O.
- 在子类中给基类加方法: 在子类中把方法标记为`@Override`, 然后在`@Override`上按Opt + Enter, 选择`Pull method xxx to YYY`.
- 改变方法参数: Cmd + F6.
- 交换方法参数: Cmd + Opt + Shift + Left/Right.
- 定位当前文件: 导航烂最左边有一个圆形小按钮可以帮你定位文件, 如果你想要自动, 可以勾选`Autoscroll from Source`.
- 把local变量改为成员变量: Cmd + Opt + F.
- 提取layout和style. (这个快捷键我实验失败了).


# LIBRARIES & CODE
## [litho-glide](https://github.com/pavlospt/litho-glide)
为litho创建的Glide图片加载组件.


## [sample-googleassistant](https://github.com/androidthings/sample-googleassistant)
Google Assistant API sample for Android Things.
