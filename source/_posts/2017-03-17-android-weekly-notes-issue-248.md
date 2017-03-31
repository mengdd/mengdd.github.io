---
title: Android Weekly Notes Issue 248
date: 2017-03-17 10:51:51
tags: [Android, Android Weekly, Crash, Dagger, Dagger2, Realm, Mockito, MockWebServer, Google Map, JUnit 5, RxJava, RxJava2, FindBugs, Lint]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #248
March 5th, 2017
[Android Weekly Issue #248](http://androidweekly.net/issues/issue-248).
本期内容包括: 为什么有时候应该让你的应用崩溃(而不是一味保护); Trello离线模式实现中两个id的问题; 如何让Dagger的component按照scope保存, 在屏幕旋转时不重建; 用Dagger构建Realm的数据库迁移逻辑; 
利用各种mock工具写单元测试; Map上markers的动画实现; JUnit5中@DisplayName的使用; RxJava中的Single和Completable使用; 举例说明如何给FindBugs写自定义的探测器; Android中静态代码分析工具的使用; Trello离线实现中sync失败情况的处理.

<!-- more -->


# ARTICLES & TUTORIALS
## [Why your app should crash](http://jeroenmols.com/blog/2017/03/08/appcrash/)
作者认为有时候让应用崩溃反而是有好处的.

以NPE为例, 有时候我们会习惯性地加很多null判断, 有的是多余的, 有的防御型的代码反而会掩盖了真实问题所在. 比如当一个不合理的情况发生时, 让用户看到一个不可理解的页面, 比如空白, 然后我们开发者根本不知道这种情况的发生.

与其这样掩盖错误, 不如让应用崩溃, 让开发者立即知道问题的原因.

实用建议:
- 永远让应用对外来输入(比如service的响应, UI的输入, 进来的intents)保持健壮性.
- 在程序的入口点保证数据的完整性. 这样不合理的数据就不会到处都是, 所以你不用到处检查.
- 如果你不确定某个错误是否会在某个地方发生, 先假装它不会发生, 在测试阶段再验证.
- 如果某个方法在产品环境不能被调用, 或者只能被调用一次等, 抛出`IllegalStateException`.
- 永远在发布之前进行完整测试, 这样你就会在用户之前, catch住可怕的崩溃.

## [The Two ID Problem](http://tech.trello.com/sync-two-id-problem/)
还是Trello开发离线模式的系列文章, 本篇讲他们遇到的一个很tricky的问题: id问题.

在他们的项目里, 所有的models都有一个id, 用以和server通信, 以及定义model之间的关系.

如果是在离线模式下, 就不能依靠server来提供这个id, 客户端需要自己生成. 

所以离线模式下有两种id, 一种是本地生成的, 一种是用来和server通信的. 

他们想过几个办法, 比如在sync的时候将local的id转化为server id; 或者干脆存储一个id的Pair类, 但是都有难以维护或者性能缺陷等种种问题.

最后他们提出了一个叫`local-server barrier`的解决方案. 基本的原则就是, 在app中, 只使用local的id, 同server通信时, 使用server id. 好处: 首先保证了客户端代码的简洁, 只有网络通信层需要考虑到server id; 重构代码量小.


## [Retaining Dagger components](https://medium.com/@Zhuinden/retaining-dagger-components-across-configuration-change-using-service-tree-3709c78bf6d2#.114aardgd)
如果你用dagger创建了component,  scope是Activity或者Fragment, 那么你可能遇到过这个问题: 旋转屏幕之后, 所有的依赖都重建了, 因为你创建了一个新的component.

如果你想要在configuration变化的时候不重建, 你就需要把component存储在一个全局的地方, 但是这样的话, 当你真的结束你的Activity和Fragment的时候, 你如何释放这些component呢?

你需要分层地(hierarchical)存储, [service-tree](https://github.com/Zhuinden/service-tree)就是用来做分层存储东西的一个工具. 

文中基本思想是把Application的component作为根节点, 然后Activity和Fragment的component作为树形结构的叶子节点逐级存储. Activity和Fragment的节点什么时候移除, 有一些判断条件和时机选择, 详见原文代码.


## [The Burden of Knowledge](https://medium.com/@trionkidnapper/the-burden-of-knowledge-52cc73508081#.1nyndcoo6)
鼓励在team里分享知识.

## [Realm Migrations Supercharged with Dagger](http://www.adavis.info/2017/03/realm-migrations-supercharged-with.html)
使用Dagger2可以大幅度改善Realm中的数据迁移处理. 具体的做法是把每一步的迁移处理都放在一个统一接口的实现类里, 然后注入它们.
```java
@Module
public class MigrationsModule {
    @Provides
    @IntoMap
    @IntKey(1)
    static VersionMigration provideVersion1Migration() {
        return new Version1Migration();
    }

    @Provides
    @IntoMap
    @IntKey(2)
    static VersionMigration provideVersion2Migration() {
        return new Version2Migration();
    }
}
```
所以最后的迁移类看起来就是这样:
```java
@Reusable
public class Migration implements RealmMigration {
    private Map<Integer, Provider<VersionMigration>> versionMigrations;

    @Inject
    Migration(Map<Integer, Provider<VersionMigration>> versionMigrations) {
        this.versionMigrations = versionMigrations;
    }

    @Override
    public void migrate(final DynamicRealm realm, long oldVersion, long newVersion) {
        for (int i = (int) oldVersion; i < newVersion; i++) {
            final Provider<VersionMigration> provider = versionMigrations.get(i);
            if (provider != null) {
                VersionMigration versionMigration = provider.get();
                versionMigration.migrate(realm, i);
            }
        }
    }
}
```

这样做的好处:
- 1.以后再有数据库升级也不需要再改这个类了.
- 2.不需要逐个check每个版本号, 自动只从需要的版本号开始做迁移.
- 3.每个迁移模块都变成了可测试的单元.


我觉得作者的这种处理结构很好, 不仅仅限于Realm数据库的迁移, 其他的数据库迁移也可以用类似的结构来处理.



## [How to be a Mock-Star…](https://medium.com/fueled-android/how-to-be-a-mock-star-fc00714d8c2f#.aff30qzf6)
介绍如何用[mockito](http://site.mockito.org/)来测试一个MVP的程序.

首先测试Presenter的部分, 这里Mock了数据源和各种错误响应.

测试Repository, 需要用到[MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver), 来模拟测试环境下的响应.

## [Animating Markers with MapOverlayLayout](https://www.thedroidsonroids.com/blog/workcation-app-part-2-animating-markers-with-mapoverlaylayout/)
作者App的动画实现讨论第二发, 如何让地图上的markers带缩放和渐变动画 -> 用`MapOverlayLayout`.

文中有详细的实现代码, 基本思路就是在这个`MapOverlayLayout`中保存一个View的列表, 然后在自定义View中实现每个marker在相应动作时的动画.


## [What Unit Tests are Trying to Tell us About Activities Pt 2](https://www.philosophicalhacker.com/post/what-unit-tests-are-trying-to-tell-us-about-activities-pt-2/)
以Activity/Fragment作为基本构建单元, 让程序难以测试, 本文举例说明了这一点.


## [JUnit 5: DisplayName](https://blog.stylingandroid.com/junit-5-displayname/)
JUnit 5提供了`@DisplayName`, 这样测试报告里case显示的名字将是`@DisplayName`定义的字符串. 

相比原先的方法名来说, 这个字符串是可以带空格的, 所以比之前的可读性增强了.


## [Clearer RxJava intentions with Single and Completable](https://medium.com/@ValCanBuild/making-your-rxjava-intentions-clearer-with-single-and-completable-f064d98d53a8#.kpyh1sal0)
RxJava中我们经常用到的类就是`Observable`, 然后处理三个事件: `onNext()`, `onError()`和`onCompleted()`.

但是有些时候我们并不需要关心全部这三个事件, 这时候我们就可以用`Single<T>`和`Completable`.

`Single<T>`返回一个值或者一个error.
它和`Observable`之间可以互相转换: 用`toObservable()`和`singleOrError()`方法.

`Completable`, 只有`onCompleted()`和`onError()`. 它不发射任何值, 可以在它之后用`andThen()`来添加另一个Observable, 进行后续其他操作.

`Observable`不能直接转换为`Completable`, 因为不知道`Observable`到底会不会停止. 可以把`Single`转换为`Completable`, 用`toCompletable()`方法.


## [Custom FindBugs detectors in Android](https://rhye.org/post/custom-android-findbugs/)
Android中有两种工具可以做进一步的编译期检查: [Android Lint](https://developer.android.com/studio/write/lint.html)和[FindBugs](http://findbugs.sourceforge.net/). 

FindBugs是一个静态的分析工具. 本文的主要任务是讲解如何实现一个自定义的检测器来检测一种特定的错误.

作者的例子是`try-with-resources`模式的代码.
这是Java 7新加的模式[try-with-resources](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html):

```java
try (Cursor c = db.query(...)) {
    c.moveToFirst();
    while (!c.isAfterLast()) {
        return new Foo(
                c.getString(c.getColumnIndex(...))
                ...
        );
    }
}
```
try语句中声明的资源将会在这个block结束的时候自动close.

但是这个特性最低需要API 19.
所以为了兼容旧版本, 我们不得不使用finally来自己close:

```java
Cursor c = db.query(...);
try {
    c.moveToFirst();
    while (!c.isAfterLast()) {
        return new Foo(
                c.getString(c.getColumnIndex(...))
                ...
        );
    }
} finally {
    c.close();
}
```

但是有时候我们会忘记close导致了泄露, 所以需要实现一个自定义的findbugs检测器来检查这种错误. 具体实现步骤和讨论见原文.


## [Static Code Analysis Tools](https://medium.com/@dmytrodanylyk/configuring-android-project-static-code-analysis-tools-b6dd83282921#.e7sc5x1if)
Android中流行的静态代码检测工具:
- Lint
- PMD
- FindBugs

本文介绍它们如何配置和使用.


## [Sync Failure Handling](http://tech.trello.com/sync-failure-handling/)
Trello离线模式文章, sync失败的处理.

在发请求的时候可能会发生各种各样的错误, 分为暂时性的和永久性的两类.
对于永久性的错误, 我们可以直接放弃delta; 但是对于暂时性的错误, 我们需要重试. 这里就需要考虑重试的时间和重试的次数.

另外还有一种情况是客户端发了请求, server也收到了, 但是客户端在收响应的时候失败了, 所以客户端可能会找机会重新发请求, 为了保证幂等性, 我们的每一个请求都有一个唯一的id, 如果server发现同样的id, 只处理第一个.

对于多个用户编辑的冲突处理, 当前用的是简单的以后者为准的方式.

撤销本地不合理数据, 以server数据为准, 更新本地数据, 这就需要在sync开始的时候先讲本地改动上传. 


# DESIGN
## [LottieFiles](http://www.lottiefiles.com/)
免费的Lottie动画.


# LIBRARIES & CODE
## [DiscreteScrollView](https://github.com/yarolegovich/DiscreteScrollView)
可滚动的列表, 中间的项目放大. (基于RecyclerView, 长得有点像ViewPager.)


## [SimpleRatingBar](https://github.com/borjabravo10/SimpleRatingBar)
五星评价View, 用kotlin实现的.


## [InstaCropper](https://github.com/yasharpm/InstaCropper)
剪切图像的View, 类似于Instagram的crop.


## [GuildWars2_APIViewer](https://github.com/huhx0015/GuildWars2_APIViewer)
一个app, 用来查看Guild Wars 2的API响应.
用了Dagger2, Retrofit2, RxJava2, MVVM架构.


## [here-be-dragons](https://github.com/anupcowkur/here-be-dragons)
一个Intellij/Android Studio插件, 你可以在一个方法上标记`@SideEffect`, 之后你调用这个方法的代码行左边会显示出一个龙的图标.


## [RoboGif](https://github.com/izacus/RoboGif)
一个python的小工具, 可以把Android设备上的录屏生成一个GIF图.


## [service-tree](https://github.com/Zhuinden/service-tree)
一个存储service的树形结构. (本期文章[Retaining Dagger components](https://medium.com/@Zhuinden/retaining-dagger-components-across-configuration-change-using-service-tree-3709c78bf6d2#.114aardgd)有讲.)

