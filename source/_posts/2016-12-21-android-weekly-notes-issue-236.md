---
title: Android Weekly Notes Issue 236
date: 2016-12-21 10:27:39
tags: [Android, Android Weekly, IoT, Android Things, FileProvider, IDE, Android Studio, Retrofit2, Sign In, SmartLock, Security, NDK, Design, Color]
categories: [Android, Android Weekly, Design]
---


# Android Weekly Issue #236
December 18th, 2016
[Android Weekly Issue #236](http://androidweekly.net/issues/issue-236)

本期内容包括: Google的物联网平台Android Things; FileProvider; Android Studio的Layout Preview使用; Retrofit2使用; Google Sign-In和SmartLock; 把敏感信息放入NDK的解决方式.

设计部分讨论了调色板的灵感来源和几个开发app的时候应该注意的问题.


<!-- more -->

# ARTICLES & TUTORIALS
## [Getting started with Android Things](https://medium.com/@alexsimo/getting-started-with-android-things-b73be3295b42#.c1arra4ps)
Internet of Things (物联网, IoT), 是互联网, 传统电信网等咨询承载体, 让所有能行使独立功能的物品之间实现互联互通的网络.

2016年12月, Google发布了Android Things的开发者预览版, 这是一个专门为IoT设备定制的Android系统.

本篇文章一步一步地教你如何写一个IoT的基本程序, 跑在Raspberry Pi 3 Model B上.

## [FileProvider](https://blog.stylingandroid.com/fileprovider/)
上次我们提到了用`DownloadManager`下载的东西可以和其他应用分享, 那么如果我们下载的时候没有用`DownloadManager`呢? 

比较常见的情况是我们的应用需要分享内容到其他应用, 或者是文件的类型是我们应用不能自己处理的, 需要找一个支持这种文件类型的其他应用来帮我们打开它.

怎么解决呢? 答案是用`FileProvider`.

上一期有一篇文章也说过Android 7开始废弃了"file://", 解决方案就是用`FileProvider`, 所以实现是一样的, 这里就不重复了.

## [Working with the Layout Preview](https://www.novoda.com/blog/layout-preview-101/)
Layout Preview向你展示了你的xml将如何在设备上显示. 你可以用它查看布局在不同的配置下如何显示, 比如可以切换横竖屏, 语言等等.

但是它同样也有一些问题:

**Issue #1: Preview显示空白**
当你的布局是由动态获取的数据来填充的, preview不知道如何填充, 所以你看到的是空白的. 

一个好的practice是使用`tools`命名空间, 指定一些只在preview阶段使用的属性. 这样你就可以指定一些text或src用来预览.

**Tip #2: 使得动态内容在Preview可见**
如果你的图片是动态资源, 你也可以设置一些最大宽高给parent view, 以防真实的图片比期待的大太多或者是比例不对. 你可以设置`tools:layout_height` 和`tools:layout_width`, 还有`tools:background`在preview中查看view占多大.

本文还推荐了另一个阅读资料: [Tools of the trade — Part 1](https://tips.seebrock3r.me/tools-of-the-trade-part-1-f3c1c73de898#.e038jlqyy)

**Tip #3: 修复坏掉的Previews**
当你创建一个自定义View的时候, 你需要确保你的View不需要任何外部依赖即可被实例化, 否则Preview可能看不到你的View. 因为Preview不是运行在你的app上的, 它只是运行在IDE的JVM上, 所以View framework之外的东西它是访问不到的.

解决办法是在你的自定义View中做一些特殊处理, 比如把依赖注入放在`!isInEditMode()`里, 或者用`tools:`命名空间加一些默认值.

**Tip #4: <merge> 布局没有被渲染**
<merge>里面的控件在preview里会被重叠在一起.
解决的办法是使用`tools:showIn="layout"`, 指定<merge>具体是显示在哪个布局里. 如果你有多个布局都用到这个<merge>, 你可以选一个.

从Android Studio 2.2开始, 你可以使用`tools:parentTag`来指定parent的类型, 比如`tools:parentTag="LinearLayout"`.

**Tip #5: 在Preview中显示隐藏的View**
如果你在layout中把view的visibility设置为gone, 那么它是不会在Preview中显示的. 

解决办法: 使用`tools:visibility="visible"`.

## [Android Things Tutorials](https://blog.mindorks.com/android-things-tutorials-getting-started-8464c11009ff#.dhacx13kq)
Android Things教程.

## [Get Started With Retrofit 2 HTTP Client](https://code.tutsplus.com/tutorials/getting-started-with-retrofit-2--cms-27792)
本篇文章以实例讲述如何使用Retrofit, 虽然都是基础内容, 但讲解很详细.

## [Improving sign-in experience with Google Sign-In and SmartLock](https://medium.com/@p.tournaris/android-improving-sign-in-experience-with-google-sign-in-and-smartlock-f0bfd789602a#.dqh1aptm4)
Google提供了两种方式来帮助我们改善用户的登录体验:
Google Sign-In(之前被称为Google+ Sign-In)和SmartLock.

这篇文章举例解释了Google Sign-In和SmartLock的实现.

Google Sign-In的部分比较简单.

SmartLock让我们可以:
- 让用户保存credentials.
- 在打开应用的时候请求credentials.
- 使用存在Chrome上的credentials, 这样我们的网站和app就可以共享credentials.
- 显示Email提示, 让用户选择email地址.
- 所有的这些信息都保存在Google的server里, 用户可以保存或删除.

Demo app: [charbgr/AuthManager](https://github.com/charbgr/AuthManager)

## [Storing your secure information in the NDK](https://www.androidsecurity.info/2016/12/15/storing-your-secure-information-in-the-ndk/)
这篇文章说敏感信息放在Java代码里不安全, 很容易被人反编译查看出来, 如果放在NDK里面就好一些, 你打开查看的只能是二进制文件, 很难找到.

# DESIGN
## [Introduction to Natural palettes](https://stories.uplabs.com/introduction-to-natural-palettes-9503bfeee3d5#.z9y0xf7zc)
作者从大自然的图像中得到颜色组合的灵感.
文章中举了几个例子, 如何用相关的照片找到相关主题的调色板.

另推荐一个网站: [IN COLOR
 BALANCE](http://color.romanuke.com/)

## [Make your Android app look better](https://hackernoon.com/make-your-android-app-look-less-shitty-5dd63c4938f1#.4q5ro3ty8)
让你的App看起来更好的几点建议:
- 使用同一个图标集的图标.
(这里推荐了一些图片工具和网站.)
- 使用Material Design设计的keylines, 使用固定的格子大小.
- 使用颜色的时候小心一些. (这里推荐了一些调色板网站)
- 选择字体要明智一些.


# LIBRARIES & CODE
## [Material Components](https://github.com/material-components)
模块化和可定制的Material Design UI组件. Android, iOS, Web.

## [Android-oss from Kickstarter](https://github.com/kickstarter/android-oss)
Kickstarter开源了他们的Android应用.

## [stencil](https://github.com/thoughtbot/stencil)
一个kotlin写的Android库, 实现一种文字路径的动画.

## [AuthManager](https://github.com/charbgr/AuthManager)
包装了Google Sign-In和SmartLock的Manager.

## [FolioReader-Android](https://github.com/FolioReader/FolioReader-Android)
一个ePub阅读器和解析框架.

## [BufferTextInputLayout](https://github.com/bufferapp/BufferTextInputLayout)
对Support Library中的`TextInputLayout`的扩展, 增加了字数统计.

## [TextLayoutBuilder](https://facebookincubator.github.io/TextLayoutBuilder/)
使用Builder模式来配置创建一个Layout的属性.
