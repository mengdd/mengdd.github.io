---
title: Google IO 2019 Android 太长不看版
date: 2019-05-17 14:05:23
categories: [Android]
tags: [Android, Google IO 2019, Architecture Components]
---

Google I/O 2019, 这里有个playlist是所有Android开发相关的session视频合集:
[Android & Play at Google I/O 2019](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM)
当然啦每个视频都看不太现实了, 就挑几个看看吧.
这里是我个人的一点笔记, 可以作为一个太长不看版, 感兴趣的点再自己了解下.

<!-- more -->

## [CameraX](https://www.youtube.com/watch?v=kuv8uK-5CLY&list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM&index=4&t=0s)
* 更易用的API.
* 隐藏底层细节.
* 兼容各种设备.
* 自动化测试套件.

## [Android Studio UI design tools and Debugging Tools](https://www.youtube.com/watch?v=oWTG5g5rT4s&list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM&index=6&t=0s)

Design Toolchain:
### Layout Editor
* Blueprint mode.
* context menu来提供一些更方便的工作. constraint popup. constraint menu.
* 预览改进: RecyclerView的预览改进. 预览sample数据.
* attribute界面可以指定dimen中定义的自定义尺寸.

相关文章:
https://medium.com/androiddevelopers/android-studio-project-marble-layout-editor-608b6704957a

### Navigation Editor
* navigation components的使用. 
* navigation editor中的预览.


### Resource Manager: 
* Batch Import, 可以直接把图片资源拖拽整理进去. * 可以把svg拽进去变成vector drawable. 
* 提供了layout, color等的小图预览. 
* 颜色的alpha终于可以用百分比设置了.

### Layout Inspector: 
* 改进了属性显示.
* 可以直接预览修改. 
* 3D显示, 可以查找某个背景颜色到底是哪一层设置的.

(我的思考: 现在Android Studio越来越鼓励开发者直接利用图形界面来设置layout了, 总是喜欢直接编辑xml算不算是早期Android遗老遗少的一个陋习?)

## [What's New in Architecture Components](https://www.youtube.com/watch?v=Qxj2eBmXLHg&list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM&index=6)
Kotlin first.

### Data binding:
* Faster complilation:
`android.databinding.incremental = true`.
增量注解处理.
* 错误信息改善了. (耶!)

### How to access views?
* 几种find view的方法比较.
* View Binding: coming soon in Android Studio 3.6.

### Lifecycle
* ViewModel和Saved State.
`SavedStateHandle`: 传入ViewModel, 用于保存一些在应用被杀死后重启仍然需要恢复的值.
* 一些代码的写法被优化了.

### WorkManager
优点: deferrable, persistent, constraint-based, backwards compatible.
* 性能: on-demand initialization.
* Google Play Services integration.
* 兼容性.
* 测试.
* Future: foreground service.

### Room
* Coroutines, 协程支持: suspend方法.
* Full text search: `@Fts4`, `MATCH`.
* Database Views. `@DatabaseView`. 重新组织一个可查询的数据结构, 类似于重新组装一个表, 用来查询.
* 扩展了Rx支持: 数据库操作方法可以返回`Single`, `Completable`等Rx类型.
* Future: 注解处理; 关系改善; migration改善; 协程Channel&Flow.

### Paging
What's next in Paging?
* Built in network support with error handling.
* Headers & footers
* 更好的Rx和协程集成.

### Navigation
* ViewModels scoped to Navigation Graphs.
* Navigate by URI.
* Dialog as destinations.
* Safe Args.
* Future: Better support for dynamic features.

最后广告了一下这个课程:
https://www.udacity.com/course/developing-android-apps-with-kotlin--ud9012

## [What's New in Android](https://www.youtube.com/watch?v=td3Kd7fOROw&list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM&index=8&t=0s)
What's new in Android Q:
### System UI
* SAW: System Alert Window. -> 安全问题. -> 引入Bubbles (API 29).
* Dark theme.
* Share sheet: 内容预览, 粘贴, 性能改善.
* 通知分区域: Priority, Gentle.
* Notification actions: 自动生成回复.
* 手势导航.

### Platform
* WebView: Trichrome: Separate WebView/Chrome, hung renderer检测.
* Accessibility.

### Text
* API 23默认开启了连字符, 性能下降, 所以在Q默认关闭了.
* API for fonts.
还单独有个text的演讲专门讲这个.

### Magnifier
### private APIs
要么用public的, 要么private改成public的, 要么加新的. 
gray list中加了更多的(以后的版本将不能用了).

### Android Runtime (ART)
Profile, 启动改善, GC改善.

### Kotlin
Q的新API有nullability注解.

### Security
TLS 1.3 默认开启.
生物识别改进.

### Jetpack Compose
全新的UI组件: Kotlin, Reactive.
(很像Flutter.)

### Privacy
* 外部存储限制.
* Location: 权限设置更新. 新增只有前台时允许的选项.
* 后台启动Activity限制.

### 其他
* CameraX
* Architecture Components
* ViewPager2
* ViewBindings
* Blend Modes
* RenderNode
* HardwareRenderer: 可以控制光源(阴影)
* Hardware Bitmaps
* Audio Playback Capture
* 应用不能控制wifi开关
* Settings Panel

## [Build a Modular Android App Architecture](https://www.youtube.com/watch?v=PZBg5DIzNww&list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM&index=12)
### 为什么我们要模块化呢?
* scale: 更易分组开发人员; 更易查找资源.
* 更快的编译速度.
* 更快的CI. 只执行有修改的module相关的测试.
* Good for business. App Bundles: 更小的apk.
* 分离的feature测试. 快速AB Test一个新功能.

### 两种Modules:
* Library Module
* Dynamic Feature Module

### 如何分离模块呢?
* by feature
* by layer
* feature & layer.

### 模块间的依赖
Module A依赖Module B:
* `api`: Module B is part of my **Public API**. 依赖A的可以看到B.
* `implementation`: Module B is my **Implementation Detail**. 依赖A的看不到B.

这里讨论了一个分层模块化构架中的问题: repo模块对数据库模块的依赖, 用`api`, 上层可以直接使用数据库模块中的实例类, 但是同时暴露了DAO, 破坏了模块化分离的意义.
两个选择:
* 规定在上层不要操作数据库.
* 新建一个通用的数据模块, 存放实体类, 数据库模块`implementation`依赖它, repo模块`api`依赖它.

### 测试.
### Dynamic feature modules
几个挑战:
* 导航. 需要hardcode类名.
* 找依赖. Dynamically loaded modules.

代码例子: [android-dynamic-code-loading](https://github.com/googlesamples/android-dynamic-code-loading)
结合dagger.

### 数据库
数据库的几种选择:
* 一个数据库: 好维护, 好共享, 但是没有分离.
* 一个核心数据库 + 每个feature自己的数据库: 有了分离, 但是共享成了问题. 如果hybrid: 让有交互的模块共享, 也是挺复杂的.
* 一个Room的bright feature: Multi-module数据库. (Not Available now.)

最后闲聊了几句关于构架的讨论.
Android team只是提供options, 最后的选择还是depends.
* 考虑: short time costs, long time benefits.
* 用户不会因为app的构架好而给你5星.

## [What's New in the Android OS User Interface](https://www.youtube.com/watch?v=nWbW58RMteI&list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM&index=34)
三个方面:
* Sharing: 性能改进; UI改进; 更多自定义选项.
* Notifications: gentle; actions.
* Multitasking: Bubbles.

## [Improving App Performance with Benchmarking](https://www.youtube.com/watch?v=ZffMCJdA5Qc&list=PLWz5rJ2EKKc9FfSQIRXEWyWpHD6TtwxMM&index=57)
Jetpack Benchmark Library:
https://developer.android.com/studio/profile/benchmark.md

* 可以测量工作方法的时间: 重复测量的Rule;
合理排除初始化时间.
* UI测试.
* 新增module: Benchmark Module.

工具实现细节介绍.

Best Practices:
* Start with tracing.
* Synchronous blocks.
* Small blocks
* Hot code.
* Caches.
* @RunWith(Parameterized)
* 不要比较设备.

例子: [googlesamples/android-performance](https://github.com/googlesamples/android-performance)