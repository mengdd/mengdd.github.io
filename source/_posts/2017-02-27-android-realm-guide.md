---
title: Android Realm数据库使用指南
date: 2017-02-27 17:32:23
tags: [Android, Database, Realm]
categories: [Android, Database, Realm]
---


# Android Realm数据库使用指南
Realm数据库, 目前有Java, Objective‑C, React Native, Swift, Xamarin的几种实现, 是一套用来取代SQLite的解决方案. 

本文面向Android开发, 所以只讨论Java实现.
目前Realm Java的最新版本是2.3.1.

官方文档在此: [realm java doc](https://realm.io/docs/java/latest/), 花一个下午就可以基本过一遍, 之后随时查用. 

我写了一个小程序[TodoRealm](https://github.com/mengdd/TodoRealm), 使用Realm做数据库实现的一个To-do应用,  在实际使用的过程中也有一些发现.

本文是我自己看文档的时候的一些记录, 有一些实际使用时的发现也穿插在对应的章节了.

<!-- more -->


## Setup
在项目的根build.gradle的文件中添加:

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "io.realm:realm-gradle-plugin:2.3.0"
    }
}
```

然后在app的build.gradle文件中添加:
```
apply plugin: 'realm-android'
```
Done.

## Models
Model类只要继承`RealmObject`即可.

### 字段类型
Model类中可以包含的字段类型包括基本数据类型(及它们的装箱类型)和Date类, 另外也可以包含`RealmObject`的子类或者是`RealmList<? extends RealmObject>`.

### 字段性质
在字段上加注解可以定义字段的性质:

`@Required`表明字段非null.
原生类型和`RealmList`类型默认是非null的.
`RealmObject`字段永远是可以为null的.

`@Ignore`表示字段不会被存储.

`@Index`加索引.

`@PrimaryKey`加主键, 主键只能有一个, 主键默认加索引.

但是注意主键默认没有加`@Required`, 如果主键要求非null, 需要显式添加`@Required`.

### 主键使用
有主键才能使用`copyToRealmOrUpdate()`这个方法.
主键类型必须是String或者整型(byte, short, int, long)或者它们的装箱类型(Byte, Short, Integer, Long).

有主键的对象创建的时候不能使用`createObject(Class<E> clazz)`方法, 而应该使用`createObject(Class<E> clazz, Object primaryKeyValue)`附上主键.

或者用
`copyToRealm(obj)`或`copyToRealmOrUpdate(obj)`, 前者遇到主键冲突时会崩溃, 后者遇到主键冲突会更新已有对象.

### 自动更新的对象
Realm中的数据对象是自动更新(Auto-Updating)的, 对象一旦被查询出来, 后续发生的任何数据改变也会立即反映在结果中, 不需要刷新对象.

这是一个非常有用的特性, 结合数据变化的通知可以很方便地刷新UI.


## 关系
Realm model对象间可以很方便地建立关系.
你可以在Model中存储另一个对象的引用, 建立多对一的关系; 也可以存储一组对象`RealmList<T>`, 建立一对多或多对多的关系.

`RealmList<T>`的getter永远也不会返回null, 它只会返回一个为空的list.
把这个字段设置为null可以清空这个list.

## 初始化
Realm在使用之前需要调用初始化:
```java
Realm.init(context);
```
建议把它放在Application的`onCreate()`里.

### 配置
配置类: `RealmConfiguration`定义了Realm的创建配置.
最基本的配置:
```java
RealmConfiguration config = new RealmConfiguration.Builder().build();
```
它会创建一个叫`default.realm`的文件, 放在`Context.getFilesDir()`的目录下.

如果我们想自定义一个配置, 可以这样写:
```java
// The RealmConfiguration is created using the builder pattern.
// The Realm file will be located in Context.getFilesDir() with name "myrealm.realm"
RealmConfiguration config = new RealmConfiguration.Builder()
  .name("myrealm.realm")
  .encryptionKey(getKey())
  .schemaVersion(42)
  .modules(new MySchemaModule())
  .migration(new MyMigration())
  .build();
// Use the config
Realm realm = Realm.getInstance(config);
```
所以我们是可以有多个配置, 访问多个Realm实例的.

我们可以把配置设置为默认配置:
```java
Realm.init(this);
RealmConfiguration config = new RealmConfiguration.Builder().build();
Realm.setDefaultConfiguration(config);
```
之后用`Realm.getDefaultInstance()`取到的就是这个默认配置对应的实例.

## 数据库迁移
迁移的策略是通过config指定的:
```java
RealmConfiguration config = new RealmConfiguration.Builder()
    .schemaVersion(2) // Must be bumped when the schema changes
    .migration(new MyMigration()) // Migration to run instead of throwing an exception
    .build()
```
其中`MyMigration`实现了`RealmMigration`接口, 在`migrate()`方法中根据新旧版本号进行一步一步地升级.

具体例子见[Migration](https://github.com/realm/realm-java/blob/master/examples/migrationExample/src/main/java/io/realm/examples/realmmigrationexample/model/Migration.java).


## 关于Realm的close()
一个打开的Realm实例会持有一些资源, 有一些是Java不能自动管理的, 所以就需要打开实例的代码负责在不需要的时候将其关闭.

Realm的instance是引用计数的(reference counted cache), 在同一个线程中获取后续实例是免费的, 但是底层的资源只有当所有实例被释放了之后才能释放. 也即你调用了多少次`getInstance()`, 就需要调用相应次数的`close()`方法.

比较建议的方法是在Activity或Fragment的生命周期中处理Realm实例的开启和释放:

- 在Activity的`onCreate()`中`getInstance()`, `onDestroy()`中`close()`.
- 在Fragment的`onCreateView()`中`getInstance()`, `onDestroyView()`中`close()`.

如果多个Fragment相关的都是同一个数据库实例, 那么在Activity中处理更好一些.

## 写
写操作一般的流程是这样:
```java
// Obtain a Realm instance
Realm realm = Realm.getDefaultInstance();

realm.beginTransaction();

//... add or update objects here ...

realm.commitTransaction();
```
这里创建对象可以用`createObject()`方法或者`copyToRealm()`方法.
前者是先创建再set值, 后者是先new对象再更新数据库.

如果不想自己处理`beginTransaction()`, `cancelTransaction()`和`commitTransaction()`, 可以直接调用`realm.executeTransaction()`方法:
```java
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        User user = realm.createObject(User.class);
        user.setName("John");
        user.setEmail("john@corporation.com");
    }
});
```
### 异步
因为transactions之间是互相阻塞的.
异步执行可以用这个方法:
```java
realm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm bgRealm) {
                User user = bgRealm.createObject(User.class);
                user.setName("John");
                user.setEmail("john@corporation.com");
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                // Transaction was a success.
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                // Transaction failed and was automatically canceled.
            }
        });
```
这两个回调是Optional的, 它们只能在有Looper的线程调用.

注意: 这个方法的返回值对象可以用于在Activity/Fragment生命周期结束的时候取消未完的操作.

### 删除和更新
所有的写操作都要放在transaction中进行, 如上, 不同的操作只是其中具体方法不同.

删除操作:
```java
final RealmResults<User> users = getUsers();
// method 1:
users.get(0).deleteFromRealm();
// method 2:
users.deleteFromRealm(0);

// delete all
users.deleteAllFromRealm();
```

更新操作:
```java
realm.copyToRealmOrUpdate(obj);
```
注意: 这个方法需要Model有主键.

## 查询
查询可以流式地写:
```java
// Or alternatively do the same all at once (the "Fluent interface"):
RealmResults<User> result2 = realm.where(User.class)
                                  .equalTo("name", "John")
                                  .or()
                                  .equalTo("name", "Peter")
                                  .findAll();
```
查询条件默认是and的关系, or则需要显式指定.

这个`RealmResults`是继承Java的`AbstractList`的, 是有序的集合, 可以通过索引访问.
`RealmResults`永远不会为null, 当查不到结果时, 它的`size()`返回0.

### 查询的线程
基本上所有的查询都是很快进行的, 足够在UI线程上同步进行.
所以绝大多数情况在UI线程上使用`findAll()`是没有问题的.

如果你要进行非常复杂的查询, 或者你的查询是在非常大的数据集上进行的, 你可以选择异步查询, 使用`findAllAsync()`.


### 查询条件是一个集合 -> `in()`
如果想要查询的某一个字段的值是在一个集合中, 比如我有一个id的集合, 我现在想把id在这个集合中的项目全都查出来, 这就可以使用in操作符:
```java
RealmResults<TodoList> toDeleteLists = realm.where(TodoList.class).in("id", ids).findAll();
```

### 链式查询
查询的时候可以利用link或关系来查询, 比如一个Person类中含有一个`RealmList<Dog> dogs`的字段.
查询的时候可以这样:
```java
RealmResults<Person> persons = realm.where(Person.class)
                                .equalTo("dogs.color", "Brown")
                                .findAll();
```
利用字段名`dogs.`来查询一个dog的属性, 再查出拥有这种特定属性dog的人.

但是反向地, 我们能不能查询主人是满足特定属性的人的所有dogs呢? 目前(2017.2.17)这种查询仍是不支持的. 这里有讨论:  [realm-java-issue-607](https://github.com/realm/realm-java/issues/607).

 所以两种解决办法: 一是做两次查询; 二是在Dog类的model里加入对Person的引用.

## Notifications
可以添加一个listener, 在数据改变的时候收到更新.
```java
public class MyActivity extends Activity {
    private Realm realm;
    private RealmChangeListener realmListener;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      realm = Realm.getDefaultInstance();
      realmListener = new RealmChangeListener() {
        @Override
        public void onChange(Realm realm) {
            // ... do something with the updates (UI, etc.) ...
        }};
      realm.addChangeListener(realmListener);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // Remove the listener.
        realm.removeChangeListener(realmListener);
        // Close the Realm instance.
        realm.close();
    }
}
```
注意listener需要在不用的时候删除掉. 

可以用这样删除所有的listeners:
```java
realm.removeAllChangeListeners();
```
Listener不一定要和Realm绑定, 也可以和具体的`RealmObject`或者`RealmResults`绑定.
当Listener被调用的时候, 它绑定的对象是自动更新的, 不需要手动刷新.

## 查看数据库的工具
用Stetho不能直接查看Realm的数据库, 看不到.
需要用这个工具配置一下: [stetho-realm](https://github.com/uPhyca/stetho-realm).
之后就可以在浏览器中查看Realm的数据库了.

(但是感觉这个工具不是很好用, 有时候不显示数据, 有时候显示的是旧数据.)

也可以用官方提供的Realm Browser来查看, 但是只有Mac版.
如何查看看这里: [StackOverflow answer](http://stackoverflow.com/questions/28465706/how-to-find-my-realm-file/28465803#28465803).

## 实际使用的感想和遇到的问题
### 优点
- 建立Model之间的关系很方便也很直接, 查询的时候自动关联了其中的关系.
- 自动更新(Auto-Updating)的特性很有用, 不用再关心数据的刷新, 只用关心UI的刷新. 

比如一旦给Adapter绑定了数据, 之后的数据更新只需要在onChange()里面通知Adapter调用`notifyDataSetChanged()`即可.

当然我并没有用`RealmBaseAdapter`和`RealmRecyclerViewAdapter`, 估计这两个更好用, 官方有例子, 这里不再赘述.

### 缺点
这里有的也不能说是缺点, 只是使用起来觉得不方便的地方. 

- 限制了创建对象和操作对象必须在同一个线程.
违反了这条会报错: `java.lang.IllegalStateException: Realm access from incorrect thread. Realm objects can only be accessed on the thread they were created.` 比如我们在UI线程查询出来的对象, 想要异步地删除或者更新, 我们必须在新的线程重新查询.
- 没有主键自增的功能, 见[Issue #469](https://github.com/realm/realm-java/issues/469), 需要自己控制主键自增.
- 从List中删除了一项之后, 最后的一项会移动过来补到被删除的那一项原来的位置. 这是因为人家就是这么设计的[stackoverflow](http://stackoverflow.com/questions/37480785/realm-order-of-records-was-changed). 默认情况下是没有排序的, 数据按照添加的顺序返回, 但是这并不是一种保证, 所以当删除了中间的元素, 后面的会补上这个位置, 以保证底层的数据是放在一起的. 解决办法就是指定一个排序规则.
- 查询出来的对象不可以临时改变其数据, 否则会报错: `java.lang.IllegalStateException: Changing Realm data can only be done from inside a transaction.`
- 不支持反向link的查询. (见前面链式查询部分的介绍).
- 不支持级联删除. 即从数据库中删除一个对象的时候, 不会删除其中`RealmObject`子类或`RealmList`类型的字段在数据库中对应的数据. [Issue #1104](https://github.com/realm/realm-java/issues/1104), [Issue #2717](https://github.com/realm/realm-java/issues/2717). 这点也可以理解, 因为model之间的关系可能是多对多的. 所以需要实现级联删除的地方需要手动处理.
- 测试不方便: `RealmResults`对象即不能被mock也不能被new; 所有的Model对象也不能被mock. 因为`Mockito can only mock non-private & non-final classes.`



## 资源
- [Github repo realm-java](https://github.com/realm/realm-java)
- [Realm Java Doc](https://realm.io/docs/java/latest/)

我的练习Demo:
- [TodoRealm](https://github.com/mengdd/TodoRealm)