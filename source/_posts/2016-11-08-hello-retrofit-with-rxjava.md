---
title: Retrofit结合RxJava使用指南
date: 2016-11-08 10:36:59
tags: [Android, Retrofit, RxJava]
categories: [Android, Retrofit, RxJava]
---


# Retrofit结合RxJava使用指南
Retrofit是一个当前很流行的网络请求库, 官网的介绍是: "Type-safe HTTP client for Android and Java". 本文介绍Retrofit的使用.
先介绍单独使用Retrofit进行网络请求, 后面主要介绍和RxJava结合的请求, 有实例代码.

<!-- more -->

## Retrofit单独使用
### Setup
首先在manifest中加上网络权限:
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
然后在`app/build.gradle`中加上依赖:
```gradle
compile 'com.squareup.retrofit2:retrofit:2.1.0'
compile 'com.google.code.gson:gson:2.8.0'
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
```

### 准备API和model类
本例子中使用[Github API](https://developer.github.com/v3/)做请求.

以Github的Root Endpoint为例:
`https://api.github.com`.
首先, 我们在命令行发送:
```bash
curl https://api.github.com
```
或者在Postman发送这个请求, 两种方法都可以得到结果.

这个请求返回的是一个json.

利用这个网站: [jsonschema2pojo](http://www.jsonschema2pojo.org/), 可以用json生成一个java类, 比如上面这个, 我们给它起名字叫`Endpoints.java`.

之后例子中的API都是这种方式, 先发送请求得到json, 然后转成java的model类.

### 利用Retrofit发送请求并得到结果
首先写一个`ServiceGenerator`类, 用于生成service:
```java
public class ServiceGenerator {
    public static final String API_BASE_URL = "https://api.github.com";

    private static OkHttpClient.Builder httpClient = new OkHttpClient.Builder();

    private static Retrofit.Builder builder =
            new Retrofit.Builder()
                    .baseUrl(API_BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create());

    public static <S> S createService(Class<S> serviceClass) {
        Retrofit retrofit = builder.client(httpClient.build()).build();
        return retrofit.create(serviceClass);
    }
}
```
这里指定了我们的base url.
ConverterFactory指定的是`GsonConverterFactory`.

`createService()`方法返回的是一个泛型.

然后我们创建`GithubService`, 注意这是一个**接口**:
```java
import com.ddmeng.helloretrofit.data.models.Endpoints;

import retrofit2.Call;
import retrofit2.http.GET;
import retrofit2.http.Url;

public interface GitHubService {
    @GET
    Call<Endpoints> getAllEndpoints(@Url String url);
}
```
这里`@GET`指定了是一个GET请求, 因为我们请求的就是base url, 所以是这样写的.
`Endpoints`类是这个请求所返回的json转化的java类.

好了, 准备工作做完了, 现在就可以请求并得到结果:
```java
GitHubService gitHubService = ServiceGenerator.createService(GitHubService.class);
Call<Endpoints> endpointsCall = gitHubService.getAllEndpoints("");
endpointsCall.enqueue(new Callback<Endpoints>() {
    @Override
    public void onResponse(Call<Endpoints> call, Response<Endpoints> response) {
        Endpoints endpoints = response.body();
        String repositoryUrl = endpoints.getRepositoryUrl();
        LogUtils.i(repositoryUrl);
        Toast.makeText(MainActivity.this, "repository url: " + repositoryUrl, Toast.LENGTH_LONG).show();
    }

    @Override
    public void onFailure(Call<Endpoints> call, Throwable t) {

    }
});
```

首先利用前面的ServiceGenerator来创建Service, 然后调用接口中定义的`getAllEndpoints()`方法, 此处传入了空字符串, 因为我请求的就是base url.


这里注意这个Call<T>, 泛型T是model类型, 它有两个方法:
- `execute()`是同步方法, 返回`Response<T>`;
- `enqueue()`是异步方法, 在上面的例子中用的就是这个, 在回调`onResponse()`中返回了`Response<T>`.

### Path和参数
从上面返回的endpoints可以看到, user_url是: `https://api.github.com/users/{user}`
这是一个带path参数的url, 我们发请求的时候在{user}处写一个github用户名, 即可得到该用户的信息, 比如:
`https://api.github.com/users/mengdd`.

那么用Retrofit如何处理呢?
只需要在`GithubService`中增加一个方法, 这样写:
```java
public interface GitHubService {
    @GET
    Call<Endpoints> getAllEndpoints(@Url String url);

    @GET("users/{user}")
    Call<User> getUser(@Path("user") String user);
}
```
使用时的方法完全一样, 不再赘述, 同理, 如果要在后面加参数, 可以用`@Query`.


## Retrofit + RxJava
RxJava近年来很流行, 主要优势是流式操作, 可以处理并行发送请求, 使用灵活, 线程切换容易.
当你要处理的逻辑比较复杂时, 就会发现使用RxJava的优势.

以我们的例子来说, 当前我们利用一个请求可以得到一个用户的信息并显示出来.
如果我们想得到这个用户的所有repo的所有者或者其他信息, 所有他follow的人的信息, 以及他们的repo的信息呢?

这就需要发很多个请求, 并且其中有些请求是并行发送的, 如果按照前面的方法, 不断地在callback里面嵌套, 那就太难看了.

### Setup with RxJava
#### 添加RxJava依赖
首先, 添加RxJava和RxAndroid的依赖:
```gradle
compile 'io.reactivex:rxjava:1.2.2'
compile 'io.reactivex:rxandroid:1.2.1'
```
注: 虽然在我写这篇文章的时候(2016.11.4)RxJava2.0刚刚release, 但是我们还是先用RxJava1来写这个demo.

然后添加retrofit的adapter-rxjava:
```gradle
compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
```
所以现在我们的依赖总的看起来是这样:
```gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile "com.android.support:appcompat-v7:${supportLibVersion}"
    compile "com.android.support:design:${supportLibVersion}"
    compile "com.jakewharton:butterknife:${butterKnifeVersion}"
    apt "com.jakewharton:butterknife-compiler:${butterKnifeVersion}"
    compile 'com.squareup.retrofit2:retrofit:2.1.0'
    compile 'com.google.code.gson:gson:2.8.0'
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
    compile 'io.reactivex:rxjava:1.2.2'
    compile 'io.reactivex:rxandroid:1.2.1'
    testCompile 'junit:junit:4.12'
}
```

#### Retrofit结合RxJava
Retrofit.Builder()中加入这一行:
`
.addCallAdapterFactory(RxJavaCallAdapterFactory.create());
`

ServiceGenerator变成这样:
```java
public class ServiceGenerator {
    public static final String API_BASE_URL = "https://api.github.com";

    private static OkHttpClient.Builder httpClient = new OkHttpClient.Builder();

    private static Retrofit.Builder builder =
            new Retrofit.Builder()
                    .baseUrl(API_BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .addCallAdapterFactory(RxJavaCallAdapterFactory.create());

    public static <S> S createService(Class<S> serviceClass) {
        Retrofit retrofit = builder.client(httpClient.build()).build();
        return retrofit.create(serviceClass);
    }
}
```
这样我们在`GithubService`中定义的接口方法, 既可以像原来一样返回`Call`, 也可以返回`Observable`.

### Retrofit + RxJava请求实例
以单个请求为例,
**不用RxJava的时候**:
```java
@GET("users/{user}/following")
Call<List<User>> getUserFollowing(@Path("user") String user);
```
请求的时候是这样的:
```java
GitHubService service = ServiceGenerator.createService(GitHubService.class);
Call<List<User>> userFollowing = service.getUserFollowing(inputUserNameView.getText().toString());
userFollowing.enqueue(new Callback<List<User>>() {
    @Override
    public void onResponse(Call<List<User>> call, Response<List<User>> response) {
        List<User> followingUsers = response.body();
        peopleListAdapter.setUsers(followingUsers);
        peopleListAdapter.notifyDataSetChanged();
    }

    @Override
    public void onFailure(Call<List<User>> call, Throwable t) {

    }
});
```
现在改用RxJava了, 返回的不是Call而是Observable:
```java
@GET("users/{user}/following")
Observable<List<User>> getUserFollowingObservable(@Path("user") String user);
```
结合RxJava请求的时候变为这样:
```java
GitHubService service = ServiceGenerator.createService(GitHubService.class);
String username = inputUserNameView.getText().toString();
service.getUserFollowingObservable(username)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Subscriber<List<User>>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(List<User> users) {
                LogUtils.i("onNext: " + users.size());
                peopleListAdapter.setUsers(users);
                peopleListAdapter.notifyDataSetChanged();
            }
        });
```

其中`.subscribeOn(Schedulers.io())`指定请求在io线程, `.observeOn(AndroidSchedulers.mainThread())`指定最后onNext()回调在主线程.

这是单个请求的例子, 所以RxJava的优势不是很明显, 如果我们有多个请求, 用RxJava进行变换组合显然就是更好的选择.

### RxJava处理多个请求的例子
设计这样一个场景, 我们现在取到了一个用户follow的所有人, 但是取回的信息中并不包含每个人拥有的repo个数, 只有一个url可用户查看所有repo.

接下来我们要取其中每一个人的详细信息, 就要查询另一个API, 重新查询这个人的完整信息.
```java
subscription = service.getUserFollowingObservable(username)
        .flatMap(new Func1<List<User>, Observable<User>>() {
            @Override
            public Observable<User> call(List<User> users) {
                return Observable.from(users);
            }
        })
        .flatMap(new Func1<User, Observable<User>>() {
            @Override
            public Observable<User> call(User user) {
                return service.getUserObservable(user.getLogin());
            }
        })
        .toList()
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Subscriber<List<User>>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(List<User> users) {
                peopleListAdapter.setUsers(users);
                peopleListAdapter.notifyDataSetChanged();
            }
        });
```
可以看到我们加了两个`flatMap()`和一个`toList()`来做这个事情.

首先, 第一步我们用`getUserFollowingObservable()`得到的是一个`Observable<List<User>>`;
我们之后用`.flatMap()`, 它的输入是`List<User>`, 返回的是`Observable<User>`. 我们在其中用了一个`.from()`来生成一个发射一组User的`Observable`;

之后第二个`.flatMap()`里, 输入是前一个`Observable`的输出, 即User, 调用了`getUserObservable()`, 返回的结果是`Observable<User>`, 之后加一个`.toList()`, 把输出的结果从单个的User变为List<User>, 即和我们最初的例子一样. 

只不过此时得到的用户信息是更详细的用户信息, 包含了他的repo数据和follow数据. 因为它们是通过单独查询每一个人得到的.

运行, 虽然可以得到我们想要的结果, 但是这个例子仍然是有问题的.

#### 线程问题处理
上面多个请求的例子, 发现虽然实现了我们的需求, 但是结果回来得很慢. 
我们加上一个`.map`操作符来加上log:
```java
...
subscription = service.getUserFollowingObservable(username)
        .flatMap(new Func1<List<User>, Observable<User>>() {
            @Override
            public Observable<User> call(List<User> users) {
                return Observable.from(users);
            }
        })
        .flatMap(new Func1<User, Observable<User>>() {
            @Override
            public Observable<User> call(User user) {
                return service.getUserObservable(user.getLogin())
                        .map(new Func1<User, User>() {
                            @Override
                            public User call(User user) {
                                // this .map is used to output log information to check the threads
                                LogUtils.i("getUserObservable: " + user.getLogin());
                                return user;
                            }
                        });
            }
        })
        .toList()
        ...
```
由Log可以发现(log中的线程号是一样的)单独取每一个用户详细信息的请求都发生在同一个线程, 是顺次进行的.

查看代码: 
Demo地址: https://github.com/mengdd/HelloRetrofit.
`git checkout multiple-requests-in-single-thread`

回头梳理一下我们的需求, 请求一个所有follow的人, 返回一个人的List, 然后对List中的每一项, 单独请求详细信息.

那么按理来说, 第二个批量的请求是可以同时发送, 并行进行的.
所以我们想要的行为其实是平行发送多个请求, 然后最后统一结果到UI线程. 

改动如下:
```java
subscription = service.getUserFollowingObservable(username)
        .subscribeOn(Schedulers.io())
        .flatMap(new Func1<List<User>, Observable<User>>() {
            @Override
            public Observable<User> call(List<User> users) {
                LogUtils.i("from");
                return Observable.from(users);
            }
        })
        .flatMap(new Func1<User, Observable<User>>() {
            @Override
            public Observable<User> call(User user) {
                return service.getUserObservable(user.getLogin())
                        .subscribeOn(Schedulers.io())
                        .map(new Func1<User, User>() {
                            @Override
                            public User call(User user) {
                                // this map operation is just used for showing log
                                LogUtils.i("getUserObservable: " + user.getLogin());
                                return user;
                            }
                        });
            }
        })
        .toList()
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Subscriber<List<User>>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(List<User> users) {
                LogUtils.i("onNext: " + users.size());
                peopleListAdapter.setUsers(users);
                peopleListAdapter.notifyDataSetChanged();
            }
        })
```


这样一改, 我们的显示时间不再是所有请求时间之和, 而是只取决于最慢的那个请求时间.

查看代码:
Demo地址: https://github.com/mengdd/HelloRetrofit
`git checkout multiple-requests-in-multiple-threads`


### 取消请求
上面的代码中已经出现了, 订阅方法`subscribe()`的返回值是一个`Subscription`对象, 我们保存了这个对象的引用, 然后在`onPause()`的时候取消了请求, 防止内存泄露.
```java
@Override
public void onPause() {
    super.onPause();
    if (subscription != null && subscription.isUnsubscribed()) {
        subscription.unsubscribe();
    }
}
```
当然也可以选别的生命周期回调, 比如`onDestroyView()`或者`onDestroy()`.

如果有多个请求, 可以用:
```java
private CompositeSubscription compositeSubscription = new CompositeSubscription();

...
// 在发请求的地方, 返回subscription
compositeSubscription.add(subscription);
...

// 选一个生命周期注销所有请求
@Override
public void onPause() {
    super.onPause();
    compositeSubscription.unsubscribe();
}
```

## Demo说明
Demo地址: https://github.com/mengdd/HelloRetrofit

本Demo只用于展示Retrofit和RxJava结合的使用, 为了清晰起见所以没有采用MVP构架, 也没有用Dagger进行依赖注入, 有的请求也没有在生命周期结束时取消, 也没有UI的loading效果, 大家使用时请根据实际需要做一些处理.

这些没有的东西会在我最近在做一个应用repo中出现: https://github.com/mengdd/GithubClient, 还在开发中, 可以关注一下. 

另, Demo使用有时候用着用着请求就返回了:
```
{"message":"API rate limit exceeded for xxx ip...
```
这是因为没授权的用户每小时最多只能发60个请求:https://developer.github.com/v3/#rate-limiting
解决办法就是..查following的人时, 不要查那种follow了很多人的账号. orz.




## References
- [Retrofit Github](https://github.com/square/retrofit)
- [Retrofit Website](https://square.github.io/retrofit/)
- [CodePath- Consuming APIs with Retrofit](https://guides.codepath.com/android/Consuming-APIs-with-Retrofit)
- [Future Studio 系列教程](https://futurestud.io/tutorials/retrofit-getting-started-and-android-client)
- [RxJava 与 Retrofit 结合的最佳实践](http://gank.io/post/56e80c2c677659311bed9841)