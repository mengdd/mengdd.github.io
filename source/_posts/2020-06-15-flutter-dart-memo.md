---
title: Dart Memo for Android Developers
date: 2020-06-15 00:34:57
tags: [Flutter, Dart]
categories: [Flutter]
---


# Dart Memo for Android Developers
Dart语言一些语法特点和编程规范.

本文适合: 日常使用Kotlin, 突然想写个Flutter程序的Android程序员.

<!-- more -->
## Dart语言
完整的请看[A tour of the Dart language](https://dart.dev/guides/language/language-tour)
* 创建对象可以不用`new`. -> 并且规范不让用`new`, lint会报错.
* 声明变量可以用`var`, 也可以用具体类型如`String`. 不变量用`final`, 常量用`const`.
* 没有访问修饰符, 用`_`来表示私有: 文件级别.
* 字符串可以用单引号`'`.
* 语句结尾要用`;`.
* 创建数组可以用: `var list = [1, 2, 3];`.
* `assert()`常用来断定开发时不可能会出现的情况.
* 空测试操作符: `??`.
* 过滤操作符: `where`.
* 两个点`..`表示链式调用.
* `dynamic`说明类型未指定.
* 除了throw异常, 还可以throw别的东西, 比如字符串.

### 函数
* 函数返回值在函数最开头, 可以不标. -> 但是规范会建议标注返回值.
```
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```
* `=>`箭头符号, 用来简化一句话的方法.
```
bool isNoble(int atomicNumber) => _nobleGases[atomicNumber] != null;
```
### 构造函数
* 构造函数`{}`表示带名字, 参数可选, 若要必选加上`@required`.
```
const Scrollbar({Key key, @required Widget child})
```
* 构造函数名可以是`ClassName`或者`ClassName.identifier`.
* 空构造函数体可以省略, 用`;`结尾就行:
```
class Point {
  double x, y;
  Point(this.x, this.y);
}
```
这里会初始化相应的变量, 也不用声明具体的参数类型.
* 有`factory`构造, 可以用来返回缓存实例, 或者返回类型的子类:
```
factory Logger(String name) {
    return _cache.putIfAbsent(name, () => Logger._internal(name));
}
```

### 异步代码
```
Future<String> lookUpVersion() async => '1.0.0';

Future checkVersion() async {
  var version = await lookUpVersion();
  // Do something with version
}
```

## 编程规范类
完整的规范在这里: [Effective Dart](https://dart.dev/guides/language/effective-dart).

有一些Good和Bad的举例, 这里仅列出比较常用的几项.

* 文件名要蛇形命名: `lowercase_with_underscores`. 类名: `UpperCamelCase`.
* 对自己程序的文件, 两种import都可以(package开头或者相对路径), 但是要保持一致.
* Flutter程序嵌套比较多, 要用结尾的`,`来帮助格式化.



## 本文缘由
年初的时候学了一阵子Flutter, 写了各种大小demo. 结果隔了两个月之后, 突然心血来潮想写个小东西, 打开Android Studio, 首先发现创建Flutter程序的按钮都不见了. (估计是Android Studio4.0升级之后Flutter的插件没跟上).

接着用命令行创建了工程, 打开之后稍微整理了一下心情, 然后就....懵逼了. 

突然不知道如何下手.
宏观的东西还记得, 要用什么package, 基本常用的几个Widget都是啥, 但是微观的, 忘了函数和数组都是咋定义的了.
这种懵逼的状态令我很愤怒, 果然是上年纪了吗, 无缝切换个语言都不行.

于是就想着还是写个备忘录吧.

## References
* [A tour of the Dart language](https://dart.dev/guides/language/language-tour)
* [Effective Dart](https://dart.dev/guides/language/effective-dart)