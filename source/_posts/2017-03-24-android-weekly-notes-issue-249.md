---
title: Android Weekly Notes Issue 249
date: 2017-03-24 17:13:37
tags: [Android, Android Weekly, ConstraintLayout, ViewSwitcher, Transition, Kotlin, Coroutines, RxJava, SDK, Libraries, OpenGL, JBox2D, OkHttp, Retrofit, Network, Etag, If-Modified-Since, ClassyShark, Fingerprint, MVI, TensorFlow, Machine Learning]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #249
March 19th, 2017
[Android Weekly Issue #249](http://androidweekly.net/issues/issue-249)

本期内容包括: 一个设计的实现Demo讨论; Kotlin的Coroutines可能还是没有RxJava好用; 在构建SDK/Libraries时需要注意的事项; 如何用OpenGL和JBox2D实现一个好看的多气泡选择器效果; 
网络请求中Etag, If-Modified-Since的工作原理和用OkHttp的客户端实现; 用ClassyShark导出类型分析; 指纹认证实现代码; MVI模式对调试带来的好处; 用TensorFlow做一个图像识别处理器.

<!-- more -->


# ARTICLES & TUTORIALS
## [From design to Android](http://saulmm.github.io/from-design-to-android-part1)
作者想建立一个项目, 把从[Dribbble](https://dribbble.com/)和[MaterialUp](https://material.uplabs.com/)上看到的一些设计实现出来, 再讲解一些实现细节和UI/UX的tips等.

本文是此系列文章的第一篇, 选择的设计是[preferred-date-and-time](https://material.uplabs.com/posts/preferred-date-and-time), 实现的demo在这里:[from_design_to_android_part1](https://github.com/saulmm/from_design_to_android_part1).

实现中涉及到的点: Bottom Sheets; `ConstraintLayout`和其中的链式约束[Chains](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html#Chains); [ViewSwitcher](https://developer.android.com/reference/android/widget/ViewSwitcher.html); Databinding; Scene和Transition.


## [Why Im Skeptical about Kotlin Coroutines](https://www.philosophicalhacker.com/post/why-im-skeptical-about-kotlin-coroutines-for-android-development/)
Kotlin为了处理异步发布了Coroutines特性, 本文作者对Coroutines持怀疑态度, 认为RxJava的方式更好.

首先, Observables为我们要处理的问题建立了很好的模型, 但coroutines并没有起到这样的作用. (coroutines只是把异步的操作写成了看起来顺序的样子, 同时不阻塞主线程.)

其次, Observables让我们在同步和异步数据上都可以进行同等的抽象, 而coroutines的同步处理和异步处理明显不同.

最后, Observables让我们在更高的抽象层中工作, 比如对元素进行遍历处理的`.map()`.

当然, 本文并不是说Coroutines一无是处, 它肯定是有自己的用处的, 但是作者只是持怀疑态度, 觉得它的地位远不及RxJava.



## [Things I wish I knew when I started building Android SDK/Libraries](https://android.jlelse.eu/things-i-wish-i-knew-when-i-started-building-android-sdk-libraries-dba1a524d619#.tc5qkaglp)
当你遇到一个问题并且解决了, 有时候你会把解决方案作为一个库发布出去.

作者发布了一些库[Android Libraries](https://github.com/nisrulz/android-tips-tricks#extra--android-libraries-built-by-me), 他提出了一些基本的问题和几个应该注意的点.

### 为什么要创建这个库呢?
如果已经有现成的解决方案, 尝试使用已有的方案, 或者给已有的库提Pull Request. 如果没有解决方案, 好吧, 那创建自己的库吧.

### 你的artifacts可选的类型是什么? 
- Library Project: 直接项目引用.
- JAR: 包含了Java class文件和metadata.
- AAR: 除了Java class文件, 还包含了Android的资源和manifest.

### 你的库放在哪里?
- 本地.
- 私有的代码库.
- 公有的代码库: Maven Central, Jcenter or JitPack.

(每一种方式都有教程链接).

上面三个基本问题说完了, 在建立这个库的时候还有一些注意事项:

### 避免多个参数
参数最好不要多于三个, 可以用setter或者Builder模式来解决.

### 容易使用
- 直观: 任何发生在库里的行为, 最好有一些反馈, 比如打印出logs或者显示在UI上.
- 一致性: 遵从[semantic versioning](http://semver.org/).
- 容易使用, 不容易滥用. 最好一眼能看出它的用途. public的方法应该有足够的验证, 确保用户不会滥用. 当不存在依赖关系时, 提供默认值并处理场景.


### 最小化权限
尽量少地要求权限, 可以发送Intent让更专业的应用帮你做一些处理然后返回结果. 根据权限的获取情况来使能你的feature, 不要仅因为没有权限就crash.

你也可以提供一些不需要权限的fallback的实现, 让库的使用者去获取权限.

### 最小化要求
有时候我们需要设备具有某项功能, 比如蓝牙.
这时候就需要在manifest中写`uses-feature`.

如果我们在库中这样写, 它会被merge进应用的manifest, 在Play Store上, 整个应用对无蓝牙的设备都变为不显示. 这样只是引入了一个库, 却失去了一部分用户, 这肯定不是我们所希望看到的.

解决方案就是: 不要写在manifest里, 换为在代码中动态检查. 对于不支持的情况, 库可以关掉这个功能, 提供fallback的实现.


### 支持不同的版本
如果你有一个特定版本才支持的功能, 应该做版本检查, 然后对于更低的版本关掉它.


### Production版本不要打log
### 不要悄悄crash, 另外fail fast
遇到崩溃时应该总是输出错误信息.
如果你不想在production输出任何log, 你至少应该提供flag, 让初始化的时候可以使能它.

如果你的库遇到异常, 应该立即失败, 想开发者输出Exception, 而不是卡在那里. 要避免写出会阻塞主线程的代码.

### 优雅地处理错误
当你的库出错的时候, 尽量做检查, 使得代码不会让整个应用崩溃, 而是只有你的库提供的功能被关闭了.

### 捕获特定的exceptions
### 处理不良的网络连接
如果你的库中有网络请求, 请处理网络连接不良的情况.

如果有可能, 批处理你的网络请求, 这会节约很多电量. 看[这里](https://developer.android.com/training/efficient-downloads/efficient-network-access.html).

使用[FlatBuffers](https://google.github.io/flatbuffers/)而不是json或xml, 来减小网络请求的数据量.

更多的网络优化看这里: [Reducing Network Battery Drain](https://developer.android.com/topic/performance/power/network/index.html).


### 尽量不要依赖很大的库
主要是因为方法数的限制.

### 不要依赖你不需要的库
除了不依赖没有用到的库, 还可以把添加依赖的选择权留给你的用户.

让用户来选择性地添加你依赖的库, 如果他选择不添加, 你的相关feature可以关闭.

可以这样实现:
```java
private boolean hasOKHttpOnClasspath() {
   try {
       Class.forName("com.squareup.okhttp3.OkHttpClient");
       return true;
   } catch (ClassNotFoundException ex) {
       ex.printStackTrace();
   }
   return false;
}
```

而你添加的时候可以这样:
```
dependencies {
   // for gradle version 2.12 and below
   provided 'com.squareup.okhttp3:okhttp:3.6.0'
   // or for gradle version 2.12+
   compileOnly 'com.squareup.okhttp3:okhttp:3.6.0'
}
```

但是这种只能用于纯Java的依赖, 如果是aar就不行.


### 不要拖慢启动时间
在应用启动初始化你的库时, 不要花太多时间.

两种解决方案: 一种是新启一个线程来做初始化; 另一种是到使用之前才进行初始化.


### 删除功能的时候要优雅
升级版本的时候, 不要删除public的方法.
可以把方法标记为`@Deprecated`, 然后在未来的版本中慢慢删除它.


### 让你的代码可测试
使用mock来测试你的代码, 在代码中国避免final的类和static的方法.

写代码的时候public的API用接口, 这样更容易更换实现, 更好测试.


### 文档记录所有的事
包括如何使用你的库, 库中每一个feature都是什么.

- Repo根目录有一个Readme.
- 所有的public方法应该有javadoc注释. 说明目的, 参数, 返回值.
- 有一个sample app, 展示如何使用你的库.
- 在你的release界面, 确保有一个详尽的change log.


### 提供一个最简单的sample
越简单越容易让人明白.

### 考虑加一个Licence
### 收集反馈


## [How to Create a Bubble Selection Animation on Android](https://medium.com/@igalata13/how-to-create-a-bubble-selection-animation-on-android-627044da4854#.1ncs9qy84)
作者他们想要在Android上实现Apple music中的选择气泡效果.

这种动画效果用于让用户在一系列的选择项中做出选择, 气泡自由浮动, 一旦被选中就会变大一点.

作者选择的是用Kotlin, OpenGL和JBox2D(物理引擎).

详细介绍的内容包括: 如何用GLSL写vertex shader和fragment shader; 如何贴图; 用JBox2D来实现气泡的动画(需要自己实现重力); 检测用户手势移动气泡; 发现用户点击的气泡.

项目在Github: [Bubble-Picker
](https://github.com/igalata/Bubble-Picker).


## [Reducing networking footprint with OkHttp, Etags and If-Modified-Since](https://android.jlelse.eu/reducing-your-networking-footprint-with-okhttp-etags-and-if-modified-since-b598b8dd81a1#.260pws449)

### If-Modified-Since和Last-Modified
Header中使用了If-Modified-Since和Last-Modified, 如果两次请求之间内容并未改变, 第二次, server就会返回`304 NOT MODIFIED`, 并且响应不含body.

![if-modified-since and last-modified](/images/if-modified-since-and-last-modified.png)

### Etag和If-None-Match
Etag工作的原理类似, 它实现起来不容易出错, 但是需要server跑一个完整的查询, 并且每次都创建一个hash.

![Etag and if-none-match](/images/Etag-and-If-None-Match.png)

server将会在返回response之前根据响应内容创建一个hash, 然后把它作为Etag header; 客户端在做下一次请求时, 把这个Etag作为If-None-Match header发给server. 客户端在准备下一个响应的时候, 比较新的hash和请求中发来的是否相同, 如果相同, 则返回无内容的`304 NOT MODIFIED`.


### 客户端实现
如果你使用了Retrofit2, 或OkHttp3, 在客户端使能Last-Modified或Etags是很容易的:

```java
private final static int CACHE_SIZE_BYTES = 1024 * 1024 * 2;
...
OkHttpClient.Builder builder = new OkHttpClient().newBuilder();
builder.cache(new Cache(context.getCacheDir(), CACHE_SIZE_BYTES));
...
```

根据server的响应, Last-Modified或Etags将会自动启用.

如果你还想减少处理的时间:

**减少处理**:
在304状态下, Retrofit2和OkHttp3将会假装这个响应和上一次的相同, 所以被缓存的响应会被返回, 你可以检测响应返回值, 如果是304就不做处理.
但是有时候你可能需要每次都重新parse, 这就不用检查`HTTP_NOT_MODIFIED`了, 看你的需要.

注意在Retrofit2中要用raw()中的response来检查, 因为`response.networkResponse().code()`返回的是被缓存了的状态值:

```java
if (response.isSuccessful() &&
    response.raw().networkResponse() != null &&
    response.raw().networkResponse().code() ==
           HttpURLConnection.HTTP_NOT_MODIFIED) {
    // not modified, no need to do anything.
    return;
}
// parse response here
```

### 问题解决
如果你的Etag或Last-Modified不工作.

**检查你的Headers.**

可以用[Stetho](http://facebook.github.io/stetho/)或[OkHttp logging interceptor](https://github.com/square/okhttp/tree/master/okhttp-logging-interceptor)来检查你的headers.

正确的输出是这样:
```
Cache-Control: private, must-revalidate
```
所有请求和响应的Last-Modified和Etag headers都会被显示出来.

**同时使用Etags和Last-Modified.**

OkHttp3会按照严格的顺序检查cache headers:
- 1.如果上一个响应包含Etag, 那么同样的Etag值将会被加在下一个请求的If-Not-Match中.
- 2.如果第一点不满足, 但上一个响应包含Last-Modified, 那么这个值将会被记载下一个请求的If-Modified-Since中.

所以如果同时使用了两种, Etag会屏蔽Last-Modified.


## [Exporting types from Android app using ClassyShark](https://medium.com/@BorisFarber/exporting-types-from-android-app-using-classyshark-7cd2be18cdf7#.5q7p9qoya)
如何导出`ClassyShark`的类型分析.


## [Fingerprint authentication](http://josiassena.com/android-fingerprint-authentication/)
一个指纹认证的代码例子.


## [Reactive Apps with MVI - Part 5](http://hannesdorfmann.com/android/mosby3-mvi-5)
MVI模式系列文章第五篇.

之前在第一篇讲过单向数据流的重要性, 应用状态应该由业务逻辑驱动. 本篇我们将看到这样做带来的好处: 调试程序变得简单了.

我们经常会遇到无法复现的bug, 这往往是因为你只知道崩溃栈, 却不知道用户在出现这个bug之前的实际状态.

当我们用MVI的时候, 我们可以把每次用户激发的intent和model(也即状态)都打出log(用Crashlytics或者其他工具).
这样做以后, 我们从收集到的log中不仅能看到崩溃前最近的状态, 还能看到用户的整个操作历史.

而且用户的应用状态截图都被当做json发送过来, 我们可以拿到任何状态当做我们的初始状态.

这样做以后, 不仅复现崩溃更加容易, 我们还可以利用这些序列化的状态来写一些回归测试.

这样做也是有缺点的: 状态的序列化需要额外花费一些毫秒; 崩溃时传递的数据量增大了; 对用户的敏感信息, 要么忽略, 会导致信息不完整; 要么加密, 那就会需要更多的处理时间.


## [Add some machine learning to your apps, with TensorFlow](http://nilhcem.com/android/custom-tensorflow-classifier)
[TensorFlow](https://github.com/tensorflow/tensorflow)是一个开源的机器学习的库, 由Google开发.

一个简单快速的开始方法就是用TensorFlow来建立一个图像分类器. 相对于使用[Google’s Cloud Vision API](https://cloud.google.com/vision/)来说, 我们可以做一个离线和简化版本, 在Android设备上检测和识别图像中物体.

本篇文章中, 我们会创建一个app, 来识别游戏中的角色.

官方有一个[Demo](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/android), 如果你要跑它, 你需要安装NDK和Bazel.

作者创建了这个[Repo](https://github.com/Nilhcem/tensorflow-classifier-android), 可以直接clone下来build, 更快.

不管你选择哪种方式, 能够运行之后, 这个sample使用了Inception, 一个提前训练好的可以检测1000个物体的model.

之后作者重新下载了一些图片, 对Inception进行了重新训练, 优化, 最后导入新的model并运行, 文中详细记录了过程.


# LIBRARIES & CODE
## [AutoplayVideos](https://github.com/Krupen/AutoplayVideos)
在RecyclerView中显示url对应的Video, 当view出现时自动播放, view不见或部分可见时自动暂停.

## [PreferenceHolder](https://github.com/MarcinMoskala/PreferenceHolder)
一个Kotlin的SharedPreferences的包装库.

## [ActivityStarter](https://github.com/MarcinMoskala/ActivityStarter)
提供了一种简化的方式来启动多参数的Activity.
用注解简化了从Bundle拿参数的过程, 也有相应的存取状态的方法. 可以用于Activity, Fragment, Service和BroadcastReceiver.


## [BlockCanaryEx](https://github.com/lqcandqq13/BlockCanaryEx)
基于[BlockCanary](https://github.com/markzhai/AndroidPerformanceMonitor)的扩展, 用于检测UI阻塞, 打印出了更多的方法信息, 并显示出最耗时的方法.


## [EasySP](https://github.com/WhiteDG/EasySP)
一个简单的SharedPreferences辅助类, 支持流式操作.

