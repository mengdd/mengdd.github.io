---
title: Android Weekly Notes Issue 222
date: 2016-09-15 17:44:39
tags: [Android, Android Weekly]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #222
September 11th, 2016
[Android Weekly Issue #222](http://androidweekly.net/issues/issue-222)

<!-- more -->

# ARTICLES & TUTORIALS

## [Forcing bytes download in Okio](http://jakewharton.com/forcing-bytes-downward-in-okio/)

这是Jake Wharton的文章, 关于Okio的`BufferedSink`.

[okio](https://github.com/square/okio) 是一个java io库, 包装了一套API用来读写和处理数据. 文档见: [okio doc](http://square.github.io/okio/1.x/okio/).

很多库都是在其之上写的, 比如okhttp, Retrofit, Moshi. 这里有个视频: [A Few OK libraries](http://jakewharton.com/a-few-ok-libraries/)

有三个方法可以强制把bytes放入底层的`Sink`.

### flush(), emit() and emitCompleteSegments()

`flush()`

调用这个方法会使得所有缓存的字节移动到Sink, 然后Sink立即flush自己. 所以调用`flush()`方法返回后, 你可以保证所有的字节都到了目的地Sink.

当用多级buffer时, `flush()`会清除所有level的buffer. Okio中多级buffer的花销很小, 一个flush会让每一级把自己的segments移到下一级.

java io中的多级buffer每一级都自己管理自己的字节, 所以flush操作会让每一级都拷贝数据到下一级.

流类型的`close()`和`flush()`的行为类似, 在关闭流之前把所有的字节都写了.

`emit()`

发射字节的行为和flush类似, 但是不是递归的. 调用`emit()`会使得所有缓存的字节移动到Sink, 但是与`flush()`不同, 此时Sink并不会做其他的操作.

`emitCompleteSegments()`

调用这个`emitCompleteSegments()`方法仅仅移动那些完整`segment`的字节移动到Sink.

`segment`是okio的一个概念.

原文中有动图图解, 看起来更清楚一些.

### Use cases

Writing messages to a WebSocket -> `flush()`

Encoding a video to a file -> `emitCompleteSegments()` + `emit()`/`flush()` at the end

Serializing an object to JSON -> `emit()`

## [Using Android Studio's Performance Monitors](https://medium.com/@andreworobator/debugging-without-a-stacktrace-using-android-studios-performance-monitors-a0d601afd814#.wnqu72q21)
作者他的音乐应用遇到了不响应的问题, 然后他打开Android Studio的performance monitor看到了是内存问题, 然后他track了一下出问题的时候的memory allocation, 最后发现是有一张要下载的图太大了.

## [The hidden cost of code coverage](http://jeroenmols.com/blog/2016/09/01/coveragecost/)
测试覆盖率是一个很好的方法来激励你和你的团队多写一些测试, 但是你知道吗, 打开测试覆盖率的检测会让你的build变慢.

要测量build时间:

`./gradlew clean assembleDebug --profile`

然后作者发现其中`:app:transformClassesWithJacocoForDailyDebug`占用的时间达到了build时间的14%.

就是因为这句:
```
buildTypes {
    debug {
        ...
        testCoverageEnabled true
    }
}
```

**解决办法**
只在需要测试覆盖率的时候用它, 所以加一个变量:
`/gradlew -Pcoverage clean connectedDebugAndroidTest`

gradle里这样写:
```
buildTypes {
    debug {
        ...
        testCoverageEnabled (project.hasProperty('coverage') ? true : false)
    }
}
```
当你不加这个flag的时候就不会检查测试覆盖率.

## [Android Development Useful Tools](https://medium.freecodecamp.com/android-development-useful-tools-fd73283e82e3#.ru47kg8kt)

[methodscount](http://www.methodscount.com/) 可以统计library的方法数, 因为Android有64k的方法数限制.

[Stetho](http://facebook.github.io/stetho/)
可以看网络请求, 数据库, shared preferences等.

[LeakCanary](https://github.com/square/leakcanary) 检测内存泄露的库.

[apk-method-count](http://inloop.github.io/apk-method-count/) 这个网站可以检测apk中的方法数, 根据包分别显示.

[Android Asset Studio](http://romannurik.github.io/AndroidAssetStudio/)
生成图片, 9-patch等的一系列工具.

[Buck](https://buckbuild.com/) Facebook开发的一套build系统.

[Gradle, please](http://gradleplease.appspot.com/) 输入你要的库, 然后就可以找到对应的dependencies里应该写的 compile xxx.

[Proguard](https://developer.android.com/studio/build/shrink-code.html)或[DexGuard](https://www.guardsquare.com/dexguard)

[Genymotion](https://www.genymotion.com/)

[Material Design Icons](https://materialdesignicons.com/)

## [Introduction to Automated Android Testing - Part 6](https://riggaroo.co.za/introduction-automated-android-testing-part-6/)
系列文章的最后一篇, 作者讲用Espresso写UI测试.

如果数据是动态的, 那么要测试View包含指定的信息是很难的. 数据有可能变化, 但是我们的测试不能因此而失败. 所以为了测试可靠和可重复, 我们不应该调用production的API.

所以第一步是mock API, 有几种方法:
- 1. 用[WireMock](http://wiremock.org/) (需要server).
- 2. 用OkHttp的[MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver), 在你的设备上跑一个webserver.
- 3. 创建一个Retrofit REST接口, 返回一些dummy对象.

Mock数据的时候利用了Gradle的flavor, 可以创建一个mockDebug flavor, 然后在src目录下建一个mock目录, 放一些mock的代码进去, 然后再建一个prod目录, 把真实的实现移动到那里去.

### Espresso基础
用[Espresso](https://google.github.io/android-testing-support-library/docs/espresso/index.html)写测试, 基本格式是这样:
```java
onView(withId(R.id.menu_search))      // withId(R.id.menu_search) is a ViewMatcher
  .perform(click())               // click() is a ViewAction
  .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion
```

`ViewMatchers` 是用来定位View的 有withId, withText, withTag等.

`ViewActions` 是用来跟View交互的, 比如点击, 输入等.

`ViewAssertations` 是用来做出断言的.

这里有个cheat-sheet可以查: [android-expresso-testing.pdf](https://google.github.io/android-testing-support-library/downloads/espresso-cheat-sheet-2.1.0.pdf)

### Writing Espresso UI Tests
这部分具体讲了作者例子的UI测试写法, 见原文.
最后作者还看了一下测试覆盖率.

## [What 2 years of Android Development Have Taught Me](https://blog.aritraroy.in/what-my-2-years-of-android-development-have-taught-me-the-hard-way-52b495ba5c51#.ur0ez1ouq)
作者讲了他两年来的心路历程, 以及他总结的一些To do和Not to do.

1. 不要重复造轮子.
可以用[android-arsenal](https://android-arsenal.com/)来查库.

2. 明智地选择库.
选择库的时候看看星多不多, 作者有没有什么其他流行的库. 再看看issues, 有时间的话可以看代码.
[Dryrun](https://github.com/cesarferreira/dryrun)可以用来跑sample.

3. 坐下来看代码. 我们应该花费多数的时间阅读别人的代码, 而不是写自己的代码.
你能写出来的代码是你已经知道的东西的一个反映, 要不断成长和提高自己, 只能不断阅读和学习别人的代码.
这里有[Library列表](https://snowdream.github.io/awesome-android/)和[app列表](https://github.com/pcqpcq/open-source-android-apps).

4. 保持一个好的代码风格. 这里有一些参考: [code-style](https://source.android.com/source/code-style.html), [android-guidelines](https://github.com/ribot/android-guidelines/blob/master/project_and_code_guidelines.md).

5. 用Proguard, 这样release版本不但缩小了代码也做了混淆. 这里还有一个[DexGuard](https://www.guardsquare.com/dexguard).

6. 使用一个合理的架构. 可以用MVP来解耦, 这里有个demo: [Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture), 这里是它相关的Guide文章: [Guide on Clean Architecture](https://medium.com/@dmilicic/a-detailed-guide-on-developing-android-apps-using-the-clean-architecture-pattern-d38d71e94029#.j970cogwt).
更多资源:
[androidmvp](https://github.com/antoniolg/androidmvp), 
[mosby](https://github.com/sockeqwe/mosby), 
[android-architecture](https://github.com/googlesamples/android-architecture).

7. 对于独立工作的开发, 还需要注意一些UI/UX相关的原则. 可以上[Dribble](https://dribbble.com/)和[MaterialUp](https://material.uplabs.com/)多看看.

8. 分析是你的好朋友. 分析包括了crash报告和app使用记录, 可以用[Firebase](https://firebase.google.com/).

9. 做一个市场忍者. 对于独立开发者来说, 需要marketing. 这是一个市场分析工具: [sensor tower](https://sensortower.com/#).

10. 优化你的App. 推荐检查内存泄露的工具: [leakcanary](https://github.com/square/leakcanary).

11. 优化gradle的build时间. 这里有两个Guides: [speeding up gradle build](https://medium.com/@cesarmcferreira/speeding-up-gradle-builds-619c442113cb#.6ezl2rgec), [making gradle builds faster](http://zeroturnaround.com/rebellabs/making-gradle-builds-faster/)

12. 多做测试. 发布前花时间测试, 不要急.

13. 兼顾多种机型, 包括屏幕尺寸, API level, 不同厂商的OS等.

14. 用Git. 这是一个[Git branch model](http://nvie.com/posts/a-successful-git-branching-model/). 如果你不能负担github上的private repo, 你可以试试[BitBucket](https://bitbucket.org/)

15. 让黑客觉得难处理. Android很容易被攻击, app可以被轻易地反编译和分析. 你需要知道如何处理你的API keys, 如果你需要处理用户的敏感信息, 你应该知道如何加密. 秘钥也要妥善存储. 任何存储在数据库中的敏感信息也都要加密. 相关的资料可以看[Adding Tampering Detection to Your app](https://www.airpair.com/android/posts/adding-tampering-detection-to-your-android-app)和[Hiding Secrets in Android Apps](https://rammic.github.io/2015/07/28/hiding-secrets-in-android-apps/).

16. 在低级设备上开发. 低级的设备容易暴露问题.

17. 学习设计模式. 这里有一个repo讲了所有的Java中的设计模式: [java-design-patterns](https://github.com/iluwatar/java-design-patterns). 另外还推荐书籍: GoF的设计模式, Martin Fowler的重构, Joshua Bloch的Effective Java.

18. 贡献自己的力量. StackOverflow, Github, blog posts...

## [Android Support Annotations](http://mayojava.github.io/android/android-support-annotations/)

使用Annotation library:

```
compile 'com.android.support:support-annotations:<latest-library-version>'
```

如果你已经用了`appcompat library`, 你就已经可以用annotations了, 因为appcompat自己就用了它.

annotations按照用法和功能来分组:

- Nullness Annotations
- Resource Annotations
- Thread Annotations
- Value Constraints Annotations
- Others : Permissions Annotations, CheckResults Annotations and CallSuper Annotations.


### Nullness Annotations
`@Nullalbe`和`@NonNull`用来检查变量, 参数和方法返回值为null与否.

`@NonNull`表示变量, 参数或返回值不能为null, 如果为null了编译器会给出警告.

`@Nullalbe`表示可能为null, 用这个注解的时候表示代码中应该加上null check.

### Resource Annotations
因为资源号都是int值, 所以如果你把一个drawable的int传给一个期待string resource的代码, 编译器是会接受的.
资源注解就是用来做这种情况的类型检查的.

比如用`@StringRes`来标记参数, 如果你传入一个drawable的id, IDE就会把它标记出来.

每一个Android的资源类型都有一个对应的资源注解, 比如类型是`Foo`, 那么对应的资源类型注解就是`FooRes`.

有一个特殊的`@AnyRes`, 用来表示任意的资源类型.

### Thread Annotations
这种注解用来检查方法是不是在特定的线程调用的. 有:
- `@UiThread`
- `@MainThread`
- `@WorkerThread`
- `@BinderThread`

`@UiThread`和`@MainThread`是可互换的.

如果一个类中的所有方法都是从同样的线程调用搞得, 那么可以直接把类给标记了.

### Value Constraints Annotations
`@IntRange`, `@FloatRange`和`@Size`注解是用来验证参数的值的. 比如`@IntRange`就验证参数是在一个给定的int范围之内.
比如下面的方法确保传入的参数是0到255:
```java
public void setAlpha(@IntRange(from=0, to=255) int alpha) {
    //set alpha
}
```
相应的`@FloatRange`检查参数是在一个指定范围内的浮点数.

`@Size`注解是用来检查集合的大小, 还有字符串的长度. 比如`@Size(min=1)`用来检查集合不为空, `@Size(2)`检查集合有两个元素.

### CheckResult Annotations
这个注解是用来检查一个方法的返回值确实被使用了. 
一个比较好的例子是`String.trim`方法, 当这个方法被`@CheckResult`标注, 如果它的返回值没有被使用, IDE就会报错.

另外一些比较值得看的注解有`@CallSuper`, `@Keep`和`@RequiresPermission`.
可以直接查看support annotations的[reference](https://developer.android.com/reference/android/support/annotation/package-summary.html).

其他参考资料:

[Improve code Inspection with Annotations](https://developer.android.com/studio/write/annotations.html#adding-nullness)

[Support Annotation documentation](http://tools.android.com/tech-docs/support-annotations)

[Improving your code with android support annotations](http://michaelevans.org/blog/2015/07/14/improving-your-code-with-android-support-annotations/)

## [ActivityTestRule: Espresso's Test "Lifecycle"](https://jabknowsnothing.wordpress.com/2015/11/05/activitytestrule-espressos-test-lifecycle/)
作者这篇文章的目的是讲讲用Espresso的`ActivityTestRule`写的测试中的操作顺序, 讨论像`beforeActivityLaunched()`, `afterActivityLaunched()`, 和 `afterActivityFinished()`这些方法相对于测试和Activity的生命周期都是什么时候被调用的. 

首先作者介绍了Espresso 2.0及之前的旧的`ActivityInstrumentationTestCase2`.

### `ActivityTestRule`的"生命周期".
新的写法是这样:
```java
@RunWith(AndroidJUnit4.class)
public MyNewTest {
  @Rule
  public MyCustomRule<MyActivity> testRule = new MyCustomRule<>(MyActivity.class);
  
  @Test
  public void testStuff() {
    // Wow where's all the boilerplate code?
    
    // Verify Oscar Grouch is no longer grouchy.
  }
}
```
而其中的MyCustomRule:
```java
public class MyCustomRule<A extends MyActivity> extends ActivityTestRule<A> {
  public MyCustomRule(Class<A> activityClass) {
    super(activityClass);
  }
  
  @Override
  protected void beforeActivityLaunched() {
    super.beforeActivityLaunched();
    // Maybe prepare some mock service calls
    // Maybe override some depency injection modules with mocks
  }
    
  @Override
  protected Intent getActivityIntent() {
    Intent customIntent = new Intent();
    // add some custom extras and stuff
    return customIntent;
  }
  
  @Override
  protected void afterActivityLaunched() {
    super.afterActivityLaunched();
   // maybe you want to do something here 
  }
    
  @Override
  protected void afterActivityFinished() {
    super.afterActivityFinshed();
    // Clean up mocks
  }
}
```
### `ActivityTestRule`: launchActivity=false;
`ActivityTestRule`的第三个参数允许开发者明确指定对每一个test case启动一个Activity.

`public ActivityTestRule(Class activityClass, boolean initialTouchMode, boolean launchActivity)`.

把第三个参数设置为false, 就可以写出这样的测试:
```java
@RunWith(AndroidJUnit4.class)
public class MultipleIntentsTest {
  @Rule
  public ActivityTestRule<MyActivity> testRule = new ActivityTestRule<>(MyActivity.class,
          false,    // initialTouchMode
          false);  // launchActivity. False to set intent per test);

  @Test
  public void testOscarGrouchy() {
    Intent grouchyIntent = new Intent();
    // intent stuff
    grouchyIntent.putExtra("EXTRA_IS_GROUCHY", true);
    testRule.launchActivity(grouchyIntent);
    // verify Oscar is grouchy
  }
  
  @Test
  public void testOscarNotGrouchy() {
    Intent happyIntent = new Intent();
    // intent stuff
    happyIntent.putExtra("EXTRA_IS_GROUCHY", false);
    testRule.launchActivity(happyIntent);
    // verify Oscar is not grouchy
  }
}
```

这个lauchActivity的值默认是为true. 设置为false之后我们在每一个test case里面自己启动activity. 对生命周期产生的影响作者也做了图对比分析, 见原文.

作者最后还讲了几个点, 关于测试迁移, 以及Activity的生命周期方法中启动Intent相关的需要注意的地方.

## [People and resources to learn Android programming from](https://m.signalvnoise.com/my-favorite-people-and-resources-to-learn-android-programming-from-293f249e2b4e#.6j6qqirem)
作者分享了关于Android编程学习中他积累的人和资源.

### Twitter
[Chiu-Ki Chan](https://twitter.com/chiuki)

[Donn Felker](https://twitter.com/donnfelker), 他的博客: [blog](http://www.donnfelker.com/). 他和[Kaushik Gopal](https://twitter.com/kaushikgopal)一起弄了[Fragmented Podcast](http://fragmentedpodcast.com/). 这里还有一个视频教程的网站: [Caster.io](https://caster.io/).

[Jake Wharton](https://twitter.com/JakeWharton), 这个大家都知道啦, 这是他的博客: [blog](http://jakewharton.com/).

[Kristin V Marsicano](https://twitter.com/kristinmars), 这里有一个她的关于Activity生命周期的演讲[Activities in the Wild](https://realm.io/news/activities-in-the-wild-exploring-the-activity-lifecycle-android/), 可能有一些你没有想过的东西.

[Ryan Harter](https://twitter.com/rharter)

[The Practical Dev](https://twitter.com/ThePracticalDev)

最后这有一个列表: [Tweet Android List](https://twitter.com/dankim/lists/androids)

### Podcasts
[Fragmented](http://fragmentedpodcast.com/) 两个独立开发者办的.

[Android Developers Backstage](http://androidbackstage.blogspot.hk/) 写Android的那些人办的.

[Material](https://www.relay.fm/material) 这不是一个技术广播, 讲一些Google新闻.

### Videos
[Caster.io](https://caster.io/)

[Realm.io](https://realm.io/news/)

[Android Dialogs (YouTube)](https://www.youtube.com/channel/UCMEmNnHT69aZuaOrE-dF6ug)

### Newsletters
[Android Weekly](http://androidweekly.net/)

[Kotline Weekly](http://us12.campaign-archive2.com/home/?u=f39692e245b94f7fb693b6d82&id=93b2272cb6)

### General Reading
Medium: 标签[androiddev](https://medium.com/tag/androiddev)和[android-app-development](https://medium.com/tag/android-app-development)

### Conferences
[GoogleIO](https://events.google.com/io2016/)

[360|AnDev](https://360andev.com/)

[Droid Con NYC](http://droidcon.nyc/)

## [ThirtyInch - a new MVP library for Android](https://medium.com/@passsy/thirtyinch-a-new-mvp-library-for-android-bd1a27262fd6#.dvjdbuvfx)

近年来MVP已经变成了Android社区中很流行的一种设计模式, MVC和MVVM也有人用. 这些模式的共同点就是把业务逻辑从Activity中抽取来.

这样做的好处首先是我们可以尽量把需要测试的逻辑用JVM上的单元测试测, 而不是用模拟器上的androidTests.
当然有些需要UI测试的地方仍然会用Espresso.

作者介绍了他们的MVP库: [ThirtyInch](https://github.com/grandcentrix/ThirtyInch/).

这个库开始的时候[mosby](https://github.com/sockeqwe/mosby)还没有release, 建议读一下mosby作者关于MVP的文章: [mosby](http://hannesdorfmann.com/android/mosby), 其中关于passive View的概念也在ThirtyInch中用到.

### ThirtyInch:
[ThirtyInch](https://github.com/grandcentrix/ThirtyInch/).

有`TiPresenter`, `TiView`.
`TiView`是一个接口, 可以被attach和detach.

`TiPresenter`有四个生命周期事件:
- `onCreate()`: 初始化的时候调用一次, 此时view还没有attach.
- `onWakeUp()`: view attach了, 并且对用户变为可见.
- `onSleep()`: 在这个调用之后, view将被detach, 并且变为对用户不可见.
- `onDestroy()`: 在Activity/Fragment完全销毁的时候调用一次.


`onWakeUp()`和`onSleep()`对应了`onStart()`和`onStop()`, onResume/onPause没有对应的回调支持, 因为这些生命周期回调应该在View层处理, 见: [Presenters dont need lifecycle](http://hannesdorfmann.com/android/presenters-dont-need-lifecycle)

### ThirtyInch有什么不同
- 可配置. 可以传`TiConfiguration`对象给TiPresenter, 去掉一些features.
- 所有Presenter的生命周期都按照正确的顺序调用, `onCreate()`和`onDestroy()`只调用一次.
- 不依赖RxJava. 它有一个独立的Rx module.
- View接口的方法注解. 比如`@CallOnMainThread`和`@DistinctUntilChanged`.
- Public API. 一些基层API可以被所有人利用起来.
- 不用继承`TiActivity`. 你可以利用[CompositeAndrodi](https://github.com/passsy/CompositeAndroid), 把plugin module作为你的依赖, 然后把`TiActivityPlugin`加到你的Activity.


之后作者举了一个Hello World的例子, 附图讲解很好, 见原文.

### 不是严格的MVP, MVVM也可以
作者又列出了一个ViewModel的图.
ViewModel存储了当前UI的状态数据.
当ViewModel中的数据变化时立即应用到View, 这里`@DistinctUtilChanged`的使用避免了数据不变时候的重复操作.

### 测试, Keep Android At Arm's Length
MVP的初衷之一是为了方便写测试, 因为Android SDK中的一些方法和类不好mock, 所以Presenter中应该是没有Android相关的东西的, 比如Context和Fragment等, Presenter只知道View接口和其中的方法, 是纯java的.

这就是"Keep Android At Arm's Length."的意思, 不要把Android和逻辑代码绑在一起, 库的名字ThirtyInch也是来自于这个原则, 因为三十寸是人类手臂, 肩膀到手指的平均长度.

### How does the Presenter survive the configuration change?
Activity在屏幕方向旋转时会被重建. 此时没有被序列化保存的信息就会丢失, 网络请求要么被取消, 要么被忽略, 重新请求.

序列化数据会费时, 而且在这种情况下, 序列话的数据几秒之后就要被反序列化.

Android Framework提供了两个方法来避免这种不必要的序列化:
- Fragment的`setRetainInstance(true)`, 之前的那个Fragment实例会被保存.
- 使用`Activity#onRetainNonConfigurationInstance()`和`Activity#getLastNonConfigurationInstance()`来存储和恢复对象. 这也是Android保存上面retained Fragments的方法. 这个方法最近被废弃了.

在Java中还有一个比较简单粗暴的方法是保存一个应用级别的单例.

ThirtyInch使用了上述的三种方法来确保TiPresenter在configuration变化的时候不死. 单例的解决方法尽在一些边缘情况必须.

当使用`TiPresenter`的时候不需要再实现`onSaveInstanceState(Bundle)`方法了, 因为数据都存在Presenter中.

## [Firebase Analytics VS Google Analytics](https://medium.com/google-developer-experts/firebase-analytics-vs-google-analytics-b2010f34d2bb#.cjqb9n505)
作者之前对比过 [Firebase Crash Reporting和Crashlytics](https://medium.com/google-developer-experts/firebase-crash-reporting-vs-crashlytics-a6c287c4b792#.m4ubzrcds).

在这篇文章里, 作者对比Firebase和Google的分析工具, 下面简称FA和GA.

首先GA是2005年就推出了, 那时候根本没有Android和iOS, 所以GA最开始是网站用的, FA是今年推出的, 从一开始目标就是移动应用.

然后作者总结了FA的优势和当前存在的几个不足.

最后的结论就是:
如果你只有app, 用FA; 如果你只有网站, 用GA; 如果你两个都有, 则两个都用.

## [VectorDrawable Fill Windings](https://blog.stylingandroid.com/vectordrawable-fill-windings/)
作者有一个Sketch做的资源, 是一个空洞图, 结果放在程序里看的时候中间的洞没有了, 变成实心的了.

作者分析并详细解释了出现这种问题的原因, 并提供了两种解决方案.

# DESIGN
## [Basic Patterns for Mobile Navigation](https://uxplanet.org/basic-patterns-for-mobile-navigation-d12a87686efe#.1jleu0y6v)

这篇文章作者分析了三种导航模式: hamburger menu, tab bar和gesture-base navigation.

### Hamburger Menu
Pros: 导航选项多, 设计干净, 给主要内容留出了更多空间.

Cons: 不易被发现. 
在iOS实现时, 和iOS的基本导航元素冲突.
hamburger的icon并没有给出上下文.
需要点两下才到目标页面.

Tips: 给选项排列优先级. 如果你的高优先级选项不多, 可以考虑用tabs或者tab bar. 重审你的信息结构, 有没有必要划分成多个简单的app.

### Tab Bar
Pros: Tab bar可以反映出当前在哪. 它们是永久存在的, 用户可以单击访问.

Cons: 有限的选项数. iOS和Android可能会有不同的设计规范.

Tips: 让点击区域足够大. icon要经过可用性测试. icon和label一起用.

### Gesture-Based Navigation
Pros: 移除了UI杂项, 节约了屏幕空间. 自然的人机交互接口.

Cons: 导航不可见. 增加了用户教育成本.

Tips: 确保不要必须教给用户一种全新的交互方式, 设计相似的体验. 使用过程动画的形式教用户如何使用. 

## [Design Reviews: Going beyond the surface](https://design.google.com/articles/going-beyond-the-surface/)
关于设计的review.

# LIBRARIES & CODE
## [green-coffee](https://github.com/mauriciotogneri/green-coffee)
一个Android库, 让你可以在instrumentation测试中跑Cucumber.

## [ThirtyInch](https://github.com/grandcentrix/ThirtyInch/)
一个Android MVP库.

# Tools
## [Exynap](http://exynap.com/)
一个Android Studio插件, 可以生成实现代码.

