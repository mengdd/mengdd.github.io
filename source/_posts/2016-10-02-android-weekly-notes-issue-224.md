---
title: Android Weekly Notes Issue 224
date: 2016-10-02 12:17:13
tags: [Android, Android Weekly, pre-launch, Wear, Handler, RxAndroid, Profile, Methods Count, APK, Redux, Reductor, Memory Leak, Annotation, Adapter, Intro Screen]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #224

September 25th, 2016
[Android Weekly Issue #224](http://androidweekly.net/issues/issue-224)

本期内容包括: Google Play的pre-launch报告; Wear的Complications API; Android Handler解析; RxAndroid; 测量性能的库: Pury; 方法数限制; APK内容分析; Redux for Android; 一种view造成的泄露; 注解处理; 更好的Adapter; Intro屏等等.

<!-- more -->

# ARTICLES & TUTORIALS
## Apk的pre-launch报告 [Awesome pre-launch reports for Alpha/Beta APK's](https://medium.com/@AruLNadhaN/awesome-pre-launch-reports-for-alpha-beta-apks-9960ac5c403c#.5qhy3bbqc) 

Google Play team在I/O 2016的时候宣布了很多新features, 其中有一个pre-launch report.

这个report是干什么的呢, 它会报告在一些设备上测试你的应用的时候发现的issues.

要生成这种报告, 你应该在Developer console上enable它. 然后[上传alpha/beta apk](https://support.google.com/googleplay/android-developer/answer/3131213). 上传到beta channel之后, 5-10分钟就会生成报告.

报告主要包括三个部分:
- Crashes
- Screenshots
- Security

官方文档: [pre-launch](https://support.google.com/googleplay/android-developer/answer/7002270#sources)

## [Wear Complications API](https://medium.com/@danybony_/wear-complications-api-16ab65290aa1#.w6lt2q3rx)
在钟表的定义里, complications是指表上除了小时和分钟指示之外其他的东西.

在Android Wear里面我们已经有一些complications的例子, 比如向用户显示计步器, 天气预报, 下一个会议时间等等.

但是之前有一个很大的限制就是每一个小应用都必须实现自己的逻辑来取数据, 比如有两个应用都取了今天的天气预报信息, 将会有两套机制取同样的数据, 这明显是一种浪费.

Android Wear 2.0推出了Complications API解决了这个问题.

通信主要是**Data providers**和**Watch faces**之间的, 前者包含取数据的逻辑, 后者负责显示.

Complications API定义了一些Complications Types, 见[官方文档](https://developer.android.com/wear/preview/features/complications.html#using_complication_types).

作者在他朋友的开源应用里用了新的API: [Memento-Namedays](https://github.com/alexstyl/Memento-Namedays), 这个应用是生日或者日期提醒类的.

首先, 作者用[Wearable Data Layer API](https://developer.android.com/training/wearables/data-layer/index.html)同步了手机和手表的数据. 然后在Wear module里继承`ComplicationProviderService`创建了complication data provider, 这里就提供了`onComplicationActivated`, `onComplicationDeactivated`, `onComplicationUpdate`等回调. 

用户也可以点击Complications, 可以用`setTapAction()`指定点击后要启动的Activity.

可以指定`ComplicationProviderService`的更新频率, 是在manifest里用这个key:
`android.support.wearable.complications.UPDATE_PERIOD_SECONDS`.

更新得太频繁会比较费电. 
需要注意的是这并不是一个常量, 因为系统也会根据手机的状况进行一些调节, 不必要的时候就不需要频繁更新.

本文作者采用的方式是用`ProviderUpdateRequester`. 在manifest里面设置0.

```java
ComponentName providerComponentName = new ComponentName(
    context, 
    MyComplicationProviderService.class
);
ProviderUpdateRequester providerUpdateRequester = new
    ProviderUpdateRequester(context, providerComponentName);
providerUpdateRequester.requestUpdateAll();
```

最后, 这里是官网文档:
[Complications](https://developer.android.com/wear/preview/features/complications.html).

这里是作者PR: [PR](https://github.com/alexstyl/Memento-Namedays/pull/40)

## [Android Handler Internals](https://medium.com/@jagsaund/android-handler-internals-b5d49eba6977#.xuogmm2c0)

首先, 作者举了一个简单的例子, 用两种方法, 用Handler来实现下载图片并显示到ImageView上的过程.

主要是因为网络请求需要在非UI线程, 而View操作需要在UI线程. Handler就用来在这两种线程之间切换调度.

**Handler的组成**
- Handler
- Message
- Message Queue
- Looper

**Handler**

[Handler](https://developer.android.com/reference/android/os/Handler.html)是线程间消息传递的直接接口, 生产者和消费者线程都是通过调用下面的操作和Handler交互:
* creating, inserting, removing Messages from Message Queue.
* processing Messages on the consumer thread.

每一个Handler都是和一个Looper和一个Message Queue关联的. 有两种方法来创建一个Handler:
* 用默认构造器, 将会使用当前线程的Looper.
* 显式地指明要用的Looper.

Handler不能没有Looper, 如果构造时没有指明Looper, 当前线程也没有Looper, 那么将会抛出异常.

因为Handler需要Looper中的消息队列.

一个线程上的多个Handler共享同一个消息队列, 因为它们共享同一个Looper.

**Message**

[Message](https://developer.android.com/reference/android/os/Message.html)是一个包含任意数据的容器, 它包含的数据信息是callback, data bundle和obj/arg1/arg2, 还有三个附加数据what, time和target.

可以调用Handler的`obtainMessage()`方法来创建Message, 这样message是从message pool中取出的, target会自动设置成Handler自己. 所以直接可以在后面调用`sendToTarget()`方法.

Message pool是一个最大尺寸为50的LinkedList. 当消息被处理完之后, 会放回pool, 并且重置所有字段.

当我们使用Handler来`post(Runnable)`的时候, 实际上是隐式地创建一个Message, 它的callback存这个Runnable.


**Message Queue**

[Message Queue](https://developer.android.com/reference/android/os/MessageQueue.html) 是一个无边界的LinkedList, 元素是Message对象. 它按照时间顺序来插入Message, 所以timestamp最小的最先分发. 

MessageQueue中有一个`dispatch barrier`表示当前时间, 当message的timestamp小于当前时间时, 被分发和处理.

Handler提供了一些方法在发message的时候设置不同的时间戳:

`sendMessageDelayed()`: 当前时间 + delay时间.

`sendMessageAtFrontOfQueue()`: 把时间戳设为0, 不建议使用.

`sendMessageAtTime()`.


Handler经常需要和UI交互, 可能会引用Activity, 所以也经常会引起内存泄漏.
作者举了两个例子, 略.

需要注意:
非静态内部类会持有外部类实例引用.
Message会持有Handler引用, 主线程的Looper和MessageQueue在程序运行期间是一直存在的.

建议的是, 内部类用static修饰, 另用WeakReference.


**Debug Tips**
显示Looper中dispatched的Messages:
```java
final Looper looper = getMainLooper();
looper.setMessageLogging(new LogPrinter(Log.DEBUG, "Looper"));
```
显示MessageQueue中和handler相关的pending messages:
```java
handler.dump(new LogPrinter(Log.DEBUG, "Handler"), "");
```

**Looper**

[Looper](https://developer.android.com/reference/android/os/Looper.html) 从消息队列中读取消息, 然后分发给target handler. 每当一个Message穿过了`dispatch barrier`, 它就可以在下一个消息循环中被Looper读.

一个线程只能关联一个Looper. 因为Looper类中有一个静态的ThreadLocal对象保证了只有一个Looper和线程关联, 企图再加一个就会抛出异常.

调用`Looper.quit()`会立即终止Looper, 丢弃所有消息.
而`Looper.quitSafely()`会将已经通过`dispatch barrier`的消息处理了, 只丢弃pending的消息.

Looper是在Thread的`run()`方法里setup的, `Looper.prepare()`会检查是否之前存在一个`Looper`和这个线程关联, 如果有则抛异常, 没有则建立一个新的`Looper`对象, 创建一个新的MessageQueue. 见[代码](https://github.com/android/platform_frameworks_base/blob/e71ecb2c4df15f727f51a0e1b65459f071853e35/core/java/android/os/Looper.java#L83).

现在`Handler`可以接收或者发送消息到`MessageQueue`了. 执行`Looper.loop()`方法将会开始从队列读出消息. 每一个loop迭代都会取出下一个消息.

## [Crunching RxAndroid - Part 10 ](https://medium.com/crunching-rxandroid/crunching-rxandroid-part-10-cc0c33108ee2#.ri2xoc35c) 细细咀嚼RxAndroid 

作者这个是个系列文章, 本文是part 10.

Android的listener很多, 我们可以通过RxJava把listener都变成发射信息的源, 然后我们subscribe.

本文举例讲了`Observable.fromCallable()`和`Observable.fromAsync()`方法的用法.

## [Pury a new way to profile your Android application](https://medium.com/@nikita.kozlov/pury-new-way-to-profile-your-android-application-7e248b5f615e#.a7a9lsexj)

在做任何优化之前我们都应该先定位问题. 首先是收集性能数据, 如果收集到的信息超过了可以接受的阈值, 我们再进一步深究, 找到引起问题的方法或者API.

幸运的是, 有一些工具可以帮我们profiling:

- [Hugo](https://github.com/JakeWharton/hugo) 用`@DebugLog`注解来标记方法, 然后参数, 返回值, 执行时间都会log出来.
- Android Studio toolset. 比如System Trace, 非常准确, 提供了很多信息, 但是需要你花时间来收集和分析数据.
- 后台解决方案, 比如[JMeter](https://jmeter.apache.org/), 它们提供了很多功能, 需要花时间来学习如何使用, 第二就是高并发profile也不是常见的需求.


**Missing tool**

关于我们关心的应用的速度问题, 大多数可以分为两种:
- 特定方法和API的执行时间, 这个可以被Hugo cover.
- 两个事件之间的时间, 这可能是独立的两段代码, 但是在逻辑上关联. Android Studio toolset可以cover这种, 但是你需要花很多时间来做profile.

作者意识到下面的需求没有被满足:
- 开始和结束profiling应该是被两个独立的事件触发的, 这样才可以满足我们灵活性的需求.
- 如果我们想监控performance, 仅仅开始和结束事件是不够的. 有时候我们需要知道这之间发生了什么, 这些阶段信息应该被放在一个报告里, 让我们更容易明白和分享数据.
- 有时候我们需要做重复操作, 比如loading RecyclerView的下一页, 那么一个回合的操作显然是不够的, 我们需要进行多次操作, 然后显示统计数据, 比如平均值, 最小最大值.

基于上面的需求, 作者创建了Pury.

**Introduction to Pury**

Pury是一个profiling的库, 用于测量多个独立事件之间的时间.
事件可以通过注解或者方法调用来触发, 一个scenario的所有事件被放在同一个报告里.


然后作者举了两个例子, 一个用来测量启动时间, 另一个用来测量loading pages.

**Inner structure and limitations**

性能测量是`Profilers`做的, 每一个`Profiler`包含一个list, 里面是`Runs`. 多个`Profilers`可以并行运行, 但是每个`Profiler`中只有一个`Run`是active的. 

**Profiling with Pury**

Pury可以测量多个独立事件之间的时间, 事件可以用注解或者方法调用触发.
基本的注解有: `@StartProfiling`, `@StopProfiling`, `@MethodProfiling`

方法:
```java
Pury.startProfiling();

Pury.stopProfiling();
```

最后作者介绍了一些使用细节. 
项目地址: [Pury](https://github.com/NikitaKozlov/Pury)

## 处理方法数限制问题 [Dealing With the 65K Methods limit on Android](http://bytes.schibsted.com/dealing-65k-methods-limit-android/)

作为Android开发, 你可能会看到过这种信息:
```
Too many field references: 88974; max is 65536.
You may try using –multi-dex option.
```

首先, 为什么会存在65k的方法数限制呢?

Android应用是放在APK文件里的, 这里面包含了可执行的二进制码文件(DEX - Dalvik Executable), 里面包含了让app工作的代码.

DEX规范限制了单个的DEX文件中的方法总数最大为65535, 包括了Android framework方法, library方法, 还有你自己代码中的方法. 如果超过了这个限制你将不得不配置你的app来生成多个DEX文件(multidex configuration). 

但是开启了multidex配置之后有一些随机性的兼容问题, 所以我们在决定开启multidex之前, 首先采取的第一步是减少方法数来避免这个问题.

在我们开始改动之前, 先提出了这些问题:
- 我们有多少方法?
- 这些方法都是从哪里来?
- 主要的方法来源是谁?
- 我们真的需要所有这些方法吗?

在搜寻这些问题的答案的过程中, 我们发现了一些有用的工具和tips:

[MethodsCount.com](http://www.methodscount.com/) 将会告诉你一个库有多少方法, 还提供了每个方法的依赖.

[JakeWharton/dex-method-list utility](https://github.com/JakeWharton/dex-method-list) 可以显示.apk, .aar, .dex, .jar或.class文件中的所有方法引用. 这可以用来发现一个库中到底有多少方法是被你的app使用了.

[mihaip/dex-method-counts](https://github.com/mihaip/dex-method-counts) 这个工具可以按包来输出方法, 计算出一个DEX文件中的方法数然后按包来分组输出. 这有利于我们明白哪些库是方法数的主要来源.

[Gradle build system](https://gradle.org/) 提供了关于项目结构很有价值的信息. 一个有用的task是`dependencies`, 让你看到库的依赖树, 这样你就可以看到重复的依赖, 进而删除它们来减少方法数.

[Classyshark](http://classyshark.com/) 是一个Android可执行文件的浏览器. 用这个工具你可以打开Android的可执行文件(.jar, .class, .apk, .dex, .so, .aar, 和Android XML)来分析它的内容.

[apk-method-count](http://inloop.github.io/apk-method-count/) 这是一个工具, 用来快速地查apk中的方法数, 拖拽apk之后就会得到结果.

## [What's in the APK](http://crushingcode.co/whats-in-the-apk/) APK中有什么
APK: Android application package 是Android系统的一种文件格式, 实际上是一种压缩文件, 如果把.apk重命名为.zip, 就可以取出其内容.

但是此时我们直接在文本编辑器打开AndroidManifest.xml的时候看到的全是机器码.

当然是有工具来帮我们分析这些东西的, 这个工具从一开始就有, 那就是aapt, 它是Android Build Tool的一部分.

**aapt - Android Asset Packaging Tool** 这个工具可以用来查看和增删apk中的文件, 打包资源, 研究PNG文件等等.

它的位置在: `<path_to_android_sdk>/build-tools/<build_tool_version_such_as_24.0.2>/aapt`.

aapt能做的事情, 从man可以看出:
- aapt list - Listing contents of a ZIP, JAR or APK file.
- aapt dump - Dumping specific information from an APK file.
- aapt package - Packaging Android resources.
- aapt remove - Removing files from a ZIP, JAR or APK file.
- aapt add - Adding files to a ZIP, JAR or APK file.
- aapt crunch - Crunching PNG files.

用这个工具来分析我们的apk:

输出基本信息:
`aapt dump badging app-debug.apk `

输出声明的权限:
`aapt dump permissions app-debug.apk`

输出配置:
`aapt dump configurations app-debug.apk`

还有其他这些:
```
# Print the resource table from the APK.
aapt dump resources app-debug.apk

# Print the compiled xmls in the given assets.
aapt dump xmltree app-debug.apk

# Print the strings of the given compiled xml assets.
aapt dump xmlstrings app-debug.apk

# List contents of Zip-compatible archive.
aapt list -v -a  app-debug.apk    
```
## [Reductor - Redux for Android](https://yarikx.github.io/Reductor-prologue/)

Redux是一个当前JavaScript中很火的构架模式. Reductor把它的概念借鉴到了Java和Android中.

关于状态管理到底有什么好方法呢, 作者想到了前端开发中的SPA(Single-page application), 和Android应用很像, 有没有什么可借鉴的呢? 答案是有.

[Redux](http://redux.js.org/) 是一个JavaScript应用的可预测的状态容器, 可以用下面三个基本原则来描述:
- 单一的真相来源
- 状态只读
- 变化是纯函数造成的

Redux的灵感来源有[Flux](http://facebook.github.io/flux/)和[Elm Architecture](https://github.com/evancz/elm-architecture-tutorial/).
强烈建议阅读一下它的[文档](http://redux.js.org/docs/introduction/Motivation.html).

[Reductor](https://github.com/Yarikx/reductor)是作者用Java又实现了一次Redux.

作者用了一个Todo app的例子来说明如何使用, 以及它的好处.

作者先写了一个naive的实现, 然后不断地举出它的缺点, 然后改进它.

其中作者用到了[pcollection](https://github.com/hrldcpr/pcollections)来实现persistent/immutable的集合.

最后还把代码改为对测试友好的.

## [Android leak pattern: subscriptions in views](https://medium.com/@pyricau/android-leak-pattern-subscriptions-in-views-18f0860aa74c?swoff=true)

开始作者举了一个例子, 一个自定义View, subscribe了Authenticator单例的username变化事件, 从而更新UI.

```java
public class HeaderView extends FrameLayout {
  private final Authenticator authenticator;

  public HeaderView(Context context, AttributeSet attrs) {...}

  @Override protected void onFinishInflate() {
    final TextView usernameView = (TextView) findViewById(R.id.username);
    authenticator.username().subscribe(new Action1<String>() {
      @Override public void call(String username) {
        usernameView.setText(username);
      }
    });
  }
}
```

但是代码存在一个主要的问题: 我们从来没有unsubscribe. 这样匿名内部类对象就持有外部类对象, 整个view hierarchy就泄露了, 不能被GC.

为了解决这个问题, 在View的`onDetachedFromWindow()`回调里调用`unsubscribe()`.

作者以为这样解决了问题, 但是并没有, 还是检测出了泄露, 并且作者发现View的`onAttachedToWindow()`和`onDetachedFromWindow()`都没有被调用.

作者研究了`onAttachedToWindow()`的调用时机:
- When a view is added to a parent view with a window, onAttachedToWindow() is called immediately, from addView().
- When a view is added to a parent view with no window, onAttachedToWindow() will be called when that parent is attached to a window.

而作者的布局是在Activity的`onCreate()`里面`setContentView()`设置的.
这时候每一个View都收到了`View.onFinishInflate()`回调, 却没有调`View.onAttachedToWindow()`.

`View.onAttachedToWindow()` is called on the first view traversal, sometime after `Activity.onStart()`.

`onStart()`方法是不是每次都会调用呢? 不是的, 如果我们在`onCreate()`里面调用了`finish()`, `onDestroy()`会立即执行, 而不经过其中的其他生命周期回调.

明白了这个原理之后, 作者的改进是把订阅放在了`View.onAttachedToWindow()`里, 这样就不会泄露了. 对称总是好的.

## [Annotation Processing in Android Studio](https://medium.com/@aitorvs/annotation-processing-in-android-studio-7042ccb83024#.khjikdf51) 注解和其处理器
作者用例子说明了如何自定义注解和其处理器, 让被标记的类自动成为Parcelable的.
看了这个有助于理解各种依赖和了解相关的目录结构.

建议使用: [android-apt](https://bitbucket.org/hvisser/android-apt).

[Parcelable](https://developer.android.com/reference/android/os/Parcelable.html).
相关库代码: [aitorvs/auto-parcel](https://github.com/aitorvs/auto-parcel).

## [Writing Better Adapters](https://medium.com/@dpreussler/writing-better-adapters-1b09758407d2#.ngas0y7j1) 写出更好的Adapter
在Android应用中, 经常需要展示List, 那就需要一个Adapter来持有数据.

RecyclerView的基本操作是: 创建一个view, 然后这个ViewHolder显示view数据; 把这个ViewHolder和adapter持有的数据绑定, 通常是一个model classes的list.

当数据类型只有一种时, 实现很简单, 不容易出错. 但是当要显示的数据有很多种时, 就变得复杂起来.

首先你需要覆写:
```
override fun getItemViewType(position: Int) : Int
```
默认是返回0, 实现以后把不同的type转换为不同的整型值.

然后你需要覆写:
```
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder
```

为每一种type创建一个ViewHolder.

第三步是:
```
override fun onBindViewHolder(holder: ViewHolder, position: Int): Any
```

这里没有type参数.


**The Uglyness**
好像看起来没有什么问题?
让我们重新看`getItemViewType()`这个方法. 系统需要给每一个position都对应一个type, 所以你可能会写出这样的代码:
```java
if (things.get(position) is Duck) {
    return TYPE_DUCK
} else if (things.get(position) is Mouse) {
    return TYPE_MOUSE
}
```

这很丑不是吗?

如果你的ViewHolder没有一个共同的基类, 在binding的时候也是这么丑:
```
override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
    val thing = things.get(position)
    if (thing is Animal) {
        (holder as AnimalViewHolder).bind(thing as Animal)
    } else if (thing is Car) {
        (holder as CarViewHolder).bind(thing as Car)
    }
...
}
```

很多的instance-of和强制类型转换, 它们都是code smells. 违反了很多软件设计的原则, 并且当我们想要新添一种类型时, 需要改动很多方法. 我们的目标是添加新类型的时候不用更改Adapter之前的代码.
开闭原则: Open for Extension, Closed for Modification.

**Let's Fix It**
用一个map来查询? 不好.
把type放在model里? 不好.

解决问题的一种办法是: 加入ViewModel, 作为中间层.

但是如果你不想创建很多的ViewModel类, 还有其他的办法: [Visitor模式](https://en.wikipedia.org/wiki/Visitor_pattern)

```
interface Visitable {
    fun type(typeFactory: TypeFactory) : Int
}

interface Animal : Visitable
interface Car : Visitable

class Mouse: Animal {
    override fun type(typeFactory: TypeFactory) 
        = typeFactory.type(this)
}
```

工厂:
```
interface TypeFactory {
    fun type(duck: Duck): Int
    fun type(mouse: Mouse): Int
    fun type(dog: Dog): Int
    fun type(car: Car): Int
}
```

返回对应的id:
```
class TypeFactoryForList : TypeFactory {
    override fun type(duck: Duck) = R.layout.duck
    override fun type(mouse: Mouse) = R.layout.mouse
    override fun type(dog: Dog) = R.layout.dog
    override fun type(car: Car) = R.layout.car
```

## [Material Intro Screen for Android Apps](https://medium.com/tangoagency/material-intro-screen-for-android-apps-c4317fbac923?source=latest)
现在有两个主流的libraries为Android 应用提供了好看的intro screens, 但是感觉并不是很好用, 所以作者他们发布了一个新的欢迎界面的库[TangoAgency/material-intro-screen
](https://github.com/TangoAgency/material-intro-screen/), 好用易扩展.

## [Testing Legacy Code: Hidden Dependencies](https://medium.com/@corneliu/testing-legacy-code-hidden-dependencies-9b8cd617953f)

本文讨论[God Object](https://en.wikipedia.org/wiki/God_object), [Blob](https://sourcemaking.com/antipatterns/the-blob), 这种很大的类和方法, 做了很多事情. 如果你想要重构, 先加点测试, 也发现很难, 因为它的依赖太多了, 做了太多事情.

首先, 实例化:
加set方法, 让数据库依赖抽离出来, 这样测试的时候可以传一个Fake的进去.

第二, 更多依赖:
把UserManger和网络请求等依赖也抽为成员变量, 加上set方法或者构造参数, 这样在测试的时候易于把mock的东西传进去.

第三, 清理: 要牢记[单一职能原则](https://en.wikipedia.org/wiki/Single_responsibility_principle), 进行职能拆分.

最后, 现实: 清理是一个持续化的过程, 得一步一步来, 有时候小步的改动会帮助你发现另外需要改动的地方. 

# LIBRARIES & CODE
## [EncryptedPreferences](https://github.com/PDDStudio/EncryptedPreferences)
AES-256加密的SharedPreferences.

## [Pury](https://github.com/NikitaKozlov/Pury)
报告多个不同事件之间的时间, 可用于性能测量.

## [Floating-Navigation-View](https://github.com/andremion/Floating-Navigation-View)
Floating Action Button, 展开后是一个NavigationView.

## [Material Intro Screen](https://github.com/TangoAgency/material-intro-screen)
易用易扩展的欢迎界面.

# SPECIALS
## [Huge list of useful resources for Android development](http://www.anysoftwaretools.com/best-android-development-resources/)
资源分享, 包括博客论坛Video社区等等.