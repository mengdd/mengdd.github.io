---
title: Android Weekly Notes Issue 322
date: 2018-08-14 10:30:16
tags: [Android, Android Weekly, Keyboard, Security, Network, Kotlin, MotionLayout, AR, Paging Library, aapt2, Accessibility, Android P]
categories: [Android, Android Weekly]
---


# Android Weekly Issue #322
August 12th, 2018
[Android Weekly Issue #322](http://androidweekly.net/issues/issue-322).
本期内容包括: 键盘的图像支持; 网络安全实现; Kotlin Native插件; MotionLayout实现折叠Toolbar; MotionLayout的关键帧和路径动画; 用Sceneform渲染3D物体; Paging Library的使用; 如何在文字底部绘制一个带圆角的背景, 可跨行; Google Play的新计划; aapt2的更准确的控制; 系统和应用中关于Accessibility的实现讨论. 
新闻部分: Android 9 Pie发布啦!

<!-- more -->

# ARTICLES & TUTORIALS
## [Exploring Image Keyboard Support on Android](https://overflow.buffer.com/2018/08/09/exploring-image-keyboard-support-on-android/)
[Image Keyboard Support (IKS)](https://developer.android.com/guide/topics/text/image-keyboard)是Android 7.1 (API level 25)引入的, 允许我们用输入法查询和发送更丰富的内容.

注: 这个API同样在support库中支持: v13 Support Library as of revision 25.0.0.

本文讨论了这个API如何工作和使用.

## [Securing Network Data Tutorial for Android](https://www.raywenderlich.com/5634-securing-network-data-tutorial-for-android)
保护网络数据:
* 用HTTPS来做网络请求.
* 用证书来信任一个连接.
* 验证传输数据的完整性.

第一部分解释了为什么要用HTTPS请求.

如何强制应用所有的网络请求都用HTTPS(Android N and higher):
添加一个文件: `res\xml\network_security_config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
  <domain-config cleartextTrafficPermitted="false">
    <domain includeSubdomains="true">github.io</domain>
  </domain-config>
</network-security-config>

```
在Application中使用它:
```xml
<application android:networkSecurityConfig="@xml/network_security_config"
```

第二部分解释了证书是什么, 以及`Certificate pinning`.

一个查看证书的网站: [SSL Labs]( https://www.ssllabs.com/ssltest/analyze.html).

这个public key同样也是添加在上面那个`network_security_config.xml`文件里. 但如果想要在Android N以下支持, 可以使用第三方库, 比如[TrustKit](https://github.com/datatheorem/TrustKit-Android).

本文后面还有详细解释加密验证等方面的内容.

## [Droidcon App with Kotlin Native Gradle](https://medium.com/@kpgalligan/droidcon-app-with-kotlin-native-gradle-898bafeba2f9)
在Droidcon App中使用了gradle插件[kotlin-native](https://github.com/JetBrains/kotlin-native).

## [MotionLayout – Collapsing Toolbar – Part 1](https://blog.stylingandroid.com/motionlayout-collapsing-toolbar-part-1/)
作者推荐一个关于`MotionLayout`的介绍文章: [Introduction to MotionLayout (part I)](https://medium.com/google-developers/introduction-to-motionlayout-part-i-29208674b10d).

本文介绍如何用`MotionLayout`实现Collapsing Toolbar的效果.

之前可以用`CoordinatorLayout`和`CollapsingToolbarLayout`来实现这个效果. 也没什么不对. 

`MotionLayout`提供了更多的自由性.

文本详细解释了实现细节.

## [Defining motion paths in MotionLayout](https://medium.com/@camaelon/defining-motion-paths-in-motionlayout-6095b874d37)
这个作者写了一系列关于`MotionLayout`的文章.

本文讨论`MotionLayout`中的关键帧和路径动画相关.

## [Render 3D objects at Runtime using Sceneform](https://medium.com/@harivigneshjayapalan/arcore-cupcakes-2-render-3d-objects-at-runtime-using-sceneform-9627d28f81fe)
ARCore Cupcakes是一系列的博客文章, 主要是ARCore和Sceneform的Android开发相关. 

本文教大家如何使用Sceneform来渲染3D物体.


## [7 Steps to implement Paging library in Android](https://proandroiddev.com/8-steps-to-implement-paging-library-in-android-d02500f7fffe)
7步在Android中实现Paging Library:
* 增加依赖.
* 用Retrofit取数据.
* 建立[DataSource](https://developer.android.com/reference/android/arch/paging/DataSource), 有三种选择.
* 建立`DataSourceFactory`.
* 建立`ViewModel`.
* 写好Adapter.
* 写好Activity.

作者的Demo在[这里](https://github.com/anitaa1990/PagingLibrary-Sample).


## [Drawing a rounded corner background on text](https://medium.com/google-developers/drawing-a-rounded-corner-background-on-text-5a610a95af5)
如何给文字加上带圆角的背景呢? 可以跨行, 也支持从右到左.

分析了需求之后, 最终的解决方案是写一个自定义的TextView.

例子代码: [RoundedBackground-Kotlin](https://github.com/googlesamples/android-text/tree/master/RoundedBackground-Kotlin).


## [Looking forward with Google Play](https://android-developers.googleblog.com/2018/08/looking-forward-with-google-play.html)
总结了Google Play过去这一年做出的重大改变以及下一年的计划目标等.

## [Increased accuracy of aapt2 "keep" rules](https://jakewharton.com/increased-accuracy-of-aapt2-keep-rules/)
aapt2提供了更加精细的控制力度, 可以明确指定哪个构造函数被保留, 这样可以减少APK中最终的方法数.

## [How VRT puts accessibility first](https://medium.com/vrt-digital-studio/how-vrt-puts-accessibility-first-f96eff2fba5f)
Android系统提供的一些辅助设置:
* 放大字体: Settings > Display > Font Size.
* 放大显示: Settings > Display > Display Size. (Android 7.0).
* 颜色校正: Settings > Accessibility > Color Correction.

这些都是系统级的, 开发者不需要在应用中实现.

盲人需要借助另一个应用, 比如[TalkBack](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback), 这种应用会给用户震动反馈, 读出屏幕上的内容. 所以作为应用的开发者, 我们应当确保所有的view都有正确的内容描述, 并且有一个合理的布局, 遵循Material的规定和一些惯例.

注: 在开发时可以打开这个: Talkback settings > Developer Settings > Display speech output. 这样读出的内容就会显示成文字, 不会打扰到其他人.

其他辅助应用还有: Brailleback, Switch Access.

一些开发者工具:
* [Accessibility Scanner App](https://play.google.com/store/apps/details?id=com.google.android.apps.accessibility.auditor): 扫描你的应用, 给出更好地提供辅助的建议.
* Android Studio也会给出一些建议: 颜色对比度不够, ImageView没有contentDiscription, 字太小等.


文章后面是作者他们应用的一些实践.

# LIBRARIES & CODE
## [Ferris Wheel](https://github.com/iglaweb/Ferris-Wheel)
一个会动的摩天轮.

## [folding-cell-android](https://github.com/Ramotion/folding-cell-android)
一个可以折叠的View, 动画效果很炫.

## [multiplatform-settings](https://github.com/russhwolf/multiplatform-settings)
Kotlin写的跨平台保留键值对设置的工具.

# News
## [Introducing Android 9 Pie](https://android-developers.googleblog.com/2018/08/introducing-android-9-pie.html)
### 更加智能
A smarter smartphone, with machine learning at the core.
* [Adaptive Battery](https://developer.android.com/about/versions/pie/power).
* [Slices](https://developer.android.com/guide/slices/).
* [App Actions](https://developer.android.com/guide/actions/).
* [Text Classifier](https://developer.android.com/reference/android/view/textclassifier/package-summary)和[Smart Linkify](https://developer.android.com/reference/android/text/util/Linkify).
* 神经网络API 1.1.

### 更加方便
Getting the most from your phone -- more easily.
* 新的系统导航. 可以滑动显示全屏预览, 然后点击进入. 
* 显示cutout(刘海).
* 通知和智能回复.
* 字体放大.

### 安全和隐私
* 生物识别认证提供了系统弹框.
* [Protected Confirmation](https://developer.android.com/training/articles/security-android-protected-confirmation).
* Stronger protection for private keys.
* DNS over TLS.
* 默认HTTPS.
* Compiler-based security mitigations.
* 用户隐私方面: 闲置的应用不能访问传感器, 读取build.serial现在需要权限.

### 相机, 音频和图像的新体验
* 相机: 多相机API, Session parameters等.
* HDR VP9 Video和HEIF图像压缩.
* 音频的动态处理API: [DynamicsProcessing](https://developer.android.com/reference/android/media/audiofx/DynamicsProcessing).
* 图像解码API: [ImageDecoder](https://developer.android.com/reference/android/graphics/ImageDecoder).

### 网络连接和地理位置
* Wi-Fi RTT室内定位.
* [JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler)根据网络状态更好地处理任务.
* Open Mobile API for NFC payments and secure transactions.

### 性能
* ART performance.
* 为Kotlin做的优化.
* Modern Android.