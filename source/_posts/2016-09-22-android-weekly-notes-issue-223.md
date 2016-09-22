---
title: Android Weekly Notes Issue 223
date: 2016-09-22 17:34:27
tags: [Android, Android Weekly, Accessibility, Test, Fingerprint, Kotlin, MVP, RxJava, RxJava2, Nougat, GCM, Memory Leak, Gradle]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #223
September 18th, 2016
[Android Weekly Issue #223](http://androidweekly.net/issues/issue-223)

本期内容包括:
Offline时间戳处理;  Accessibility的安全问题可能并不是个问题; 如何在单元测试和UI测试之间共享代码; Android中的指纹认证; 编译时间Kotlin vs Java; MVP结合RxJava, 让View来处理生命周期; RxJava2预览; 内存泄露处理; Gradle相关等等.


<!--more-->

# ARTICLES & TUTORIALS
## [Offline First: Introducing TrueTime for Android](https://tech.instacart.com/truetime/)
TrueTime是一个NTP library for [Swift](https://github.com/instacart/truetime.swift) and [Android](https://github.com/instacart/truetime-android).

其中NTP是Network Time Protocol.

作者他们有一个购物app, 但是时断时续的网络降低了用户体验, 所以他们进行了离线迁移, 准备出一系列文章分享相关的想法和在此过程中学到的东西.

本文是第一篇, 关于时间.

由于在设置里可以设置设备的日期和时间, 所以设备的时间并不一定是真实的时间, 我们在程序里`new Date()`得到的其实是设备时间.

关于真实时间的计算, 他们开源了TrueTime库, Android和iOS都能用.

TrueTime如何计算真实时间的呢? 它其实是向NTP的server发了请求, 然后计算出的.

文中和库都说明了用法.

## [Android Security and Accessibility](https://medium.com/@ataulm/a-few-weeks-ago-android-weekly-promoted-a-post-highlighting-a-security-issue-with-the-android-5eae7ff6b8aa#.p25cbw2wl)

之前有一个[文章](https://android.jlelse.eu/android-accessibility-75fdc5810025#.94tpbl6z2)说Accessiblity存在安全隐患, 这个服务可能可以访问到一些隐私信息, 比如密码.

但是这篇文章的作者觉得前一篇文章作者的解决方案不是很好.

因为当用户开启Accessibility权限的时候, Android就已经给出了警告, 说明敏感信息可能会被观察到. 第三方的keyboard也可以访问这些信息, Android也是在开启的时候给出了警告.

另外对于前一篇文章作者提出的解决方案: `View.IMPORTANT_FOR_ACCESSIBILITY_NO`
这样真正有视觉障碍的那部分用户也无法看到密码, 可能就无法登陆了.

所以本文作者建议的解决方案是, 可以弹一个对话框来提醒用户, 如果用户允许了, 再继续输入.

## [Sharing code between UI & unit tests](http://trickyandroid.com/android-test-tricks-sharing-code-between-unit-ui-tests/)
Android的测试分两种:

一种是Unit tests. 单元测试, 在JVM上跑.

另一种是UI测试, 需要Android设备.

在Android Studio中对应`test`和`androidTest`文件夹.

这两个测试文件夹之间是不共享代码的, 即一个文件夹里不能访问另一个里面的代码.

但是如果我们想要共用一些代码, 是有办法解决的.

首先在app/src下新建一个文件夹, 比如叫`testShared`. 里面添加要共享的代码.

然后在`app/build.gradle`里面添加这个:
```
android.sourceSets {  
    test {
        java.srcDirs += "$projectDir/src/testShared"
    }

    androidTest {
        java.srcDirs += "$projectDir/src/testShared"
    }
}
```
就可以在UI测试和单元测试中共享同一份代码了.

## [Synchronously Animating Colors on Android](https://kylewbanks.com/blog/animating-toolbar-tablayout-floatingactionbutton-and-statusbar-background-color-on-android)

作者想做的一个效果是, 在切换tab的时候, 把`Toolbar`, `TabLayout`, `FloatingActionButton`还有`StatusBar`的颜色都动画地改变到另一个颜色.

实现很简单, 首先用当前颜色和目标颜色建立一个`ValueAnimator`, 然后`addUpdateListener()`在更新的过程中把值set给相应的控件:
```java
colorAnimation.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animator) {
        int color = (int) animator.getAnimatedValue();

        toolbar.setBackgroundColor(color);
        tabLayout.setBackgroundColor(color);
        floatingActionButton.setBackgroundTintList(ColorStateList.valueOf(color));

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            getWindow().setStatusBarColor(color);
        }
    }

});
colorAnimation.start();

```

其中FloatingActionButton要用`setBackgroundTintList()`.

StatusBar在21及以上才支持`getWindow().setStatusBarColor(color);`

## [Android Fingerprint Authentication](https://medium.com/@aitorvs/android-fingerprint-authentication-44c047179d9a#.3og72boir)
其实用户都不喜欢验证, 因为用户都比较懒, 不喜欢一次又一次地输入密码或者手势pattern, 但是不锁屏又不安全.

指纹验证[Fingerprint Authentication](https://developer.android.com/about/versions/marshmallow/android-6.0.html#fingerprint-authentication)是Android M (Android 6.0, API 23)引入的. 它就是为了解决这个问题, 提升用户体验. 这种non-disturbing和easy的方式, 让我们不用在安全和用户体验之间做出妥协.

如果你的应用需要做一些关键操作, 比如支付, 你需要用户在操作前授权, 那么指纹验证会很有帮助.

然后作者介绍了实现的细节.

最后作者附上了自己的相关库: [fingerlock](https://github.com/aitorvs/fingerlock).

## [Kotlin vs Java: Compilation Speed](https://medium.com/keepsafe-engineering/kotlin-vs-java-compilation-speed-e6c174b39b5d#.nep65secf)

这是作者关于Kotlin的第三篇文章, 作者在这篇文章里测试了Kotlin和Java的编译时间.

**Clean build with No Gradle daemon**
Java编译比Kotlin快17%.

**Clean build + Gradle daemon**
`org.gradle.daemon=true`

Java编译比Kotlin快13%.

**Incremental builds**
`kotlin.incremental=true`

在clean build的时候, Java可能快10-15%, 但是在增量build + gradle daemon时, kotlin和Java一样快, 甚至可能比Java更快一些.

## [Let the view handle the lifecycle in MVP by using RxJava](https://medium.com/@ferhatparmak/let-the-view-handle-the-lifecycle-in-mvp-by-using-rxjava-694d67923871#.vt21pzfr0)

问题:
作者举了一个例子, 在Fragment作为View的MVP中, 如果P从service取一些数据, 然后调用View的显示方法, 则还需要知道`onViewCreated()`是不是已经调用过了.

解决方案:

首先创建一个Lifecycle的BehaviorSubject, 在`onViewCreated()`的时候调用`onNext(null)`.

把View的方法改成返回一个Observable, presenter的方法调用View的方法时实际上是subscribe了一下:
```java
class ProductsFragment implements ProductsView {
  private ProductsPresenter presenter;
  //Lifecycle subject. It is BehaviourSubject because it can be subscribed after onViewCreated call.
  private final BehaviorSubject<Void> onViewCreatedSubject = BehaviorSubject.create();
  
  @Override
  public Observable<Void> showProducts(List<Product> productList) {
    return onViewCreatedSubject. // Wait for onViewCreated
        doOnNext(new Action1<Object>() {
          @Override
          public void call(Object o) {
            //Updates recyclerview adapter items
          }
        });
  }
  
  @Override
  public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    onViewCreatedSubject.onNext(null);
  }
}
```

Presenter:
```java
class ProductsFragmentPresenter implements ProductsPresenter {
  private ProductsView view;
  
  public void loadProducts(){
    productsService.getProducts()
      .flatMap(new Func1<Object, Observable<Void>>() {
          @Override
          public Observable<Void> call(List<Product> productList) {
            //Return the view's observable to show products. 
            //No need to check if the view is created!
            return view.showProducts(productList); 
          }
        }) 
      .subscribe();
  }
}
```

当然这并不是一个完整的例子, 完整的例子还需要考虑`onDestroyView()`还有注销等情况的处理.

## [Nougat - GCM Network Manager](https://blog.stylingandroid.com/nougat-gcm-network-manager/)

作者搞了一个message app来研究Android 7的新特性.

他用到了[AutoValue](http://ryanharter.com/blog/2016/03/22/autovalue/).

关于Android 7的另一篇文章: [Random Musings on the N Developer Preview](https://commonsware.com/blog/2016/03/09/random-musings-n-developer-preview.html)

他们的应用首先需要周期性地生产一些消息, 关于生产消息的实现, 作者没有用`AlarmManager`, 也没有用`JobScheduler`(因为只支持API 21及以上), 而是选用了`GCMNetworkManager`.

具体实现见原文, 有详细说明.
另: [代码](https://github.com/StylingAndroid/Nougat/tree/GCMNetworkManager)

这只是系列文章的第一篇, 后续应该会写更多.

## [TransactionTooLargeException crashes on Nougat](http://blog.sqisland.com/2016/09/transactiontoolargeexception-crashes-nougat.html)

作者自己的应用在Activity转换的时候遇到了一个crash: `java.lang.RuntimeException: android.os.TransactionTooLargeException: data parcel size 700848 bytes.`

之前应用里有相关的Warning log, 但是
Android 7 Nougat (API 24)把它作为异常抛出来了.

产生这个问题的原因是在`onSaveInstanceState()`里面存了太多数据. 作者做了一个测试, 想看看这个限制大概是多少, 大概是500K左右. 

所以这里是不应该用来存储太多数据的, 应该只存状态.

底下回复说每个进程都有1M的buffer来接收transactions, 但是是在没有任何其他IPC的情况下. 所以建议存储的状态数据少于100K或者50K, 当然越少越好.

## [Building a blazing fast ETC2 compressor](https://medium.com/@duhroach/building-a-blazing-fast-etc2-compressor-307f3e9aad99#.ixumn3e2v)
作者是Google的, 以前做游戏的, 所以致力于Performances, GPU, 数据压缩等内容.

作者关注VR, 但是VR中要提升体验, 必定会增加图像的大小和质量.

[ETC textures](https://en.wikipedia.org/wiki/Ericsson_Texture_Compression) 是OpenGLES 3.0的一种标准格式.

编码一个高质量的ETC2 texture会花费很多时间.
以在游戏界最流行的压缩工具[Mali GPU Texture Compression tool](http://malideveloper.arm.com/resources/tools/)为例, 作者做了实验, 证明确实要花费很多时间(平均10分钟)来encode一个图.

所以作者他们开发了一个新的库: [etc2comp](https://github.com/google/etc2comp), 一个很快的texture encoder.

然后和之前的工具做了比较, 平均时间提高到了10秒.

后来他说的技术细节我就看不懂了. 文后还有其他图像格式(JPG, PNG, WebP)相关的文章链接.

## [Low Coupling With Rx and Dagger2 in Android](http://www.ottodroid.net/?p=479)
作者举例展示Android程序的解耦.

首先, 他展示一个高度耦合的Android程序, 然后加入Rx, 最后加入Dagger2, 从而一步一步地解耦这个项目.

项目的内容是发现Network中的Services. 这里有官方的Training: [Network Service Discovery](https://developer.android.com/training/connect-devices-wirelessly/nsd.html).

## [RxJava2: An Early Preview](https://medium.com/@theMikhail/rxjava2-an-early-preview-5b05de46b07#.ftflhi48n)
最近RxJava2有了第一个Release Candidate. 所以作者在这里先预览一下有哪些有趣的更新和新加的功能:

**New Dependency**: 
添加了依赖: [ReactiveStreams](http://www.reactive-streams.org/).

**Imports**:
RxJava2放在了一个不同的package下:

RxJava:

`compile ‘io.reactivex:rxjava:1.0.y-SNAPSHOT’`

RxJava2:

`compile ‘io.reactivex.rxjava2:rxjava:x.y.z’`

这意味着, 你可以同时用两个版本的库. 如果你要完全迁移的话, 你需要把所有的import都改到新包.

**Null Emissions No Longer Permitted**:
不允许再发送null值了, 会直接抛出空指针异常.

```java
Observable.just(null); //don’t do this
subject.onNext(null); //don’t do this either
```

**Under(Back)Pressure**:

`Backpressure`是当`Observable`发射值的速度比`Observer`能处理的速度快时发生的.

RxJava2引入了一个新的Observable类`Flowable`, with backpressure support.

**Single Old and New**:
订阅一个Single现在可以用这个:
`SingleObserver<T>`.

**Hit Me Maybe One More Type**:
一个新的类型叫`Maybe`, 它是`Single`和`Completable`的混合体. 用来发射0或1个值.

**New BackPressured Subject: Processor**:
引入了一个新类型, `Processor`, 它是一个有backpressure support的`Subject`.

**New Names for Function and Action**:
- `Func1` -> `Function`
- `Func2` -> `BiFunction`
- `FuncN` -> `Function<Object[], R>`
- `Func1<T, Boolean>` -> `Predicate<T>`
- `Action0` -> `Consumer`
- `Action1` -> `BiConsumer`
- `ActionN` -> `Consumer<Object[]>`

**Subscriber is Now Disposable**:

因为和Reactive-Streams的命名冲突, 所以`Subscriber`改名为`Disposable`. 它有一个`.dispose()`方法, 类似于`Subscription`的`.unsubscribe()`方法.

`onCompleted()`也将变为`onComplete()`.


**Composite Subscriptions Changes**:

`CompositeSubscription` + `subscribe()`-> `CompositeDisposable` + `subscribeWith()`

**Blocking Calls**:
RxJava2加了一些新的操作符来变异步为同步. 
`.toBlocking.first()` -> `.blockingFirst()`

**Better Hooks for Plugins**:
plugin系统被重写了. 现在你可以覆写内置schedulers返回的值了. 这样你就可以在做单元测试的时候覆写`Schedulers.io()`来返回同步的值, 甚至debug Schedulers.

**Summary**

目标Release日期: October 29.

Retrofit已经支持RxJava2了:
[retrofit-rxjava2-adapter](https://github.com/JakeWharton/retrofit2-rxjava2-adapter)

这里还有一个Library用来把RxJava1转换到RxJava2: [RxJava2Interop](https://github.com/akarnokd/RxJava2Interop)

Sources:
[RxJava 2.x javadoc](http://reactivex.io/RxJava/2.x/javadoc/),
[Github Wiki: What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0), 
[Stackoverflow](http://stackoverflow.com/questions/38423079/differences-between-rxjava1-and-rxjava2)

## [Eight Ways Your Android App Can STOP Leaking Memory](http://blog.nimbledroid.com/2016/09/06/stop-memory-leaks.html)

之前作者有个文章叫[Eight Ways Your Android App Can Leak Memory](http://blog.nimbledroid.com/2016/05/23/memory-leaks.html), 讲的是Android应用中8种内存泄露的原因, 主要是泄露了Activity.

这篇文章主要讲解决方法:

### Static Activities
错误原因: 把Activity存在一个静态引用里, Activity生命周期结束后仍然持有.

解决方法:
使用[WeakReference](https://developer.android.com/reference/java/lang/ref/WeakReference.html).


### Static Views
错误原因: 静态引用了View, 因为attached View引用了Activity, 所以等于间接引用了Activity.

解决方法:
1. 使用WeakReference;
2. 在onDestroy()里面把引用置为null.

### Inner Classes
内部类分两种, 静态内部类和非静态内部类: [Nested Class](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)

错误原因: 在Activity里有一个内部类(非静态), 创建内部类的对象, 然后静态引用之. 因为内部类持有外部类的应用, 所以会造成内存泄露.

解决方法:
尽量不要存static引用.

### 匿名内部类 AsyncTask, Handler, Thread, TimerTask
错误原因:

如果你不在超出生命周期的地方引用它, 匿名内部类的对象是无害的.

但是上面的这些内部类对象全都是用来产生一些线程的, 这些线程是app全局的, 而且会引用创建它们的对象.

解决方法: 
1. 把上面的这些类改成静态内部类, 静态的内部类对象不会引用外部类的对象.
2. 如果你坚持使用匿名内部类, 可以在Activity的onDestroy()里面终止线程.


### Sensor Manager
错误原因:

把Activity作为listener注册给了系统服务, 但是在Activity生命周期结束之前没有注销listener.

解决方法: 在生命周期结束前注销listener.

## [Auto rename Android versionName in Gradle](https://medium.com/@jagonzalez.develop/auto-rename-android-versionname-by-creating-custom-gradle-plugin-2922bbaaaed6#.cds2wvd05)
在应用release的时候, 版本号是确定的, 这没问题. 在应用开发的时候, 如果每一个apk也有一个特定的版本号, 将会非常有帮助.

**自定义Gradle Plugin**:
`com.android.application`就是一个gradle plugin.

有三种方式可以创建gradle plugin: [doc](https://docs.gradle.org/current/userguide/custom_plugins.html#sec:packaging_a_plugin).

本文作者选择了`buildSrc`的方式, 因为这很容易, 而且可以被加到repo里, 但是这样将依附于你的project, 不能复用.

具体代码见原文.

这么做了之后, 每一次build的apk都自带了分支信息, Jira卡号, 或者任何你想带的信息.
## [Is your custom view interactive aware?](https://renaudcerrato.github.io/2016/09/15/is-your-custom-view-interactive-aware/)

什么是**Interactive View**?
当View是可见的, 即可以和用户交互, 即为interactive.

当你的自定义View做一些很重的工作, 比如循环的动画或者loading, 或者依赖于传感器, 当这种View变为不可见时,你需要做一些工作来节约电量.

作者写了一个辅助类: [InteractiveViewHelper](https://gist.github.com/renaudcerrato/746e039700ac5eeaaea40808666e239f) 来做这个.

具体利用了View的这几个回调:
```java
void View::onVisibilityChanged(View, int)
void View::onWindowVisibilityChanged()
void View::onAttachedToWindow()
void View::onDetachedFromWindow()
```

还有两个ACTION:
```java
Intent.ACTION_SCREEN_ON
Intent.ACTION_SCREEN_OFF
```

## [Beta Testing Your Android App With Build Variants](http://chikemgbemena.com/2016/09/16/beta-testing-your-app-with-build-variants/)
讲了如何用Build Variants, 添加不同的Flavors.

## [Make your build.gradle great again](https://medium.com/@sergii/make-your-build-gradle-great-again-c84cc172a654#.y66yudcxc)

### 1. 把你的build.gradle分成小份, 更加模块化, 用`apply`应用.
### 2. 在build file里指明application id.
applicationId是apk最终会用的包名.
packageName是用来找代码中的R, 和activity/service组件的相对路径.
如果不在build文件里指明applicationId可能会有一些问题.

### 3. 给debug版使用一个不同的applicationId.
```
buildTypes {    
    debug {
        applicationIdSuffix ".debug"
    }
    // ...
}
```
好处是同一个机器上可以同时安装debug和release版.

### 4. 统计build时间.

用--profile命令. 或[Build Scans](https://scans.gradle.com/get-started?type=project)

还可以用[build-time-tracker-plugin](https://github.com/passy/build-time-tracker-plugin)

### 5. 配置release.

Proguard在Java层面工作, 对于资源是不管的, 只把R中的id删了.
如果想进一步处理不用的资源, 需要加:
` shrinkResources true`.

更深一步的居然还可以拆分apk: [config-apk-splits](https://developer.android.com/studio/build/configure-apk-splits.html)

### 6. 发现一些有用的tasks, 或者自己开发. [Reddit page](https://www.reddit.com/r/androiddev/comments/3ig3gm/show_us_your_gradle_tasks).

### 7. 把依赖的版本号抽出来.
### 8. 使用jcenter, 响应更快.
### 9. 在开发时把最小sdk设为21或以上, 会build得更快.

# LIBRARIES & CODE
## [Android Amazing Open Source Apps](https://medium.com/@amitshekhar/android-amazing-open-source-apps-e44f520593cc)
这篇文章列举了一些好的开源app.
包括[google/iosched](https://github.com/google/iosched), [android-architecture](https://github.com/googlesamples/android-architecture), [Telegram](https://github.com/DrKLO/Telegram), [Plaid](https://github.com/nickbutcher/plaid), [wire-android](https://github.com/wireapp/wire-android), [ribot/ribot-app-android](https://github.com/ribot/ribot-app-android), [PocketHub](https://github.com/pockethub/PocketHub).

## [DoorSignView](https://github.com/renaudcerrato/DoorSignView)
一个自定义View, 显示门牌. AnimatedDoorSignView可以根据传感器进行动画.

## [Java Error Handler](https://github.com/Workable/java-error-handler/)
一个统一的错误处理器. 为每一种错误建立全局默认的处理方式.