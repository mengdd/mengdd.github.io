---
title: Using Dagger2 in Android
date: 2016-06-24 13:22:44
tags: [Android, Dagger2]
categories: Android
---

[Dagger2](https://github.com/google/dagger)是一个Java和Android的依赖注入框架.
本文介绍Android中dagger2的基本使用.
其中包括`@Inject`, `@Component`, `@Module`和`@Provides`注解的使用.

<!-- more -->

# 使用依赖注入的好处
1.使用类和被依赖的对象构造分开,这样如果我们需要改变被依赖类的构造方法,不必改动每一个使用类.
2.对各种被依赖类的实例,可以只构造一次.
3.当我们需要更换一种实现时,只需要保证接口一致.
4.利于单元测试,我们可以方便地mock依赖类的对象.

优点总结: 创建对象和使用对象分离, 模块化增强.

# Dagger2的使用
## Set Up
在项目的**build.gradle**里加这个:
classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'

然后**app的build.gradle**加这三行:
```
    compile 'javax.annotation:jsr250-api:1.0'
    compile 'com.google.dagger:dagger:2.2'
    apt 'com.google.dagger:dagger-compiler:2.2'
```

## 常用注解
最常使用的主要是以下这几个注解:

[@Component](http://google.github.io/dagger/api/latest/dagger/Component.html)
Annotates an interface or abstract class for which a fully-formed, dependency-injected implementation is to be generated from a set of modules(). The generated class will have the name of the type annotated with @Component prepended with Dagger. For example, @Component interface MyComponent {...} will produce an implementation named DaggerMyComponent.

[@Module](http://google.github.io/dagger/api/latest/dagger/Module.html)
Annotates a class that contributes to the object graph.

[@Inject](http://docs.oracle.com/javaee/7/api/javax/inject/Inject.html)
Dagger constructs instances of your application classes and satisfies their dependencies. It uses the javax.inject.Inject annotation to identify which constructors and fields it is interested in.

Use @Inject to annotate the constructor that Dagger should use to create instances of a class. When a new instance is requested, Dagger will obtain the required parameters values and invoke this constructor.

[@Provides](http://google.github.io/dagger/api/latest/dagger/Provides.html)
Annotates methods of a module to create a provider method binding. The method's return type is bound to its returned value. The component implementation will pass dependencies to the method as parameters.

## Dagger2基本使用
### 最简单的一个实例
首先写一个Component

```java
@Component(modules = MyApplicationModule.class)
public interface MyApplicationComponent {
    // this should be an interface or abstract class

    // write like this, and Make Project, then a DaggerMyApplicationComponent class will be generated
}
```
此时里面的Module内容可以暂时为空:

```java
@Module
public class MyApplicationModule {
}
```

写好后make一下,就生成了

```java
package com.ddmeng.dagger2sample.component;

import com.ddmeng.dagger2sample.module.MyApplicationModule;
import dagger.internal.Preconditions;
import javax.annotation.Generated;

@Generated(
  value = "dagger.internal.codegen.ComponentProcessor",
  comments = "https://google.github.io/dagger"
)
public final class DaggerMyApplicationComponent implements MyApplicationComponent {
  private DaggerMyApplicationComponent(Builder builder) {
    assert builder != null;
  }

  public static Builder builder() {
    return new Builder();
  }

  public static MyApplicationComponent create() {
    return builder().build();
  }

  public static final class Builder {
    private Builder() {}

    public MyApplicationComponent build() {
      return new DaggerMyApplicationComponent(this);
    }

    /**
     * @deprecated This module is declared, but an instance is not used in the component. This method is a no-op. For more, see https://google.github.io/dagger/unused-modules.
     */
    @Deprecated
    public Builder myApplicationModule(MyApplicationModule myApplicationModule) {
      Preconditions.checkNotNull(myApplicationModule);
      return this;
    }
  }
}
```

需要切换到project视图下才能看见.
生成的这个实现,名字是在我们自己的Component名前面加了Dagger.
如果我们的类名不是顶级的,即还有外部类,则会以下划线分隔连接.

现在我们的生成的myApplicationModule()方法被标记为`@Deprecated`,这是因为我的module里面什么都还没有呢,所以被认为是没有必要的.

现在我们添加一个要用的LogUtils类. 想要在MainActivity里面用.
写好LogUtils类,在构造函数上标记`@Inject`. 这时候就将LogUtils加入了dependency graph中, 相当于作为预备队员.

想要在MainActivity作为一个字段用,
在Component里面写一句:
`void inject(MainActivity activity);`

因为此时还是没有用到Module,所以在application里面可以直接build,保存component:

```java
public class SampleApplication extends Application {

    private MyApplicationComponent component;

    @Override
    public void onCreate() {
        super.onCreate();
        component = DaggerMyApplicationComponent.builder().build();

    }

    public MyApplicationComponent getComponent() {
        return component;
    }
}
```

在MainActivity使用的时候, 先get到Component, 然后调用inject()方法, 字段就被注入了.

```java
public class MainActivity extends AppCompatActivity {

    @Inject
    LogUtils logUtils;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ((SampleApplication) getApplication()).getComponent().inject(this);
        logUtils.i("tag", "hi, I'm an instance of LogUtils");
    }
}
```

运行程序后可以看到打出log,证明注入成功.

此时我们看到生成的代码有三个类:

```java
public final class DaggerMyApplicationComponent implements MyApplicationComponent {
public enum LogUtils_Factory implements Factory<LogUtils> {
public final class MainActivity_MembersInjector implements MembersInjector<MainActivity> {
```
可以通过查看调用栈来看调用关系.

### 单例`@Singleton`
如果我们想让工具类是单例,只需要在上面的基础上,在类名前加上`@Singleton`.

此时对应的Component也需要加上`@Singleton`.否则编译会不通过.
加好之后,可以打印hashCode()看出, 标记了`@Singleton`的这个对象,不论被注入几次,都是同一个对象.

在我们的例子中, 可以让FileUtils作为一个单例被注入:
```java
@Singleton
public class FileUtils {

    @Inject
    public FileUtils() {
        Log.i(LogUtils.TAG, "new FileUtils: " + hashCode());
    }

    public void doSomething() {
        Log.i(LogUtils.TAG, "do sth with FileUtils " + hashCode());
    }
}
```

查看生成的代码,可以看见`DaggerMyApplicationComponent`为单例的类多保存了一个字段:
`private Provider<FileUtils> fileUtilsProvider;`

它在init的时候被初始化为:
`this.fileUtilsProvider = ScopedProvider.create(FileUtils_Factory.create());`

包了一层之后,在ScopeProvider里实现了单例:

```java
  @Override
  public T get() {
    // double-check idiom from EJ2: Item 71
    Object result = instance;
    if (result == UNINITIALIZED) {
      synchronized (this) {
        result = instance;
        if (result == UNINITIALIZED) {
          instance = result = factory.get();
        }
      }
    }
    return (T) result;
  }
```

### `@Module` 和 `@Provides`的使用
上面的注入都是用`@Inject`, 在构造函数和要使用的字段上标记.
有些情况下`@Inject`是不能满足需求的.

**But @Inject doesn’t work everywhere**.

1. Interfaces can’t be constructed. 接口类型不能直接被构造.
2. Third-party classes can’t be annotated. 第三方的类不能改动它的代码.
3. Configurable objects must be configured! 需要配置的对象需要被配置.

这些情况下, `@Inject`不够用啦, 这时候就要用`@Provides`标记的方法.
方法的返回值返回了它满足的依赖, 它实际返回的对象可以是返回值接口的实现,或者是返回值类型的子类.
`@Provides`方法也可以有依赖, 即它的参数.
Dagger会注入它的参数值, 如果它的参数值不能被注入, 则编译会失败.
注意这个寻找参数注入的过程是在`@Component`级别的, 只要这个Component里面有这个参数类型的注入, 即便可能是在另一个Module, 就会自动采用.

所有的`@Provides`方法都需要放在`@Module`里面.
按照命名习惯(By convention), 一般`@Provides`标记的方法都有一个provide前缀, 而module类都有一个Module后缀.
例子:

```java
@Module
public class MyApplicationModule {

    private Context context;

    public MyApplicationModule(Context context) {
        this.context = context;
    }

    @Provides
    @Singleton
    public Context providesContext() {
        return context;
    }

    // Inject interface, return implementation class instance
    @Provides
    public HttpUtil provideHttpUtil() {
        Log.i(LogUtils.TAG, "provideHttpUtil");
        return new MyHttpUtil();
    }

    // Inject class from third-party, or Android framework service
    // This provide method need a parameter, Dagger will obtain the parameter value (injected it)
    // If the parameter is not injectable, then compilation failed
    @Provides
    @Singleton
    ConnectivityManager provideConnectivityManager(Context context) {
        return (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
    }
}
```

# Dagger2中几种常用注解总结
### `@Inject`
`@Inject`的用法分为三种

- 构造函数上的`@Inject`:

如果构造函数是有参数的, 则它的所有参数都会自动从dependency graph中找到并注入.
同时构造的这个类也被作为dependency graph的一部分.
但是我们在一个类中最多只能用@Inject标记一个构造方法.

- 字段上的`@Inject`: 从dependency graph中找到并注入字段.

这里需要手动调用
`(SampleApplication) getApplication()).getComponent().inject(this);`
类似的方法, 在这个方法被调用之前, 字段都是null.
注意这里的字段不能是private的.

- public方法上的`@Inject`:

所有方法的参数都会由dependency graph提供.
方法注入在构造函数*之后*立即调用, 意味着我们可以用一个构建好的this对象.

### `@Module` 和 `@Provides`
`@Module`标记了提供依赖的类, 其中包含了一些`@Provides`标注的方法, 返回值即依赖.

### `@Component`
用`@Component`标记的接口负责将所有的事情联系起来, 可以看做是`@Module`和`@Inject`之间的桥梁.
我们可以定义我们用的依赖来自哪些Module或者Component.

在Component里可以定义哪些依赖是公有的 (提供返回值为某种依赖的无参数方法) , 也可以定义我们的component可以去哪里inject对象 (void inject()方法, 参数是去注入的地方) .

`@Component`可以有自己的子Component, 也可以有lifecycle.

先就这么多吧, 更多更高级的使用可以期待下文, 也可以参见后面的参考资料.

本文地址: [Using Dagger2 in Android](http://www.cnblogs.com/mengdd/p/5613889.html)
本文Demo: [dagger2-sample](https://github.com/mengdd/dagger2-sample)

# 参考资料:
[dagger2 repo](https://github.com/google/dagger)
[dagger2 website](http://google.github.io/dagger/)
[User Guide](http://google.github.io/dagger/users-guide.html)

[Dagger 2.0文档](https://docs.google.com/document/d/1fwg-NsMKYtYxeEWe82rISIHjNrtdqonfiHgp8-PQ7m8/edit)
[dagger 2的sample](https://github.com/google/dagger/tree/master/examples/simple/src/main/java/coffee)

这里有些guides:
[Code Path Guides: DI with dagger2](https://guides.codepath.com/android/Dependency-Injection-with-Dagger-2)
[Dagger2](https://blog.gouline.net/dagger-2-even-sharper-less-square-b52101863542#.me0ieiaph)

这里有一系列关于Dagger2的文章还挺好的:
[Froger_mcs dev blog](http://frogermcs.github.io/)
[dagger 1 to dagger 2 migration](http://frogermcs.github.io/dagger-1-to-2-migration/)
[Introduction to DI](http://frogermcs.github.io/dependency-injection-with-dagger-2-introdution-to-di/)
[Dagger2 API](http://frogermcs.github.io/dependency-injection-with-dagger-2-the-api/)
[Inject everything - ViewHolder and Dagger2 (with Multibinding and AutoFactory example)](https://medium.com/@froger_mcs/inject-everything-viewholder-and-dagger-2-e1551a76a908#.e3zxrynq4)
[Custom Scope](http://frogermcs.github.io/dependency-injection-with-dagger-2-custom-scopes/)

作者的例子:
[Github Client](https://github.com/frogermcs/GithubClient)