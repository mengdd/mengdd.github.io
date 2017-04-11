---
title: 网络使用和电池消耗 原因和改进
date: 2017-04-11 17:39:56
tags: [Android, Network, Battery]
categories: [Android]
---


# 网络使用和电池消耗 原因和改进
你的app发送的网络请求是电量消耗的主要原因, 本文先教你如何使用IDE工具来分类分析应用中的网络请求, 之后按照三种不同的网络请求分类, 分别给出优化建议, 减少电量消耗.

本文是对Android官网[Reducing Network Battery Drain](https://developer.android.com/topic/performance/power/network/index.html)系列文章的翻译, 略有删减, 可以作为摘要看看. (翻译不当的地方还请见谅).

<!-- more -->


## 收集网络数据 [Collecting Network Traffic Data](https://developer.android.com/topic/performance/power/network/gather-data.html)
使用[Network Traffic tool](https://developer.android.com/studio/profile/ddms.html#network)可以看到你的app如何以及何时通过网络发送数据.
本节教你如何通过在代码中加tag来测量和分类网络请求, 然后教你如何部署, 测试和可视化你的网络请求.

可以把网络请求分三类:
- 用户发起的.
- App发起的.
- Server发起的. 比如notification.

对这三个分类定义三个常量:
```java
public static final int USER_INITIATED = 0x1000;
public static final int APP_INITIATED = 0x2000;
public static final int SERVER_INITIATED =0x3000;
```

全文搜索:
```
extends GcmTaskService|extends JobService|extends AbstractThreadedSyncAdapter|HttpUrlConnection|Volley|Glide|HttpClient
```
勾选Regular expression, File mask(s) 写 *.java.

找出所有的网络请求后, 加上下面的代码:
```java
if (BuildConfig.NETWORK-TEST && Build.VERSION.SDK_INT >= 14) {
    try {
        TrafficStats.setThreadStatsTag(USER_INITIATED);
        // make network request using HttpClient.execute()
    } finally {
        TrafficStats.clearThreadStatsTag();
    }
}
```

其中build type的配置:
```
android {
    ...
    buildTypes {
        debug {
            // debuggable true is default for the debug buildType
        }
        network-test {
            debuggable true
        }
    }
    ...
}
```

运行测试:
Tools > Android > Android Device Monitor.
选择tab: Network Statistics.
选择你的应用, 然后按开始按钮.
你可以用下面的命令来清除应用的数据:
```
adb shell pm clear package.name.of.app
```

然后在你的应用中跑你想测试的应用场景即可.

使用了不同tag的网络请求会用不同的颜色显示出来.


## 分析网络数据 [Analyzing Network Traffic Data](https://developer.android.com/topic/performance/power/network/analyze-data.html)
有效应用网络资源的特点: 应该有大段的时间网络硬件没有在使用中. 因为在移动设备上, 发送和接收数据, 长时间保持数据连接都会有巨大的花销. 如果你的应用访问网络很有效率, 那么网络通信看起来就应该是紧耦合的, 被无请求的休息时段合理地隔开.

### 分析网络数据类型
如果你的网络请求图看起来存在这方面的问题, 我们就需要根据上一节中加tag而分三种类型形成的网络图来进行分析, 提供一些优化意见.

#### 分析用户发起的网络请求
用户发起的网络请求: 当用户在执行一项特定的活动时, 可能可以有效地耦合在一起; 或者当用户不断请求附加信息时, 非均匀地展开. 当在分析用户发起的请求是, 你的目标是: 寻找频繁的网络使用模式, 还有试图创建或增加不使用网络的时段.

这种网络请求优化的挑战是用户请求的不确定性. 另外, 用户在使用app的时候总是期待快速的响应, 所以推迟请求会降低用户体验. 一般来说, 当用户和app直接交互时, 快速响应的优先级是高于有效的网络请求使用的.

这里是一些对于用户网络请求的优化手段:
- 预取. [Pre-fetch Network Data](https://developer.android.com/topic/performance/power/network/action-user-traffic.html#pre-fetch-data). 当用户执行一项操作的时候, 应用预计一下它下一步可能需要的数据, 在单个连接中批量获取它们, 然后持有它们直到用户请求它们.
- 检测连接或监听变化. [Check for Connectivity or Listen for Changes](https://developer.android.com/topic/performance/power/network/action-user-traffic.html#check-or-listen). 在执行更新前, 检测网络连接, 或者监听网络连接变化.
- 减少连接数. [Reduce the Number of Connections](https://developer.android.com/topic/performance/power/network/action-user-traffic.html#reduce-connections). 使用允许数据按集合下载的server APIs

#### 分析app发起的网络请求
由你的应用发起的网络请求通常是一个对有效网络带宽有很大影响的方面. 在分析应用的网络活动的时候, 寻找空闲时段, 决定它们是否能被扩展. 如果你看到一致的网络请求的模式, 你应该寻找一些方法来改进, 以允许切换到低功耗模式.

一些优化app发起请求的手段:
- 批处理和定时. [Batch and Schedule Network Requests](https://developer.android.com/topic/performance/power/network/action-app-traffic.html#batch-schedule) 推迟你的网络请求以便它们可以一起处理, 并且在一个对电池来说有优势的时间.
- 允许系统检查连接. [Allow System to Check for Connectivity](https://developer.android.com/topic/performance/power/network/action-app-traffic.html#check-connect). 应该避免仅仅为了检查网络连接而引起的电池耗损, 在应用休眠的时候, 你可以让系统来帮你做检查.

#### 分析server发起的网络请求
由server对应用发起的网络请求通常也是一个对有效网络带宽有很大影响的方面. 在分析来自server的网络活动的时候, 寻找非活跃的时期, 看它们是否能被增加. 

优化手段:
- 使用GCM. [Use GCM for Server Updates](https://developer.android.com/topic/performance/power/network/action-server-traffic.html#gcm). 考虑使用Google Cloud Messaging service来做server端的更新, 而不是轮询.


## 优化用户网络请求 [Optimizing User-Initiated Network Use](https://developer.android.com/topic/performance/power/network/action-user-traffic.html)
快速处理用户的请求保证了良好的用户体验, 所以与此相比, 节约能量的优先级比较低.

### 预取 Pre-fetch Network Data
预取数据是一个减少数据传输的session数量很有效的方法. 使用预取, 在用户执行一个行为的时候, app预测下一步行为最有可能会用到的数据, 然后批量取出相关数据. 电池能量消耗因为两个原因被降低了:
- 因为预取数据发生在mobile radio已经唤醒的状态, 所以不用再次唤醒. 
- 应用预取了数据, 不然可能需要重新发另外的请求或者唤醒radio.

Tip: 为了查看是否你的应用可以从预取中获利, 你可以检查你的网络traffic, 寻找永远导致多个网络请求的特定系列的用户行为. 比如那些当用户浏览时增量下载文章内容的应用, 可以在用户当前查看的分类下预取一个或多个文章.


更多可以查看: [Optimizing Downloads for Efficient Network Access](https://developer.android.com/training/efficient-downloads/efficient-network-access.html#PrefetchData).


### 检查连接 Check for Connectivity or Listen for Changes
在移动设备上来说, 搜寻信号是最费能量的操作之一. 你的应用应该总是在发出用户请求之前检查连接状态. [Schedulers](https://developer.android.com/topic/performance/power/network/action-app-traffic.html#choosing-scheduler)可以帮你自动做这个.

- 如果你的一些按钮依赖于网络连接, 用[ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager.html)来检查连接. 如果没有网络连接, app就可以节省下强制连接的电量. 具体做法见[Monitor for Changes in Connectivity](https://developer.android.com/training/monitoring-device-state/connectivity-monitoring.html#MonitorChanges).
- 如果没有网络的情况下, 你的应用整个界面都处于不可用状态, 那么你可以根据需要使用Broadcast Receivers. 在你的Activity处于前台时监听网络连接变化, 当没有连接的时候, 就不再发请求. 如果你的app检测到网络连接丢失, 除了检测网路连接的那个receiver, 它关闭掉其他所有的receivers.

对于用户发起的请求, 一个最佳实践就是在请求之前检查连接, 如果网络连接不存在, 可以调度请求在连接上以后再进行. 调度器可以用一些方式来节省电量, 比如每次检查的时候连接失败, 那么就加倍延迟时间下次再检查.

### 减少连接数 Reduce the Number of Connections
一般来讲, 复用已存在的网络连接比启动新的更高效. 复用连接还允许网络更智能地对待拥堵和相关网络数据问题. 
更多信息请看[Optimizing Downloads for Efficient Network Access](https://developer.android.com/training/efficient-downloads/efficient-network-access.html#ReduceConnections).


## 优化App发起的网络请求 [Optimizing App-Initiated Network Use](https://developer.android.com/topic/performance/power/network/action-app-traffic.html)
你的应用发起的网络请求通常可以大幅度改善, 因为你可以对需要的资源做出计划并且设置访问它们的时间. 通过合理地调度时间, 可以创建出大段的设备radio空闲时间, 从而节约电量. 有一些Android的API可以用来调度网络访问, 并且其中的一些功能可以用来调节对其他应用的网络访问, 从而进一步优化电池性能.

### 批处理和定时 Batch and Schedule Network Requests
随机处理单个的请求会花掉很多的能量, 一个更高效的方法是把一系列的请求放入一个队列一起处理.

使用一个网络请求scheduler API用来管理和处理你的请求队列可以提高app的能量效率. Schedulers保存能量是通过把请求组合在一起, 让系统来处理. 它们还可以通过延迟一些请求, 等其他请求唤醒radio的时候, 或者等设备在充电的时候再做请求, 来进一步优化. Schedulers延迟和批处理请求是在系统范围的, 可以跨多个应用, 相比单个应用中的优化, 这给了它们更多优势.

#### 选择一个batch-and-scheduling API
Android提供了三种API, 对于大多数操作功能相似, 按推荐性从高到低排列:
- [GCM Network Manager](https://developers.google.com/cloud-messaging/network-manager). 要求使用Google Play services.
- [Job Scheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html). Android 5.0 (API 21)及以上.
- [Sync Adapter for scheduled syncs](https://developer.android.com/training/sync-adapters/index.html). 相比于前两种来说, 这种实现起来比较复杂.


### 允许系统检验连接 Allow System to Check for Connectivity
一个最严重且意外的电池消耗原因是当用户超出信号塔或接入点范围时. 在这种情况下, 用户可能并没有使用手机, 但是他们注意到设备变热, 电池电量变少.

在这种情形下, 可能是app正在跑一个后台进程, 其中以固定时间间隔搜寻信号. 搜寻信号是一个很耗电的操作.

避免这个问题的方法是用一种更高效的方式来查询信号连接情况. 对于app发起的请求, 可以用一个scheduler, 在其中用[Connectivity Manager](https://developer.android.com/training/monitoring-device-state/connectivity-monitoring.html)来检查连接. 如果没有网络连接, Connectivity Manager节约了能量, 因为它是自己检查了连接, 而不是启动了app来做检查. 可以进一步使用[exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff)来做优化, 如果没有连接, 那么扩大检查的时间间隔.


## 优化来自server的网络请求 [Optimizing Server-Initiated Network Use](https://developer.android.com/topic/performance/power/network/action-server-traffic.html)
由server发往app的请求优化起来比较有难度. 一种解决方案是客户端轮询, 检查server是否有更新, 这种方式很浪费. 一种更有效率的方式是当有新数据的时候通知你的app.

[Google Cloud Messaging](https://developers.google.com/cloud-messaging/gcm) GCM就是为了解决这个问题. 让你的server可以向app发送通知, 提高了网络效率, 降低了能量消耗.

### 用GCM发送服务器更新 Send Server Updates with GCM
Google Cloud Messaging (GCM)是一个轻量级的机制, 用于server向app传输一些简短的信息. 使用了GCM之后, server就可以在有数据更新的时候通知app, 这个方法减少了app查询更新却没有数据的能量消耗, 也避免了周期性的轮询请求.

注意: 通常情况使用[Normal priority](https://developers.google.com/cloud-messaging/concept-options#setting-the-priority-of-a-message)即可, 这样不会在设备非活跃或者低电量的时候唤醒设备.


## 优化一般的网络使用 [Optimizing General Network Use](https://developer.android.com/topic/performance/power/network/action-any-traffic.html)
一般情况下, 减少网络请求会对节约电量有帮助. 除了之前提到的改进方法, 你还应该知道一些一般性的方法. 

### 压缩数据 Compress Data
减少发送或接受的数据量会帮助减少连接时间, 从而节约电量.
你可以:
- 压缩数据, 用一些压缩方法, 比如GZIP压缩.
- 使用简洁的数据协议. JSON和XML提供了可读性和语言灵活性, 但是它们都是很占带宽的模式, 并且在Android上有一些序列化花销.
二进制序列化格式, 比如[Protocol Buffers](https://developers.google.com/protocol-buffers/)或[FlatBuffers](https://google.github.io/flatbuffers/), 提供了更小的数据包大小, 以及更快的编解码时间. 如果你的应用经常需要传输序列化数据, 这些格式会帮你获得解码时间和传输大小方面的优势.


### 本地缓存文件 Cache Files Locally
应用可以通过缓存来避免下载重复的数据. 始终缓存静态资源, 包括要求下载的全尺寸图像, 并尽可能长时间缓存它们.

缓存实现见: [Cache Files Locally](https://developer.android.com/training/efficient-downloads/redundant_redundant.html#LocalCache).


### 优化预取缓存大小 Optimize Pre-Fetch Cache Size
根据本地文件系统的尺寸和当前的网络连接来优化预取缓存大小. 你可以使用connectivity manager来确定处于活动状态的网络类型(Wi-FI, LTE, HSPAP, EDGE, GPRS), 并修改你的预取程序来最小化电池负载.

更多信息请看[Use Modifying your Download Patterns Based on the Connectivity Type](https://developer.android.com/training/efficient-downloads/connectivity_patterns.html#Bandwidth).
 