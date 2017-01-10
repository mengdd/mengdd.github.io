---
title: Android Weekly Notes Issue 239
date: 2017-01-10 14:10:31
tags: [Andorid, Android Weekly, Android Things, Gradle, Looper, Handler, HandlerThread, Wear, PDF, Test, Robolectric, RxJava, RxJava2]
categories: [Android Weekly]
---
# Android Weekly Issue #239
January 8th, 2017
[Android Weekly Issue #239](http://androidweekly.net/issues/issue-239)
本期内容包括: Android Things开发; Android中有用却不常见的一些API介绍(拼写检查, 文字识别, 时间log, 截图, 创建PDF); Gradle依赖管理冲突和解决办法; Looper, Handler和HandlerThread; 兼顾Wear1.0和2.0的部署方式; 打开PDF的实现方法; 单元测试的命名; Robolectric的弊病; 迁移到RxJava2的好处和面临的挑战. 

<!-- more -->

# ARTICLES & TUTORIALS
## [Preparing your computer for Android dev](https://medium.com/@rafael_toledo/preparing-your-android-environment-for-development-android-tutorials-pt-1-5f76ca2b8a32#.yxa0bvkp5)
在Windows, OSX和Linux上设置开发环境.

## [Beginner’s guide to Raspberry Pi 3 B and Android Things](http://www.andtuts.com/a-beginners-guide-to-raspberry-pi-3-b-and-android-things/)
如何Set up with Android Things.

## [Creating new project and emulator on Android Studio](https://medium.com/@rafael_toledo/creating-a-new-project-and-an-emulator-on-android-studio-android-tutorials-2-35bd965ac42b#.lwlhy72wf)
如何创建新项目和模拟器.

## [Discovering Android API](https://blog.autsoft.hu/discovering-the-android-api-part-1/)
Android是基于Java的, Java本身已经有四千多个类, Android API也有很多个类, 有一些不太常见但是却很有用的API我们应该了解一下.

这篇文章就旨在发现那些不常见却很有用的API, 并且附有一个[Demo](https://github.com/peekler/GDG).

### No.1 - Spell Checker
[TextServicesManager](https://developer.android.com/reference/android/view/textservice/TextServicesManager.html)从API 14开始支持, 可以发现拼写错误, 并返回正确的单词拼写.
```java
TextServicesManager tsm = (TextServicesManager) getSystemService(Context.TEXT_SERVICES_MANAGER_SERVICE);  
SpellCheckerSession spellCheckerSession = tsm.newSpellCheckerSession(null, null, this, true); 
```

### No.2 - Text Recognizer
Google Play Services Vision API中的文字识别.

Version API中包括了人脸识别, 二维码扫描和文字识别.

这个例子中可以从图像中扫描出文字信息.

### No.3 - TimingLogger
[TimingLogger](https://developer.android.com/reference/android/util/TimingLogger.html)是用来测量流逝时间的一个好工具.
```java
TimingLogger timings = new TimingLogger(TAG, "methodA");
// ... do some work A ...
timings.addSplit("work A");
// ... do some work B ...
timings.addSplit("work B");
// ... do some work C ...
timings.addSplit("work C");
timings.dumpToLog();
```
最后一句执行后, 会在log中一次性输出下面的log:
```
D/TAG     ( 3459): methodA: begin
D/TAG     ( 3459): methodA:      9 ms, work A
D/TAG     ( 3459): methodA:      1 ms, work B
D/TAG     ( 3459): methodA:      6 ms, work C
D/TAG     ( 3459): methodA: end, 16 ms
```
注意需要在adb中使能TAG: 
```
setprop log.tag.TAG_MYJOB VERBOSE 
```
假设本例子中TAG = "TAG_MYJOB".

### No.4 - Taking screenshots
有一些库可以提供截屏, 比如[Falcon](https://github.com/jraska/Falcon). 

在Android 21之上用[MediaProjection](https://developer.android.com/reference/android/media/projection/MediaProjection.html)甚至可以录屏.

但是得到屏幕图像更简单的一种方法是:
```java
View viewRoot = getWindow().getDecorView().getRootView();  
viewRoot.setDrawingCacheEnabled(true);  
Bitmap screenShotAsBitmap = Bitmap.createBitmap(viewRoot.getDrawingCache());  
viewRoot.setDrawingCacheEnabled(false);  
// use screenShotAsBitmap as you need
```

### No.5 - PDF Creation API
从API 19开始Android就提供了API可以创建PDF文档.
文中的例子是创建了一个PDF文档, 然后把当前屏幕内容放进去.

## [Avoiding Conflicts in android gradle dependencies](https://blog.mindorks.com/avoiding-conflicts-in-android-gradle-dependencies-28e4200ca235#.x66q6p4v4)
如果两个依赖又都依赖了同一个库, 但是是不同的版本, 那会发生什么呢?

比如下面这个例子:
```
androidTestCompile 'junit:junit:4.12' //(Depends on version 1.3)
androidTestCompile 'org.mockito:mockito-core:1.10.19' //(Depends on version 1.1)
```
这两个库都依赖于"org.hamcrest:hamcrest-core", 但是版本却不同.

这种情况下, 最终被包含进build的是最高版本的库.

在依赖被声明的module里运行下面这个命令:
```
./gradlew dependencies.
```
会显式地看到gradle自动把第二个依赖中的hamcrest库从1.1升级到了1.3.

前面, 两个依赖都是test依赖, 所以gradle自动解决了冲突.  如果两个依赖属于不同的配置, 如, 把第一个`androidTestCompile`改为`compile`, gradle将会报错.

原因是:
> When instrumentation tests are run, both the main APK and test APK share the same classpath. Gradle build will fail if the main APK and the test APK use the same library (e.g. Guava) but in different versions. If gradle didn’t catch that, your app could behave differently during tests and during normal run (including crashing in one of the cases).


**解决依赖冲突**
一旦有了依赖冲突, 就需要开发者决定最后到底用什么版本的库, 有几种解决方法:
- 方法1: 从依赖中排除这个库.
```
// Now junit will not include hamcrest library. Therefore there will be no 
//dependency conflict. 
 compile ('junit:junit:4.12'){
    exclude group: 'org.hamcrest', module:'hamcrest-core'
}
```
或者
```
androidTestCompile ('org.mockito:mockito-core:1.10.19'){
    exclude group: 'org.hamcrest', module:'hamcrest-core'
}
```
真实的项目中可能有多个依赖依赖于同一个库, 那么我们就需要写多个排除语句.

- 方法2: 显式地声明冲突库的版本.
```
compile 'junit:junit:4.12' //(Depends on version 1.3)
androidTestCompile 'org.mockito:mockito-core:1.10.19' //(Depends on version 1.1)
androidTestCompile 'org.hamcrest:hamcrest-core:1.3' //(We explictly mention 
//that include version 1.3)
```
这种方法不用写exclude, 但是当升级直接依赖库的时候需要注意更新这种间接依赖库的版本.

- 方法3: 强制解析.

强制设置所有configuration的依赖版本:
```
android {
    configurations.all {
        resolutionStrategy.force 'org.hamcrest:hamcrest-core:1.1'
    }
}
```
使用这种方法的时候需要小心, 因为与方法2不同, 它是把所有configuration中的版本都强制设置了. 当直接依赖更新的时候, 要注意更新间接依赖的版本.

## [Looper, Handler, and HandlerThread](https://blog.mindorks.com/android-core-looper-handler-and-handlerthread-bd54d69fe91a#.zf2nhog1x)
这篇文章讨论Looper, Handler和HandlerThread.

首先, 读者应该了解Java的[Thread](https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.html)和[Runnable](https://docs.oracle.com/javase/7/docs/api/java/lang/Runnable.html).

然后让我们带着问题来探索和复习:
### Java的Thread存在什么问题?
Java的Thread是一次性的, 当它执行完`run()`方法之后, 它就死了.
### 我们可以改善这个问题吗? 
线程本身是个双刃剑, 我们可以把任务分发到多个线程来加速, 但同时线程过多又会降低速度. 线程创建也会花费时间, 所以最好我们能有一个固定优化数量的线程, 然后用它们来执行任务.

**线程复用模型**:
- 1.一个线程保持活跃, 通过它的`run()`方法不断循环.
- 2.任务由该线程连续执行, 并保持在一个队列中(MessageQueue).
- 3.当完成之后结束这个线程.

### Android是以什么方式来做这件事的呢?
Android用`Looper`, `Handler`和`HandlerThread`实现了上述模型.
系统可以用这样的图表示:
![Android Looper Handler](/images/Android-Looper-Handler.jpeg)

- 1.`MessageQueue`是一个队列, 里面含有需要被处理的任务(消息).
- 2.`Handler`利用`Looper`往队列中加任务, 同时也在任务出队列的时候进行处理.
- 3.`Looper`是一个工人, 保持一个线程的活跃, 循环消息队列, 把消息发给对应的handler去处理.(一个线程只能对应一个唯一的`Looper`, 但是可以有多个关联的`Handler`).
- 4.最后`Looper.quit()`会让线程终止.

为一个线程创建`Looper`和`MessageQueue`:
```java
class LooperThread extends Thread {
    public Handler mHandler; 

    public void run() { 
        Looper.prepare();

        mHandler = new Handler() { 
            public void handleMessage(Message msg) { 
               // process incoming messages here
               // this will run in non-ui/background thread
            } 
        }; 

        Looper.loop();
    } 
}
```

创建`Handler`的时候自动和当前线程的Looper关联, 但是也可以通过构造传入Looper来使Handler关联到特定的线程.

用`Handler`发消息有两种方式: `Message`和`Runnable`.

自己创建一个线程并提供Looper和消息队列的方式是不好的, 所以Android提供了`HandlerThread`来简化这个过程, 它内部的实现和我们之前做的差不多, 但是是以一种更加稳健的方式. 所以我们应该使用`HandlerThread`而不是自己实现.

大多数时候你只需要继承`HandlerThread`来创建它的子类.
```java
private class MyHandlerThread extends HandlerThread {

    Handler handler;

    public MyHandlerThread(String name) {
        super(name);
    }

    @Override
    protected void onLooperPrepared() {
        handler = new Handler(getLooper()) {
            @Override
            public void handleMessage(Message msg) {
                // process incoming messages here
                // this will run in non-ui/background thread
            }
        };
    }
}
```
注意:
- `Looper`只有在`HandlerThread`的`start()`方法被调用(线程开始跑)后才会进入prepared状态.
- 只有在`HandlerThread`的`Looper`处于parepared状态以后, `Handler`才可以关联.

另一种创建`HandlerThread`的方式:
```java
HandlerThread handlerThread = new HandlerThread("MyHandlerThread");
handlerThread.start();
Handler handler = new Handler(handlerThread.getLooper());
```
注意: `HandlerThread`需要调用`quit()`方法来停止线程执行和释放资源.

作者文后还附有一个练习的demo.

## [Android Wear packaging](https://www.novoda.com/blog/android-wear-packaging/)
Android Wear 2.0将在2017年发布, 本文讨论了如何同时支持1.0和2.0的应用部署.

## [Simple Things – Part 1](https://blog.stylingandroid.com/simple-things-part-1/)
Android Things应用.

## [Options for Viewing PDFs](https://commonsware.com/blog/2017/01/04/options-viewing-pdfs.html)
显示PDF传统的方法是通过ACTION_VIEW发出去, 用一个第三方应用打开.

但是有一些开发者不愿意这样做.

还有一种方法是把文件上传, 然后用Google Docs URL用WebView打开它. 

除了这些传统的方法, 还有一些选项, 虽然它们各自都有一些问题.

比如用`PdfRenderer`, 它是Android 5.0加入的.

Mozilka在Firefox上用的是[PDF.js](https://mozilla.github.io/pdf.js/), 在Android 4.4+的WebView上可用, 它会给apk带来2MB左右的增加.

Google在Chrominum上用的是[pdfium](https://pdfium.googlesource.com/pdfium/), 这是C++. [barteksc/AndroidPdfViewer](https://github.com/barteksc/AndroidPdfViewer)封装了Pdfium, 处理了渲染和基本的手势, 在一些较老的Android版本上也使用, 但是大约每个CPU架构会给APK增加5MB, 默认情况下你会增加30MB.

## [Clean tests, Part 1: Naming](https://android.jlelse.eu/clean-tests-part-1-naming-cce94edf0522#.5nmmqx81y)
关于怎么写干净的单元测试, 本文作者提出了几点他对于命名的看法.
- 不要用"test"开头写测试名.
- 不要把被测试的方法名字写在测试名里.
- 测试是一种规范.

## [Why I Don't use Robolectric](http://www.philosophicalhacker.com/post/why-i-dont-use-roboletric/)
作者觉得Robolectric不好的几点:
- Robolectric mock了一些我们并不拥有的type.
- Robolectric turns TDD on its head.

所以最好的做法是我们在写Android代码的时候将逻辑代码抽象出来, 与framework分离, 这样在测试的时候不依赖于Android SDK的类, 也不需要robolectric来模拟一个中间层.

## [The Next Step for Reactive Android Programming](http://futurice.com/blog/the-next-step-for-reactive-android-programming)
RxJava 2已经推出了, 本篇文章讨论从RxJava 1迁移到RxJava 2会带来的好处和挑战.

好处:
- 兼容了[Reactive Streams](https://github.com/reactive-streams/reactive-streams-jvm).
- Backpressure的处理.
- Performance.

挑战:
- 流中不能再使用null.
- 方法数限制. RxJava 1(5500), RxJava 2(9200).
- 自定义操作符变得很难写.

Note: 可以对比简单的操作符[map](https://github.com/ReactiveX/RxJava/blob/2.x/src/main/java/io/reactivex/internal/operators/observable/ObservableMap.java)和复杂的操作符[flatMap](https://github.com/ReactiveX/RxJava/blob/2.x/src/main/java/io/reactivex/internal/operators/observable/ObservableFlatMap.java).

# DESIGN
## [Introducing Auto-Layout for Sketch](https://medium.com/sketch-app-sources/introducing-auto-layout-for-sketch-24e7b5d068f9#.tusju2z7k)
一个Sketch插件, 让你查看不同屏幕上的效果.

## [Designing for Both Android and iOS](https://webdesign.tutsplus.com/articles/a-tale-of-two-platforms-designing-for-both-android-and-ios--cms-23616)
Android和iOS的设计, 很详细的介绍, 最后附有一些resources.

# LIBRARIES & CODE
## [flowless](https://github.com/Zhuinden/flowless)
一个框架, 给你Activity的UI状态命名, 管理并记录状态转换.

## [Store](https://github.com/NYTimes/Store)
异步数据加载和缓存的库.