---
title: Android 7.1 App Shortcuts使用
date: 2016-10-25 14:00:01
tags: [Android, Android 7, Nougat, Shortcuts]
categories: [Android, Nougat]
---

# Android 7.1 App Shortcuts使用
Android 7.1已经发了预览版, 这里是API Overview: [API overview](https://developer.android.com/preview/api-overview.html).
其中App Shortcuts是新提供的一种快捷访问方式, 形式为长按应用图标出现的长条.

![app shortcuts](/images/app-shortcuts.png)
图来自: [Exploring Android Nougat 7.1 App Shortcuts](https://catinean.com/2016/10/20/exploring-android-nougat-7-1-app-shortcuts/)

点击快捷方式可以访问应用功能, 并且这种快捷方式也可以被拖拽到桌面单独放置.

<!-- more -->

## App Shortcuts 是什么
其中App Shortcuts是指在桌面长按app图标而出现的快捷方式, 可以为你的app的关键功能添加更快速的入口而不用先打开app.

![app shortcuts](/images/app-shortcuts-doc.png)

点击快捷方式可以访问应用功能, 并且这种快捷方式也可以被拖拽到桌面单独放置, 变成单独的桌面快捷方式(pinned shortcuts).

有两种shortcuts:
- 静态的: 在xml中定义, 适用于一些通用的动作.
- 动态的: 由[ShortcutManager](https://developer.android.com/reference/android/content/pm/ShortcutManager.html)发布, 可以根据用户的行为或者偏好添加, 可以动态更新.

每一个应用目前最多可以有5个shortcuts(静态 + 动态).

运行条件: 
应用添加App Shortcuts是Android 7.1(API 25)的API, 所以只能在Android 7.1的设备上显示, 同时需要launcher支持, 比如Pixel launcher(Pixel设备的默认launcher), Now launcher(Nexus设备上的launcher)现在就支持, 其他launcher也可以提供支持.

## 静态Shortcuts使用
静态的Shortcuts是写在xml中的, 直到下一次应用升级, 不能被改变.
要添加静态shortcuts只需两步:
首先, 在应用的Manifest中启动Activity上添加`<meta-data>`:

```xml
    <activity android:name=".MainActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>

        <meta-data
            android:name="android.app.shortcuts"
            android:resource="@xml/shortcuts" />
    </activity>
```

然后在`res/xml/`目录下创建`shortcuts.xml`文件, 里面包含静态的shortcuts:
```xml
<?xml version="1.0" encoding="utf-8"?>
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:enabled="true"
        android:icon="@drawable/ic_check_circle_black_24dp"
        android:shortcutDisabledMessage="@string/static_shortcut_disabled_message"
        android:shortcutId="static"
        android:shortcutLongLabel="@string/static_shortcut_long_label_1"
        android:shortcutShortLabel="@string/static_shortcut_short_label_1">
        <intent
            android:action="android.intent.action.VIEW"
            android:targetClass="com.ddmeng.hellonougat.shortcuts.StaticShortcutActivity"
            android:targetPackage="com.ddmeng.hellonougat" />
    </shortcut>
    <shortcut
        android:enabled="true"
        android:icon="@drawable/ic_android_black_24dp"
        android:shortcutDisabledMessage="@string/static_shortcut_disabled_message"
        android:shortcutId="static_2"
        android:shortcutLongLabel="@string/static_shortcut_long_label_2"
        android:shortcutShortLabel="@string/static_shortcut_short_label_2">
        <intent
            android:action="android.intent.action.MAIN"
            android:targetClass="com.ddmeng.hellonougat.MainActivity"
            android:targetPackage="com.ddmeng.hellonougat" />
        <intent
            android:action="com.ddmeng.hellonougat.action.STATIC_SHORTCUT_2"
            android:targetClass="com.ddmeng.hellonougat.shortcuts.StaticShortcutActivity"
            android:targetPackage="com.ddmeng.hellonougat" />
    </shortcut>
</shortcuts>
```
这就好了, 这个文件添加了两个shortcuts, 点击都将打开指定的Activity, 本例子中是`StaticShortcutActivity`.

### 用多个Intent构建back stack
上面这个文件里添加了两个静态的shortcuts, 第一个关联了一个Activity, 点击shortcut将直接打开这个Activity, 回退的时候回到桌面.


如果你想要的效果是点击back键回到应用里的某个界面, 那么可以利用多个intents来构建back stack, 比如在第二个shortcut里面, 点击shortcut还是打开目标Activity, 这个指定目标Activity的Intent放在最后, 但是回退会返回到MainActivity, 即之前的那个Intent.


## 动态Shortcuts使用
动态的shortcuts可以在用户使用app的过程中构建, 更新, 或者删除. 
使用[ShortcutManager](https://developer.android.com/reference/android/content/pm/ShortcutManager.html)可以对动态shortcuts完成下面几种操作:
- Publish发布: `setDynamicShortcuts()`, ` addDynamicShortcuts(List)`;
- Update更新: `updateShortcuts(List)`;
- Remove删除: `removeDynamicShortcuts(List)`, `removeAllDynamicShortcuts()`.

比如添加一个动态shortcut:
```java
ShortcutManager shortcutManager = getSystemService(ShortcutManager.class);

ShortcutInfo shortcut = new ShortcutInfo.Builder(this, "id1")
    .setShortLabel("Web site")
    .setLongLabel("Open the web site")
    .setIcon(Icon.createWithResource(context, R.drawable.icon_website))
    .setIntent(new Intent(Intent.ACTION_VIEW,
                   Uri.parse("https://www.mysite.example.com/")))
    .build();

shortcutManager.setDynamicShortcuts(Arrays.asList(shortcut));

```
点击这个shortcut会发出一个打开网页的Intent, 让你选择浏览器, 从而打开网址.

### 多个Intent构建back stack
动态的shortcut仍然可以用多个Intent来指定一个back stack, 那么打开目标Activity之后就可以返回到应用中的指定界面而不是回到launcher:
```java
ShortcutInfo dynamicShortcut2 = new ShortcutInfo.Builder(this, "shortcut_dynamic")
        .setShortLabel("Dynamic Shortcut")
        .setLongLabel("Open Dynamic shortcut 2")
        .setIcon(Icon.createWithResource(this, R.drawable.ic_favorite_border_black_24dp))
        .setIntents(
                // this dynamic shortcut set up a back stack using Intents, when pressing back, will go to MainActivity
                // the last Intent is what the shortcut really opened
                new Intent[]{
                        new Intent(Intent.ACTION_MAIN, Uri.EMPTY, this, MainActivity.class).setFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK),
                        new Intent(DynamicShortcutActivity.ACTION_OPEN_DYNAMIC)
                        // intent's action must be set
                })
        .build();
```
和静态一样, 最后一个Intent对应的是shortcut打开的界面`DynamicShortcutActivity`, 前面的都是用来构建back stack, 即back退回到MainActivity.
注意这里的Intent必须指定Action, 否则会抛出异常.

## Shortcuts的个数限制
Shortcuts的总数不能超过5个, 即静态和动态shortcuts加起来总数最多是五个.
当我们尝试添加第六个shortcut时, 应用会抛出异常: `java.lang.IllegalArgumentException: Max number of dynamic shortcuts exceeded`.

虽然总数限制是5个, 但是当我正好有5个(2个静态 + 3个动态)的时候, 长按只显示了4个shortcuts.
如图:

![app shortcuts](/images/app-shortcuts-demo-screenshot.png)

本文完整代码见: Demo地址: [HelloNougat](https://github.com/mengdd/HelloNougat).


## Shortcuts的次序
当我们有多个Shortcuts之后, 默认它们是按照添加顺序排列的, 即按照添加顺序rank递增.

可以通过`setRank()`来改变长按时它们显示的排序:
```java
@TargetApi(25)
private void updateDynamicShortcuts() {
    ShortcutInfo webShortcut = new ShortcutInfo.Builder(MainActivity.this, "shortcut_blog")
            .setRank(1)
            .build();

    ShortcutInfo dynamicShortcut = new ShortcutInfo.Builder(MainActivity.this, "shortcut_dynamic")
            .setRank(0)
            .build();
    // the rank value can not be set to negative, otherwise will throw
    // java.lang.IllegalArgumentException: Rank cannot be negative or bigger than MAX_RANK

    // the static shortcuts have the rank 0, so they will always be closest to launcher icon

    shortcutManager.updateShortcuts(Arrays.asList(webShortcut, dynamicShortcut));
}
```
这样更改之后, 原先排在最远端的`shortcut_dynamic`被移到了第三个, `shortcut_blog`被移到了它的后面.

`setRank()`不接受负值, 会抛出异常.

我们只能改变动态shortcuts的排序, 静态的shortcuts等级为0, 它们是按照xml中写定的先后顺序排的, 所以: 
`静态的shortcuts永远离应用icon最近, 动态shortcuts在其之上排序, rank越大的离应用icon越远.`
如果没有指定rank, 则按生成的顺序递增.


## 参考
App Shortcuts的官方文档: [App Shortcuts](https://developer.android.com/preview/shortcuts.html)
[Exploring Android Nougat 7.1 App Shortcuts](https://catinean.com/2016/10/20/exploring-android-nougat-7-1-app-shortcuts/)

Demo地址: [HelloNougat](https://github.com/mengdd/HelloNougat).
近期考虑加入更多Android 7 Nougat特性sample.




