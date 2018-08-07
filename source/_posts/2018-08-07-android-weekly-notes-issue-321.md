---
title: Android Weekly Notes Issue 321
date: 2018-08-07 13:18:04
tags: [Android, Android Weekly, Kotlin, Plaid, TensorFlow Lite, Permission, Text, Style, Cutouts, Redux, LeakCanary, Tools, Resources, Modularizing, Rendering Engine]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #321
August 5th, 2018.
[Android Weekly Issue #321](https://androidweekly.net/issues/issue-321)
本期内容包括: 开源项目Plaid的改版; 使用TensorFlow Lite来做一个自定义的机器学习的例子; 危险的权限是怎么不小心出现在应用中的; 控件文字样式定义的种种机制; 多平台应用Droidcon NYC的构建;
Kotlin标准库中一些隐藏的好东西; 在cutouts的设备上的应用的显示; 基于RxJava的Redux实现; LeakCanary的新版1.6发布了, 有什么新功能; 一些对Android开发者很有用的在线工具; 如何模块化Android应用; 一个基于物理的渲染引擎.
<!-- more -->

# ARTICLES & TUTORIALS
## [Restitching Plaid](https://medium.com/@crafty/restitching-plaid-9ca5588d3b0a)
[plaid](https://github.com/nickbutcher/plaid)是一个展示material design实现的开源项目.

它是一个2014年的项目, 目前有几个问题(tech debts): API改版了; 没有用architecture component, 也没有什么架构设计; 没有用上一些后来才出现的开源库; 虽然用了一些Kotlin但是大部分还是Java.

所以作者想要对Plaid进行一个改版, 主要是使用architecture component和kotlin. 文中提到了一些新版的设计和想法, 目前项目还在活跃开发中.


## [Building a Custom Machine Learning Model on Android with TensorFlow Lite](https://riggaroo.co.za/building-a-custom-machine-learning-model-on-android-with-tensorflow-lite/)
[ML Kit](https://developers.google.com/ml-kit/)是Firebase提供的一个工具集, 提供了人脸检测, 二维码扫描, 文字识别, 地标检测和图像标记功能.

但是如果你想要一些特别的使用情形, 你就需要使用[TensorFlow Lite](https://www.tensorflow.org/mobile/tflite/).

本文用具体的例子介绍如何使用TensorFlow Lite做一个简单的图片检测.

## [How dangerous permissions sneak into apps](https://jeroenmols.com/blog/2018/08/02/phonestatepermission/)
背景: 作者他们的应用发布了新版以后, 有一个用户抱怨为什么请求了获取device id和电话号码的权限, 却没有任何解释.

开发者也不知道为什么, 打开manifest文件, 点击底部的`Merged manifest`按钮就可以看见merge之后的manifest, 果然有.

有一个log文件在: `build/outputs/logs`可以看见merge的每一个东西都是从哪里来的.

这是因为作者他们之前做了一个工作: 把所有的翻译移动到一个单独的module中, 然后并没有指定`targetSdk`, 所以取了默认值1.
直接导致加上了这个权限: [READ_PHONE_STATE](https://developer.android.com/reference/android/Manifest.permission#READ_PHONE_STATE).

为了更好地修复这个错误, 作者选择在顶级的build.gradle文件中这样做:
```
subprojects {
    afterEvaluate { project ->
        if (project.plugins.findPlugin('android') ?: project.plugins.findPlugin('android-library')) {
            android {
                buildToolsVersion Config.buildTools
                compileSdkVersion Config.compileSdk

                defaultConfig {
                    minSdkVersion Config.minSdk
                    targetSdkVersion Config.compileSdk
                }

                compileOptions {
                    sourceCompatibility Config.javaVersion
                    targetCompatibility Config.javaVersion
                }
            }
        }
    }
}
```

为了进一步防止有什么第三方的library也犯了不指定targetSdk而引进这个危险权限的错误: 
```
<uses-permission android:name="android.permission.READ_PHONE_STATE" tools:node="remove"/>
```


## [What’s your text’s appearance?](https://medium.com/google-developers/whats-your-text-s-appearance-f3a1729192d)
这篇文章详细分析了能够改变View及文字样式的各种方式, 它们的作用域, 应用场景以及覆盖的顺序.

覆盖的顺序:
```
Span > Setters > View > Style > Default Style > Theme > TextAppearance
```
最前面的优先级最高.


## [Droidcon NYC App!](https://medium.com/@kpgalligan/droidcon-nyc-app-da868bdef387)
Droidcon的app: [DroidconKotlin](https://github.com/touchlab/DroidconKotlin/).
这个应用有Android版和iOS版, UI是分别在它们各自的native平台编写的, Android用Kotlin, iOS用Swift. 逻辑和大部分的架构是共享的.

这个应用用到了: [knarch.db](https://github.com/touchlab/knarch.db), [sqldelight](https://github.com/square/sqldelight)和[knarch.threads](https://github.com/touchlab/knarch.threads), 是一个多平台的流式架构.

本文介绍了他们所依赖的一些第三方库, 以及一些基本的想法.


## [Hidden Gems In Kotlin StdLib](https://tech.okcupid.com/hidden-gems-in-kotlin-stdlib/)
一些Kotlin语言中不易被发现的好东西. 分为两类: 
* Included methods
* Language features

### Included methods
首先是关于String, 在Java时代, 我们经常需要借助于[Apache的StringUtils类](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html), 但是在Kotlin中, 这些方法都被内置了:
```
val blank = "   ".isBlank() // Also: CharSequence?.isNullOrBlank
val first = "Adam.McNeilly".substringBefore('.') // "Adam"
val last = "Adam.McNeilly".substringAfter('.') // "McNeilly"
val withSpaces = "1".padStart(2) // " 1"
val endSpaces = "1".padEnd(3, '0') // "100"
val dropStart = "Adam".drop(2) // "am"
val dropEnd = "Adam".dropLast(2) // "Ad"
```
这样我们就不用借助于静态辅助类了.
还有一些神奇的方法:
```
"A\nB\nC".lines() // [A, B, C]

"One.Two.Three".substringAfterLast('.') // "Three"
"One.Two.Three".substringBeforeLast('.') // "One.Two"

"ABCD".zipWithNext() // [(A, B), (B, C), (C, D)]

val nullableString: String? = null
nullableString.orEmpty() // Returns ""
```
其中后两个方法还可以作用于集合上.

Kotlin中的方法很注重对称性, 比如:
```
substringBefore()
substringAfter()

isEmpty()
isNotEmpty()

padStart()
padEnd()

drop()
dropLast()

trimStart()
trimEnd()
```

关于集合, 以前在Java中我们有`Collections`类来提供一些有用的方法. 现在可以直接在集合对象上进行这些操作:
```
myList.sort()
myList.max()
myList.min()
myList.shuffle()
myList.reverse()
myList.swap(1, 2)
```
关于循环遍历查找, 在Kotlin中我们可以这样做:
```
fun getAdam(people: List<Person>): Person? {
    return people.firstOrNull { it.name == "Adam" }
}
```
关于集合, 还有很多有用的操作, 同样也是具有对称性的:
```
myList.filter { }
myList.filterNot { }
myList.filterIsInstance()
myList.filterNotNull { }

myList.first { } // Also: indexOfFirst { }
myList.firstOrNull { }
myList.last { } // Also: indexOfLast { }
myList.lastOrNull { }
myList.single { }
myList.singleOrNull { }

myList.any { }
myList.none { }
myList.all { }

myList.partition { } // Pair<List<T>, List<T>>
```
建议有空可以看看这个[文档](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html), 会发现很多没见过但是很有用的方法.

### Language features
这一部分是关于那些你在Kotlin中能做却在Java中不能做的事情.

* [高阶函数, lambda和匿名函数](https://kotlinlang.org/docs/reference/lambdas.html#lambda-expressions-and-anonymous-functions).
* [lambda作为最后一个参数](https://kotlinlang.org/docs/reference/lambdas.html#passing-a-lambda-to-the-last-parameter).
* [分解声明](https://kotlinlang.org/docs/reference/multi-declarations.html).


## [Supporting display cutouts on edge-to-edge screens](https://android-developers.googleblog.com/2018/07/supporting-display-cutouts-on-edge-to.html)
Android新出了一些cutouts设备.

(笔者注: 这类设备往往是正面全屏, 实现了更大的屏幕, 同时在顶部留出一小块区域来放置前置的摄像头. 这块突出就叫cutout, 其所占领的这个横条区域就叫cutout区域.)

官方在Android 9.0 (API level 28)以上支持display cutouts. 一些Android 8.1 (API level 27)或更早的机器也可能选择性地提供了special mode.

这里有一些官网的[指导](https://developer.android.com/guide/topics/display-cutout/#best_practices_for_display_cutout_support).

在cutout区域绘制应用内容可以更好地提升用户体验, 尤其是像照片, 视频, 地图, 游戏这类应用.

可以通过这个[layoutInDisplayCutoutMode](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#layoutInDisplayCutoutMode)你的内容相对于cutout如何显示.

为了更好地在各个API兼容支持cutout, AndroidX中增加了[DisplayCutoutCompat](https://developer.android.com/reference/androidx/core/view/DisplayCutoutCompat).


## [RxRedux](https://github.com/freeletics/RxRedux)
基于RxJava的Redux实现.

## [When lambdas and strong typing collide](https://medium.com/victoriagonda/when-lambdas-and-strong-typing-collide-18ba065631f1)
作者在开发中碰到了一个编译问题, 后来发现当有多个同名函数接受不同的lambda时, 需要明确指定类型, 不能省略.


## [LeakCanary 1.6](https://medium.com/square-corner-blog/leakcanary-1-6-91fad513b5cf)
[LeakCanary](https://github.com/square/leakcanary)的新版本发布了. 一些新特性:
* 造成leak的可能原因会被红色波浪下划线标注.
* leak分析现在在一个foreground service运行, 以支持Android O.
* 可以在每个UI测试的末尾都运行leak测试, 如果有泄露则测试失败. 具体做法见[这里](https://github.com/square/leakcanary/wiki/Customizing-LeakCanary#running-leakcanary-in-instrumentation-tests). 
* 可以向server报告leak traces, 就像报告crash一样, 见[这里](https://github.com/square/leakcanary/wiki/Customizing-LeakCanary#uploading-to-a-server).


## [Awesome List Of Online Tools For Android Developers](https://medium.com/@naveentp/awesome-list-of-online-tools-for-android-developers-f40af8f46299)
一些有用的在线工具.
### 设计类
* [Figma](https://www.figma.com/)
* [InvisionApp](https://www.invisionapp.com/)
* [Zeplin](https://zeplin.io/)
* [Mockflow](https://mockflow.com/)
* [Draw.io](https://www.draw.io/)

### 开发类
* [Android Starters](http://androidstarters.com/): 选一个架构, 然后就可以生成一个Android项目.
* [Material palette](https://www.materialpalette.com/): 生成色彩方案.
* [AndroidAssetStudio](http://romannurik.github.io/AndroidAssetStudio/): 生成icon.
* [Android SDK Search](https://chrome.google.com/webstore/detail/android-sdk-search/hgcbffeicehlpmgmnhnkjbjoldkfhoin): Chrome插件, 可以搜索Android sdk.
* [Gradle, Please](http://gradleplease.appspot.com/): 寻找gradle依赖.
* [Kotlin extensions](http://kotlinextensions.com/): 最常用的Kotlin extensions.
* [JsonStub](http://jsonstub.com/): 一个假的JSON REST API.


### 测试和产品类
* [APK method count](http://inloop.github.io/apk-method-count/): 计算方法数.
* [Appetize](https://appetize.io/): 在浏览器中运行native app.
* [Appstore screenshot generator](https://www.appstorescreenshot.com/).
* [App Launch Pad](https://theapplaunchpad.com/mockup-generator/).

### 辅助效率类
* [Android arsenal](https://android-arsenal.com/).
* [Mindorks App Store](https://mindorks.com/android/store).
* [Octotree](https://chrome.google.com/webstore/detail/octotree/bkhaagjahfmjljalopjnoealnfndnagc): Chrome插件, 可以显示github项目的树形结构.
* [RegExr](https://regexr.com/): 正则工具, 类似的还有: [RegEx101](https://regex101.com/)和[RegExtester](https://www.regextester.com/).


## [Modularizing Android Applications](https://medium.com/@hitherejoe/modularizing-android-applications-9e2d18f244a0)
很多时候, 一个module的程序似乎就可以解决问题.

但是如果我们想要做Google提供的一些features, 比如Instant apps, app bundles, 或者我们的程序规模增长了, 我们想要一个更加清晰的层次, 模块化程序将帮助我们实现上面所有提到的东西.

后面详细介绍了应该如何模块化程序.

## [Filament](https://google.github.io/filament/Filament.md.html)
Filament是一个基于物理的渲染引擎, 本篇文章提供了很多相关的数学知识.

# LIBRARIES & CODE
## [CrunchyCalendar](https://github.com/CleverPumpkin/CrunchyCalendar)
一个material的calendar, 支持无限滚动, 范围选择等.
## [Language-Switcher-Tile](https://github.com/AzimoLabs/Language-Switcher-Tile)
一个快速改变设备语言的插件.
## [EmojiSlider](https://github.com/bernaferrari/EmojiSlider)
一个带有emoji表情的滑动条控件, 可高度定制.
## [Filament](https://github.com/google/filament)
一个基于物理的渲染引擎.