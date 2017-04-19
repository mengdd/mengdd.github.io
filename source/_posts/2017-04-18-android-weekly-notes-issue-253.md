---
title: Android Weekly Notes Issue 253
date: 2017-04-18 18:02:29
tags: [Android, Android Weekly, Android O, Font, Espresso, SQLite, FileProvider, Glide, Physics-based Animation, SpringAnimation, Animation, Broadcast, RxJava, DiffUtil, Mockito, Transition, Shared Element]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #253
April 16th, 2017
[Android Weely Issue #253](http://androidweekly.net/issues/issue-253).
本期内容包括: Android O新推出的自定义字体支持; 用Espresso测试自定义View; 在添加测试的过程中解耦在Activity中写的程序; SQLite的性能研究; 用FileProvider分享Glide的缓存文件; 基于物理的动画; Android O的隐式广播限制; RxJava和DiffUtil的结合; 向Mockito 2.x的迁移; 用Transition scene framework实现的shared element transition.
<!-- more -->

# ARTICLES & TUTORIALS
## [Android O: Fonts – Part 1](https://blog.stylingandroid.com/android-o-fonts/)
Android O中的自定义字体支持.

[Google Fonts](https://fonts.google.com/)是一个很好的资源网站, 里面的字体都是开源的, 可以在app中免费试用.

下载了字体资源(.ttf)之后, 加入项目资源字体文件夹:`res/font/`, 点击会显示字体的preview.

使用的时候只需要这样: 
```xml
android:fontFamily="@font/pacifico"
```


## [Testing Views in Isolation with Espresso on Android](https://www.novoda.com/blog/testing-views-in-isolation-with-espresso/)
作者他们的项目用了很多自定义的View, 把逻辑包含进去. (其实我个人不是很赞成这种做法).

本文介绍用Espresso来测试这种自定义View.


## [Test Driving away Coupling in Activities](https://www.philosophicalhacker.com/post/test-driving-away-coupling-in-activities/)
目标代码是一个Activity中有动态权限请求模块, 本文首先企图给它加上测试, 在加测试的过程中对原来的代码进行重构(抽取了View接口), 然后说明了这样做的好处: 解耦, 易于修改; 减少了重复代码.


## [Squeezing Performance from SQLite](https://medium.com/@JasonWyatt/squeezing-performance-from-sqlite-insertions-971aff98eef2)
本文介绍当你需要在SQLite数据库中插入大量数据的时候, 各种做法以及它们的性能比较.

### 实验1: 比较`db.insert`包不包transaction.
结果: 把一系列`insert()`操作包在transaction里可以大幅度地改善性能.

为什么呢? 根据[文档](https://sqlite.org/lang_transaction.html):
如果你不显示地指定transaction, 那么SQLite自己会在每一个操作上加一个隐式的transaction.

要知道SQLite只在transaction被提交的时候把插入数据写入磁盘, 所以, 如果你能减少transaction的数量, 你就可以减少磁盘访问, 从而提高效率.

### 实验2: 比较`db.execSQL`和`db.insert`.
结果: 使用`db.execSQL`可以轻微地改善性能.

原因: `db.insert()`包装抽象了SQL语句的创建, 这一层还是需要一点花销的.


### 实验3: 用`db.execSQL()`批处理插入.
结果: 可以提升一部分性能.

但是注意每个语句中有最大的变量数限制: 999. 所以如果你的表有越多列, 那么批处理所获得的性能提升就越少.

### 实验4: 直接使用`SQLiteStatement`.
`SQLiteStatement`比`insert()`和`execSQL()`更底层.

作者在这里做了两个实验: 一个是复用单条记录插入的`SQLiteStatement`语句对象; 另一个是复用批处理的`SQLiteStatement`语句对象.

结果: 跟用`db.execSQL()`批处理相比: 复用单条记录插入的语句对象并没有获得性能改善; 但是复用批处理的插入语句可以提供一点儿性能的提高. 


## [Share the Cache](http://emuneee.com/blog/2017/04/12/share-the-cache/)
很多app都有看图片的功能, 其中很多用了图片缓存库, 比如[Glide](https://github.com/bumptech/glide)来处理图片的缓存. 分享这些缓存的图片就有点tricky了, 要考虑到这些问题:
- 我如何访问cache中的文件? 
- 其他应用也有权限访问这些文件吗?
- 如果我需要手动拷贝文件到其他地方去, 我需要什么权限? 

有一种做法是: 首先把图片再缓存到其他公有的位置, 生成一个URI, 然后把这个URI用intent发送出去分享给其他应用. 这个听起来很简单, 但是实际上有点复杂, 因为读写外部存储都需要用户允许权限(Android 6.0以上).

所以本文要介绍的是用`FileProvider`的方法.

首先在Manifest中注册`FileProvider`.
然后在Glide中修改了缓存文件的路径.
(值得注意的是用Glide缓存的文件都是没有后缀名的, 所以作者在他的项目中强加了后缀名.) 最后再通过Intent分享出去:
```java
String authority = “com.yourdomain.android.fileprovider”;
Uri uri = FileProvider.getUriForFile(context, authority, file);
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_STREAM, uri);
intent.setType("image/jpeg");
context.startActivity(Intent.createChooser(intent, “Share via”);
```


## [Physics-based Animation](https://developer.android.com/guide/topics/graphics/physics-based-animation.html)
Google的官方文档, 介绍如何使用新的基于物理的动画系统. (physics-based animation).

基于物理原理的动画看起来更自然, 更真实. 

文档中举了一个例子, 中途改变动画的目标值, 比较了基于Animator的动画和基于物理的动画有什么区别: 前者需要取消当前动画设定新的目标值, 速度发生了突变; 后者只是改变了受力, 速度是连续的, 看起来更自然.

有关弹簧动画, 详情请见: [Spring Animation](https://developer.android.com/guide/topics/graphics/spring-animation.html).


## [Android O and the Implicit Broadcast Ban](https://commonsware.com/blog/2017/04/11/android-o-implicit-broadcast-ban.html)
Android O和隐性广播限制.

首先解释了这种限制是什么; 为什么需要加上这条限制(RAM, Battery); 都有哪些广播受到影响; 以及对这一系统改动可以采取的一些应对政策(动态广播, 显式广播, 先不要升级, 改用其他的通信方式等).


## [A nice combination of RxJava and DiffUtil](https://android.jlelse.eu/a-nice-combination-of-rxjava-and-diffutil-fe3807186012)
使用[DiffUtil](https://developer.android.com/reference/android/support/v7/util/DiffUtil.html), 你只需要提供新旧数据的比较.

但是如果你的数据集很大, 或者你要做的比较很复杂, 你应该避免在主线程调用`DiffUtil`的计算. 你可以把比较工作放在后台线程, 然后在主线程通知UI.

因为你需要比较新旧数据, 所以你需要在后台线程访问当前数据, 这就意味着这个数据会在多个线程被访问, 可能你就需要同步或者线程安全的数据结构, 如何避免这些呢? -> 使用RxJava.

里面涉及到了对`.scan()`操作符的使用, 核心代码如下:
```java
disposable = ThingRepository
    .latestThings(2, TimeUnit.SECONDS)
    .scan(initialPair, (pair, next) -> {
      MyDiffCallback callback = new MyDiffCallback(pair.first, next);
      DiffUtil.DiffResult result = DiffUtil.calculateDiff(callback);
      return Pair.create(next, result);
    })
    .skip(1)
    .subscribeOn(computation())
    .observeOn(mainThread())
    .subscribe(listDiffResultPair -> {
      adapter.setThings(listDiffResultPair.first);
      listDiffResultPair.second.dispatchUpdatesTo(adapter);
    });
```


完整代码见: [RxJavaAndDiffUtil](https://github.com/ErikHellman/RxJavaAndDiffUtil).


## [Mockito 2.x over PowerMock Migration Tips and Tricks](https://www.linkedin.com/pulse/mockito-2x-over-powermock-migration-tips-tricks-top-ten-hazem-saleh)

Mockito 2.x发布了, 包括了可以mock final的的功能和对Java 8的支持等. 

如果你之前在用Mockito 1.x写测试, 现在如何迁移呢? 如果你之前的测试中还用了`PowerMock`, 那么迁移起来将会更加困难.

本篇文章提到了你在把Mockito从1.x迁移到2.x的过程中可能会遇到的问题及解决方案.


## [Shared Element Transition with RecyclerView and Scenes - Part 4](https://www.thedroidsonroids.com/blog/workcation-app-part-4-shared-element-transition-recyclerview-scenes/)
使用transition scene framework实现的shared element transition动画效果.


# LIBRARIES & CODE
## [MaterialChipsInput](https://github.com/pchmn/MaterialChipsInput)
Material Design中Chips组件的Android实现. 提供了两种Views: `ChipsInput`和`ChipView`.


## [AdaptiveTableLayout](https://github.com/Cleveroad/AdaptiveTableLayout)
一个让你可以阅读, 编辑CSV文件的库.


# NEWS
## [Java 8 Language Features Support Update](https://android-developers.googleblog.com/2017/04/java-8-language-features-support-update.html)
Android Studio从2.4开始就要支持Java 8了! 预览版已经发布.