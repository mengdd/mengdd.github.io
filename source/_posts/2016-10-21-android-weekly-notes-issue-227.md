---
title: Android Weekly Notes Issue 227
date: 2016-10-21 14:52:53
tags: [Android, Android Weekly, Mobile Vision, Face Detection, Firebase, Optimization, HashMap, ArrayMap, RecyclerView, FPS, AutoValue, Abstract class, Interface, Bottom Sheet, Android Studio, ConstraintLayout, Bottom Navigation, Nougat, Notification, font, Reductor, Redux]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #227
October 16th, 2016
[Android Weekly Issue #227](http://androidweekly.net/issues/issue-227).

本期内容包括: Google的Mobile Vision API 人脸检测; Firebase的Remote Config; 与HashMap有关的优化; 提高RecyclerView帧率的优化; 使用AutoValue生成model代码; 开源库中抽象类和接口的使用讨论; Bottom Sheet的使用; Android Studio中的版本控制系统; ConstraintLayout的使用; 应用换Bottom Navigation; Nougat的Messaging Style Notification; 自定义字体; Reductor的使用等.

<!-- more -->

# ARTICLES & TUTORIALS
## [Face Detection Concepts Overview](https://developers.google.com/vision/face-detection-concepts)
这篇文章来自Mobile Vision, 讲人脸检测及相关概念. 
API使用[Tutorial](https://developers.google.com/vision/android/detect-faces-tutorial).
[Sample](https://github.com/googlesamples/android-vision).

## [Exploring Firebase on Android & iOS: Remote Config](https://medium.com/@hitherejoe/exploring-firebase-on-android-ios-remote-config-3e1407b088f6#.ozr0s8s5q)
Remote config是Firebase提供的一个feature, 让我们可以定义参数, 在firebase的console管理, 从而在server端控制应用的UI或者行为, 并且可以选择生效的用户范围.

之前还有这个文章是关于[Firebase Analytics](https://medium.com/exploring-android/exploring-firebase-on-android-ios-analytics-8484b61a21ba#.lu7cv7ejz)的.

本篇文章介绍了Firebase的Remote Config可以干什么, 以及怎么做, 解说很详细.

**参数**
我们用Remote Config定义的键值对叫参数(parameters).  它提供了这个参数相关的what信息(key, the identifier), 和how信息(value, the configuration).

**条件**
条件值(conditional value)也是一个键值对, 其中condition指定了需要满足的条件, value指定了满足条件时需要返回的值.

**优先级**
如果单个条件被满足, 那么返回对应的值; 如果多个条件都被满足, 那么返回主导条件(list上方的条件)对应的值; 如果没有条件满足, 则返回默认值; 如果没有定义默认值, 则什么也不返回.

文中还详细介绍了Android和iOS端的实现, 以及console的配置.

## [Android App Optimization Using ArrayMap and SparseArray](https://medium.com/@amitshekhar/android-app-optimization-using-arraymap-and-sparsearray-f2b4e2e3dc47#.29qai8u8j)
当我们需要存储键值对的时候, 我们总是首先想到用`HashMap`, 然而IDE(Android Studio)有时候会警告提醒你, 应该用`ArrayMap`或`SparseArray`.


### HashMap vs ArrayMap
`ArrayMap`比传统的`HashMap`更节省内存, 因为它把自己的映射放在数组结构中: 一个整型数组放每一个item的hash code, 一个Object数组放key/value对. 这样避免了为每一个entry创建额外的对象, 而且数组增长也好控制.

注意`ArrayMap`并不是为很大的数据集设计的, 并且它会比`HashMap`慢一些, 以为查找需要二分查找, 增删需要在数组中操作.

### HashMap
`HashMap`是一个`HashMap.Entry`的数组, 其组成是key, value, HashCode, 还有一个指针.

当进行插入时: 首先计算出key的HashCode, 然后用这个hashCode找到对应的bucket, 如果已经存了元素, 则把旧元素的指针指向新元素, 即把bucket变为一个`LinkedList`.

当进行查询时: 复杂度为O(1), 但是这样是牺牲了更多的空间复杂度得到的.

HashMap的缺点:
- 因为key和value都不能是原生类型, 所以插入时可能会有自动装箱, 导致创建额外的对象.
- `HashMap.Entry`本身就是一层额外的对象.
- 每次HashMap的收缩或者扩张, Buckets都要重新排列, 随着对象变多, 这个操作变得越发昂贵.


### ArrayMap
`ArrayMap`使用两个数组: 
`int[] mHashes`用来存哈希值; `Object[] mArray`来存对象.

当插入键值对时: Key/Value被自动装箱, Key被插入到`mArray[]`数组的下一个位置, Value也被插入到`mArray[]`, 在Key的下一个位置.
计算出的哈希值被放在`mHashes[]`的下一个位置.

当查询一个Key时: 首先计算出Key的哈希值, 在`mHashes`中二分查找这个hashCode(时间复杂度(logN)), 当得到hash的index之后, 我们就知道`mArray`中`2*index`和`2*index+1`的位置对应的是查找的key和value.

虽然时间复杂度提升了, 但是这样却更省空间.

### 推荐的数据结构:

- `ArrayMap<K,V>` in place of `HashMap<K,V>`
- `ArraySet<K,V>` in place of `HashSet<K,V>`
- `SparseArray<V>` in place of `HashMap<Integer,V>`
- `SparseBooleanArray` in place of `HashMap<Integer,Boolean>`
- `SparseIntArray` in place of `HashMap<Integer,Integer>`
- `SparseLongArray` in place of `HashMap<Integer,Long>`
- `LongSparseArray<V>` in place of `HashMap<Long,V>`

## [RecyclerView: How we achieved 60 FPS in Workable’s Android App](https://medium.com/@p.tournaris/recyclerview-how-we-achieved-60-fps-tips-in-workables-android-app-recyclerviews-c646c796473c#.h4gimmdkp)

我们经常会用RecyclerView来显示一个list的数据.
作者他们做的是一个招聘应用: Workable, 其中会用list来显示candidates.
他们还使用了DataBinding.
本文是作者他们关于RecyclerView的帧率所做的一些优化.


首先他们使用了Android Studio的Allocation Tracking, 然后上下滚动, 从报告发现, 他们布局中使用的`TableLayout`花费了很多资源, 于是后来他们改为`LinearLayout`加权重的方式来解决, 摆脱了耗费资源的`TableLayout`.

另一个引起很多资源分配的问题是, 对于需要大写的文字, xml中的:
```xml
<TextView
          ...
  android:textAllCaps="true"
          ...
/>
```
`TextView`的代码中会为此生成一个对象:
```java
  if (allCaps) {
      setTransformationMethod(new AllCapsTransformationMethod(getContext()));
  }
```
这个在静态的布局中可能没有问题, 但是在一个滚动的list中可能会有些影响.

改进方法是改为用java String的`.toUpperCase()`.


然后他们使用了RecyclerView的`.onViewRecycled()`方法. 这个方法让我们知道了RecyclerView中的一行何时被回收, 这样我们就可以释放一些不需要的资源.
他们使用了DataBinding, 所以这是一个合适的时机来删除ViewModel中的`OnPropertyChangedCallbacks`, 然后清理ViewModel自身, 我们还可以清理之前用Glide load到ImageView中的图片.

```java
@Override
public void onViewRecycled(Candidates holder) {
    if(holder != null) {
        holder.binding.getCandidateVM().removePropertyChangedCallback();
        holder.binding.setCandidateVM(null);
        holder.binding.setHighlightTerm(null);
        holder.binding.setShowJobTitle(false);
        holder.binding.setShowStage(false);
        holder.binding.executePendingBindings();
        Glide.clear(holder.binding.candidateBrowserAvatar);
        holder.binding.candidateBrowserAvatar.setImageDrawable(null);
    }

    super.onViewRecycled(holder);
}
```

作者他们的应用还有一些cache设置:
```java
binding.fragmentCandidateBrowseList.setItemViewCacheSize(30);
binding.fragmentCandidateBrowseList.setDrawingCacheEnabled(true);
binding.fragmentCandidateBrowseList.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH);
```

之后作者测量了他们的FPS, 显示是60 FPS, 并且发现去掉这些cache设置仍然是60.

测量帧率FPS的工具: [TinyDancer](https://github.com/friendlyrobotnyc/TinyDancer).

## [No more value classes boilerplate — The power of AutoValue](https://medium.com/rocknnull/no-more-value-classes-boilerplate-the-power-of-autovalue-bbaf36cf8bbe#.r72rsbe34)
在Java/Android编程中经常需要写model对象来存放一些数据, 使用Google的库[AutoValue](https://github.com/google/auto/tree/master/value)可以帮你自动生成这些类, 你需要做的就是定义你的字段, 然后给类加上注解.

### Setup
在project的`build.gradle`中:
```
buildscript {
    [...]
    dependencies {
        [...]
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
```
在app的`build.gradle`中:
```
apply plugin: 'com.neenbedankt.android-apt' // At the beginning
[...]
dependencies {
    [...]
    provided "com.google.auto.value:auto-value:1.2"
    apt "com.google.auto.value:auto-value:1.2"
}
```

### 基本使用
比如要创建Film类, 你可以写一个这样的抽象类:
```java
@AutoValue
abstract class Film {
    static Film create(String name, int year) {
        return new AutoValue_Film(name, year);
    }

    abstract String name();
    abstract int year();
}
```

每一个字段都对应一个抽象方法.
build一下, `AutoValue_Film`类就会自动生成, 加上静态工厂方法(上面的`create()`方法) 然后就可以使用工厂方法来得到model:
```java
Film matrix = Film.create("The Matrix", 1999);
```

点进自动生成的类`AutoValue_Film`里可以看到, 连`hashCode()`和`equals()`方法都生成了.

### 用builder模式
上面的例子随着字段的增多, `create()`方法的参数会变得很多, 用起来不方便, 那么此时就需要用Builder模式:
```java
@AutoValue
abstract class Film {

    static Builder builder() {
        return new AutoValue_Film.Builder();
    }

    @AutoValue.Builder
    abstract static class Builder {
        abstract Builder setName(String value);
        abstract Builder setYear(int value);
        abstract Film build();
    }

    abstract String name();
    abstract int year();
}
```
这样就可以很方便地加参数了:
```java
Film matrix = Film.builder()
  .setName("The Matrix")
  .setYear(1999)
  .setCategory(Category.FANTASY)
  .setRating(8.7f)
  .setDuration(136)
  .setReleaseDate(releasedDate)
  .setDirectors(directorsList)
  .setCast(castList)
  .build();
```

### AutoValue扩展 Parcelable
有时候你需要在Activity之间传数据, 需要你的model是`Parcelable`的, 此时你就可以用这个[auto-value-parcel](https://github.com/rharter/auto-value-parcel), 在代码里也只需要实现这个接口:
```java
@AutoValue
abstract class Film implements Parcelable {
   [...]
}
```
还有很多的扩展库: [extensions for AutoValue](http://search.maven.org/#search%7Cga%7C1%7Cauto-value), 比如AutoValue-Gson, AutoValue-Cursor, AutoValue-With, AutoValue-Redacted等.

## [Consider abstract class instead of interface](http://hannesdorfmann.com/android/library-abstract-class)
这篇文章的作者说, 在library开发中, 应该考虑用抽象类而不是接口. 他的库是[AdapterDelegates](https://github.com/sockeqwe/AdapterDelegates).

作者先介绍了通用的概念比较:
- class vs. interface
接口更解耦, 更灵活, 只是定义了一个协议, 不限制实现.
- interface vs. abstract class
抽象类会有继承的问题, 基类和子类会共享一些实现, 所以子类的编写者最好能清楚基类的实现, 这样才不会在写子类实现抽象方法的时候打破了基类作者的意图. 另外就是基类作者仍然可能会更新基类, 所以得时刻检查子类是否还是符合基类的设计意图.

但是为什么作者还是要把自己库中的接口改为抽象类呢? 这是因为作者的库依赖于Android的库, Android的库中相关代码改了, 作者就得改自己的public接口, 加一个方法, 导致所有新版的使用者也都必须实现这个方法.

还有一个情况就是比如一个开发者使用了2.1版本, 但是他项目里依赖的另一个第三方库使用了2.0版本. 编译不会出错, 最终的apk中打包的是2.1版本. 然后在这个第三方库的组件里调用2.1才有的新方法时就会抛出错误.

为了解决这个问题, Jake Wharton建议在库的主要更新(major update)中更改发布的package name和group id: [http://jakewharton.com/java-interoperability-policy-for-major-version-updates/](http://jakewharton.com/java-interoperability-policy-for-major-version-updates/)

作者觉得那每次Android RecyclerView的Adapter更新都会导致自己的库major update, 所以他决定把自己的`AdapterDelegate`接口改为抽象类. 这样他就可以对新增的方法提供默认空实现.

这样定义的抽象类只有抽象方法和一些空实现的方法, 并没有状态和行为的共享可能会传播给子类, 其实和接口是一样的.

## [Android BottomSheetDialog](https://medium.com/@anitas3791/android-bottomsheetdialog-3871a6e9d538#.462vjndmp)
实现bottom sheet的时候, 有三种选择: container view + `BottomSheetBehavior`,  `BottomSheetDialogFragment`, `BottomSheetDialog`. 前两种的例子比较多, 作者要介绍的是第三种.

如何选择取决你的用途, container view + `BottomSheetBehavior` 适用于[persistent bottom sheet](https://material.google.com/components/bottom-sheets.html#bottom-sheets-persistent-bottom-sheets), 而`BottomSheetDialogFragment`和`BottomSheetDialog`适用于[Modal bottom sheets](https://material.google.com/components/bottom-sheets.html#bottom-sheets-modal-bottom-sheets).

之后作者提供了实现代码, 附有theme定制和状态callback的设置.

## [The VCS client of Android Studio](http://saulmm.github.io/vcs-android-studio)
这篇文章介绍Android Studio的版本控制系统.

在Android Studio 2.2开始, 加入了一个`Create command line launcher`, 这样你就可以在命令行或者第三方的版本控制客户端使用Android Studio的diff/merge tool了.
作者使用的客户端是[SourceTree](https://www.sourcetreeapp.com/).


用`cmd + shift + A`可以用来find action, 然后就可以找到`Compare with branch`:  可以比较当前文件和某个分支上的文件的diff; 
另外还可以`Compare with...`, 来比较和之前某一个特定提交的diff;  以及`Compare with Clipboard`来和剪贴板做比较.

还有一些其他有用的快捷键, 请看原文吧.

## [Constraint Layout: Icon Label Text](http://blog.sqisland.com/2016/10/constraint-layout-icon-label-text.html)
作者想做这样一个UI, 左边是一个icon, 右边是两行字, icon的top和bottom分别和第一行字的top和bottom对其.
![ConstraintLayout: Icon Label Text](/images/icon-label-text.jpg)
怎么做呢? 她想到了用[ConstraintLayout](https://developer.android.com/training/constraint-layout/index.html).
代码在这里: [iconlabeltext](https://github.com/chiuki/iconlabeltext)

## [Bye, Bye Burger](https://medium.com/startup-grind/bye-bye-burger-5bd963806015#.emir2u5kv)
作者他们的应用从burger menu改为bottom navigation, 此篇为心得分享和他们改版时设计中的一些细节讨论.

其中状态保存是一个最主要的技术问题.

改版之后, 作者他们的应用数据表明有以下几个好处:
- 用户参与度提升了;
- 在底部导航有入口的功能使用率提高了;
- 并没有用户反馈说新的导航不好.

## [Nougat – Messaging Style Notifications](https://blog.stylingandroid.com/nougat-messaging-style-notifications/)
Messaging Style Notifications是为信息应用特殊设计的, 提供了一个像对话一样的view.

Messaging-style notifications和Bundled notifications的主要区别是, Bundled notifications中我们持续创建新的notification, 然后它们被grouped together. 但是用Messaging-style notifications的时候, 我们只有一个notification, 然后我们把所有的信息添加进去.

作者展示了实现代码和效果, 注意这个Messaging style并不是后项兼容的, 只在Nougat及以后的版本才支持.

## [Bottom sheet everything](http://www.hidroh.com/2016/06/17/bottom-sheet-everything/)
作者介绍了他的应用中对于Bottom sheet的使用.

**Deep linking with bottom sheet Activity**
作者用它处理Deep linking, 这样用户就不用每次都全屏打开, 只先提供一个peek, 如果真的感兴趣再打开.

实现是用一个透明的Activity, 还有状态栏处理的细节.

**Bottom sheet settings menu**
关于Settings, 为了节省用户的trip, 作者它们的应用用了options menu的弹出菜单. 后来他们改用bottom sheet来实现, 并且结合了`PreferenceFragmentCompat`, 省去了一些SharedPreferences的读写操作.

**Supporting tablet users**
bottom sheet在平板上使用, 尤其是横屏的时候, 看起来不太好.
所以作者定制了Bottom sheet的宽度, 在平板上时是一个指定宽度, 在手机上维持原状.

## [Machine Learning for with the Mobile Vision API— Part 1](https://hackernoon.com/machine-learning-for-android-developers-with-the-mobile-vision-api-part-1-face-detection-e7e24a3e472f?gi=e6e15107d4d6#.8kmih1fyd)

基于Google的Mobile Vision APIs现在Android开发者可以在应用里用上机器学习了.  现在这个Mobile Vision API包括三种类型Face Detection API, Barcode Detection API和Text API.

本文主要讲人脸检测部分, 后面会讲二维码检测和文字的API.

作者的demo展示了如何从一个静态照片中检测出人脸区域, 并且标记出landmark(眼睛, 鼻子, 嘴巴等), 之后可以根据这些特征位置加上一些覆盖标记.

[sample code](https://github.com/moyheen/face-detector).

## [Custom fonts formatting, the simple way](https://medium.com/@andrei.rosca/custom-fonts-formatting-the-simple-way-c1a0e4f6687f#.kjv5uaaix)
在Android中自定义字体的一个库: [Calligraphy](https://github.com/chrisjenx/Calligraphy).

如果你的输入是html文字, 你想自动处理里面的tag(比如<b>), 用另一种字体, 怎么处理呢, 作者给出了代码. 
![custom font in one textview](/images/custom-fonts-in-one-textview.png)

完整的例子代码见: [sample code](https://github.com/andrei-egeniq/android-tibits/tree/master/StyleSpan).

## [Reductor - Redux for Android. Part 1: Introduction](https://yarikx.github.io/Reductor-introduction/)
之前这个[文章](https://yarikx.github.io/Reductor-prologue/)介绍过Reductor, 在Android Weekly之前也出现过, 我的笔记: [Android Weekly Notes Issue 224](http://mengdd.github.io/Android/Android-Weekly/2016/10/02/android-weekly-notes-issue-224/#Reductor-Redux-for-Android).

[Reductor](https://github.com/Yarikx/reductor)是一个状态管理的库, 用Java重新实现了JavaScript的库Redux.
它的中心思想: 
![redux idea](/images/redux-idea.png)

之前的一篇文章做了一个TODO app, 然后作者发现这种mutable的数据会导致失控的数据改变, 然后可能会出现无法预测的行为. 做了一些改动之后, 我们发现可以通过只保存一个immutable的对象和mutable的reference来避免这个问题.

这篇文章用Reductor来重新实现应用, 文中详细说明了代码实现.

# LIBRARIES & CODE
## [ImageTransition](https://github.com/vikramkakkar/ImageTransition)
一个很小的库, Activity直接的shared element transition动画, 把一个圆形的ImageView变换到下一个Activity的方形ImageView.
## [Design-Patterns-In-Kotlin](https://github.com/dbacinski/Design-Patterns-In-Kotlin/)
用Kotlin实现的设计模式.
