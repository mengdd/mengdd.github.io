---
title: 取代SharedPreferences的多进程解决方案
date: 2016-09-07 11:19:32
tags: [Android, Content Provider, SharedPreferences, Mutilprocess]
categories: [Android]
---

Android的SharedPreferences用来存储一些键值对, 但是却不支持跨进程使用.
跨进程来用的话, 当然是放在数据库更可靠啦, 本文主要是给作者的新库[PreferencesProvider](https://github.com/mengdd/PreferencesProvider)打个广告.
这是一个用ContentProvider实现的, 可以像SharedPreferences一样用于存储键值对, 支持跨进程使用.


<!-- more -->

## SharedPreferences不支持多进程
SharedPreferences对多进程的支持不好, 你用什么mode也没用, 所以官方已经废弃了原先的MODE_MULTI_PROCESS, 并且建议跨进程存取值还是用ContentProvider之类的更靠谱一些.
说明见:
[Context#MODE_MULTI_PROCESS](https://developer.android.com/reference/android/content/Context.html#MODE_MULTI_PROCESS)

## 用ContentProvider来取代SharedPreferences 心路历程
之前项目中为了解决跨进程存取值的问题, 找了一个解决方案: [grandcentrix/tray](https://github.com/grandcentrix/tray), 感觉还挺好用.

我们最后一次用的版本是tray的v0.10.0, 因为项目发布以后后台的崩溃里总是有相关的crash, 也是它的一个issue: https://github.com/grandcentrix/tray/issues/50
这个crash不是必现的, 概率比较低, 但是还是影响了一部分用户, 当我们解决了项目中的其他更重要的crash之后, 这个crash的排名就越来越靠前了.

后来作者做了一些改动, 说是在v0.11.0这个issue将会被修复, 但是这个版本却迟迟没有发布, 似乎作者做了一些很大的改动.

为了及时补救, 不再让用户体验到这个随机的崩溃, 我们决定放弃等待Tray的下个版本, 自己实现用ContentProvider来存取preferences.

实现过程用了[BoD/android-contentprovider-generator](https://github.com/BoD/android-contentprovider-generator)来生成ContentProvider相关的代码.
我们把存preferences的表放在了自己的数据库里, 然后借鉴了Tray的接口, 封装了读取方法, 使之用起来和SharedPreferences类似.
之后我们就用自己写的新代码全面取代了Tray, 当然数据库升级时还需要对原来存在Tray里的重要数据进行迁移.

做完了这些以后, 发现可以做一个像Tray一样的库, 更简单, 造福其他人, 那么何乐而不为呢.

## PreferencesProvider优势
- 基于ContentProvider实现, 支持跨进程使用;
- 采用模块化的管理方式, 可以将preferences分组管理;
- 没有Tray在v0.10.0版本的crash, 因为实现比Tray简单, 没有升级等功能.
(其实在我们实际项目的使用中, 基本上用不到对存preferences的表进行数据库升级的情况).
- 使用方式简单, 见项目README说明:[PreferencesProvider](https://github.com/mengdd/PreferencesProvider).


## 有用的工具
生成ContentProvider相关代码:
[BoD/android-contentprovider-generator](https://github.com/BoD/android-contentprovider-generator)
只要定义数据库基本信息, 在json中定义表结构, 就可以生成所有相关代码.

查看数据库: 
[Stetho](http://facebook.github.io/stetho/)
在Chrome中像调试网页一样看Android应用的资源, 这个真是太好用了.


最后再次附上本文推荐的解决方案库: [PreferencesProvider](https://github.com/mengdd/PreferencesProvider)
