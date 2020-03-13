---
title: Flutter json 2 model with Built Value
date: 2020-03-13 16:18:13
tags: [Flutter]
categories: [Flutter]
---

# Flutter json 2 model with Built Value
flutter中json转换model, 除了手动转之外, 就是利用第三方库做一些代码生成. 
流行的库有: [json_serializable](https://pub.dev/packages/json_serializable)和[built_value](https://pub.dev/packages/built_value)

本文介绍[built_value](https://pub.dev/packages/built_value)的实际使用及问题处理.

<!-- more -->

## Flutter中的json转model方法
Flutter中json到model类型的转换可以有多种方式:

* 利用官方自带的dart convert中的json解码. 该方法只能将json转换为List或Map, 剩下的工作需要手动完成, 根据key取值赋值给model的字段.
* 利用第三方的库, 做代码生成, 流行的库有: [json_serializable](https://pub.dev/packages/json_serializable)和[built_value](https://pub.dev/packages/built_value). 原理都是相同的, 先写一些模板代码, 说明一下model是什么样子的, 然后运行命令行生成一些代码, 之后就可以很方便地调用, 将json转换为model了.

使用json_serializable可以看:

* 官网的例子: [Serializing JSON using code generation libraries](https://flutter.dev/docs/development/data-and-backend/json#code-generation).
* Flutter实战中文的: [11.7 Json转Dart Model类](https://book.flutterchina.club/chapter11/json_model.html)

本篇文章主要介绍built value的使用.

## built value使用指南
实例: 用github api拿到的events: https://api.github.com/events?per_page=10
如何转化成model对象呢?

### TDD
先写个测试, 明确一下我们想要的目标.

在`test`下建立一个文件, 比如叫`json_test.dart`.

里面写main函数和两个测试:
```
void main() {
  test("parse events list", () {
    const jsonString = """replace with events list json string""";

    expect(Event.fromEventsListJson(jsonString).first.id, "11732023561");
  });

  test("parse event", () {
    const jsonString = """replace with event json string""";

    expect(Event.fromJson(jsonString).id, "11732036753");
  });
}
```
这里面应该放json字符串的, 太长了我就省略了, 这样看比较清晰.

用`"""`之后可以支持多行. (IDE里面可以折叠的.)

这个`Event`类和方法我们都还没有写, 所以暂时报错.

### setup

添加依赖, 去package页面看添加什么版本: https://pub.dev/packages/built_value

在`pubspec.yaml`中添加:
```
dependencies:
  flutter:
    sdk: flutter

  # other dependencies here

  built_value: ^7.0.9
  built_collection: ^4.3.2

dev_dependencies:
  flutter_test:
    sdk: flutter

  # other dev_dependencies here

  build_runner: ^1.8.0
  built_value_generator: ^7.0.9
```
然后点`Packages get`.


### Live Templates
这个是IntelliJ系IDE(包括Android Studio)的快捷设置, 目的是为了减少手动输入. (可选.)

打开`Preferences`, 搜`Live Templates`.
在`Dart`的部分点+号新增一个Live Template.

下面Abbreviation选一个适当的缩写, 比如`built`.
Template text贴入这段:
```
abstract class $CLASS_NAME$ implements Built<$CLASS_NAME$, $CLASS_NAME$Builder> {
  $CLASS_NAME$._();
  factory $CLASS_NAME$([void Function($CLASS_NAME$Builder) updates]) = _$$$CLASS_NAME$;
}
```
Applicable in Dart选: top-level.

建好之后以后就直接用啦.

### 建立models抽象类
输入刚才建立的live template的关键字`built`, 就会出现要生成的代码, 其中写好自己的类名.

比如我们要建立的model类型是`Event`类. 
新建`event.dart`文件. 

在其中输入`built`按确认之后, 输入类名`Event`, 就建好了:
```
abstract class Event implements Built<Event, EventBuilder> {
  Event._();
  factory Event([void Function(EventBuilder) updates]) = _$Event;
}
```
包括一个私有构造和一个工厂方法. 此时会有一些红色的报错.
这里`import 'package:built_value/built_value.dart'`消除`Built`类的报错.

根据观察API: https://api.github.com/events 返回的json, 发现还应该有`Actor`, `Repo`, `Payload`三个类.
也都按这个方法建立好.

然后在其中添加字段, 现在看起来是这样了:
```
import 'package:built_value/built_value.dart';
// imports for models

part 'event.g.dart';

abstract class Event implements Built<Event, EventBuilder> {
  String get id;

  String get type;

  Actor get actor;

  Repo get repo;

  Payload get payload;

  bool get public;

  String get createdAt;

  Event._();

  factory Event([void Function(EventBuilder) updates]) = _$Event;
}
```

很重要的一步, 就是在类前面添加上一句: `part 'event.g.dart';`.
`g.dart`是一个惯例, 表明这个文件是生成的代码. `part`表示目前这个文件是另一个文件的一部分.

按照同样的方法把几个类都建好.

注意如果有列表字段, 要声明为`BuiltList`类型.

### 运行生成命令
生成命令: 
```
flutter packages pub run build_runner build
```

需要持续构建和可以用:
```
flutter packages pub run build_runner watch
```
这样就不用每次改完代码都需要跑一次命令了.

我们这里用watch, 因为还没有改完.
运行完成之后, 可以看到`.g.dart`的文件们都生成了, 报错也消失了.

### 写Serializers
新建文件`serializers.dart`.

```
import 'package:built_value/serializer.dart';
import 'package:built_value/standard_json_plugin.dart';

// imports for models

part 'serializers.g.dart';

@SerializersFor(const [
  Event,
  Actor,
  Repo,
  Payload,
])
final Serializers serializers =
    (_$serializers.toBuilder()..addPlugin(StandardJsonPlugin())).build();
```

`@SerializersFor`里面列出想要序列化的类.

注意这里要加上`StandardJsonPlugin`, 因为built value的json格式不是标准的, 而是所有字段逗号分隔的.
用了`StandardJsonPlugin`之后就转换成了标准的JSON格式.

因为我们跑命令的时候用的是`watch`, 所以保存修改后`serializers.g.dart`文件此时自动生成了.

### 添加model序列化和反序列化代码
在Event类中添加:
```
  static Serializer<Event> get serializer => _$eventSerializer;

  String toJson() {
    return json.encode(serializers.serializeWith(Event.serializer, this));
  }

  static Event fromJson(String jsonString) {
    return serializers.deserializeWith(
        Event.serializer, json.decode(jsonString));
  }
```

import中除了model类还有:
```
import 'dart:convert';

import 'package:built_value/built_value.dart';
import 'package:built_value/serializer.dart';
import 'serializers.dart';
```

此时其他几个model类也要添加`serializer`, 比如`Actor`类中添加:
```
static Serializer<Actor> get serializer => _$actorSerializer;
```
重新build生成代码, 报错消失.


现在可以运行测试:

```
  test("parse event", () {
    const jsonString = """replace with event json string""";

    expect(Event.fromJson(jsonString).id, "11732036753");
  });
```
来检验单个的Event model建立.

可能会遇到的失败情况:
* 一些字段需要被标记为可为空`@nullable`.
* 一些字段名和key不匹配, 用`@BuiltValueField`的`wireName`标记.
详见后面的`Troubleshooting`部分.

### 如何反序列化顶层列表?
Event的API返回的是一个Event的数组: `[]`. 这种怎么做呢?

这里有个issue就是关于这个问题, 里面的解决办法挺好: https://github.com/google/built_value.dart/issues/565

在`serializers.dart`中添加方法:
```
T deserialize<T>(dynamic value) =>
    serializers.deserializeWith<T>(serializers.serializerForType(T), value);

BuiltList<T> deserializeListOf<T>(dynamic value) => BuiltList.from(
    value.map((value) => deserialize<T>(value)).toList(growable: false));
```


其中`BuiltList`需要`import 'package:built_collection/built_collection.dart';`.

反序列化`Event`数组的方法:
```
  static List<Event> fromEventsListJson(String jsonString) {
    final BuiltList<Event> listOfEvents =
        deserializeListOf<Event>(json.decode(jsonString));
    return listOfEvents.toList();
  }
```

到这一步, 跑我们开头写的两个测试应该都绿了. 如果没绿见`Troubleshooting`部分.

### 泛型的fromJson方法.
上面给`serializers`中添加了两个方法. 其中第一个方法是一个泛型的`fromJson`方法.

我们测试中的:
```
expect(Event.fromJson(jsonString).id, "11732036753");
```
也可以这样写:
```
expect(deserialize<Event>(json.decode(jsonString)).id, "11732036753");
```

这样不用给每一个类都写一个`fromJson`方法了.

## Troubleshooting
可能会有的报错, 问题原因和解决方式.

### 报错1: `failed due to: Invalid argument(s): Unknown type on deserialization. Need either specifiedType or discriminator field.`

比如这个样子:
```
Deserializing '[id, 11732036753, type, PushEvent, actor, {id: 54496419, login: supershell201...' to 'Event' failed due to: Invalid argument(s): Unknown type on deserialization. Need either specifiedType or discriminator field.
```

这是因为`Event`中依赖的类(`Actor`, `Repo`, `Payload`)没有添加serializer.

比如`Actor`中:
```
static Serializer<Actor> get serializer => _$actorSerializer;
```
添加上重新build生成代码即可.


### 报错2: `Tried to construct class "XXX" with null field`
比如: 
```
Deserializing '[id, 11732036753, type, PushEvent, actor, {id: 54496419, login: supershell201...' to 'Event' failed due to: Deserializing '[id, 54496419, login, supershell2019, display_login, supershell2019, gravatar...' to 'Actor' failed due to: Tried to construct class "Actor" with null field "displayLogin". This is forbidden; to allow it, mark "displayLogin" with @nullable.
```
此时, 先不要着急把字段标记为`@nullable`.
而是要看这个字段是否真的为null, 很有可能是因为字段名称和json中的key不匹配造成的, 比如json中是个蛇形命名.

查看了一下果然就是, 解决办法:
```
  @BuiltValueField(wireName: 'display_login')
  String get displayLogin;
```

如果字段真的是有可能为null的情况, 那么加上`@nullable`:
比如:
```
  @BuiltValueField(wireName: 'ref_type')
  @nullable
  String get refType;
```

### 报错3: `FormatException: Control character in string`
比如: 
```
FormatException: Control character in string (at line 25, character 129)
... replica::on_client_write(dsn::message_ex *request, bool ignore_throttling)
```

相关issue: https://github.com/dart-lang/convert/issues/10

解决的办法就是在测试的字符串声明前加一个`r`:
```
const jsonString = r"""replace with events list json string""";
```

## Model生成工具推荐
有个很棒的工具: https://charafau.github.io/json2builtvalue/
左边输入json字符串, 写好命名, 点击之后右边就会出现那些本来需要手动写的代码.

生成的`Event`类是这样:
```
library event;

import 'dart:convert';

import 'package:built_collection/built_collection.dart';
import 'package:built_value/built_value.dart';
import 'package:built_value/serializer.dart';

part 'event.g.dart';

abstract class Event implements Built<Event, EventBuilder> {
  Event._();

  factory Event([updates(EventBuilder b)]) = _$Event;

  @BuiltValueField(wireName: 'id')
  String get id;
  @BuiltValueField(wireName: 'type')
  String get type;
  @BuiltValueField(wireName: 'actor')
  Actor get actor;
  @BuiltValueField(wireName: 'repo')
  Repo get repo;
  @BuiltValueField(wireName: 'payload')
  Payload get payload;
  @BuiltValueField(wireName: 'public')
  bool get public;
  @BuiltValueField(wireName: 'created_at')
  String get createdAt;
  String toJson() {
    return json.encode(serializers.serializeWith(Event.serializer, this));
  }

  static Event fromJson(String jsonString) {
    return serializers.deserializeWith(
        Event.serializer, json.decode(jsonString));
  }

  static Serializer<Event> get serializer => _$eventSerializer;
}
```

哈哈, 看到这里是不是有种被骗了的感觉.

有了这个很棒的工具之后根本不用自己很小心地写一个一个model类了, 只需要写一个`serializers.dart`文件:
```
part 'serializers.g.dart';

@SerializersFor(const [
  Event,
  Actor,
  Repo,
  Payload,
])
final Serializers serializers =
    (_$serializers.toBuilder()..addPlugin(StandardJsonPlugin())).build();

T deserialize<T>(dynamic value) =>
    serializers.deserializeWith<T>(serializers.serializerForType(T), value);

BuiltList<T> deserializeListOf<T>(dynamic value) => BuiltList.from(
    value.map((value) => deserialize<T>(value)).toList(growable: false));
```
然后把要反序列化的类加进来, 再跑命令行生成代码, 就可以了.

经历一下前面的手动过程可能理解得更好一些, 也知道各种问题的原因. 
以后使用直接用工具就方便多了.


## 参考资料
* 官方文档: https://flutter.dev/docs/development/data-and-backend/json
* Flutter实战11.7: https://book.flutterchina.club/chapter11/json_model.html
* [built value github](https://github.com/google/built_value.dart)
* 生成工具: https://charafau.github.io/json2builtvalue/