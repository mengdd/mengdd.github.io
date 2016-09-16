---
title: Android Weekly Notes Issue 220
date: 2016-09-01 14:05:48
tags: [Android, Android Weekly, Gradle, Docker, Bottom Sheet, Certificate, Retrofit, AnimatedVectorDrawable, Fragment, RxJava, MVP, AsyncLayoutInflater, DiffUtil, Recorder, tiger]
categories: [Android, Android Weekly]
---

# Android Weekly Issue #220
August 28th, 2016
[Android Weekly Issue #220](http://androidweekly.net/issues/issue-220)

<!-- more -->

## ARTICLES & TUTORIALS
### [Manage dependencies versions with gradle extra properties](https://segunfamisa.com/posts/android-gradle-extra-properties)
依赖管理的小Tip: 把依赖的版本号作为变量管理.
改造之后, build.gradle文件变成这样:
```
apply plugin: 'com.android.application'
android {
    ...
}
...

ext {
    supportLibraryVersion = '23.4.0'
    playServicesVersion = '9.2.1'
}

dependencies {
    // support libraries
    compile "com.android.support:appcompat-v7:$supportLibraryVersion"
    compile "com.android.support:design:$supportLibraryVersion"
    compile "com.android.support:percent:$supportLibraryVersion"
    compile "com.android.support:cardview-v7:$supportLibraryVersion"
    compile "com.android.support:gridlayout-v7:$supportLibraryVersion"

    //play services
    compile "com.google.android.gms:play-services-location:$playServicesVersion"
    compile "com.google.android.gms:play-services-gcm:$playServicesVersion"

    // other dependencies
    ...
}
```
定义了版本号变量, 原来hardcode时的单引号变成了双引号, 然后用$符号取变量值.

上面这个是app module里面使用的例子, 如果你的应用有多个module怎么办呢?
当然一种办法是每个module里定义一组版本号变量, 更方便的办法是在项目工程总目录的build.gradle文件里定义变量.
可以在工程的build文件里写
```
ext {
    // sdk and tools
    minSdkVersion = 14
    targetSdkVersion = 23
    compileSdkVersion = 23
    buildToolsVersion = '23.0.2'

    // dependencies versions
    supportLibraryVersion = '23.4.0'
    playServicesVersion = '9.2.1'
}
```
也可以这样定义:
```
project.ext.supportLibVersion = '24.0.0'
```
使用的时候可以这样取值: `$rootProject.supportLibraryVersion`.
也可以省略前面的rootProject, 直接取`$supportLibraryVersion`

### [Android CI with Docker](https://medium.com/@Malinskiy/android-ci-with-docker-a2f522086640#.ud9unt793)
作者讲了他怎么用Docker搭建CI.
1. 环境:
首先, CI需要Android环境(JDK 7&8, Android SDK, Gradle, Release keychain, google-services.json, etc).
装了这些环境之后, 需要保证他们在每一个CI实例上都是同步更新的.
用了Docker之后, 更新环境的步骤变为:
更新你的Dockerfile -> Push到版本管理系统 -> CI会build新的image, 然后push到docker registry.

2. Build:
`docker run -v ./app:/opt/app docker-ci-android:latest gradle assembleRelease`

3. Test:
有两种测试, 一种是单元测试, 只需要JVM; 另一种是UI或者功能测试, 需要Android.
emulator会有一些问题: [why](https://developer.android.com/training/articles/smp.html)
所以你可能想要在更真实的机器上测试: [STF](https://github.com/openstf/stf)提供了服务, 你只需要用这个[stf-client](https://github.com/Malinskiy/stf-client).

4. Deploy:
部署用一些gradle的task就可以完成.
[fabric](https://docs.fabric.io/android/beta/gradle.html#distribution-with-gradle)
[gradle-play-publisher](https://github.com/Triple-T/gradle-play-publisher)

后面还提到了一些扩展和问题.

### [Bottom Sheets in Android](http://mayojava.github.io/android/bottom-sheets-android/)
BottomSheet是support library 23.2加入的, 是从底部滑上来的一个块块, 用来向用户展现更多内容.
Support Library提供了:
`BottomSheetBehavior`: 加在`CoordinatorLayout`的直接child view上, 然后在java代码里get出来, 设置state控制其状态.
有HIDE, COLLAPSED和EXPANDED三种状态, 分别对应隐藏, 展开到指定高度(peekHeight)和完全展开.
`BottomSheetDialog`:
`BottomSheetDialogFragment`.
Behaviour是给View加行为, 后面这两种是更加模块化的dialog, 状态控制都一样.

这里推荐一下笔者自己的demo: [AndroidDesignWidgetsSample](https://github.com/mengdd/AndroidDesignWidgetsSample)
再推荐一下这篇文章里面的Bottom Sheets部分: [CodePath-Handling-Scrolls-with-CoordinatorLayout](https://guides.codepath.com/android/Handling-Scrolls-with-CoordinatorLayout)

### [Certificate public key pinning using Retrofit 2](https://medium.com/@sreekumar_av/certificate-public-key-pinning-in-android-using-retrofit-2-0-74140800025b#.9ajsf36qp)
SSL handshake, 交换了证书(Certificate), 这样客户端就可以通过证书来验证服务器的身份.
什么是Certificate public key pinning呢? 也叫作SSL pinning.

把host name和[public key](https://tools.ietf.org/html/rfc7469#section-2.4)关联起来, 这个public key将用来和证书中的public key比较, 如果匹配了, 就证明你正在和正确的server通信.
而直接pinning证书相比pinning public key更容易一些, 但是也有不好的地方, 如果网站(比如Google)经常轮换证书(rotate its certificate), 你的应用就也得经常更新, 而这种情况一般证书里面的public keys是保持不变的.
如何在Android中用Retrofit实现pinning呢?
首先需要网站的public key的hash, 有很多获取方法, 参见[okhttp3-CertificatePinner](https://github.com/square/okhttp/blob/master/okhttp/src/main/java/okhttp3/CertificatePinner.java).
然后构建CertificatePinner类对象, 加到OkHttpClient上.
```java
 CertificatePinner certificatePinner = new CertificatePinner.Builder()
                    .add("api.github.com", "sha256/6wJsqVDF8K19zxfLxV5DGRneLyzso9adVdUN/exDacw=")
                    .build();
 final OkHttpClient client = httpBuilder.certificatePinner(certificatePinner).build();

  Retrofit retrofit = new Retrofit.Builder()
                    .baseUrl(END_POINT)
                    .addConverterFactory(GsonConverterFactory.create())
                    .client(client)
                    .build();
```
TLSv1.2从Android16+开始支持, 但是对于20+的设备默认是disabled的, 为了强制获取支持, 可以继承[SSLSocketFactory](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/6-b14/javax/net/ssl/SSLSocketFactory.java), 强制设置为enabled, 代码见原文吧.
Github上有完整的代码[PublicKeyPinning](https://github.com/pollux-/PublicKeyPinning)
作者最后还推荐了一个测试的工具[mitmproxy](https://mitmproxy.org/).

### [Isometric AnimatedVectorDrawable - Part 3](https://blog.stylingandroid.com/isometric-animatedvectordrawable-part-3/)
作者继续讲了他如何构建方块地形图的动态效果.
一个AnimatedVectorDrawable的xml文件实际上是用来建立一个映射关系, 关联objectAnimators和VectorDrawable上的独立元素. 我们可以建立一个objectAnimator, 操纵我们的一块元素的动画效果.
文中实现了让方块地形动起来的动画效果.

### [The many flavors of commit()](https://medium.com/@bherbst/the-many-flavors-of-commit-186608a015b1#.uwl2v86cx)
**FragmentTransaction的提交方法**:
support library的`FragmentTransaction`现在提供了四种不同的方法来commit一个transaction:
[commit()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commit%28%29)
[commitAllowingStateLoss()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commitAllowingStateLoss%28%29)
[commitNow()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commitNow%28%29)
[commitNowAllowingStateLoss()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commitNowAllowingStateLoss%28%29)
这篇文章分析了这四个方法的不同.

**commit() vs commitAllowingStateLoss()**:
用`commit()`提交有时候会遇到`IllegalStateException`, 说你在`onSaveInstanceState()`之后提交, 这里有另一个文章很好地分析了这个问题:[Fragment Transactions & Activity State Loss](http://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html)
`commit()`和`commitAllowingStateLoss()`在实现上唯一的不同就是当你调用`commit()`的时候, FragmentManger会检查是否已经存储了它自己的状态, 如果已经存了, 就抛出`IllegalStateException`.
那么如果你调用的是`commitAllowingStateLoss()`, 并且是在`onSaveInstanceState()`之后, 你可能会丢失掉什么状态呢?
答案是你**可能**会丢掉FragmentManager的状态, 即save之后任何被添加或被移除的Fragments.
举例说明:
1.在Activity里显示一个FragmentA;
2.然后Activity被后台, `onStop()`和`onSaveInstanceState()`被调用;
3.在某个事件触发下, 你用FragmentB replace FragmentA , 使用的是 `commitAllowingStateLoss()`.
这时候, 用户再返回应用, 可能会有两种情况发生:
1.如果系统杀死了你的activity, 你的activity将会重建, 使用了上述步骤2保存的状态, 所以A会显示, B不会显示;
2.如果系统没有杀死你的activity, 它会被提到前台, FragmentB就会显示出来, 到下次Activity stop的时候, 这个包含了B的状态就会被存下来.
(上述测试可以利用开发者选项中的”Don’t Keep Activities”选项).
那么你要选择哪一种呢? 这就取决于你提交的是什么, 还有你是否能接受丢失.

**commit(), commitNow() 和 executePendingTransactions()**:
使用`commit()`的时候, 一旦调用, 这个commit并不是立即执行的, 它会被发送到主线程的任务队列当中去, 当主线程准备好执行它的时候执行.
`popBackStack()`的工作也是这样, 发送到主线程任务队列中去. 也即说它们都是异步的.

但是有时候你希望你的操作是立即执行的, 之前的开发者会在`commit()`调用之后加上 `executePendingTransactions()`来保证立即执行, 即变异步为同步.
support library从v24.0.0开始提供了 `commitNow()`方法, 之前用`executePendingTransactions()`会将所有pending在队列中还有你新提交的transactions都执行了, 而`commitNow()`将只会执行你当前要提交的transaction. 所以`commitNow()`避免你会不小心执行了那些你可能并不想执行的transactions.

但是你不能对要加在back stack中的transaction使用`commitNow()`, 即`addToBackStack()`和`commitNow()`不能同时使用.
为什么呢?
想想一下, 如果你有一个提交使用了`commit()`, 紧接着又有另一个提交使用了`commitNow()`, 两个都想加入back stack, 那back stack会变成什么样呢? 到底是哪个transaction在上, 哪个在下? 答案将是一种不确定的状态, 因为系统并没有提供任何保证来确保顺序, 所以系统决定干脆不支持这个操作.

前面提过`popBackStack()`是异步的, 所以它同样也有一个同步的兄弟`popBackStackImmediate()`.

所以实际应用的时候怎么选择呢?
1. 如果你需要同步的操作, 并且你不需要加到back stack里, 使用`commitNow()`.
support library在FragmentPagerAdapter里就使用了commitNow()来保证在更新结束的时候, 正确的页面被加上或移除.
2. 如果你操作很多transactions, 并且不需要同步, 或者你需要把transactions加在back stack里, 那就使用`commit()`.
3. 如果你希望在某一个指定的点, 确保所有的transactions都被执行, 那么使用`executePendingTransactions()`.

### [Break circular dependency with RxJava](https://medium.com/@ferhatparmak/break-your-circular-dependency-with-rxjava-8a487345061#.4718laogc) 用RxJava打破循环依赖.
当你把代码分成各个部分, 比如用MVP, 这些各个部分之间可能会有相互依赖, 比如View需要Presenter, Presenter也需要View.
作者也没有说双向关联有什么缺点, 但是他说RxJava可以把这种双向的依赖改成单向的.
作者的办法是使用[RxBinding](https://github.com/JakeWharton/RxBinding)把button的click事件变成一个Observable, 然后Presenter监听click这个Observable, 后面接一个flatMap, 里面发网络请求, 得到结果之后再调用view的方法.
这么一改以后View中就不需要再持有Presenter的引用了.
举这个例子, 最后是想说, 如果你想从A中调用B的异步方法, 你不用总是在A中保存一个B的引用, 你可以把A中的事件作为一个Observable. 这样只需要B保存了A的引用就可以了.

### [Asynchronous layout inflation](https://medium.com/@lupajz/asynchronous-layout-inflation-7cbca2653bf#.lld73d5uq) 异步解析layout
最近的support library revision 24中, Google的开发者在v4包中加入了一个新的辅助类[AsyncLayoutInflater](https://developer.android.com/reference/android/support/v4/view/AsyncLayoutInflater.html), 来实现布局的异步解析.

我们现在常用的布局解析inflate方法都是同步的, 那什么时候需要异步地做这件事情呢?
比如你想延迟加载布局中的一块, 或者你想把布局解析作为用户某个交互的一个响应. 这样就可以用这个异步布局解析类, 保证了主线程在inflation进行的时候仍然可响应.
怎么使用呢?
首先, 在主线程创建对象`AsyncLayoutInflater(this)`,
用它inflate布局的时候第三个参数是一个`OnInflateFinishedListener`回调.
以前同步方法的第三个参数是一个boolean, 说布局是否需要attach到parent上, 现在没有这个boolean参数了.
当然, 使用异步解析也有缺点:
- 父类方法`generateLayoutParams()`必须是线程安全的.
- 被创建的所有View不能创建Handler,或者调用`Looper.myLooper()`方法.
- 不支持设置`LayoutInflater.Factory`和`LayoutInflater.Factory2`
- 不支持布局里有Fragment.
如果我们要异步inflate的布局不能支持异步, inflate的过程将会自动转化为在UI线程的解析.
作者文中附有Kotlin的例子.

### [Introduction to Automated Android Testing - Part 5](https://riggaroo.co.za/introduction-automated-android-testing-part-5/)
系列文章的第五篇, 之前第四篇的时候写了Presenter, 定义了V和P的接口, 本篇接着写View接口的实现.

这里Presenter和View关联作者写了两个attachView()和detachView()方法, 前者在Presenter构造之后调用, 后者在Activity的onDestroy()里调用. 这里同时会unregister RxJava的subscriptions, 避免了内存泄露的发生.

作者在布局时用了`ConstraintLayout`, 关于这个layout的使用她有另一个[blog](https://riggaroo.co.za/constraintlayout-101-new-layout-builder-android-studio/)
另外作者还加了Toolbar上的SearchView, 到此, 作者的这个app就基本完成了.
作者的代码里还有一个Injection类, 用来提供retrofit的service, 即代码中UserRepo的获取, 在Presenter构造时传入.

作者的代码: [GithubUsersSearchApp](https://github.com/riggaroo/GithubUsersSearchApp/tree/testing-tutorial-part5-complete)
预告下一篇将会加入UI测试.

### [DiffUtil is a must!](https://medium.com/@nullthemall/diffutil-is-a-must-797502bc1149#.lqfl9xikm)
support library 24.2.0推出了一个新的辅助类`DiffUtil`, 它是用来解决什么问题的呢?
如果你的RecyclerView.Adapter第一次接收到了新的数据, 这很简单, 只需要将它们显示出来, 但如果已经有了数据, 新的数据又来了, 这时候怎么做才是最好的呢?
[DiffUtil](https://developer.android.com/reference/android/support/v7/util/DiffUtil.html)来了, 它就是专门为了解决RecyclerView的Adapter更新而设计的, 他可以计算出前后两个list的不同, 然后返回一组更新操作, 把第一个list变为第二个list.
`DiffUtil`需要知道你的两个list的基本信息: 长度, 基本item的比较.
`DiffUtil.Callback`是用来向`DiffUtil`提供这些基本信息的, 它是一个抽象类, 你需要继承它, 然后覆写里面的几个方法. 它的构造传入了两个待比较的list, 覆写的方法主要是get它们的size, 比较它们的内容.
Callback里还有一个`getChangePayload()`方法, 它不是抽象的, 这个方法在`areItemsTheSame()` 返回`true`, 但是`areContentsTheSame()`返回`false`的时候被调用.
这意味着我们的item还是之前的那个item,但是可能里面的字段变化了.
这个方法的返回值即为两个对应item的diff, 基本来说, 这个方法返回的是为什么我们认为list变化了.
文中的代码例子返回了一个Bundle, 把compare不相等的字段都放进去了, 用的是new item的值.

一旦我们写好了这个Callback类, 剩下的事情就很简单了, 我们只需要在新数据到来的时候计算一下diff, 然后更新.
```java
@ Override
public void onNewProducts(List<Product> newProducts) {
    DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new ProductListDiffCallback(mProducts, newProducts));
    diffResult.dispatchUpdatesTo(mProductAdapter);

}
```

当然上面`getChangePayload()`返回的对象还得我们自己利用起来, 它会被`DiffResult`分发到Adapter.
用的是`notifyItemRangeChange(position, count, payload)`方法, 传到了Adapter的`onBindViewHolder()`方法, 我们判断payload不为空时, 从里面拿出diff做更新.

文档里说`DiffUtil`对很大的数据集可能比较费时, 所以建议把计算放在后台线程.

作者还给出了一个RxJava的例子, 各种flatMap.

## DESIGN
[Diverse Device Hands](http://facebook.design/handskit)
Facebook的design资源, 很多拿着手机的手的照片.

## LIBRARIES & CODE
### [unipiazza-android-twostepslogin](https://github.com/unipiazza/unipiazza-android-twostepslogin)
一个实现两步登录的库, 比如Google web登录, Material Design.
要用它的布局, 然后设置一些属性, 还有UI交互事件的Listener.

### [Om Recorder](https://kailash09dabhi.github.io/OmRecorder/)
一个简单的Pcm / Wav 录音机, API简单, 可以录制Pcm和Wav音频, 可以配置输出, 有暂停功能.

### [tiger](https://github.com/google/tiger)
又一个依赖注入库, 但是README里说这不算一个Google的官方产品, 官方的是[Dagger](https://github.com/google/dagger)和[Guice](https://github.com/google/guice).
这个tiger好像自称是目前最快的java依赖注入.

## NEWS
[Taking the final wrapper off of Android 7.0 Nougat](http://android-developers.blogspot.com.au/2016/08/taking-final-wrapper-off-of-nougat.html)
Android 7.0已经问世了, 从Nexus开始, 同时API 24的source code已经push到AOSP了.