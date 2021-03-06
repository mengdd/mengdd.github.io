---
title: Android App安装包瘦身计划
date: 2019-07-19 16:34:49
tags: [Android]
categories: [Android]
---

# Android App安装包瘦身计划
Android App安装包体积优化: 理由, 指标和可以采用的方法.

本文内容归纳如下图:
![Reduce Apk Size](/images/reduce-apk-size.png)

<!-- more -->

## 为什么要安装包瘦身
- 安装包需要瘦身吗? 
- 不需要吗?

安装包要瘦身的主要原因就是考虑应用的下载转化率和留存率. 
应用太大了, 用户可能就不下载了, 尤其是移动网络或者流量收费的情况下.
再者, 因为手机空间问题, 用户有时候可能需要选择卸载一些应用, 就会先盯上那些占空间大的, 所以应用大小也会也影响留存率.

但现在什么时代了, 到处都是WiFi, 流量包又大又便宜, 网速快到飞起, 手机硬件也在不断升级, 更大更快更高级, 安装包真的还需要瘦身吗? 有人在乎吗?

那当然还是有一定必要的, 不然这篇文章写到这里已经结束了. 
因为就算用户不在乎你这个应用有多少大, 决定要下载安装了, 安装包越大, 下载时间就会越长, 没等下载完成就因为各种原因取消或者断网了.
或者因为下载时间稍微有点长, 等完成以后, 用户已经忘了要用这个app了.

现代人的耐心很有限, 不然也不会有小程序这样轻量级的解决方案.

安装包瘦身可以地定量, 在一定程度上改善问题.

除了下载下载转化率和留存率, 安装包体积优化还有一些理由:
* 预装应用的推广成本.
* 满足应用市场的最大包体积限制.
* 在技术甚至业务层面, 我们可以重新审视我们的codebase, 是否需要删除一些低价值的业务, 清理无用的代码和资源, 进行进一步的重构和改善.

## 安装包的构成
Apk安装包本质上是一个zip压缩文件, 解压之后我们可以看到其中包含的内容, 通常包含:
* 一个或多个`classes.dex`文件: 代码编译结果.
* `assets/`文件夹.
* `resources.arsc`: 编译后的二进制资源文件, 包含了所有`res/values`中的XML内容, 如字符串, style等, 也包括对没编译进来的资源(比如layout和图片)的路径.
* `res/`: 没有被编译进`resources.arsc`文件的资源: layout, drawable等.
* `AndroidManifest.xml`: merge后的manifest文件.
* `META-INF/`: 包含签名文件等.

如果有native的库, 可能还会有`lib/`文件夹.

## 尺寸指标
安装包大小有几个指标呢? 
有三个:
* 文件尺寸: 这可能是最直观的一个, 就是我们build出来的文件大小. 但是其实它是最不重要的一个指标.
* 下载尺寸: 下载app需要的大小, 最重要的指标. 因为下载市场(Google Play)会帮我们做优化, 下载尺寸会比文件的尺寸小. 
一般增量安装和全新安装需要的大小是不一样的.
* 安装尺寸: app安装到手机上, 启动解压之后的大小, 关系到app的留存率.

## 安装包分析工具
在做安装包体积优化之前, 建议用工具看一下当前的安装包情形, 找到当前的bottle neck, 这样才能有的放矢.
优化之后也好做一个比较.

### APK Analyzer
[APK Analyzer](https://developer.android.com/studio/build/apk-analyzer)是Android Studio自带的工具.
使用APK Analyzer的三种方法: 
* `Build > Analyze APK..`, 选择`.apk`文件.
* 切换到Project View, 双击build/outputs/apk目录下的apk文件.
* 把apk文件拖到Android Studio的编辑窗口中.

利用这个工具可以查看apk的各个组成部分以及它们各自的大小, 包括文件大小和预估的下载大小.

这个工具还可以比较两个APK, 从而可以看出具体是哪个部分有尺寸的变化.

### 官方插件: Android Size Analyzer
在Android Studio中可以添加Plugin: `Android Size Analyzer`.

安装之后重启Android Studio, 接着就可以点击菜单`Analyze > Analyze App Size`.
点击之后会给出建议, 显示出项目中比较大的文件, 建议转化一些图片到webp格式, 建议开启代码压缩混淆等.


## 安装包瘦身措施: Coding阶段
### 减少代码(native和Java代码)
* 控制第三方库的使用: 依赖第三方库的时候, 尽量减小范围, 或者使用更加友好的替代. (ProGuard会移除不用的代码, 但是它不会移除库的内部依赖.)
* 减少不必要的生成代码.
* 减少枚举的使用.
* 减小native库: 移除debug symbols和关闭so库压缩.

### 控制资源使用
要控制资源的使用: 减少资源的数量, 减小资源的大小.

#### 设备屏幕支持
Android有很多屏幕密度类型:
```
ldpi, mdpi, tvdpi, hdpi, xhdpi, xxhdpi and xxxhdpi
```
并不是需要每个资源都提供多密度版本的. 
当一个屏幕密度下没有提供特定的资源, Android会自动根据这个资源其他密度的版本进行缩放.

如果你的app仅需要提供缩放图像, 你可以只在`drawable-nodpi`中提供单个资源. (建议还是至少有一个`xxhdpi`.)

#### shape的使用
很多时候我们并不需要一个图片, 比如纯色或渐变色的背景, 带边框, 带圆角等.

用`<shape>`尺寸就会比较小, 也不用多个密度的版本.

#### 复用资源
这里的复用可以是改变了颜色和旋转方向等的复用.
比如改资源的tint, 或者把一个图旋转了之后再用. (thumb up变成thumb down了).

#### 压缩图片
可以用一些工具对PNG和JPEG图片进行无损压缩: 图片质量无损但是尺寸变小.

#### 使用WebP格式的图片
可以使用[WebP](https://developers.google.com/speed/webp/)格式的图片, 比JPEG和PNG压缩得更好.

Android Studio提供了转换工具: [Create WebP images](https://developer.android.com/studio/write/convert-webp.html)

注意: Google Play只接受PNG作为launcher icon.

#### 使用矢量图
可以使用矢量图来作为可伸缩的资源, 在Android中是[VectorDrawable](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable.html)对象.

但是矢量图的渲染需要系统的时间花销, 所以推荐只有小的icon使用矢量图.

#### 逐帧动画
不要再使用很多个图片逐帧播放来实现一个动画效果了.
尽量用改变属性的动画来节省资源使用.

另外还有一些手段比如在程序中绘制, 而不是使用图片.


## 安装包瘦身措施: Post-coding阶段
### Lint静态分析
Lint是静态分析工具, 会提示项目中的各种问题.

针对不用的资源, 可以专门这样检查: 
在Android Studio中: `Analyze -> Run Inspection by Name... -> Unused Resources`.

扫描后会对所有未使用的资源给出警告.

注意, 因为是Lint静态检查, 所以会有一些错报和漏报的情况:
* `assets/`目录不被扫描.
* 检测不到第三方库中带来的不用的资源.
* 错报: 资源在项目中实际上用了, 但是被检测出来了. 如果资源是被动态引用的, 比如用`getIdntifer()`方法, 拼名字使用, 会被lint错误地报告说没有用到.
* 漏报: 实际上没有用到, 但是没有检测出来. 这是因为还有另一个没有用到的代码引用了这个资源. 
比如有一个没有人用的Fragment, 它的布局文件就不会被检测出来, 因为写在了代码里. (注意: 如果是不用的布局中include了另一个布局, 这两个布局都能被检测出来.)

静态检查只是帮我们检测并报告, 真正要移除这些资源还得靠我们手动删除.

### 代码压缩和混淆
ProGuard -> R8会帮我们删除不用的代码, 进行名称改写和代码优化.
```
minifyEnabled true
```

不用的代码被移除了, 类和成员的名称都被改得很短, 从而有效地减小了dex文件的大小.
这里需要注意rules的编写, 不要过度保护.

比较追求极致的方法会从这个方面下手, 进一步细化优化ProGuard Rules, 以达到更深度的混淆和压缩.

### shrinkResources
移除不用资源的一个有利工具`shrinkResources`:
```
android {
    // Other settings

    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```
为了开启压缩资源, 必须先开启压缩代码, 即`minifyEnabled`要为`true`.
这样, 在打包的时候, 首先移除了不用的代码, 然后Gradle才能检测哪些资源还在被代码引用, 从而移除不用的资源.

注意这个过程并没有真的删除掉资源, 只是将不用的资源替换了一个简单版本.

跟代码压缩一样, 资源压缩也可以指定一个自定义的规则, 明确指明哪些资源要保留.

注意开启了代码压缩和混淆之后, 编译速度会变慢, 所以一般debug版本不开启.

#### 这种删除方式靠谱吗? 精确吗? 会误删吗?
常规情况下resource shrinker还挺准确的.

但是如果你的代码(或依赖的库)中有用到`Resources.getIdentifier()`方法来获取资源, (比如AppCompat中这么做), 那么说明你的代码会根据一个动态生成的字符串来查找资源.

如:
```
val name = String.format("img_%1d", angle + 1)
val res = resources.getIdentifier(name, "drawable", packageName)
```
这个时候resource shrinker默认会表现得很保守, 会保留所有符合这个名字模式的资源, 认为它们都是可能会被用到的, 从而不会删除它们.

而且resource shrinker还会扫描字符串, 查找符合资源格式的URLs, 比如`file:///android_res/drawable//ic_plus_anim_016.png`, 从而保留这些对应的资源.

所以不用担心, 默认情况下, 资源压缩都是"better safe than sorry"的模式.

如果你想激进一点, 可以在`keep.xml`里面把压缩模式指定为`shrinkMode="strict"`, 这时候你就必须手动指定需要keep的动态使用的资源了.

#### 如何查看都删了什么呢?
用命令行:
```
./gradlew clean assembleRelease --info | grep "Skipped unused resource"

```
注意这里`assembleRelease`可以替换成任何一个开启了资源压缩的type.

执行了之后会列出所有被替换成dummy file的资源.

用IDE:
在Android Studio的Build窗口下, Toggle View切换到详细信息模式下. Build, 然后查找信息, 可以看见信息中会有一句:
```
Removed unused resources: Binary resource data reduced from XXXKB to YYYKB: Removed Z%
```
显示资源压缩节省出来的大小.

文件:
在app的输出路径(`<module-name>/build/outputs/mapping/release/`)下, 和`mapping.txt`一起, 还有一个`resources.txt`文件.
文件的最后会列出一系列的`Skipped unused resource`.

同时这个文件里也详细地写了资源之所以会被保留下来的理由. 关键字: `matches string pool`. 这是前面说过的为了防止有动态使用资源的情况而采取的保守措施.

如果有空的话可以把这些`Skipped unused resource`标注的文件都手动从代码库里删除.


### 声明支持的配置, 删掉其他
Gradle的resource shrinker会删除没有被代码用到的资源, 但是不会删掉不同设备配置下的可替代资源
([alternative resources](https://developer.android.com/guide/topics/resources/providing-resources.html#AlternativeResources)).

可以利用`resConfigs`, 来删除你的app并不需要的配置资源.

原理就是声明app支持的配置, 这样其他不支持的配置的资源就会被删除, 从而减少apk的尺寸.

比如你包含了一个库, 这个库中带有各种语言的资源, 这样默认你的app就会包含所有这些库中包含的语言的字符串. 你可以明确声明app支持的语言, 那么其他语言资源就会被删除:
```
android {
    defaultConfig {
        ...
        resConfigs "en", "fr"
    }
}
```

见[Remove unused alternative resources](https://developer.android.com/studio/build/shrink-code.html#unused-alt-resources)

## 安装包瘦身措施: 发布阶段
因为有的资源会提供多个版本, 所以一个用户下载的apk中可能包含了一些他永远也用不到的内容.

为了保证每个用户的下载尺寸都尽量小, 我们可以改进发布包的形式: 拆分成多个针对不同机型的apks;
如果是在Google Play发布, 还可以用App Bundle, 由Google Play帮我们做拆分.

### Build多个apks
根据屏幕密度和指令集(ABI)的不同, 可以拆分多个apk, 每种配置只下载自己需要的资源,  从而达到减小安装包大小的目的.

Gradle就可以帮我们做这个拆分创建的工作.
具体实现见:
[Build multiple APKs](https://developer.android.com/studio/build/configure-apk-splits.html).


### Android App Bundles
用Google Play发布应用时可以使用[Android App Bundle](https://developer.android.com/guide/app-bundle)的格式, 这是一种新的发布格式, 包含了应用所有的编译代码和资源, 把apk的生成和签名留给Google Play来做.

Google Play会根据用户不同的设备配置来生成优化的apk, 这样用户下载的就只是对应自己设备的代码和资源. 开发者不用为不同设备准备多个apk, 用户也得到了更小更优化的结果.

## 安装包瘦身措施: 进阶补充
要更加极致地压缩安装包, 还有一些更加复杂的手段, 比如:
* 业务层面: 动态feature, 资源动态加载.
* Dex: 加大混淆力度, 混淆四大组件和View; 去除debug信息; 优化Dex分包; Dex压缩.
* native库: 去除debug信息; 压缩; 剪裁.
* 资源: 资源名混淆成短路径; 压缩.

另外, 还可以将对安装包体积的分析加入到CI中, 进行自动化监控, 这样可以得到各个版本的尺寸变化, 及时获知是否不经意引入大的依赖, 引入过大的资源等情况.

## 总结
总结回顾一下本文内容, 对于安装包瘦身的话题:
首先, 讨论了我们为什么要做这个事情 -> 为了增加app被下载安装使用和留存的几率. (为什么要瘦身呢? -> 为了提升自己增加竞争力.)
其次, 介绍了安装包的构成, 相关的指标和用于检测的工具. (瘦身需要关注的指标和测量方式.)
重点介绍, 如何减少安装包体积的实践方法. (瘦身的各种手段.)


## 参考资料
* [Reduce your app size](https://developer.android.com/topic/performance/reduce-apk-size.html)
* [Shrink, obfuscate, and optimize your app](https://developer.android.com/studio/build/shrink-code.html)
* [Configure APK Splits](https://developer.android.com/studio/build/configure-apk-splits.html)
* [Build multiple APKs](https://developer.android.com/studio/build/configure-apk-splits.html)
* [Screen sizes and densities](https://developer.android.com/about/dashboards/index.html#Screens)
* [Analyze your build with APK Analyzer](https://developer.android.com/studio/build/apk-analyzer)
* [支付宝 App 构建优化解析：Android 包大小极致压缩](https://mp.weixin.qq.com/s/_gnT2kjqpfMFs0kqAg4Qig)
* [美团 - Android App包瘦身优化实践](https://tech.meituan.com/2017/04/07/android-shrink-overall-solution.html)
* 极客时间: Android开发高手课, 包体积优化 (上), (下)