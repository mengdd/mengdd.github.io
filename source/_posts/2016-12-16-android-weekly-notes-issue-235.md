---
title: Android Weekly Notes Issue 235
date: 2016-12-16 14:14:34
tags: [Android, Android Weekly, Custom View, JCenter, Animation, AnimatedVectorDrawable, Test, DownloadManager, OkHttp, Nougat, Android 7, Android Studio, Offline Architecture, lint, RecyclerView, FragmentPagerAdapter, FragmentStatePagerAdapter]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #235
December 11th, 2016 
[Android Weekly Issue #235](http://androidweekly.net/issues/issue-235)
本期内容包括: 开发一个自定义View并发布为开源库的完整流程介绍; 用`AnimatedVectorDrawable`实现的动画; 什么样的程序是可测试的; `DownloadManager`介绍; Okhttp的重试; Android 7取消了`file://`; Android Studio即将推出的build cache功能; 支持离线模式的app构架; 如何写自定义的lint规则; Epoxy, 一个处理复杂RecyclerView屏的库; `FragmentPagerAdapter`和`FragmentStatePagerAdapter`的比较等. 

<!-- more -->

# ARTICLES & TUTORIALS
## [Make an android custom view, publish and open source ](https://medium.com/dualcores-studio/make-an-android-custom-view-publish-and-open-source-99a3d86df228#.zc8s14lek)
作者开发了一个环形的SeekBar, 并把它作为一个库发布到了JCenter.

**作者首先讲了自定义View的实现**:
首先是关于View生命周期的介绍, 在写自定义View的时候有几个关键的生命周期回调需要处理:
![view-lifecycle-diagram-lite-version](/images/view-lifecycle-diagram-lite-version.png)

作者实现的几个关键步骤讲解:
- 自定义属性并获取.
- 在`onMeasure()`中控制尺寸.
- 在`onDraw()`中绘制: 避免在`onDraw()`中分配内存; 用`invalidate()`方法来激发重绘.
- 在`onTouchEvent()`处理用户手势. 在他的环形SeekBar的实现里, 这里涉及到了点击坐标到角度的转换. 

**将自定义View库开源到Github**:
开源到Github有个好的README很重要, 这里有几个tips:
- 提供截图, Gif或者Video.
- 提供安装/使用说明.
作者自己的库: [SwagPoints](https://github.com/enginebai/SwagPoints)

**发布库**:
- 去[JFrog Bintray](https://bintray.com/)注册.
- 创建repository, package, 和版本号.
- 生成并上传, 用了[这个library](https://github.com/blundell/release-android-library).
- 添加到Jcenter.
- 被接受之后收到邮件, 就可以使用了.


## [Animation: Jump-through](https://medium.com/google-developers/animation-jump-through-861f4f5b3de4#.k238d5tw2)
用`AnimatedVectorDrawable`实现的一个很fancy的位置标志动画.

## [What makes Android Apps Testable](http://www.philosophicalhacker.com/post/what-makes-android-apps-testable/)
如果程序的架构不适合测试, 那么硬要写一些测试很可能就会面临这样的局面: 要么就是发现没法写测试, 要么就是为了写测试而破坏了代码, 做了一些奇怪的事情.

那么到底是什么样的程序才是适合写测试, 或者是可测试的呢?

有一个有趣的定义是seam(接缝), 在接缝处你可以改变程序的行为, 而不用编辑当前程序. 如果程序没有接缝, 你将无法设置测试的初始条件和验证测试结果.

本文中举了一个实际的例子, 开始的时候程序没有seam, 所以导致无法测试, 后来把静态方法改为实例的方法之后, 我们就可以通过Mockito来模拟行为, 设置条件, 最后通过验证某一方法的调用与否来进行验证.

## [DownloadManager – Part 3](https://blog.stylingandroid.com/downloadmanager-part-3/)
用`DownloadManager`来处理下载.
首先它在设备上有自己的UI, 还有notification, 还有Downloads app能让用户管理下载文件.

 我们可以查询到文件的一些信息, 比如MIME type, 文件尺寸, 下载状态等.
 
 我们还可以用`getUriForDownloadedFile()`方法来获取一个URI, 配合MIME type, 发送Intent, 来打开一个相关的查看程序.
 
 关于储存文件的合适地点:
 - 文件小, 仅app自己使用 -> 私有数据区域(默认行为).
 - 文件大, 仅app自己使用 -> 外部存储的私有数据区域(不需要权限). `setDestinationInExternalFilesDir()`.
 - 文件需要被别的应用访问 -> 外部存储的共有区域, 需要`WRITE_EXTERNAL_STORAGE`权限. `setDestinationInExternalPublicDir()`.
 
## [OkHttp is quietly retrying requests. Is your API ready?](https://medium.com/inloop/okhttp-is-quietly-retrying-requests-is-your-api-ready-19489ef35ace#.ldxyyly7t)
在网路较慢或不稳定的时候, OkHttp有可能会重复发送请求, 直到成功. 

这个重试的逻辑是通过[RetryAndFollowUpInterceptor.java](https://github.com/square/okhttp/blob/07309c1c7d9e296014268ebd155ebf7ef8679f6c/okhttp/src/main/java/okhttp3/internal/http/RetryAndFollowUpInterceptor.java)实现的.

那么, 我们可以关掉这个重试行为吗? 有一些issues就在讨论这个问题: [Issue # 1043](https://github.com/square/okhttp/issues/1043). 后来有两个pull requests:  [PR #1259](https://github.com/square/okhttp/pull/1259)和[PR #2479](https://github.com/square/okhttp/pull/2479)改进了这个问题, 减少(但并没有消除)了不必要的retry请求.

全局关闭重试行为: `OkHttpClient.Builder .retryOnConnectionFailure()`设置为false. 但是注意这样是很粗暴并具有破坏性的, 消除了retry逻辑带来的好处:
- 如果Url有多个IP, 失败了一个还可以试另一个.
- 连接池中的连接偶尔会time out, 减少这种意外导致的后果.
- 可以顺次查找多个代理, 如果都失败了再转向直接连接.

**解决真正的问题**: 关闭静默重试在某些情形下有帮助, 但是其实它隐藏了真正的问题, 就是你的API是否是幂等的[idempotent](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). server端可以根据客户端的GUID来检测重复, 这样server就不会多次执行操作, 会通知发送者.

## [File scheme is now not allowed with Intent on N](https://inthecheesefactory.com/blog/how-to-share-access-to-file-with-fileprovider-on-android-nougat/en)
Android N (Nougat, API 24)开始, 不再允许发送`file://`的Intent, 将会直接抛出`FileUriExposedException`异常.

所以当你把`targetSdkVersion`改为24之后, 你必须要确保你修复了这些问题再发布.

解决方案是什么呢? 用`content://`, 结合`FileProvider`:
首先在manifest里面声明:
```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="${applicationId}.provider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"/>
</provider>
```
然后在`res\xml\provider_paths.xml`文件里指明路径:
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="external_files" path="."/>
</paths>
```
最后, 把
```java
Uri photoURI = Uri.fromFile(createImageFile());
```
改为
```java
Uri photoURI = FileProvider.getUriForFile(MainActivity.this,
        BuildConfig.APPLICATION_ID + ".provider",
        createImageFile());
```
然后放在Intent里发送就好了.

注意, 如果你的`targetSdkVersion`还没有更新到24, 那么即便是在Nougat的手机上`file://`也仍然是能正常使用的.

## [Use Android Studio Gradle Build Cache for faster builds](http://zeroturnaround.com/rebellabs/using-build-cache-in-android-studio-makes-gradle-build-faster/)
Android Studio当前的最新版是2.3 Canary 2. 有一些新的改进, 但是其中最吸引人的是这个[build cache](http://tools.android.com/tech-docs/build-cache). 它会使你的clean build更快.

本文后面解析了build cache的工作原理.

## [Offline App Architecture, build for the Next Billion](https://hackernoon.com/so-you-want-to-develop-for-the-next-billion-9eb072c26bc8#.1zklimr3o)
一个好的应用应该在网络不好甚至离线的时候仍然可以使用, 我们应该做些什么呢?
- 确定连接状况. 可以使用这个[network-connection-class
](https://github.com/facebook/network-connection-class). 如果你使用的是Okhttp, 可以加一个Intercepter来进行采样.
- 有效地缓存. 从网络取数据很慢并且昂贵, 所以有效地利用之前取到的数据是很关键的优化. (Cache-Control, Etag).
- 在本地操作, 在全局同步. 等网络请求的时候可以先显示本地数据, 而不是loading.
- 有效地处理线程.
- 优化图片. 网络不好的时候先用RGB_565, 等网络变好了再取高质量图片.
- 使用大Cookie. 尽量一次传输更多的数据(big cookie), 而不是频繁发送一些小请求(small cookies).

## [Writing custom lint rules and integrating them](https://medium.com/@mosesJay/writing-custom-lint-rules-and-integrating-them-with-android-studio-inspections-or-carefulnow-c54d72f00d30#.5y0o98bor)
如何创建自定义的lint规则.
事情的由来是作者发现了一个死循环调用, 然后他想做一个什么标记以防以后其他人会犯同样的错误.

然后他想到的是[@Nullable注解](https://developer.android.com/studio/write/annotations.html#adding-nullness), 的检查, 实质是依靠[lint](https://developer.android.com/studio/write/lint.html)来实现的.

于是他自己写了一个自定义的lint规则, 来提示使用用他的注解`@CarefulNow`标记的方法时应当注意.
详细的实现方式请看原文.

## [Epoxy: Airbnb’s View Architecture on Android](https://medium.com/airbnb-engineering/epoxy-airbnbs-view-architecture-on-android-c3e1af150394#.uyvuayspc)
[epoxy](https://github.com/airbnb/epoxy)是一个Android库, 用来处理复杂的RecyclerView屏. 本文介绍了它在项目中实际的使用.

## [Adventures with FragmentStatePagerAdapter](https://medium.com/inloop/adventures-with-fragmentstatepageradapter-4f56a643f8e0#.qk6aygake)
可能有很多Android开发者对于
[FragmentPagerAdapter](https://developer.android.com/reference/android/support/v4/app/FragmentPagerAdapter.html)和[FragmentStatePagerAdapter](https://developer.android.com/reference/android/support/v4/app/FragmentStatePagerAdapter.html)的区别不是太清楚或根本不知道, 本文作者就具体介绍了二者的不同.

**基本不同**

`FragmentPagerAdapter`
适用于项目个数确定的情形.
为什么呢? 因为一旦fragment的实例被创建, 它永远也不会从`FragmentManager`中移除, 直到Activity被销毁.

当Fragment不见的时候, 仅仅是`onDestroyView()`被调用, 当fragment再次回来时, 再调用`onCreateView()`.

`FragmentStatePagerAdapter`
当fragment的实例不可达的时候, 实例就会立即从`FragmentManager`移除. 被移除的fragment实例的状态由`FragmentStatePagerAdapter`保存, 当你再次回到该项的时候, fragment会重建新实例, 并且状态被恢复. 所以这种adapter适用于项目个数不确定或的情况.

所以使用`FragmentPagerAdapter`的时候需要注意内存问题.

**notifyDatasetChanged()的问题**.

`notifyDataSetChanged()`是用来处理数据集变化的情况, 比如一些项目增删的情况. 这个方法不是用来刷新当前显示的Fragment或其中的Views的.

文章中还有一些关于数据改变实现以及现有issue的讨论. 为了解决issue作者还发布了一个库[UpdatableFragmentStatePagerAdapter](https://github.com/inloop/UpdatableFragmentStatePagerAdapter).

# LIBRARIES & CODE
## [KeepActivitiesTile](https://github.com/Stocard/KeepActivitiesTile)
一个quick settings tile来开启"Don't keep activities".

## [WaveLoading](https://github.com/race604/WaveLoading)
一个波形的loading图, 水面上涨代表loading程度.

## [coordinators](https://github.com/square/coordinators)
Simple MVWhatever for Android.

## [epoxy](https://github.com/airbnb/epoxy)
一个处理复杂的RecyclerView屏的库.

## [Screen Record for Android](https://gist.github.com/tasomaniac/93cefd97af13e2ea2b2f248affb373bd)
录屏脚本.

