---
title: Android Weekly Notes Issue 226
date: 2016-10-12 17:46:27
tags: [Android, Android Weekly, Firebase, RxJava, Animation, MVP, Proguard, Data Binding, Mockito, Kotlin, Gradle]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #226
October 9th, 2016
[Android Weekly Issue #226](http://androidweekly.net/issues/issue-226)

本期内容包括: 用Firebase做A/B Test; 用RxJava做动画; MVP; proguardFiles; RxJava和Android Data Binding的结合; Mockito的更新; Gradle configurations等.

<!-- more -->

# ARTICLES & TUTORIALS
## 用Firebase做A/B Test [A/B Test your App using Firebase Remote Config](https://riggaroo.co.za/ab-test-app-firebase-remote-config/)
作者讲了如何用Firebase的Remote Config做A/B Test.

## 用RxJava做动画 [Android animations powered by RxJava](https://pspdfkit.com/blog/2016/android-animations-powered-by-rx-java/)

**动画基础**

用
[ViewPropertyAnimator](https://developer.android.com/reference/android/view/ViewPropertyAnimator.html) 操作View的属性动画很容易也很方便.

本文讲的内容主要用[ViewPropertyAnimatorCompat](https://developer.android.com/reference/android/support/v4/view/ViewPropertyAnimatorCompat.html), 它是通过这个方法获得的: [ViewCompat.animate(targetView)](https://developer.android.com/reference/android/support/v4/view/ViewCompat.html#animate(android.view.View)).

它是这样用的: 
```kotlin
ViewCompat.animate(someButton)
    .scaleX(0f)                         // Scale to 0 horizontally
    .scaleY(0f)                         // Scale to 0 vertically
    .setDuration(300)                   // Duration of the animation in milliseconds.
    .withEndAction { removeView(view) } // Called when the animation ends successfully.
```


[Completable](http://reactivex.io/RxJava/javadoc/rx/Completable.html) 是RxJava1.1.1加入的.


作者通过RxJava来做他们的动画效果.
这在链式连接多个动画和其他操作的时候很有用.

## [Android Architecture Patterns Part 2: MVP](https://upday.github.io/blog/model-view-presenter/)
关于Android程序的架构, google提供了[Android Architecture Blueprints](https://github.com/googlesamples/android-architecture), 其中作者他们合作于[MVP & RxJava](https://github.com/googlesamples/android-architecture/tree/todo-mvp-rxjava/)的sample.

**MVP(Model-View-Presenter)模式**:
- Model: 数据层. 负责与网络层和数据库层的逻辑交互.
- View: UI层. 显示数据, 并向Presenter报告用户行为.
- Presenter: 从Model拿数据, 应用到UI层, 管理UI的状态, 决定要显示什么以及响应用户的行为.

V和P联系紧密, 所以它们通常会持有对方的引用. 为了给P做单元测试, V是一个抽象的接口. P和对应的V的关系定义在一个`Contract`接口里, 这样可以让代码可读性更好, 更容易发现二者的联系.

**MVP模式 & RxJava在Android Architecture Blueprints里的应用**

Google blueprint的Sample是一个[To Do应用](https://github.com/googlesamples/android-architecture/wiki/To-do-app-specification). 它让用户可以创建, 阅读, 更新和删除to do task, 也可以过滤显示. RxJava主要是用来进行一些非主线程的异步操作.

然后作者详细说明了代码实现.

**Model**中用RxJava在本地和网络取数据.
(他们的单元测试里是下划线和驼峰结合的方法命名方式.)

**View**有一个base接口:
```java
public interface BaseView<T> {
    void setPresenter(T presenter);
}
```

View在`onResume()`的时候调用Presenter的`subscribe()`, `onPause()`的时候调用Presenter的`unsubscribe()`. 如果View接口的实现不是Fragment或Activity, 而是Android的自定义View, 那么在`onAttachedToWindow()`和`onDetachedFromWindow()`方法里分别调用这两个方法.

View的测试是用Espresso写的.

**Presenter**也有一个base接口:
```java
public interface BasePresenter {
    void subscribe();
    void unsubscribe();
}
```
View和Model都通过构造函数传入Presenter, 在Presenter构造里还要调用View的`setPresetner()`方法.

每一个Presenter还要暴露一些其他的方法, 对应View中用户的行为.

**MVP模式的缺点**: 
MVP模式很好地分离了概念, 当然这是好的. 但是当开发很小的app或者只是做一个原型时, 确实感觉过度设计了. 为了减少所用的接口, 有一些开发者省去了`Contract`接口类, 也删掉了Presenter的接口.

当把UI的逻辑移到Presenter中时, 它就变成了一个全能的类, 代码很长. 为了解决这个问题, 可以进一步拆分代码, 并且记得创建单一职能, 并且可以被单元测试的类.

**结论**:
[Model-View-Controller MVC模式](https://upday.github.io/blog/model-view-controller/) 有两个主要的缺点: 首先, View持有Controller和Model的引用; 第二, 它没有把对UI逻辑的操作限制在单一的类里, 这个职能被Controller和View或者Model共享.

MVP模式解决了这两个问题: 砍断了View和Model之间的联系, 用Presenter来管理所有和View显示相关的逻辑(handles everything related to the presentation of the View), 并且这个类是很容易被单元测试的.

## [proguardFiles: A Cautionary Tale](https://stkent.github.io/2016/10/07/proguardfiles-a-cautionary-tale.html)
作者有三个buildTypes: debug, beta, release.

其中beta用了initWith(buildTypes.debug).
他想给不同的type加上不同的proguard files. 让debug不混淆(`-dontobfuscate`), beta和release混淆.
结果却发现beta没有混淆.

查看代码发现`proguardFiles`其实是将proguard files叠加.
作者找到的解决方式是用`setProguardFiles()`:
```
buildTypes {
  debug {
    // ...
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro', 'proguard-debug.pro'
  }

  beta {
    initWith(buildTypes.debug)
    // ...
    // New!
    setProguardFiles([getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'])
  }

  release {
    // ...
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
  }

}
```

评论区有人指出还可以这样:
在`defaultConfig`中:
```
proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
```
然后在`debug`中: 加`proguardFile 'proguard-debug.pro`, 这样更简洁一些.

相关文档: [BuildType](https://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html).

## [RxJava meets Android Data Binding](https://medium.com/tangoagency/rxjava-meets-android-data-binding-4ca5e1144107#.p13x1zwkc)
作者使用例子介绍了如何将RxJava和Android的Data Binding结合起来使用.


## [Mocking Kotlin with Mockito](http://hadihariri.com/2016/10/04/Mocking-Kotlin-With-Mockito/)
因为Kotlin默认类和方法都是final的, 如果你想要继承, 必须显式声明`open`.

当你想要在测试中Mock一些行为时, Mockito可能会报错, 因为它无法mock一个final的class/method.

于是你可能要修改源代码, 加`open`或者是接口, 仅仅是为了测试.

Mockito 2解决了这个问题: [What's new in Mockito 2](https://github.com/mockito/mockito/wiki/What's-new-in-Mockito-2).

你只需要在`resources/mockito-extensions`目录下创建一个文件: `org.mockito.plugins.MockMaker`.
里面只包含一行内容:
`mock-maker-inline`.


## [Droidcon NYC 2016 - Victor Nascimento](https://medium.com/@victor.nascimento/droidcon-ny-2016-e037cb81559#.4ncx0xcgg)
## [Droidcon NYC 2016 - Florina Muntenescu](https://upday.github.io/blog/droidcon_nyc/)
这两篇是关于Droidcon NYC 2016的感想.

## [Android Gradle configurations](http://www.thedroidsonroids.com/blog/android/android-gradle-configurations/)
Gradle中的configuration是:
```
dependencies {
	annotationProcessor 'com.jakewharton:butterknife-compiler:8.4.0'
	compile 'com.jakewharton:butterknife:8.4.0'
	compile project(':api')
	debugCompile 'com.squareup.leakcanary:leakcanary-android:1.4'
	releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.4'
	androidTestCompile 'com.android.support.test:runner:0.5'
	testCompile 'org.robolectric:robolectric:3.1.2'
	testAnnotationProcessor 'org.robolectric:robolectric-processor:3.1.2'
}
```

模式是`configuationName dependencyNotation`.

Configuration names由两部分组成:
- 可选的前缀, 指定build variant, product flavor或者build type.
- 必需的后缀, 指定scope.

比如在`debugCompile`中, debug就是一个build type.

`compile`没有前缀, 就表示它应用到所有的build类型里.

每一个正常的configuration都有一个相应的unit test版, 比如`testCompile`, `testDebugCompile`.

对于功能测试来说是`androidTest`, 只有这一种.

**Scope**

Scope是和configuration应用的阶段有关:
- annotationProcessor/kapt: 注解处理;
- provided/compileOnly: 编译期;
- compile: 编译 + 执行;
- apk: 执行期.

**继承**
Configuration可以继承, 意味着子类包含父类所有包含的项目.
比如`testCompile`就继承了`compile`.
但是注意继承必须显式声明, 并不是由名字看出来的, 比如`testAnnotationProcessor`没有继承`annotationProcessor`.

利用继承可以定义单元测试和公共测试的基类, 这样它们的共享依赖就可以只声明一次.
```
configurations {
 [androidTestCompile, testCompile].each { it.extendsFrom commonTestCompile }
}
```
# LIBRARIES & CODE
## [android-data-binding-rxjava](https://github.com/TangoAgency/android-data-binding-rxjava)
例子代码, 展示如何结合RxJava和Android data binding.

## [AnimatorDurationTile](https://github.com/nickbutcher/AnimatorDurationTile)
一个Quick Settings tile, 用于控制动画的duration scale.

## [DateTimeSeer](https://github.com/p-v/DateTimeSeer/)
一个关于日期和时间的自动提示输入框.

## [A list of all Android permissions](https://gist.github.com/Arinerron/1bcaadc7b1cbeae77de0263f4e15156f)
一个Android所有权限的列表.

# NEWS
## [What's new in Mockito 2](https://github.com/mockito/mockito/wiki/What%27s-new-in-Mockito-2)
Mockito 2发布了, 有什么新东西呢?

## [Kotlin 1.0.5 EAP](https://discuss.kotlinlang.org/t/kotlin-1-0-5-eap/2023)
Kotlin 1.0.5 EAP (Early Access Program).

## [What's next for android-apt?](http://www.littlerobots.nl/blog/Whats-next-for-android-apt/)
android-apt将不会再开发了, 因为它的功能已经被包含进了Android Gradle plugin.
