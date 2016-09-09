---
title: Android Weekly Notes Issue 221
date: 2016-09-09 11:42:07
tags: [Android, Android Weekly]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #221
September 4th, 2016
[Android Weekly Issue #221](http://androidweekly.net/issues/issue-221)

<!-- more -->

## ARTICLES & TUTORIALS
## [Android ImageView ScaleType: A Visual Guide](https://robots.thoughtbot.com/android-imageview-scaletype-a-visual-guide)

回想一下, 你是不是总是记不住ImageView的不同ScaleType的区别, 每次都要各种尝试来找到自己适合的.
这篇文章的作者也有这样的烦恼, 于是他把各种ScaleType都截了图:
![ImageView-ScaleTypes](/images/ImageView-ScaleTypes.png)


如果用了CENTER_INSIDE, FIT_CENTER, FIT_START,或者FIT_END, 而实际View的大小比图像的大, 可以使用`android:adjustViewBounds`属性为true, 就会调整View的大小.

官方文档: [ImageView.ScaleType](https://developer.android.com/reference/android/widget/ImageView.ScaleType.html)

## [5 steps to creating frustration-free Android test devices](https://m.signalvnoise.com/5-steps-to-creating-frustration-free-android-test-devices-9bb2750edd19#.56mnep7p1)
作者讲了如何统一管理测试机.

### 1.根据你要支持的API level来安装系统.
理想情况下你应该每一个API都有一个对应的机器, 更进一步可以统计一下你的用户用什么的最多来进行调整. 

作者列举了他当前的五个机器, 一般来讲, 你至少需要高中低API版本的, 也需要Samsung的机器来测试一些可能会被定制的地方, (当然作者是在国外了, 国内估计需要测试的定制机型就更多了), 另外还需要一个大屏幕的, 来查看UI的适配情况.

幸运的是除了品牌定制机, 其他都可以用模拟器来补救, 在此推荐一下genymotion, 传说中最快的模拟器.

### 2. 安装并配置测试所需的应用
为了测试你的app, 你可能需要一系列的工具app, 所以第二步你需要安装它们, 登录及设置等等.
原作者常用的工具app有:

- 1Password: 管理密码.
- AZ Screen Recorder: 截屏, 制作gif.
- Chrome Beta: 因为原作者做WebView相关的工作, 所以需要看这个.
- Dropbox: 自动上传截图, 从电脑可以方便拿, 也可以用来做一些文件相关的测试.
- Flesky / Swiftkey / Google Keyboard: 也是作者应用相关, 需要测试各种键盘.
- Keep: 很好用的笔记应用, 可以存一些小notes, url等, 跨设备同步.
- Solid Explorer: 文件管理器, 可以在系统中方便地移动文件.


### 3. 在各处都登录

需要登录的账号都登录.

### 4. 为了统一体验装个Nova Launcher
为了让每个机器都看起来一样, 原作者装了个launcher应用: [Nova Launcher](https://play.google.com/store/apps/details?id=com.teslacoilsw.launcher).

确实, 因为每个机器的launcher和组织方式不一样, 所以有时候换个机器就会很难找到你想要的东西.

用了这个Nova Launcher之后, 你可以设置好你的home, dock, drawer, 然后多个机器分享设置, 这样当你拿起另一个机器的时候, 所有的应用都在同样的位置.

### 5. 设置每个机器的系统设置
最后一件事就是一些系统上的设置, 包括:
- 所有地点的Wi-Fi;
- DND/total silence: 关声音;
- Developer options和USB debugging开关打开;
- 当插线时仍然保持屏幕唤醒;
- 亮度设置.

最后作者建议开发者平时生活中可以多玩玩各种Android应用.

## [Security issues with Android Accessibility](https://medium.com/@vedprakashrout/android-accessibility-75fdc5810025#.f10tnu6oj)

看这篇文章之前, 让我们了解一下Accessibility是什么, 搜了一下Android相关文档:

- [Guides of Accessibility](https://developer.android.com/guide/topics/ui/accessibility/index.html)
- [Training for Implementing Accessibility](https://developer.android.com/training/accessibility/index.html)
- [Design Guidelines for Accessibility](https://developer.android.com/design/patterns/accessibility.html)
- [Material Design Accessibility](https://material.google.com/usability/accessibility.html#)

Accessibility是为了扩展访问和利用应用的形式, 基本出发点是为了辅助老年人或者是有障碍的人, 增加一些听觉或触觉反馈, 也可以用来辅助一些特殊场合下的用户, 比如正在开车或照顾孩子, 或者处于非常嘈杂的环境下的情形.

可以结合Google的[TalkBack](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback), 也可以自己开发相关的服务.

好了, 话题收敛回来, 看看作者说的安全问题指的是什么.

作者一开篇以一个印度很流行的应用Voodoo为例指出, 把屏幕上的文字读出来这个功能是有安全漏洞的.

首先Voodoo向用户请求accessibility的权限, 这个权限使得应用可以从屏幕上读取文字, 但是用户会认为所有的敏感字段应该不在这个范围之内, 这就是开发者需要认真对待的了.

最近有一个新的登录设计, 已经被应用开来, 就是用户可以选择显示或者隐藏密码字段.

当我们把输入框的input type设置为密码, 那么它是不会被读取到的, 但是有一些应用为了支持显示密码的功能, 可能会把input type设置为其他类型, 这样就会导致密码暴露, 有accessibility权限的恶意应用就会借此盗用用户的敏感信息.

这样当然是不好的啦, 用户开启权限的时候还认为敏感字段总会受到保护呢, 所以我们开发者应该小心地对待用户的敏感信息, 很简单:

`ViewCompat.setImportantForAccessibility(your_view, ViewCompat.IMPORTANT_FOR_ACCESSIBILITY_NO);`

新的`TextInputLayout`的API可以实现密码显示的toggle功能, 希望这个问题在TextInputLayout中已经解决了, 但是在用这个View之前, 上面的hotfix也算一种解决办法.

## [How to fix horizontal scrolling in your Android app](http://nerds.headout.com/fix-horizontal-scrolling-in-your-android-app/)
在Android中垂直的滚动很常见, 但是如果在垂直滚动的View里嵌套一个水平滚动的View, 那滑动的体验将会非常不好.

**Problem**: 垂直滚动和内嵌的水平滚动打架了, 滚动体验不佳.

**What's happening inside**:

例子里根view是一个`RecyclerView`加垂直 `LinearLayoutManager`, 里面的child是一个`RecyclerView`加水平`LinearLayoutManager`.

但是当用户做水平滚动的时候, touch事件首先被外面的父View给拦截了.

看`RecyclerView`的代码可知, 在`onInterceptTouchEvent()`方法里, 在垂直滚动使能的情况下, 只要垂直移动的距离(dy)大于一定程度(`Math.abs(dy) > mTouchSlop`), 就会被认为是垂直滚动.

所以作者他们的解决方案是继承了RecyclerView, 覆写了这个方法, 把条件改成:
```java
if(canScrollVertically && Math.abs(dy) > mTouchSlop && (canScrollHorizontally || Math.abs(dy) > Math.abs(dx))) {...}  
```

这是他们的完整代码: [BetterRecyclerView](https://gist.github.com/manidesto/ecccd38787fa8e287a3f18bcd9867189)

**Bonus**
还有一个跟fling相关的问题: `RecyclerView`在fling之后需要挺长的一段时间来稳定(settle)下来, 当child还在这个稳定过程中时, 如果用户尝试竖直滚动, touch事件实际上是被child吃掉的.

还是从`onInterceptTouchEvent()`的代码可以看出:
```java
if (mScrollState == SCROLL_STATE_SETTLING) {
    getParent().requestDisallowInterceptTouchEvent(true);
    setScrollState(SCROLL_STATE_DRAGGING);
}
```

当child处于SETTLING状态时, child会要求它的parent不要拦截touch事件.

这在通常情况下是好的.
但是在作者的使用场景里, 他们的root中没有其他竖直滚动和拖拽的child, 所以他们又继承了刚才那个BetterRecyclerView, 写了一个`requestDisallowInterceptTouchEvent()`为空实现的View作为root.

他们的sample demo在这里: [manidesto/scrolling-demo](https://github.com/manidesto/scrolling-demo).

## [Update Dependencies. Code. Repeat.](http://crushingcode.co/update-dependencies-code-repeat/)

每一个Android开发可能都需要对他们的项目进行(更新依赖, 编码, 重复)这样的循环工作, 如果你想要你的所有项目都有同样的版本号, 这样是很浪费时间的.

原文作者就经历了这样的情景, 他想要把他这个目录[Android-Examples](https://github.com/nisrulz/android-examples)下的所有项目都更新一下. 这个目录里全是那种很小的简单例子, 但都是可独立运行的工程.
每当gradle-plugin, support library或者google play services要更新版本号, 保持这些工程全部都updated是一项很难的工作.

所以原作者想要放弃原先逐个更新的土办法, 更有效率地来更新依赖版本号.
首先想到的就是在gradle里定义一个变量, 然后双引号加$引用这个变量.
为了让所有的module都采用同一变量, 可以在根项目的build.gradle文件里定义变量, 即使用ext块.
但是到此, 只能统一管理在同一个project下的各个modules的依赖版本.

如果跨projects呢?
首先, 作者在存放这些projects的根目录下建了一个gradle文件, 然后把变量都定义在那里.
然后如何应用到各个project呢?于是原作者找啊找, 找到了这块: [gradle Subproject configuration](https://docs.gradle.org/current/userguide/multi_project_builds.html#sec:subproject_configuration)
他给每个工程的根build文件加了个这个:
```
// This is added to apply the gradle file to each module under the project
subprojects {
    apply from: '../../dependencies.gradle'
}
```
这样以后在每个工程的module里面都可以直接引用变量了.


但是, 这对于android-gradle-plugin的版本是不管用的.

这是因为上面应用的配置只对subproject起作用, 对每一个root project是没有应用到的.
所以作者在每一个项目的root build.gradle中, 在buildscript块又加了它的依赖配置文件:
```
buildscript {
    // This is added to apply the gradle file to facilitate providing variable values to root build.gradle of the project
    apply from: '../dependencies.gradle'
    ..
    dependencies {
        classpath "com.android.tools.build:gradle:$androidPluginVer"
        ..
    }
}
```

好啦, 至此, 所有的依赖配置问题就解决了, 以后改版本号只需要改一个地方就可以应用到所有项目.

## [DI 101 - Part 2](https://medium.com/di-101/di-101-part-2-9f7f4e1dcc81#.bfcmljct9)
作者上一篇的文章里介绍了Dagger的基本使用.
这篇还是教程类文章, 讲:

**多个Modules**:

写了一个Retrofit 2的ApiModule, 和一个Realm的DatabaseModule.

**多个对象**:

有时候我们需要提供一个类的不同对象, 我们可以用`@Named`注解, 然后用不同的字符串来区分它们.

文中的例子是这样:
```java
@Provides
@Named(IMAGE_URL)
String provideImageUrl() {
  return ImageApiService.ENDPOINT;
}

@Provides
@Named(URL)
String provideBaseUrl() {
  return RestApiService.ENDPOINT;
}

@Provides @Singleton @Named(REST_API_RETROFIT)
Retrofit provideRetrofit(@Named(URL) String baseUrl, OkHttpClient client) { ... }

@Provides @Singleton @Named(IMAGE_API_RETROFIT)
Retrofit provideImageRetrofit(@Named(IMAGE_URL) String baseUrl, OkHttpClient client) { ... }
```

## [Inject interfaces without provide methods on Dagger 2](https://android.jlelse.eu/inject-interfaces-without-providing-in-dagger-2-618cce9b1e29?swoff=true#.66p0l6oik)
用dagger2注入接口, 返回实现类的对象, 比较常规的方法是在Module里面写一个`@Provides`标注的providesXXX()方法, 返回值类型是接口, 实际返回的是实现类的对象, 比如:
```java
@Module
public class HomeModule {

  @Provides
  public HomePresenter providesHomePresenter(){
    return new HomePresenterImp();
  }
}
```

但是如果我们想给实现类加一个依赖UserService呢?

我们当然可以把UserService作为参数传给这个provide方法, 然后传到实现类的构造函数中, 在里面存一个字段.

又或者, 我们可以使用`@Binds`注解, 像这样:
```java
@Module
public abstract class HomeModule {

  @Binds
  public abstract HomePresenter bindHomePresenter(HomePresenterImp   
    homePresenterImp);
}
```
这是一个抽象类中的抽象方法, 这个方法的签名意思是告诉dagger, 注入`HomePresenter`接口(返回值)时, 使用`HomePresenterImpl`(方法参数)实现.

然后, 在`HomePresenterImpl`类的构造函数上加一个`@Inject`就可以了.

这样我们就不需要在provide方法上加依赖参数了.

## [Introducing ExpandableRecyclerView](https://robots.thoughtbot.com/introducing-expandablerecyclerview)
本文介绍[expandable-recycler-view](https://github.com/thoughtbot/expandable-recycler-view), 一个开源的库, 可以展开和折叠RecyclerView中的组.

`RecyclerView`作为`ListView`的升级版, 却也减少了一些功能比如`OnItemClickListener`, `ChoiseModes`, 还有扩展版的`ExpandableListView`.
本文作者就介绍这个开源库, `ExpandableRecyclerView`, 用自定义的`RecyclerView.Adapter`来实现展开关闭分组的功能.

首先明白一下Adapter的功能, 其实adapter就是一个中间人, 将一些数据按照index翻译给View, 然后显示.

当显示的list是单维度的时候, 这样的翻译很简单, 数据的index就直接对应了屏幕上view的index.

当时当你显示二维数据时, 翻译就变得有点复杂,数据和view的index可能对应, 也可能不对应.
`RecyclerView.Adapter`就只能处理一维数据的情况, 这就是为什么要对其进行一些扩展, 才能实现ExpandableRecyclerView.

后来作者简单讲了实现的原理, 用到了`ExpandableListPosition`, 是Android SDK中就有的类, 只不过有包限制, 所以拷贝到了这个库里.

最后附上repo地址: [expandable-recycler-view](https://github.com/thoughtbot/expandable-recycler-view)

## [Creating Custom Annotations in Android](https://medium.freecodecamp.com/creating-custom-annotations-in-android-a855c5b43ed9#.dq08cdjsm)
**注解是什么**

**Annotations are Metadata.**

注解是元数据, 而元数据是一些关于其他数据的信息.
所以说, 注解是关于代码的信息.

比如`@Override`注解, 即便你不在方法上标注它, 程序依然能够正常工作. 那么它是用来干什么的呢? 

`@Override`是用来告诉编译器, 这个方法覆写了一个方法, 如果父类没有这个方法, 则会报一个编译错误.

如果你不加这个注解, 有可能你方法名不小心拼错了却仍然编译通过了.

**创建自定义注解**:
比如, 创建一个:
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface Status {
  public enum Priority {LOW, MEDIUM, HIGH}
  Priority priority() default Priority.LOW;
  String author() default “Amit”;
  int completion() default 0;
}
```
其中`@Target`指定了这个注解可以放在哪里. 如果你不设置, 这个注解可以放在任何地方.
可能的值有:
* `ElementType.TYPE` (class, interface, enum)
* `ElementType.FIELD` (instance variable)
* `ElementType.METHOD`
* `ElementType.PARAMETER`
* `ElementType.CONSTRUCTOR`
* `ElementType.LOCAL_VARIABLE`

`@Retention`定义这个注解可以被保存多久.
可能的值有:
* `RetentionPolicy.SOURCE` - 到编译结束, 不会被编进.class, 只会留在源文件中. `@Override`, `@SuppressWarnings`都是这种.
* `RetentionPolicy.CLASS` - 到类加载丢弃, 注解将存储在.class文件中, 这是默认值.
* `RetentionPolicy.RUNTIME` - 不被丢弃. .class文件中有, 并且可由VM读入, 在运行时可以通过反射的方式读取到.

上面的注解使用时:
```java
// in Foo.java
@Status(priority = STATUS.Priority.MEDIUM, author = “Amit Shekhar”, completion = 0)
public void methodOne() {
 //no code
}

// get the annotation information
 Class foo = Foo.class;
 for(Method method : foo.getMethods()) {
    Status statusAnnotation = (Status)method.getAnnotation(Status.class);
    if(statusAnnotation != null) {
     System.out.println(" Method Name : " + method.getName());
     System.out.println(" Author : " + statusAnnotation.author());
     System.out.println(" Priority : " + statusAnnotation.priority());
     System.out.println(" Completion Status : " + statusAnnotation.completion());
    }
 }
```

如果你的注解中仅有一个属性, 它应该叫value, 并且使用的时候不用指定属性名.
```java
@interface Status{
 int value();
}
 @Status(50)
 public void someMethod() {
  //few codes
 }
```

最后作者还附上了另一个他的文章, 推荐他的网络请求库: [Fast Android Networking](https://medium.freecodecamp.com/simple-and-fast-android-networking-19ed860d1455#.3hqqpr1ba)

## [It's parfetti time!](https://medium.com/@jinatonic/its-parfetti-time-f40634472608#.9vf9fxn99)
作者介绍他的库: [confetti](https://github.com/jinatonic/confetti).

这个库实现了一个粒子系统, 来发射出随机的纸屑, 并且可以被定制化, 比如发射源的形状(点或者线), 初始的物理约束(速度, 加速度, 旋转等), 还可以定义消失或者拖拽行为, 感觉效果还挺好的.

关于性能, 纸屑对象是循环利用的, 每一个bitmap也只被分配一次地址, 动画参数也做了一些预计算, 所以作者说不用担心丢帧, 除非你一次性出现的片儿实在是太多了.

## [Converting callback async calls to RxJava](https://medium.com/we-are-yammer/converting-callback-async-calls-to-rxjava-ebc68bde5831#.mmtpwqgkh)
作者他们在自己Android应用里开始使用RxJava以后, 经常会遇到由于API没有follow reactive model, 导致他们必须做一些转换工作, 将它们和其他的RxJava Observable链连接起来.

API对于很重的操作通常提供这两种方式之一
* 1.同步阻塞方法调用, 通常需要后台线程调用.
* 2.异步非阻塞方法调用, 结合callback, listener, 或者broadcast receiver等.

**把同步方法变为Observable**:

用这个[Observable.fromCallable()](http://reactivex.io/RxJava/javadoc/rx/Observable.html#fromCallable%28java.util.concurrent.Callable%29)

比如:
```java
// wrapping synchronous operation in an RxJava Observable
Observable<Boolean> wipeContents(final SharedPreferences sharedPreferences) { 
    return Observable.fromCallable(new Callable<Boolean>() { 
        @Override 
        public Boolean call() throws Exception { 
            return sharedPreferences.edit().clear().commit(); 
        } 
    }); 
}
```

**把异步方法变为Observable**:

变异步没那么简单了, 之前有一些模式是用工厂方法`Observable.create()`把它们包起来, 比如[here](http://ryanharter.com/blog/2015/07/07/wrapping-existing-libraries-with-rxjava/), [here](http://andriydruk.com/post/rxdnssd/), [here](http://stackoverflow.com/questions/29679801/chaining-rxjava-observables-with-callbacks-listeners/29682801#29682801), 但是这种方法存在一些缺点.

作者举了一个传感器监听的例子.
用create()转换之后, 需要处理一些问题, 比如注销listener, 错误处理, 检查subscriber等, 这几个都可能办到, 但是还有一个backpressure的问题, 不好办.
这个backpressure是什么捏: [backpressure](http://stackoverflow.com/documentation/rx-java/2341/backpressure#t=201609081434407670206)
当生产者发射值的速率比消费者可以处理的速率快的时候, 有一个内置的buffer size, 当超出的时候就会抛出`MissingBackpressureException`.

幸运的是, RxJava v1.1.7推出了`Observable.fromAsync()`, 在v1.2.0改名为`Observable.fromEmitter()`.
这个里面对于backpressure的处理定义了好几种策略, 你只需要选一种模式就行.

然后作者给出了采用这个新方法的例子, 这里不再赘述, 可以看原文.
Sample代码在: [AndroidRxFromAsyncSample](https://github.com/murki/AndroidRxFromAsyncSample)

# LIBRARIES & CODE
## [ItemTouchHelper Extension](https://github.com/loopeer/itemtouchhelper-extension)
`ItemTouchHelper`的扩展, 加滑动settling和恢复. Sample的效果是给单个item加了滑动后出现删除和refresh两个按钮.
## [Fresco Image Viewer](https://github.com/stfalcon-studio/FrescoImageViewer)
为Fresco库加的全屏查看图像的工具, 支持双手指的zoom和滑动关闭手势.
## [ABTestGen](https://github.com/imperial-crystalline-recursion/abtestgen)
一个生成简单A/B test的库, 使用注解.
## [RecyclerViewHelper v24.2.0](https://github.com/nisrulz/recyclerviewhelper)
一个RecyclerView的辅助类, 提供滑动删除, 拖动, divider, 选中和非选中事件等的支持.
## [Paginize](https://github.com/neevek/Paginize)
一个轻量级的Android应用framework.

# SPECIALS
## [Tips and tricks for Android Development](https://github.com/nisrulz/android-tips-tricks)
一个很长的README, 包含了各种快捷键, 编码建议, 工具, 插件, 还有有一些推荐的网站等, 其中有mock api和新闻网站及其他有用的工具等.

