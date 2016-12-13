---
title: Android Weekly Notes Issue 233
date: 2016-12-01 18:23:40
tags: [Android, Android Weekly, RxJava, Mockito, Terminal, Android 7.1, Nougat, App Shortcuts, Custom View, Firebase, Remote Config, APK analyzer, Cache, MVP, push notification, transient, Java]
categories: [Android, Android Weekly, Java]
---

# Android Weekly Issue #233
November 27th, 2016
[Android Weekly Issue #233](http://androidweekly.net/issues/issue-233)
本期内容包括: 用Mockito做RxJava的单元测试; Android开发中的命令行使用; Android 7.1的App Shortcuts; 自定义View的绘制; 用Firebase的Remote Config进行feature逐步分发; APK分析工具的使用, APK瘦身讨论; RxJava处理网络请求和缓存; presenter的设计; 用Firebase发送push notification; transient关键字的使用等.

<!-- more -->

# ARTICLES & TUTORIALS
## [Testing asynchronous RxJava code using Mockito](https://medium.com/@fabioCollini/testing-asynchronous-rxjava-code-using-mockito-8ad831a16877#.yhndxn3y1)
这篇文章讲了如何用Mockito给RxJava的异步请求代码写单元测试.
内容包括了:
- 如何设置Mockito的默认返回值. (通过自定义的`MockitoConfiguration`类).
- 如何把异步变为同步测试. (1.用`blockingGet()`; 2.在RxJava2中, 可以使用`TestObserver`的`awaitTerminalEvent()`).
- [AssertJ](http://joel-costigliola.github.io/assertj/)的使用.
- 测试异步代码. 使用Rule来替换原来的scheduler.
- `flatMap()`, `concatMap()`, `concatMapEager()`操作符的使用.
- 测试Timeout.
- 测试异常和retry逻辑.


好用的工具: [AssertJ](http://joel-costigliola.github.io/assertj/)
用来更方便地写Java测试中的assert语句.

## [Mastering the Terminal side of Android development](https://medium.com/@cesarmcferreira/mastering-the-terminal-side-of-android-development-e7520466c521#.5pjzgdn2s)
作者分享了在Android开发中他是如何使用命令行的.

使用更好的命令行程序: [iTerm2](http://www.iterm2.com/).
它有很多有用的[features](https://www.iterm2.com/features.html), 比如分屏, 自定义颜色, 粘贴历史等.

**on-my-zsh**: 

[on-my-zsh](https://github.com/robbyrussell/oh-my-zsh)内置了一个[git plugin](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugin:git), 提供了很多aliases和功能.

[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)会在你输入的时候根据历史提供建议.

你可以用Ctrl + R在命令历史中进行逆向智能搜索(Reverse intelligent search). 你开始输入这个命令, 命令行会在历史中寻找并自动补全. 你可以按Enter来执行这个命令, 或者左右箭头来编辑命令, 或者继续按Ctrl + R在其他可能的命令中寻找.

**dryrun**

如果你在github上看到一个程序, 想要运行一下看看, 你不必再把它下载下来, 导入Android Studio了.

你只需要用[dryrun](https://github.com/cesarferreira/dryrun), 一句命令就可以:
```
dryrun REMOTE_GIT_URL
```

**Build faster, build offline**

在build的时候使用--offline可以让所有依赖都使用缓存版本, 不再进行网络请求, 从而加快执行速度.
```
./gradlew assembleDevelopDebug --offline
./gradlew test --offline
```
在Android Studio中也可以进行设置.
在`Settings -> Build, Execution, Deployment -> Build tools -> Gradle`中勾选`Offline work`即可.


**alfi**
[alfi](https://github.com/cesarferreira/alfi)是一个工具, 装了这个工具之后, 用一行命令就可以查到第三方库的依赖语句, 然后你就可以把它拷贝粘贴到`build.gradle`中去了.

**gradle tasks shortcuts**
gradle的task有缩写版的, 比如:
- iDD for installDevelopmentDebug
- aDD for assembleDevelopmentDebug
- cC for connectedCheck

**Android Rocket Launcher**
[Android Rocket Launcher](https://github.com/cesarferreira/android-rocket-launcher)增加新的tasks, 在命令行启动应用.

**直接在console输出单元测试结果**:
```
android {
  ...
  testOptions.unitTests.all {
    testLogging {
      events 'passed', 'skipped', 'failed', 'standardOut', 'standardError'
      outputs.upToDateWhen { false }
      showStandardStreams = true
    }
  }
}
```

这个工具[pidcat](https://github.com/JakeWharton/pidcat)可以指定包名显示log.

## [Exploring Android Nougat 7.1 App Shortcuts](https://www.novoda.com/blog/exploring-android-nougat-7-1-app-shortcuts/)
这篇文章讲Android 7.1推出的App Shortcuts如何实现.

## [The Quirks of Supporting SDK 25](http://www.zdominguez.com/2016/11/the-quirks-of-supporting-sdk-25.html)
作者分享了她在适配Nougat, API 25时学到的东西, 包括更换SDK版本, 圆形的启动icon, 还有app shortcuts. (根据文中的图标, 这个app居然是domain).

## [Android: draw a custom view](https://medium.com/@romandanylyk96/android-draw-a-custom-view-ef79fe2ff54b#.i4ipiz2u7)
作者自定义了一个ViewPager的page indicator: [PageIndicatorView](https://github.com/romandanylyk/PageIndicatorView).

这篇文章讲述了如何自定义View, 首先是View的生命周期, 然后是具体如何实现, 如何避免一些常见的错误, 最后是如何添加View的动画.
![view-lifecycle](/images/view-lifecycle.png)

**各个生命周期中应该干的事情**:
- 构造函数中: 解析自定义属性.
- `onAttachedToWindow()`中: 可以发现同一布局中相关的其他View, 其id是上一步通过自定义属性传入的.
- `onMeasure()`: 自定义View尺寸相关, 当覆盖这个方法时, 最后要调用`setMeasuredDimension(int width, int height)`.
- `onLayout()`: 一般这个方法是给ViewGroup的child指定位置和尺寸的, 对于自定义View来说, 没有child就没有必要覆盖这个方法.
- `onDraw()`: 这里是画东西的地方. 用canvas和Paint结合绘制. 需要注意的是`onDraw()`会被多次调用, 当你有一些变化, 滚动滑动等, 都会重绘, 所以这个方法中不要创建新对象. 


**View更新**
有两个方法可以让View重绘:
- `invalidate()`: 只是重新绘制, 调用`onDraw()`方法.
- `requestLayout()`: 将会从`onMeasure()`开始, 可能会改变尺寸, 然后根据新尺寸重新绘制.

**Animation**
自定义View的动画是一帧帧进行的, 这就意味着你每一步都要调用`invalidate()`来画它.

在自定义View中你的动画好助手是`ValueAnimator`, 它可以让你动画任何值.


## [How to Stage Rollout Features using Firebase Remote Config](https://riggaroo.co.za/stage-rollout-features-firebase-remote-config-ios-android/)
[Staged Rollout](https://support.google.com/googleplay/android-developer/answer/6346149?hl=en)是Google Play Store的一个feature. 让你可以慢慢地把新版App发布给一部分用户, 并逐渐增大比例. 使用Firebase Remote Config, 我们可以做的更多,  我们可以控制某个feature的发布.

## [Making the most of the APK analyzer](https://medium.com/google-developers/making-the-most-of-the-apk-analyzer-c066cb871ea2#.36ccm5y0c)
Android Studio中Build菜单有一项是`Analyze APK...`, 这是一个很有用的功能.

`Raw File Size`是apk在磁盘上的大小.
`Download size`是估计下载你的应用所需要的数据流量大小, 考虑到了Play Store的压缩.

文件和文件夹是按照大小降序排列的. 这对于Apk瘦身来说很有用, 很容易发现最占地方的原因.

比如作者发现了一些png很占地方, 于是就用[PSD support in the Vector Asset import tool](https://developer.android.com/studio/write/vector-asset-studio.html)把它们转成了`VectorDrawable`, 后向兼容用`VectorDrawableCompat`.

有一些没有压缩的WAV可以转成OGG. 

在lib/里面, 发现它们要支持的三个ABI: x86, armeabi-v7a, armeabi, 解决的办法就是利用[apk拆分](https://developer.android.com/studio/build/configure-apk-splits.html), 针对每一个ABI有一个不同的版本.

还有一个优化是把`android:extractNativeLibs` 属性设置为false, 这样系统就不会把.so文件在安装的时候从apk中拷贝到文件系统了. 这样应用的增量更新也会小一点.

这个功能有一个"Compare with"按钮, 利用它你可以比较两个apk的改变.

可以通过查看DEX文件来查看方法数限制 (Referenced Methods), 类混淆等问题.

## [`Rxify` : The Anti Cache-then-Network OR Network-then-Cache Problem](http://www.andevcon.com/news/rxify-the-anti-cache-then-network-or-network-then-cache-problem)
用RxJava处理网络请求和缓存.
- 如果先使用Cache, 没有缓存的时候再进行网络请求. -> 用`.concatWith()`和`.take(1)`.
- 如果优先取网络最新数据, 没网的时候才用缓存数据. -> `.onErrorReturn()`.

## [Your presenters don’t need all those lifecycle events](https://medium.com/@anupcowkur/your-presenters-dont-need-all-those-lifecycle-events-721f500eeef4#.f7nupw3jo)
作者认为在Presenter中放入太多生命周期的方法不太好, 他觉得最基本的只需要这两个方法:
```java
public interface Presenter {
  void onViewAttached(MVPView view); 
  void onViewDetached();
}
```
当然当你需要更多的时候可以加入更多, 但是我们不应该每个生命周期方法都加进去.

## [How to send notifications using Android Firebase](http://www.survivingwithandroid.com/2016/09/android-firebase-push-notification.html)
使用Firebase Messaging如何发送push notification.

## [RxRecipes: Wrap your way to Rx](https://hackernoon.com/rxrecipes-wrap-your-way-to-rx-fd40eb5254b6#.hbtcjp4rm)
使用`.fromCallable()`来把一个同步方法包装成一个Observable. 

并比较了和`.just()`的区别. (`.just()`发射的东西在创建的时候就确定了, 而`.fromCallable()`是在subscribe的时候确定的.)


## [Diving deeper into the Java transient modifier](https://medium.com/google-developer-experts/diving-deeper-into-the-java-transient-modifier-3b16eff68f42#.8pbk9i6fm)
`transient`修饰符加在字段上时, 在对象被序列化的时候, 这个字段将被排除在外, 反序列化时这个字段将被初始化一个默认值.

可能的使用场景: 
- 实现了Serializable的User对象中的password字段.
- 一个Serializable的类中的某个字段是通过其他字段推导或派生出来的, 这些派生的字段没有必要被序列化, 于是把它们标记为`transient`.

注意transient和static是不能并存的, 因为static默认是transient的.

# LIBRARIES & CODE
## [Tinker](https://github.com/Tencent/tinker)
腾讯的热补丁(hot-fix)解决方案, 支持不重新安装app的dex, library和资源更新.

## [Android-Debug-Database](https://github.com/amitshekhariitbhu/Android-Debug-Database)
在浏览器里看应用的数据库和shared preferences.

## [blurkit-android](https://github.com/wonderkiln/blurkit-android)
实时模糊布局. 像iOS一样.

