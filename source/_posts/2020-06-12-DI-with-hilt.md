---
title: Android官方新推的DI库 Hilt
date: 2020-06-12 10:58:56
tags: [Android, DI, Dagger]
categories: [Android]
---

# Android官方新推的DI库 Hilt
Hilt是Google Android官方新推荐的依赖注入工具.
已加入到官方文档: [Dependency injection with Hilt](https://developer.android.com/training/dependency-injection/hilt-android). 目前是alpha release阶段.

Hilt是在Dagger之上, Hilt单词的意思是: 刀把, 柄.
代码库还是这个[google/dagger](https://github.com/google/dagger). 

Hilt的出现, 让我想起了曾经昙花一现的dagger.android, 不知道hilt能不能经得住时间的考验. 

本文介绍Hilt的基本使用. 熟悉dagger的朋友可能会发现很容易.

Hilt是花里胡哨的小打小闹? 还是下一个主流工具? 让我们拭目以待.

<!-- more -->

## Codelab练习例子
* [Codelab: Using Hilt in your Android app](https://codelabs.developers.google.com/codelabs/android-hilt/#1).

我的Fork: https://github.com/mengdd/android-hilt.

这个项目最开始是一个用`ServiceLocator`手动注入的例子, 功能是记录用户点击button操作, 显示log. 有两个Fragment和Activity.

通过Codelab中一系列步骤的改造, 最终改成用Hilt做依赖注入.

本文举例就用其中的代码片段了.

原repo还贴心地附上了改造前后的对比diff: [Full codelab comparison](https://github.com/googlecodelabs/android-hilt/compare/master...solution).

## Hilt依赖添加
project root: `build.gradle`:
```
buildscript {
    ext.hilt_version = '2.28-alpha'
    ...
    dependencies {
        ...
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```
需要Android 4.0以上.

`app/build.gradle`:
```
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    ...
}

dependencies {
    // Hilt dependencies
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-android-compiler:$hilt_version"

    // Hilt testing dependency
    androidTestImplementation "com.google.dagger:hilt-android-testing:$hilt_version"
    // Make Hilt generate code in the androidTest folder
    kaptAndroidTest "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```
后面两个是测试的依赖.

## 概念, 用词解释
* Container, 对应Component.
* Bindings, 对应依赖.
```
The information Hilt has about how to provide instances of different types are also called bindings.
```
* [Component hierarchy](https://developer.android.com/training/dependency-injection/hilt-android#component-hierarchy)是指依赖在当前component中可以用, 也可以在它包含的child component中用. 举例: application容器中的依赖activity可以用, 但是反过来不行.

## Hilt基本使用
* `@HiltAndroidApp`: 标记在Application类上, 触发代码生成, application container是整个app的parent container.
```
@HiltAndroidApp
class LogApplication : Application()
```
* `@AndroidEntryPoint`: 标记在Activity和Fragment上. 创建了一个和当前Activity/fragment生命周期相关的container. 目前支持的类型是: `Activity`, `Fragment`, `View`, `Service`, `BroadcastReceiver`.
```
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() { ... }
```
每次Fragment(或Activity等)创建都会有它对应的container实例.
* 字段注入: 用`@Inject`标记字段, 注意字段不能是`private`的.
```
@AndroidEntryPoint
class LogsFragment : Fragment() {

    @Inject
    lateinit var dateFormatter: DateFormatter
    ...
}
```
从
[Component lifetimes](https://developer.android.com/training/dependency-injection/hilt-android#component-lifetimes)可以看到每种类型的component创建和注入的时间.

Activity是`onCreate()`, Fragment是`onAttach()`.

对Fragment来说, 在`onAttach()`的时候完成了对象的注入, 之后访问对象都没有问题.

* 提供依赖: 用`@Inject`标记构造器.
```
class DateFormatter @Inject constructor() {
    ...
}
```
它的依赖可以在构造参数中标明.

## Hilt的Scope支持
默认所有的依赖都是没有scope的, 每次注入依赖都创建新的实例.
Hilt自动在对应的生命周期创建/销毁对象: [Component lifetimes](https://developer.android.com/training/dependency-injection/hilt-android#component-lifetimes).

也可以把依赖scope到某个component, 这样在这个component内, 依赖就是单例.
* scope: `@Singleton`: application container的scope, 说明是application范围内的单例. `@ActivityScoped`对应activity component.


所有可用的scope: [Component scopes](https://developer.android.com/training/dependency-injection/hilt-android#component-scopes).


## module
module的使用基本和dagger一样, 用来提供一些无法用构造`@Inject`的依赖, 比如接口, 第三方库类型, Builder模式构造的对象等.

* `@Module`: 标记这是一个module. 在Kotlin代码中, module可以是一个`object`.
* `@Provides`: 标记方法, 提供返回值类型的依赖.
* `@Binds`: 标记抽象方法, 返回接口类型, 实现是方法的唯一参数.

Hilt多了一个注解: 
* `@InstallIn`: 指明module对应哪个container, 也即Component.
[Generated components for Android classes](https://developer.android.com/training/dependency-injection/hilt-android#generated-components)

```
@InstallIn(ApplicationComponent::class)
@Module
object DatabaseModule {

    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext appContext: Context): AppDatabase {
        return Room.databaseBuilder(
            appContext,
            AppDatabase::class.java,
            "logging.db"
        ).build()
    }

    @Provides
    fun provideLogDao(database: AppDatabase): LogDao {
        return database.logDao()
    }
}
```


### module的拆分
module的名字最好能传达它提供了什么类型的依赖, 所以多种依赖拆分多个modules写比较好.

```
@InstallIn(ActivityComponent::class)
@Module
abstract class NavigationModule {
    @Binds
    abstract fun bindNavigator(impl: AppNavigatorImpl): AppNavigator
}
```

`@Binds`和`@Provides`不能放在同一个module里.
因为`@Binds`需要module是一个abstract class, 而`@Provides`需要module是一个object.
放一起会报错: `error: A @Module may not contain both non-static and abstract binding methods`.

## 默认依赖
有一些默认依赖是已经有的, 比如:
* `@ApplicationContext`.
* `@ActivityContext`.

可以直接作为`@Provides`方法或`@Inject`构造的参数.

默认依赖是和component对应的.

[Component default bindings](https://developer.android.com/training/dependency-injection/hilt-android#component-default)


## Qualifier
要提供同一个接口的不同实现, 可以用不同的注解来标记. (dagger之前用的是`@Named`).

A qualifier is an annotation used to identify a binding.

举例: `LoggerDataSource`接口提供了内存和数据库两种实现. 

定义两个注解:
```
@Qualifier
annotation class InMemoryLogger

@Qualifier
annotation class DatabaseLogger
```

module中提供的时候用来标记相应的依赖:
```
@InstallIn(ApplicationComponent::class)
@Module
abstract class LoggingDatabaseModule {

    @DatabaseLogger
    @Singleton
    @Binds
    abstract fun bindDatabaseLogger(impl: LoggerLocalDataSource): LoggerDataSource
}

@InstallIn(ActivityComponent::class)
@Module
abstract class LoggingInMemoryModule {

    @InMemoryLogger
    @ActivityScoped
    @Binds
    abstract fun bindInMemoryLogger(impl: LoggerInMemoryDataSource): LoggerDataSource
}
```
这里用了两个module因为它们对应两个不同的component, 一个是application一个是activity, 依赖也是相应的scope.

注入的时候也对应加上:
```
@InMemoryLogger
@Inject
lateinit var logger: LoggerDataSource
```

## `@EntryPoint`
Hilt支持最常用的Android组件, 对于默认不支持的类型, 如果还想做字段注入, 需要用`@EntryPoint`.

注意这里只是限制了**字段注入**的情况, 自定义类型一般用构造注入比较方便(如果能用的话).

`@EntryPoint`的意思就是一个边界点, 这里可以通往Hilt的世界, 得到容器提供的依赖对象们.

[Codelab](https://codelabs.developers.google.com/codelabs/android-hilt/#10)中的例子是一个ContentProvider.

关键的部分是这段代码:
```
@InstallIn(ApplicationComponent::class)
@EntryPoint
interface LogsContentProviderEntryPoint {
    fun logDao(): LogDao
}

private fun getLogDao(appContext: Context): LogDao {
    val hiltEntryPoint = EntryPointAccessors.fromApplication(
        appContext,
        LogsContentProviderEntryPoint::class.java
    )
    return hiltEntryPoint.logDao()
}
```

## Hilt and Dagger
当初的dagger.android已被放弃, 它是为了简化dagger在Android上的使用而单独推出的. (根据Activity和Fragment的生命周期, 开发者不用手动调用inject方法, 但是确实最开始的setup code比较多.)


Hilt相对于Dagger来说, 简化了几个点:
* 不用自己创建Component.
* 不用手动调用inject()方法来完成字段注入.
* 不用自己在Application中保存component.
* 提供了一些Scope, 不用自己定义和写.
* 提供了一些默认依赖, 比如Context.

总体来说Hilt就是针对Android做的定制, 让依赖注入框架用起来更方便. 毕竟dagger是一个java注入库, 它的应用范围不限于Android.

因为Hilt和Dagger可以共存, 可以逐步迁移. 既然官方推荐了, 可以在项目内小范围地先试试.

最后推荐这个[cheat sheet](https://developer.android.com/images/training/dependency-injection/hilt-annotations.pdf)

## Reference
* [Dependency Injection on Android with Hilt](https://medium.com/androiddevelopers/dependency-injection-on-android-with-hilt-67b6031e62d)
* [Dependency injection with Hilt](https://developer.android.com/training/dependency-injection/hilt-android)
* [Dagger Hilt](https://dagger.dev/hilt/)