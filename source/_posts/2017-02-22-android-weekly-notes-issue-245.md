---
title: Android Weekly Notes Issue 245
date: 2017-02-22 15:35:47
tags: [Android, Android Weekly, Testing, Unit Test, Kotlin, Mockito, God Objects, Call, SMS, Android Things, Keystore, Dagger 2, Wear 2.0, ViewPager, RxJava, Memory Leak, Alert, Gesture, Loading]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #245
February 19th, 2017
[Android Weekly Issue #245](http://androidweekly.net/issues/issue-245)
本期内容: 写好单元测试的几条原则; 如何mock Kotlin的对象; 如何消除God Object -> Context; 如何用Android来打电话和发短信, 以及相应事件的监听; 一个监控用电情况的应用(Android Things); 
用Keystore保存敏感信息; 依赖注入和Dagger 2的使用; Wear应用向Wear 2.0的迁移; 用ViewPager构建无Fragment的应用结构; Android应用的压力测试讨论; RxJava中`Subscription`注销处理不当引起的内存泄露; 单元测试并不是完全可靠; Trello向离线模式迁移的架构变化.

本周推荐的代码里有一个顶部提示控件, 一个手势检测库, 还有一个loading view的库.

<!--more-->

# ARTICLES & TUTORIALS
## [Write awesome unit tests](http://jeroenmols.com/blog/2017/02/16/unittests/)
作者关于写好单元测试提供了三条简单的规则以及每条规则对应的一些建议.

### 1. 尽快尽早地跑测试.
尽量在每次改动之后都跑跑测试, 及早发现问题. 你的测试跑得越快你就越有可能经常跑它们.

为了让测试跑得很快:
- 让测试跑在JVM上而不是设备上.
- 仅测试独立的逻辑模块.
- 不要包含UI, 数据库, 或者网络测试在你的主测试套件中.
- 测试中不要使用wait/sleep.

### 2. 小并且关注点集中的测试
对每一个bug来说, 应该有且只有一个测试挂掉, 并且测试失败的原因应该能从测试方法名上看出来.

这样就迫使你每一个测试只检查一件事情, 导致你的测试小并且简单易懂, 也好维护. 

实现tips:
- 测试中只有一条assert/verify语句.
- 有更多的小测试, 而不是几个大测试.
- 测试的名字能清楚地描述失败的原因.

### 3. 100%的可靠性
你的测试应该是完全值得信赖的, 不应该随机失败, 否则你将会对测试失去信任, 也不再会认真对待测试的失败.

所以你的测试应该是100%可靠的, 只在真的有问题的时候才失败.

建议是:
- 在JVM上跑测试, 因为到设备的连接可能会中断.
- 在测试的时候mock网络通信.
- 把UI/集成测试移出你的单元测试套件.

## [Helping to Mock Tests in Kotlin](https://medium.com/@orogersilva/helping-androiddev-to-mock-tests-in-kotlin-ab3be5204559#.wetcvdvbt)
因为Kotlin中的类默认是`final`的, 要继承的话需要显示地声明`open`.

如果只是为了在单元测试中mock就要加个open吗? 不.

本篇文章就介绍如何如何mock Kotlin的对象, 而不用该它的声明.

首先, [Set up](https://kotlinlang.org/docs/tutorials/kotlin-android.html); 然后, 使用这个库[mockito-kotlin](https://github.com/nhaarman/mockito-kotlin).

文中详细介绍了使用细节, 以及对`any()`方法的讨论.

## [How and Why I Kill God Objects](https://www.philosophicalhacker.com/post/towards-godless-android-development-how-and-why-i-kill-god-objects/)
在面向对象编程中, God Objects是应该被避免的.

在Android开发中, 最常见的一种God对象是Context. 本文介绍如何清除这个God对象, 同样的方法也可以用来处理其他对象.

首先说为什么要干掉Context?
在做TDD的过程中, 我们希望是面向接口的, 而且我们不应该mock非我们拥有的类型. 
所以我们不应该直接mock外部的API, 而是应该创建一个自己的接口层.

作者发现很多类其实并不真正需要一个Context, 它们只是需要得到string或者存储的键值对.

之后文中举例介绍了如何通过定义接口摆脱Context.

## [How to Make Calls and Use SMS in Android Apps](https://code.tutsplus.com/tutorials/how-to-make-calls-and-use-sms-in-android-apps--cms-28168)
**如何拨打电话**:
```java
String dial = "tel:" + phoneNo;
startActivity(new Intent(Intent.ACTION_DIAL, Uri.parse(dial)));
```
(不需要权限).

如果想在app里直接拨出去电话, 需要权限`android.permission.CALL_PHON`, 并且改用`ACTION_CALL`.

**监控电话事件**:

需要权限`android.permission.READ_PHONE_STATE`.来监控来电, 打出去的电话需要这个权限: `android.permission.PROCESS_OUTGOING_CALLS`.

具体实现就是在`TelephonyManager`注册监听器`PhoneStateListener`. 如果是在Activity中需要在对应的生命周期注销监听器.

如果需要后台监控, 则需要用到`BroadcastReceiver`, 过滤actions为`android.intent.action.PHONE_STATE`和`android.intent.action.NEW_OUTGOING_CALL`.
除了获取相应的电话号码, 还可以进一步阻止电话的拨出.

**发送短信**:

发短信也是两种方法, 启动一个短信客户端程序, 或者直接从程序里发.

启动其他程序:
```java
Intent smsIntent = new Intent(Intent.ACTION_SENDTO, Uri.parse("smsto:" + phoneNo));
smsIntent.putExtra("sms_body", message);
startActivity(smsIntent);
```

自己发: 需要权限`android.permission.SEND_SMS`.

```java
SmsManager smsManager = SmsManager.getDefault();
smsManager.sendTextMessage(phoneNo, null, message, null, null);
```

注意Android 6.0以上的设备, 本文提到的这些危险权限都是需要动态请求的.

**收短信**:
通过`BroadcastReceiver`, 需要权限`android.permission.RECEIVE_SMS`.

## [Android Things - Electricity Monitoring App](https://riggaroo.co.za/android-things-electricity-monitoring-app/)
作者分享了一个她的Android Things的应用(和Github repo), 可以监控她家的用电情况.

## [Using Keystore system to store and retrieve sensitive information](https://medium.com/@josiassena/using-the-android-keystore-system-to-store-sensitive-information-3a56175a454b#.nu3gw39qp)
利用Android的Keystore来存储一些敏感信息.

## [The lost droid and the magic Dagger](https://medium.com/rocknnull/the-lost-droid-and-the-magic-dagger-an-intro-to-dependency-injection-for-android-c686f4399117#.k5vgmxsjh)
一篇依赖注入的介绍文章.
先介绍依赖注入是什么, 有什么优点, 接着介绍Dagger 2的使用.

## [Wear 2.0: Match Timer – Part 1](https://blog.stylingandroid.com/wear-2-0-match-timer-part-1/)
作者把他的Wear应用升级到了Wear 2.0.

## [ViewPager without Fragments](http://www.ottodroid.net/?p=523)
一些开发者可能不想选择Fragment, 这篇文章里有相关讨论: [Advocating Against Android Fragments](https://medium.com/square-corner-blog/advocating-against-android-fragments-81fd0b462c97#.e4k145h1b).

作者推荐了一些在不用Fragment的情况下构建App的库: [Conductor](https://github.com/bluelinelabs/Conductor), [mosby](https://github.com/sockeqwe/mosby), [flow](https://github.com/square/flow), [mortar](https://github.com/square/mortar).

而本篇文章想要展示另一种方法, 既不用Fragment, 也不用上述的第三方库来构建一个App -> 用ViewPager.

在PagerAdapter里管理了一个Presenter的List, 每一个Presenter管理一个View. 具体实现见原文.

## [Stress-testing Android apps](https://android.jlelse.eu/stress-testing-android-apps-601311ebf590#.8kqor9m39)
之前大神JakeWharton有一个Sample App: [JakeWharton/u2020](https://github.com/JakeWharton/u2020), 里面有一个debug drawer, 可以用来模拟不同的测试情形, 比如网络连接不好, 延迟, 或者网络错误等等.

作者他们的App也有一个类似的debug drawer, 他们讨论出了一个需要测试的情形的checklist:
- 网络延迟
- 错误率
- 离线模式
- 屏幕旋转
- 应用在后台被杀死
- 应用升级
- Key Bashing
- 多窗口模式 (Android N)
- TransactionTooLargeException (Android N)


作者甚至发现其中的一些项目组合起来测试非常有趣.

- 网络延迟: 可结合方向改变/app后台被杀死测试.
- 错误率: 可以检查错误是否被正确处理并被报告.
- 离线模式: 关掉网络或者打开飞行模式, 检测正在执行的网络请求是否会引起崩溃; 是否正确通知了用户连接丢失了; 所有应该被cach的内容是否被正确cach了.
- 方向改变: 检查:  正在进行的请求会怎么办? app的状态是否被正确恢复了? 是否加载了当前方向对应的正确资源?
- App在后台被杀死: 可以通过命令: `adb shell am kill YOUR_PACKAGNE_NAME`或者"Do not keep activities"来模拟这种情形. 相关阅读: [Optimizing for Doze and App Standby](https://developer.android.com/training/monitoring-device-state/doze-standby.html).
- App更新: 升级后之前的数据是否被保存了?
- Key Bashing: 剧烈的滑动和敲击可能产生一些奇怪的错误. 可以跑Monkey来测试一下你的应用: `adb shell monkey -p YOUR_PACKAGNE_NAME`.
- 多窗口模式(Android N): 列出了一些多窗口的测试项目, 详情见原文.
- TransactionTooLargeException (Android N): Bundle中的数据不能太大, 超过限制, 在Android N以上会直接抛异常.

## [How to leak memory with Subscriptions in RxJava](https://medium.com/@scanarch/how-to-leak-memory-with-subscriptions-in-rxjava-ae0ef01ad361#.20w4lbkxq)
文中举了一个例子, 用RxJava结合MVP, 做网络请求, 更新UI, 很常见的使用情形. 

在生命周期结束的时候调用RxJava的`Subscription.unsubscribe()`来注销, 以结束还在进行的网络请求.

看上去没有什么问题, 但是程序实际运行, 反复旋转屏幕进行测试, `StrictMode`报告出了Activity的`InstanceCountViolation`, dump memory的确看到了多个Activity的实例. 这是为什么呢? 

作者深究原因, 发现`Subscriber`的子类存储的都是final的字段, 比如这个类:

```java
public final class ActionSubscriber<T> extends Subscriber<T> {

    final Action1<? super T> onNext;
    final Action1<Throwable> onError;
    final Action0 onCompleted;

    public ActionSubscriber(Action1<? super T> onNext, Action1<Throwable> onError, Action0 onCompleted) {
        this.onNext = onNext;
        this.onError = onError;
        this.onCompleted = onCompleted;
    }
...
}
```

因为它们都是final的, 所以最后即便执行了注销操作, 也是没有办法把它们置为null的.

在生命周期结束的时候注销的操作是这样:
```java
public void destroy() {
    subscription.unsubscribe();
    view = null;
}
```
这个`subscription`是`subscribe()`方法的返回值, 被保存在Presenter的一个字段里, 它实际就是`Subscriber`对象.

这里的问题就是, 在`destroy()`之后, 该引用并没有被置为null, 导致了下面的引用链:

```
Presenter -> subscription字段, 也即匿名的`Subscriber`对象 -> final字段 -> 对view的引用 -> 对Activity的引用.
```

从而造成了内存泄露.

解决的办法有两个:
- 在`subscription.unsubscribe();`之后把`subscription`字段置为null.
- 使用`CompositeSubscription`, 它可以管理多个`Subscription`对象, 用它的`clear()`方法, 它会unsubscribe所有项目并且清除所有的引用.

文后还列了相关的资料, 作者发现问题并寻找原因的思路很值得学习.

## [Your Unit tests might not be as reliable as you thought](https://afterecho.uk/blog/your-unit-tests-might-not-be-as-reliable-as-you-thought.html)
作者举了个例子, 说明即便你的单元测试过了, 也不保证你的产品代码一定没问题.

他的例子是
```java
SimpleDateFormat fmt = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSXXX");
```
失败的原因是因为`XXX`格式是在Android 4.3以上才支持的, 它是在`java.text`包下的. 所以实际在高版本的设备还运行正常, 换个低版本的设备就崩溃了.

所以单元测试并不一定可靠, 因为跑单元测试的JVM和Android设备上的JVM有可能不一样.

## [Airplane Mode: Enabling Trello Mobile Offline](http://tech.trello.com/sync-architecture/)
Trello移动移动现在有离线模式了. 作者介绍了他们的心路历程和架构变化. (比较简单和笼统的介绍).

## [Self-guided resources to Android development](https://twitter.com/corey_latislaw/status/831624360175603713?s=03)
这是一条Twitter, 作者分享了Android的学习资源. (可惜我打不开里面说的链接, 不知为何.)


# LIBRARIES & CODE
## [Alerter](https://github.com/Tapadoo/Alerter)
一个加在Window的Decor View上面的顶部提示栏, 类似于Snackbar和Toast一类的东东. 可定制外观, icon, 加多行字, 可添加click事件.

## [sensey](https://github.com/nisrulz/sensey)
一个好用的手势检测库.

## [mkloader](https://github.com/nntuyen/mkloader)
好看并且平滑的自定义loading view. 目前支持好几种图案.
