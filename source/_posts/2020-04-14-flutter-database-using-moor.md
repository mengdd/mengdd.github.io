---
title: 写一个TODO App学习Flutter数据库工具Moor
date: 2020-04-14 09:20:46
tags: [Flutter]
categories: [Flutter]
---


# 写一个TODO App学习Flutter数据库工具Moor
Flutter的数据库存储, 官方文档: https://flutter.dev/docs/cookbook/persistence/sqlite
中写的是直接操纵SQLite数据库的方法.

有没有什么package可以像Android的Room一样, 帮助开发者更加方便地做数据库存储呢? 

Moor就是这种目的: https://pub.dev/packages/moor.
它的名字是把Room反过来. 它是一个第三方的package.

为了学习一下怎么用, 我做了一个小的todo app: https://github.com/mengdd/more_todo.

本文是一个工作记录.
<!--more-->

## TL;DR
用Moor做TODO app:
* 基本使用: 依赖添加, 数据库和表的建立, 对表的基本操作.
* 问题解决: 插入数据注意类型; 多个表的文件组织.
* 常用功能: 外键和join, 数据库升级, 条件查询.

代码: Todo app: https://github.com/mengdd/more_todo

## Moor基本使用
官方这里有个文档:
[Moor Getting Started](https://moor.simonbinder.eu/docs/getting-started/)
### Step 1: 添加依赖
`pubspec.yaml`中:
```
dependencies:
  flutter:
    sdk: flutter

  moor: ^2.4.0
  moor_ffi: ^0.4.0
  path_provider: ^1.6.5
  path: ^1.6.4
  provider: ^4.0.4

dev_dependencies:
  flutter_test:
    sdk: flutter
  moor_generator: ^2.4.0
  build_runner: ^1.8.1
```
这里我是用的当前(2020.4)最新版本, 之后请更新各个package版本号到最新的版本.

对各个packages的解释:
```
* moor: This is the core package defining most apis
* moor_ffi: Contains code that will run the actual queries
* path_provider and path: Used to find a suitable location to store the database. Maintained by the Flutter and Dart team
* moor_generator: Generates query code based on your tables
* build_runner: Common tool for code-generation, maintained by the Dart team
```

现在推荐使用`moor_ffi`而不是`moor_flutter`.

网上的一些例子是使用`moor_flutter`的, 所以看那些例子的时候有些地方可能对不上了.

### Step 2: 定义数据库和表
新建一个文件, 比如`todo_database.dart`:
```dart
import 'dart:io';

import 'package:moor/moor.dart';
import 'package:moor_ffi/moor_ffi.dart';
import 'package:path/path.dart' as p;
import 'package:path_provider/path_provider.dart';

part 'todo_database.g.dart';

// this will generate a table called "todos" for us. The rows of that table will
// be represented by a class called "Todo".
class Todos extends Table {
  IntColumn get id => integer().autoIncrement()();

  TextColumn get title => text().withLength(min: 1, max: 50)();

  TextColumn get content => text().nullable().named('description')();

  IntColumn get category => integer().nullable()();

  BoolColumn get completed => boolean().withDefault(Constant(false))();
}

@UseMoor(tables: [Todos])
class TodoDatabase extends _$TodoDatabase {
  // we tell the database where to store the data with this constructor
  TodoDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;
}

LazyDatabase _openConnection() {
  // the LazyDatabase util lets us find the right location for the file async.
  return LazyDatabase(() async {
    // put the database file, called db.sqlite here, into the documents folder
    // for your app.
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'db.sqlite'));
    return VmDatabase(file, logStatements: true);
  });
}
```

几个知识点:
* 要加`part 'todo_database.g.dart';`, 等一下要生成这个文件.
* 这里定义的class是`Todos`, 生成的具体实体类会去掉s, 也即`Todo`. 如果想指定生成的类名, 可以在类上加上注解, 比如: `@DataClassName("Category")`, 生成的类就会叫"Category".
* 惯例: $是生成类类名前缀. `.g.dart`是生成文件.

### Step 3: 生成代码
运行:
```
flutter packages pub run build_runner build
```
or:
```
flutter packages pub run build_runner watch
```
来进行一次性(build)或者持续性(watch)的构建.

如果不顺利, 有可能还需要加上`--delete-conflicting-outputs`:
```
flutter packages pub run build_runner watch --delete-conflicting-outputs
```
运行成功之后, 生成`todo_database.g.dart`文件. 

所有的代码中报错应该消失了.

### Step 4: 添加增删改查方法
对于简单的例子, 把方法直接写在数据库类里:
```dart
@UseMoor(tables: [Todos])
class TodoDatabase extends _$TodoDatabase {
  // we tell the database where to store the data with this constructor
  TodoDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  Future<List<Todo>> getAllTodos() => select(todos).get();

  Stream<List<Todo>> watchAllTodos() => select(todos).watch();

  Future insertTodo(TodosCompanion todo) => into(todos).insert(todo);

  Future updateTodo(Todo todo) => update(todos).replace(todo);

  Future deleteTodo(Todo todo) => delete(todos).delete(todo);
}
```
数据库的查询不但可以返回Future还可以返回Stream, 保持对数据的持续观察.

这里注意插入的方法用了Companion对象. 后面会说为什么.

上面这种做法把数据库操作方法都写在一起, 代码多了之后显然不好.
改进的方法就是写DAO:
https://moor.simonbinder.eu/docs/advanced-features/daos/

后面会改.


### Step 5: 把数据提供到UI中使用
提供数据访问方法涉及到程序的状态管理.
方法很多, 之前写过一个文章: https://www.cnblogs.com/mengdd/p/flutter-state-management.html


这里先选一个简单的方法用Provider直接提供数据库对象, 包在程序外层:
```dart
void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Provider(
      create: (_) => TodoDatabase(),
      child: MaterialApp(
        title: 'Flutter Demo',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: MyHomePage(),
      ),
    );
  }
}
```
需要的时候:
```dart
TodoDatabase database = Provider.of<TodoDatabase>(context, listen: false);
```
就拿到database对象, 然后可以调用它的方法了.

之后就是UI怎么用的问题了, 这里不再多说.

我的代码中tag: `v0.1.1`就是这种最简单的方法.
可以checkout过去看这个最简单版本的实现.

### Step 6: 改进: 抽取方法到DAO, 重构
增删改查的方法从数据库中抽取出来, 写在DAO里:
```dart
part 'todos_dao.g.dart';

// the _TodosDaoMixin will be created by moor. It contains all the necessary
// fields for the tables. The <MyDatabase> type annotation is the database class
// that should use this dao.
@UseDao(tables: [Todos])
class TodosDao extends DatabaseAccessor<TodoDatabase> with _$TodosDaoMixin {
  // this constructor is required so that the main database can create an instance
  // of this object.
  TodosDao(TodoDatabase db) : super(db);

  Future<List<Todo>> getAllTodos() => select(todos).get();

  Stream<List<Todo>> watchAllTodos() => select(todos).watch();

  Future insertTodo(TodosCompanion todo) => into(todos).insert(todo);

  Future updateTodo(Todo todo) => update(todos).replace(todo);

  Future deleteTodo(Todo todo) => delete(todos).delete(todo);
}
```
运行命令行重新生成一下(如果是watch就不用).

其实就生成了个这:
```dart
part of 'todos_dao.dart';

mixin _$TodosDaoMixin on DatabaseAccessor<TodoDatabase> {
  $TodosTable get todos => db.todos;
}
```
这里的todos是其中的table对象.

所以如果不是改table, 只改变DAO中的方法实现的话, 不用重新生成.

这时候我们提供给UI的部分也要改了.

之前是Provider直接提供了database对象, 虽然可以直接换成提供DAO对象, 但是DAO会有很多个, 硬要这么提供的话代码很快就乱了.

怎么解决也有多种方法, 这是一个架构设计问题, 百花齐放, 答案很多.

我这里简单封装一下:
```dart
class DatabaseProvider {
  TodosDao _todosDao;

  TodosDao get todosDao => _todosDao;

  DatabaseProvider() {
    TodoDatabase database = TodoDatabase();
    _todosDao = TodosDao(database);
  }
}
```

最外层改成提供这个:
```dart
    return Provider(
      create: (_) => DatabaseProvider(),
//...
    );
```
用的时候把DAO get出来用就可以了.

如果有其他DAO也可以加进去.

## Troubleshooting
### 插入的时候应该用Companion对象
插入数据的方法:
如果这样写:
```dart
Future insertTodo(Todo todo) => into(todos).insert(todo);
```
就坑了.

因为按照定义, 我们的id是自动生成并自增的:
```dart
IntColumn get id => integer().autoIncrement()();
```
但是生成的这个Todo类, 里面所有非空的字段都是`@required`的:
```dart
Todo(
  {@required this.id,
  @required this.title,
  this.content,
  this.category,
  @required this.completed});
```
要新建一个实例并插入, 我自己是无法指定这个递增的id的. (先查询再自己手动递增是不是太tricky了. 一般不符合直觉的古怪的做法都是不对的.)

可以看这两个issue中, 作者的解释也是用Companion对象:

* https://github.com/simolus3/moor/issues/62
* https://github.com/simolus3/moor/issues/68

所以insert方法最后写成了这样:
```dart
Future insertTodo(TodosCompanion todo) => into(todos).insert(todo);
```
还有一种写法是这样:
```dart
 Future insertTodo(Insertable<Todo> todo) => into(todos).insert(todo);
```

添加数据:
```dart
final todo = TodosCompanion(
  title: Value(input),
  completed: Value(false),
);
todosDao.insertTodo(todo);
```
这里构建对象的时候, 只需要把需要的值用`Value`包装起来. 没有提供的会是`Value.absent()`.

### 表定义必须和数据库类写在一起? 多个表怎么办?
实际的项目中肯定有多个表, 我想着一个表一个文件这样比较好.

于是当我天真地为我的新数据表, 比如Category, 新建一个`categories.dart`文件, 里面继承了Table类, 也指定了生成文件的名字.
```dart
part 'categories.g.dart';

@DataClassName('Category')
class Categories extends Table {
//...
}
```

运行生成build之后代码中这行是红的:
```dart
part 'categories.g.dart';
```
没有生成这个文件.

查看后发现`Category`类仍然被生成在了databse的.g.dart文件中.

关于这个问题的讨论: https://github.com/simolus3/moor/issues/480

解决方法有两种思路:
* 简单解决: 源码仍然分开写, 只不过所有的生成代码放一起.

去掉part语句. 
```dart
@DataClassName('Category')
class Categories extends Table {
//...
}
```
生成的代码仍然是方法database的生成文件中, 但是我们的源文件看起来是分开了.
之后使用具体数据类型的时候, import的还是database文件对应类.

* 使用`.moor`文件.

## 进阶需求
### 外键和join
把两个表关联起来这个需求还挺常见的. 

比如我们的todo实例, 增加了Category类之后, 想把todo放在不同的category中, 没有category的就放在inbox里, 作为未分类.

moor对外键不是直接支持, 而是通过`customStatement`来实现的.

这里Todos类里的这一列, 加上自定义限制, 关联到categories表:
```dart
IntColumn get category => integer()
  .nullable()
  .customConstraint('NULL REFERENCES categories(id) ON DELETE CASCADE')();
```
**要用主键id**.

这里指定了两遍可以null: 一次是`nullable()`, 另一次是在语句中.

实际上`customConstraint`中的会覆盖前者. 但是我们仍然需要前者, 用来表明在生成类中改字段是可以为null的.

另外还指定了删除category的时候删除对应的todo.

外键默认不开启, 需要运行:
```
customStatement('PRAGMA foreign_keys = ON');
```

join查询的部分, 先把两个类包装成第三个类.
```dart
class TodoWithCategory {
  final Todo todo;
  final Category category;

  TodoWithCategory({@required this.todo, @required this.category});
}
```

之后更改TODO的DAO, 注意这里添加了一个table, 所以要重新生成一下.

之前的查询方法改成这样:
```dart
Stream<List<TodoWithCategory>> watchAllTodos() {
final query = select(todos).join([
  leftOuterJoin(categories, categories.id.equalsExp(todos.category)),
]);

return query.watch().map((rows) {
  return rows.map((row) {
    return TodoWithCategory(
      todo: row.readTable(todos),
      category: row.readTable(categories),
    );
  }).toList();
});
}
```
join返回的结果是`List<TypedResult>`, 这里用map操作符转换一下.

### 数据库升级
数据库升级, 在数据库升级的时候添加新的表和列.

由于外键默认是不开启的, 所以也要开启一下.

PS: 这里Todo中的category之前已经建立过了.
迁移的时候不能修改已经存在的列. 所以只能弃表重建了.

```dart
@UseMoor(tables: [Todos, Categories])
class TodoDatabase extends _$TodoDatabase {
  // we tell the database where to store the data with this constructor
  TodoDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 2;

  @override
  MigrationStrategy get migration => MigrationStrategy(
        onUpgrade: (migrator, from, to) async {
          if (from == 1) {
            migrator.deleteTable(todos.tableName);
            migrator.createTable(todos);
            migrator.createTable(categories);
          }
        },
        beforeOpen: (details) async {
          await customStatement('PRAGMA foreign_keys = ON');
        },
      );
}
```
没想到报错了: `Unhandled Exception: SqliteException: near "null": syntax error`,
出错的是drop table的这句:
```
Moor: Sent DROP TABLE IF EXISTS null; with args []
```
说todos.tableName是null.

这个get的设计用途原来是用来指定自定义名称的:
https://pub.dev/documentation/moor/latest/moor_web/Table/tableName.html

因为我没有设置自定义名称, 所以这里返回了null.

这里我改成了:
```
migrator.deleteTable(todos.actualTableName);
```

### 条件查询
查某个分类下:
```dart
  Stream<List<TodoWithCategory>> watchTodosInCategory(Category category) {
    final query = select(todos).join([
      leftOuterJoin(categories, categories.id.equalsExp(todos.category)),
    ]);

    if (category != null) {
      query.where(categories.id.equals(category.id));
    } else {
      query.where(isNull(categories.id));
    }

    return query.watch().map((rows) {
      return rows.map((row) {
        return TodoWithCategory(
          todo: row.readTable(todos),
          category: row.readTable(categories),
        );
      }).toList();
    });
  }
```

多个条件的组合用`&`, 比如上面的查询组合未完成:
```dart
query.where(
        categories.id.equals(category.id) & todos.completed.equals(false));
```


## 总结
[Moor](https://github.com/simolus3/moor)是一个第三方的package, 用来帮助Flutter程序的本地存储. 由于开放了SQL语句查询, 所以怎么定制都行. 作者很热情, 可以看到很多issue下都有他详细的回复.

本文是做一个TODO app来练习使用moor.
包括了基本的增删改查, 外键, 数据库升级等.

代码: https://github.com/mengdd/more_todo


## 参考
* [Moor github](https://github.com/simolus3/moor)
* [Moor website](https://moor.simonbinder.eu/)
* [package: moor_ffi](https://pub.dev/packages/moor_ffi)
* [Moor Getting Started](https://moor.simonbinder.eu/docs/getting-started/)
* [Moor (Room for Flutter) #1 – Tables & Queries – Fluent SQLite Database](https://www.youtube.com/watch?v=zpWsedYMczM&feature=youtu.be)
* [Moor (Room for Flutter) #3 – Foreign Keys, Joins & Migrations – Fluent SQLite Database](https://resocoder.com/2019/07/17/moor-room-for-flutter-3-foreign-keys-joins-migrations-fluent-sqlite-database/)
* [SQLite Foreign Key Support](https://www.sqlite.org/foreignkeys.html)