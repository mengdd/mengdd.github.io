---
title: Android Weekly Notes Issue 255
date: 2017-05-03 15:38:46
tags: [Android, Android Weekly, RxJava, Android O, Data Model, immutable, MVP, Dagger2, Kotlin, Gradle Plugin, Starter, Firebase, Real Time Database, Resources, Litho, Picasso]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #255
April 30th, 2017
[Android Weekly Issue #255](http://androidweekly.net/issues/issue-255)
本期内容包括: 一种在RxJava中显示loading/content/error的好的处理方法; Android O中的一些隐藏宝藏; Uber app的immutable的数据升级; MVP模式下, 不要再做`view != null`的判断了; 用Dagger2实现的依赖注入; 迁移应用到Kotlin; 如何把Gradle插件从Groovy迁移到Kotlin; Activity中的静态start方法使用; Firebase的实时数据库使用.  

<!-- more -->

# ARTICLES & TUTORIALS
## [LCE: Modeling Data Loading in RxJava](https://tech.instacart.com/lce-modeling-data-loading-in-rxjava-b798ac98d80)
作者介绍了一种方法, 用RxJava来处理显示loading/内容/错误的逻辑.

核心思想是中这个结构把数据包一层:
```java
// Lce -> Loading / Content / Error
class Lce<T> {
    public static <T> Lce<T> data(T data) {
        // implementation
    }

    public static <T> Lce<T> error(Throwable error) {
        // implementation
    }

    public static <T> Lce<T> loading() {
        // implementation
    }

    boolean isLoading();

    boolean hasError();

    Throwable getError();

    T getData();
}
```
然后实际处理的代码就变成了这样:
```java
  repository.getDataEventStream().subscribe({ event ->
    if (event.isLoading) {
      view.showLoading(true)
    } else if (event.hasError()) {
      view.showError(event.getError())
    } else {
      view.showData(event.getData())
    } 
  })
```
怎么构建这个Observable呢:
```java
Observable<Lce<Data>> getDataEventStream() {
  return api.getData()
    .map(data -> Lce.data(data))
    .startWith(Lce.loading())
    .onErrorReturn(e -> Lce.error(e))
}
```
更多构建方法见原文.


## [Hidden Gems of Android O](https://medium.com/@ianhlake/hidden-gems-of-android-o-7def63136629)
作者仔细看了Android O的[API Diff](https://developer.android.com/sdk/api_diff/o-dp1/changes.html), 然后发现了一些隐藏的宝藏拿出来分享.
- Storage Access Framework的改进.
- RecoverableSecurityException.
- SharedPreferences支持更换底层实现.
- `SmsManager.createAppSpecificSmsToken()`提供的更好的短信验证流.
- 锁屏情况下的显示处理: `Keyguard.dismissKeyguard()`.
- 全屏Activity的旋转处理.


## [Engineering Stability in Migrations](https://eng.uber.com/immutable-collections/)
Uber的数据类生成及迁移到Immutable Collections的过程.


## [Don’t put view != null checks in your Presenters](https://android.jlelse.eu/dont-put-view-null-checks-in-your-presenters-4b6026c67423)
如果你使用了MVP模式, 并且你的presenter在configuration变化时是一直存在的, 那么你的presenter至少会有下面两个方法:
```java
void attachView(View)
void detachView()
```
这样的话你的`getView()`方法应该被标记为`@Nullable`, 然后你就需要在很多地方做null判断, 即便有些地方你100%地肯定View肯定不为null. 

### Presenter的方法直接从View中被调用
比如那些View中UI控件点击导致的调用.

加个`view != null`的判断有一个缺点就是如果attach时出现了问题, 这时用户点击了按钮却没有反应, 这个错误会被忽略和隐藏起来. 在这种View应该存在的情形下, 如果得到了null, 应该及时抛出异常发现错误. 

`It’s always a bad sign when the else branch is missing.`

解决方案: 加个`@NonNull View getViewOrThrow()`方法:

```java
@Nullable
public MyView getView() {...} 

@NonNull
public MyView getViewOrThrow() {
    final MyView view = getView();
    if (view == null) {
        throw new IllegalStateException("view not attached");
    }
    return view;
}
```

### 在Presenter中异步调用View
很多时候我们需要异步调用View的方法, 这时候我们就不能用`getViewOrThrow()`了, 因为View被detach是一种合理的情况.

这时候我们如果加个`if (view != null)`是可以解决这个问题的, 但是却是一个错误的选择. 因为else分支的缺失, 用户可能错过了server返回的结果, 然后永远地等下去.

一个比较好的解决方案就是[ThirtyInch](https://github.com/grandcentrix/ThirtyInch), 它有一个方法叫[sendToView(ViewAction)](https://github.com/grandcentrix/ThirtyInch/blob/master/thirtyinch/src/main/java/net/grandcentrix/thirtyinch/TiPresenter.java#L491), 它会推迟`ViewAction`的执行, 到View再次被attach的时候执行. 如果View已经处于attached的状态, 那么就立即执行.

一个例子:
```java
public class MyPresenter extends TiPresenter<MyView> {

    public void onSubmitLogin(final Credentials credentials) {
        mLoginService.login(credentials).subscribe(
                success -> {
                    sendToView(view -> view.close());
                },
                error -> {
                    sendToView(view -> view.showError(error));
                });
    }
}
```

注意请不要过度使用`sendToView()`. 

如果你用MVI模式, 维护一个ViewModel, 在变化的时候渲染到View, 同样也可以删掉`view != null`的判断. 见[My take on Model View Intent (MVI) — Part 1: State Renderer](https://hackernoon.com/model-view-intent-mvi-part-1-state-renderer-187e270db15c).


### Optional和WeakReference
这篇文章中用了`view == null`作为View被detached了的依据. 如果你使用了其他的包装, 比如`WeakReference`或者`Optional`, 你虽然不用null判断了但是并不代表你解决了问题, 你需要做其他的判断并且lint不能帮你做提示了.


### 结论
你并不需要`if (view != null)`检查:
- 当你确定View是attached时, 使用`getViewOrThrow()`.
- 当View可能会是detached时, 使用`sendToView(ViewAction)`, 来支持else的处理.


## [Dependency Injection in Android with Dagger 2](https://www.raywenderlich.com/146804/dependency-injection-dagger-2)
一个用了Retrofit和MVP模式的应用, 用Dagger2做依赖注入的例子.


## [How we made Basecamp 3’s Android app 100% Kotlin](https://m.signalvnoise.com/how-we-made-basecamp-3s-android-app-100-kotlin-35e4e1c0ef12)
作者他们如何把应用改为用Kotlin.


## [Migrate a Gradle Plugin from Groovy to Kotlin](http://adavis.info/2017/04/migrate-a-gradle-plugin-from-groovy-to-kotlin.html)
如何把一个用Groovy写的Gradle插件转化成Kotlin写的.


## [Object Oriented Tricks: #4 Starter Pattern](https://hackernoon.com/object-oriented-tricks-4-starter-pattern-android-edition-1844e1a8522d)
在Activity中定义一个静态的`start()`方法, 把需要放在intent中的参数都当做方法参数传进来.

Android Studio对此有一个内置的模板, 你只要输入`starter`, 按回车就可以生成这个方法.


## [Using Firebase as a Real Time System](https://medium.com/google-developer-experts/using-firebase-as-a-real-time-system-d360265aa678)
Firebase的Real Time Database.
数据库存储的信息以NoSQL的形式放在Google Cloud上. 

三个主要的优点: 实时,离线处理, 自动同步.

文中展示了基本的用法.

之后提供了实时数据库的几种使用思路: 
- 实时通讯: ([Firechat](https://firechat.firebaseapp.com/)); 
- 显示实时位置的地图([Realtime geolocation tracking with Firebase](https://moquet.net/blog/realtime-geolocation-tracking-firebase/), [geofire](https://github.com/firebase/geofire)); 
- 排名.


# LIBRARIES & CODE
## [Bubble-Picker](https://github.com/igalata/Bubble-Picker)
气泡选择器.

## [UltimateAndroidReference](https://github.com/aritraroy/UltimateAndroidReference)
Android资源收集, 包括库, 开源项目, 书籍博客等等.

## [litho-picasso](https://github.com/charbgr/litho-picasso)
为Litho写的picasso库.


