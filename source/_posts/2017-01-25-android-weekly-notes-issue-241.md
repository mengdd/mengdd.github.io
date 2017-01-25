---
title: Android Weekly Notes Issue 241
date: 2017-01-25 09:40:01
tags: [Android, Android Weekly, Master/Detail, APK, MVI, RxJava, Build Time, Testing, RxJava2, Toast]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #241
January 22nd, 2017
[Android Weekly Issue #241](http://androidweekly.net/issues/issue-241)
本期内容包括: 经典导航模式Master/Detail的设计和实现; APK的大小讨论和增量下载大小的预估工具; Model-View-Intent模式的讨论和实现; 分多个modules对build时间的影响; 测试中能够利用的一些Android特有的接缝设计(manifest, build config, resource).


<!-- more -->

# ARTICLES & TUTORIALS
## [Case Study. Master/Detail Pattern Revisited](http://goneremote.io/master-detail-pattern/)
Master/Detail是一种经典的导航流, master屏包含一个list, detail显示某一项的详细信息. [Android Doc](https://developer.android.com/training/implementing-navigation/descendant.html).

作者讲了他适配多种屏幕(包括平板)的设计, 以及简单的实现.

## [Tracking app update sizes](https://medium.com/google-developers/tracking-app-update-sizes-1a1f57634f7b#.v2ecs8u4t)
以前作者有一系列的[文章](https://medium.com/google-developers/smallerapk-part-1-anatomy-of-an-apk-da83c25e7003#.dv2tfqdyq)讲过apk的组成以及如何减少apk的大小.

事实上app的大小可以分下面四种:
- 提交到Google Play的APK文件大小.
- 初始的下载大小.
- 在设备上的安装大小.
- 更新下载大小.

之前的一些文章可能都在讨论如何减少初始的大小, 但是大多数情况用户可能只安装你的应用一次, 之后就只是从Play Store更新, 所以应用的更新大小也是很重要的.

事实上Android Studio(2.2+)改善了打包apk的机制, ([apk packaging](https://android-developers.googleblog.com/2016/11/understanding-apk-packaging-in-android-studio-2-2.html)), 使得每一个build都尽可能地相似, 这样Play Store就能计算出一个较小的delta更新. 另外, Play Store也引入了新的算法, 比如最近的[File-By-File patching](https://android-developers.googleblog.com/2016/12/saving-data-reducing-the-size-of-app-updates-by-65-percent.html), 同样也有效减小了更新的大小.

所以我们要注意的就是不要介入和干扰当前Android Studio和Play Store的这些优化, 比如不要用自定义的ZIP加密设置来自己压缩APK. 也不要用Zopfli来再次压缩APK.

Play Store上会显示应用的下载大小, 如果用户已经安装了, 则显示的是更新大小.

对于开发者来说, 如果能在发布前知道这些信息就更好了, 所以作者他们开源了这个库: [apk-patch-size-estimator](https://github.com/googlesamples/apk-patch-size-estimator)

这是一个命令行的工具, 可以集成到CI里, 也可以手动比较两个apk文件.

这个工具实现了当前Play Store的算法, 可以帮你估计出初始的apk下载大小和更新下载大小.

(注意下载大小和apk文件大小不同因为Play Store可能会做进一步压缩.)

同样, Android Studio中也有一个图形化的 APK Analyzer工具, 可以做apk的比较, 让你看到到底是哪一部分的尺寸增长了.

## [Reactive Apps with Model-View-Intent - Part 2](http://hannesdorfmann.com/android/mosby3-mvi-2)
上一篇文章讨论了一个好的Model层可以解决很多问题. 这篇来介绍`Model-View-Intent`模式.

### Model-View-Intent模式
Model-View-Intent模式是在一个JavaScript的framework `cycle.js`中提出的.

![view-model-intent](/images/view-model-intent.png)

- `intent()`: 这个方法接收用户输入, 然后输出将会作为参数传给`model()`.
- `model()`: 接收`intent()`的输出作为自己的输入, 来操纵Model, 这个方法的输出是一个新的Model(状态变化). 所以它不应该更新一个已经存在的Model. 因为我们想要不可变性. 注意这里是唯一一个允许创建新Model的地方.
- `view()`: 接收`model()`方法返回的model作为输入, 然后将其展示出来.

### 用RxJava连接
我们希望数据流是单向的, 于是我们用了RxJava, 它很适合这种基于事件的编程, 在这里主要是UI事件.

作者之后举了一个实现的例子, 在这个例子中他们的Model层用了ViewState后缀. `SearchInteractor`用来执行搜索, 返回的结果是`Observable<SearchViewState>`.

这个模式中定义的View接口里包含了`render()`方法, 根据传入的状态model显示UI; 这个View接口其实还包含了`intent()`的方法, 返回的是一个`Observable`, UI中用了[RxBinding](https://github.com/JakeWharton/RxBinding).

最后一步就是, 如何将View的intent和业务逻辑联系起来呢? 这里用到了一个额外的组件: `Presenter`.

这个Presenter看起来像这样:
```java
public class SearchPresenter extends MviBasePresenter<SearchView, SearchViewState> {
  private final SearchInteractor searchInteractor;

  @Override protected void bindIntents() {
    Observable<SearchViewState> search =
        intent(SearchView::searchIntent)
            .switchMap(searchInteractor::search) // I have used flatMap() in the video above, but switchMap() makes more sense here
            .observeOn(AndroidSchedulers.mainThread());

    subscribeViewState(search, SearchView::render);
  }
}
```

`MviBasePresenter`是[mosby](https://github.com/sockeqwe/mosby)中的一个类.
这个类做的事情就是当View第一次attach到Prensenter上时, 调用`bindIntent()`方法将来自view的intent绑定到业务逻辑上, 只有第一次会绑定, 当View再次attach时不会发生.

而`subscribeViewState()`方法则处理了定于管理, 避免内存泄露(具体原因见原文).

## [How modularization affects build time of an Android application](https://medium.com/@nikita.kozlov/how-modularisation-affects-build-time-of-an-android-application-43a984ce9968#.hzgv3h6cm)
一个Android应用至少有一个application module, build这个module之后得到一个.apk文件.

application module之间不能相互依赖, 它只能依赖于library, build library module的结果是得到一个.aar(Android Archive Library)文件.

build的过程可以粗略分为5个阶段:
- 1.准备依赖.
- 2.Merge资源和manifest.
- 3.编译. 从annotation processors开始, 把源码编译成字节码.
- 4.后处理. 所有以`transform`开头的gradle tasks都属于这个阶段. 其中最重要的是`transformClassesWithMultidexlist`和`transformClassesWithDex`, 它们生成了.dex文件.
- 5.打包发布. 对library来说是生成.aar, 对application来说是生成.apk.

我们都知道gradle只有在输入变化了的情况下才会重跑task. 而且如果一个module没有变化, 也不会被重新build, 那么就出现了一种假设: 多个module应用的增量build要比单个module的快, 因为只有被改变了的module才会重新编译.

作者想验证这种假设是否正确.
他用的工具就是:
```
./gradle assembleDebug --profile
```

做了一系列实验之后证明这个假设还是有道理的.

实验过程中的一些发现:

1.当应用被拆分为多个modules之后, 改变application module中的代码, build时间会减少; 但是library中的代码, build时间反而会增加. 这是因为library build的时候debug和release的tasks都执行了(并不知道为什么).


当library module被这样添加的时候:
```
dependencies {
    compile project(path: ':app2')
    compile project(path: ':app3')
}
```
不管app当前的build type是什么, app永远依赖的是library的release版本.

这是一个Gradle当前的限制. 参见[Library-Publication](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Library-Publication).

幸运的是, 我们可以改变这一行为:
首先在library中添加:
```
android {
    defaultConfig {
        defaultPublishConfig 'release'
        publishNonDefault true
    }
}
```
让它也可以发布debug版.
在app中依赖的时候:
```
dependencies {
    debugCompile project(path: ':app2', configuration: "debug")
    releaseCompile project(path: ':app2', configuration: "release")

    debugCompile project(path: ':app3', configuration: "debug")
    releaseCompile project(path: ':app3', configuration: "release")
}
```

这样debug和release都只依赖各自对应版本的library了.

2.不管我们改动的是library中的代码还是application中的代码, application module永远都会被重新编译, 所以减小app module的尺寸很有意义.

3.上面这些都是library之间互相独立的情况, 如果library之间还有相互依赖, 那么build时间也会变长.

4.如果应用超出了DEX的方法数限制, 用了multidex, 也会增加build时间, Android 5.0开始使用了一个叫做ART的runtime, 在这方面有一些优化, 可以减少build时间, 所以我们可以在开发的时候设置最小API是21: [Optimize multidex in development builds](https://developer.android.com/studio/build/multidex.html#dev-build).

## [Exploiting Android Seams for Testing and Flexibility](https://www.philosophicalhacker.com/post/exploiting-android-specific-seams-for-testing-and-flexibility/)
如何让Android应用代码可测试? 答案是创建一些接缝. 这篇文章中, 作者将一些Android特有的接缝, 来让我们的应用更加灵活和易测.

### Manifest接缝
使用[Merge rule markers](https://developer.android.com/studio/build/manifest-merge.html#merge_rule_markers)可以方便地更改manifest.

比如在build variant是mock的时候, 由于我们在src/mock/AndroidManifest.xml里这样写:
```xml
<!-- src/mock/AndroidManifest.xml -->
<activity
  android:name=".StubConfigActivity">
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
</activity>
<activity
  android:name=".MainActivity">
  <intent-filter tools:node="remove">
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
</activity>
```
所以在build mock的时候, 启动Activity会被替换成上面这个`StubConfigActivity`.

还有更多的可能值得探索, 比如你可以替换filter的内容, 从而改变默认intent启动的Activity.

### BuildConfig接缝
在gradle中可以根据不同的build variant来定义`BuildConfig`中的变量值. 

默认情况下`BuildConfig`中会包含一些有用的变量比如`DEBUG`和`FLAVOR`.

我们可以创建更多额外的变量:
```
productFlavors {
  mock {
    buildConfigField('Boolean', 'MOCK', "true")
  }
}
```

一个简单的应用case是我们可以定义不同的base url:
```
defaultConfig {
  buildConfigField('String', 'API_BASE', '\"api.awesomecompany.com\"')
}
productFlavors {
  sandbox {
    buildConfigField('String', 'API_BASE', '\"localhost:8080\"')
  }
}
```

### Resource接缝
不同build variants的资源就像manifest一样, 最后会被merged. 但是对于资源我们没有markers可以控制它们如何merge. 

我们可以利用默认的merge行为: [Resource merging](https://developer.android.com/studio/write/add-resources.html#resource_merging).

优先级是这样的:
```
build variant > build type > product flavor > main source set > library dependencies
```

所以我们可以把默认的资源放在main里, 然后在特定的build variant再创建一份覆盖它们.

# LIBRARIES & CODE
## [Reptar](https://github.com/Commit451/Reptar)
RxJava2.x的有用的类的集合.

## [Toasty](https://github.com/GrenderG/Toasty)
前面加了一个icon的Toast, 带背景颜色, 除了内置的error, info, success, warning等几种形式, 还可以自定义.

## [Google-Actions-Java-SDK](https://github.com/frogermcs/Google-Actions-Java-SDK)
非官方的Google Actions Java SDK.



