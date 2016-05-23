---
title: EventBus源码阅读记录
categories: Android
---

# EventBus源码阅读记录

repo地址: 
[greenrobot/EventBus](https://github.com/greenrobot/EventBus)


## EventBus的构造
双重加锁的单例.

```java
static volatile EventBus defaultInstance;
public static EventBus getDefault() {
    if (defaultInstance == null) {
        synchronized (EventBus.class) {
            if (defaultInstance == null) {
                defaultInstance = new EventBus();
            }
        }
    }
    return defaultInstance;
}
```

但是仍然开放了构造函数,用于构造其他别的对象.

**Builder模式**: `EventBusBuilder`.
有一个`DEFAULT_BUILDER`.

## 注册
注册即添加订阅者,调用`register()`方法:
方法参数最全时共有三个参数:

```java
private synchronized void register(Object subscriber, boolean sticky, int priority) {
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass());
    for (SubscriberMethod subscriberMethod : subscriberMethods) {
        subscribe(subscriber, subscriberMethod, sticky, priority);
    }
}
```
其中`subscriber`(订阅者)传入的是一个对象,用到了它的class.
`SubscriberMethodFinder`会去找这个类中的方法.
被找到的方法最后会被缓存到一个map里,key是`class`, value是`ArrayList<SubscriberMethod>()`.


### 寻找方法

在一个类(class)中寻找方法的过程, 首先是拿出方法:
在循环中skip了一些系统的类, 因为我们不可能在这些类里加入方法.

```java
while (clazz != null) {
    String name = clazz.getName();
    if (name.startsWith("java.") || name.startsWith("javax.") || name.startsWith("android.")) {
        // Skip system classes, this just degrades performance
        break;
    }

    // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
    try {
        // This is faster than getMethods, especially when subscribers a fat classes like Activities
        Method[] methods = clazz.getDeclaredMethods();
        filterSubscriberMethods(subscriberMethods, eventTypesFound, methodKeyBuilder, methods);
    } catch (Throwable th) {
        // Workaround for java.lang.NoClassDefFoundError, see https://github.com/greenrobot/EventBus/issues/149
        Method[] methods = subscriberClass.getMethods();
        subscriberMethods.clear();
        eventTypesFound.clear();
        filterSubscriberMethods(subscriberMethods, eventTypesFound, methodKeyBuilder, methods);
        break;
    }
    clazz = clazz.getSuperclass();
}
```
#### 反射
关于反射的性能讨论, 代码中有说:

```java
// This is faster than getMethods, especially when subscribers a fat classes like Activities
Method[] methods = clazz.getDeclaredMethods();
```

为什么呢?
`getMethods()`返回了所有的public方法,包含从所有基类继承的,也即包含了从Object类中继承的public方法.
`getDeclaredMethods()`返回了该类中声明的所有方法,包括各种访问级别的,但是只包含本类中的,不包括基类中的方法.

相关DOC:

[反射package-summary](https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/package-summary.html)
[getDeclaredMethods()](https://docs.oracle.com/javase/7/docs/api/java/lang/Class.html#getDeclaredMethods())
[getMethods()](https://docs.oracle.com/javase/7/docs/api/java/lang/Class.html#getMethods())


#### Issue of NoClassDefFoundError
这里有一个try catch主要是为了解决这个issue: <https://github.com/greenrobot/EventBus/issues/149>
本来的流程是: 

1. 从自己的class开始,每次都`getDeclaredMethods()`, 即提取自己类中的方法,不取基类.
2. 取完之后, `getSuperclass()`,获取基类的class,重新进入while循环.直到进入java包或者android包才退出.

但是`getDeclaredMethods()`会检查一些参数和返回值, 如果找不到类型则抛出NoClassDefFoundError.
`getMethods()`却不检查.

什么样的情况会抛出这个Error呢?

Android代码里可能会有一些方法标明了`@TargetApi`,表明是更高级的sdk上才会有的.
这样在低版本的机器上遇到了这些代码,就无法解析出它们的类了.

**只要你的作为subscriber的class里含有这种东西,就会出现问题.**

为了解决这个崩溃, 所以代码里catch了一把,然后采用第二种方案`getMethods()`,一次性get所有基类中的方法,这种效率虽然低,但是不会抛异常.
需要把之前的map都清理一把.

### 筛选方法
得到了所有的方法之后,开始筛选方法:

```java
private void filterSubscriberMethods(List<SubscriberMethod> subscriberMethods, HashMap<String, Class> eventTypesFound, 
StringBuilder methodKeyBuilder, Method[] methods)

```

这里第一个参数会作为最后的返回值,即我们方法选择的结果.

筛选的过程, 遍历所有找到的方法:

1. 看它是以”onEvent”开头,即为我们要找的目标方法.
2. 然后`getModifiers()`看它是一个**public**的方法,并且不是我们要忽略的方法.
注意这里用到了位操作**&**来比较. 结果不为零表示满足,为零表示不满足.
默认的忽略方法是***static, bridge, synthetic***方法.
后两个词指的其实是同一种东东,但是这是什么东东呢?
是编译器生成的方法, 见参考链接: 
<https://javax0.wordpress.com/2014/02/26/syntethic-and-bridge-methods/>
<https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html>
从上面的例子中可以看出,编译器生成***bridge***方法主要是为了保证多态的顺利进行.它和基类的签名一样,但是实现去调用了子类的方法.自己偷偷完成了其中的类型转换.

3. 获取参数类型:必须是一个参数.
4. 获取`ThreadMode`: 即看方法名中onEvent之后还是什么,一共有四种Mode,对应四种方法名:
***onEvent(), onEventMainThread(), onEventBackgroundThread(), onEventAsync()***
如果获取不到`ThreadMode`,则continue;即这个方法不是我们要找的方法.

5. 用`StringBuilder`组成一个key: ***method name>parameterType class name***.
注意这里StringBuilder的清理方式是`setLength(0)`.
然后放进了一个`eventTypesFound`的HashMap, String是key, Class是value,这里放的是`method.getDeclaringClass()`;即方法声明的那个类的类型.

注意这里还利用了`put()`方法的返回值,如果map里之前有这个key对应的值,那么老的value会作为返回值返回.
文档:
[HashMap.put()](https://docs.oracle.com/javase/7/docs/api/java/util/HashMap.html#put(K,%20V))



这里还用了这个一个方法: [isAssignableFrom](https://docs.oracle.com/javase/7/docs/api/java/lang/Class.html#isAssignableFrom(java.lang.Class))
判断是否自己的class是参数的基类或接口.如果传入的参数是当前对象的子类或自身,则返回true.

如果有old class存在,并且old class和新的class不能互相转换, 后者old是new的子类, 那么`eventTypesFound`这个map里还是保存老的值.

如果存在old class,但是old class是新加class的父类,会把新的class加进`eventTypesFound`的map,取代前者,即这个map中尽量放继承体系下层中更具体的类.
这里虽然父类没有被放进`eventTypesFound`,但是父类的方法仍然会被加进最后返回的methods的map.


筛选结束后,我们就获取到了所有的目标方法.
把它们都存在了一个cache map里面,以免同一个类下次我们又要重新筛选一遍:
```java
private static final Map<Class<?>, List<SubscriberMethod>> methodCache = new HashMap<Class<?>, List<SubscriberMethod>>();
```

### 订阅
得到了方法的list(`List<SubscriberMethod>`)之后,我们要对每一个成员调用
`private void subscribe(Object subscriber, SubscriberMethod subscriberMethod, boolean sticky, int priority)` 方法.

里面有一个新的数据类型`CopyOnWriteArrayList`:
[CopyOnWriteArrayList Java doc](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CopyOnWriteArrayList.html)
[CopyOnWriteArrayList android doc](http://developer.android.com/reference/java/util/concurrent/CopyOnWriteArrayList.html)

类说明: *A thread-safe variant of ArrayList in which all mutative operations (add, set, and so on) are implemented by making a fresh copy of the underlying array.*

这个数据类型是一个ArrayList,但是它在每次进行变异操作之前都拷贝一份新的.它底层的数组是***volatile***的.
这种数据类型的写操作代价很高.

`subscribe()`方法中主要是给这两个字段放数据:

1. `private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;`
key是eventType的Class, value是Subscription这种数据类型的数组:
`Subscription newSubscription = new Subscription(subscriber, subscriberMethod, priority);`

2. `private final Map<Object, List<Class<?>>> typesBySubscriber;`
key是subscriber,即订阅者的类的对象,value是eventType的class,即事件类.

## 注销
`unregister()`的时候, 传入subscriber:
首先从typesBySubscriber获取到事件的List,然后遍历这个List, 从subscriptionsByEventType中移除该eventType,并且subscriber是当前subscriber的Subscription.
遍历完成之后,从typesBySubscriber移除该subscriber.

## 事件触发
好了,注册和反注册到这里就结束了,看起来也就是找到一些方法和类型,放在一些map里面,注销的时候再从map里面拿出来而已.
真正做事情的代码呢?

首先看事件的触发: `post()`方法, 这里传入的参数是事件类对象.

```java
public void post(Object event) {
    PostingThreadState postingState = currentPostingThreadState.get();
    List<Object> eventQueue = postingState.eventQueue;
    eventQueue.add(event);

    if (!postingState.isPosting) {
        postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            while (!eventQueue.isEmpty()) {
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```

大致地看上去好像就是加入了一个队列,然后发送出去直到队列为空.

对每一个事件来说,是调用了`postSingleEvent()`这个方法.

`postSingleEvent()`这个方法里`eventInheritance`为true时(默认行为)会把event的class拿出来,然后取出它的所有基类和接口,和它自己一起放在一个map里.
这是可以理解的,因为可能我们本来的需求是监听了一个灾难事件,后来的需求发展,又写了个它的子类事件叫地震.
那么当我post地震事件的时候,除了地震事件后来新加的处理,当然也要采取原先灾难事件的相关措施.

取出所有基类和接口的方法:`lookupAllEventTypes()`

```java
/** Looks up all Class objects including super classes and interfaces. Should also work for interfaces. */
private List<Class<?>> lookupAllEventTypes(Class<?> eventClass) {
    synchronized (eventTypesCache) {
        List<Class<?>> eventTypes = eventTypesCache.get(eventClass);
        if (eventTypes == null) {
            eventTypes = new ArrayList<Class<?>>();
            Class<?> clazz = eventClass;
            while (clazz != null) {
                eventTypes.add(clazz);
                addInterfaces(eventTypes, clazz.getInterfaces());
                clazz = clazz.getSuperclass();
            }
            eventTypesCache.put(eventClass, eventTypes);
        }
        return eventTypes;
    }
}
```

所有这些费时的遍历查找操作都是有一个map作为cache的.
注意这里添加接口的时候,因为**接口是多继承的**,所以除了去重以外,还需要深入遍历:

```java
/** Recurses through super interfaces. */
static void addInterfaces(List<Class<?>> eventTypes, Class<?>[] interfaces) {
    for (Class<?> interfaceClass : interfaces) {
        if (!eventTypes.contains(interfaceClass)) {
            eventTypes.add(interfaceClass);
            addInterfaces(eventTypes, interfaceClass.getInterfaces());
        }
    }
}
```

获取到所有类型之后,进行遍历, 对每一个eventClass进行处理, 真正的对每一个类型post的方法是这个:
`
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) 
`

这里,从之前那个`subscriptionsByEventType`里面,根据eventClass把`CopyOnWriteArrayList<Subscription>`拿出来.
这里拿出来的就是一个List,里面是一个一个的*onEventXXX*方法的个体,
对每一个`Subscription`,执行了:
`private void postToSubscription(Subscription subscription, Object event, boolean isMainThread)`


### 线程模式
这里根据线程模式不同,有一个switch case.

```java
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
    switch (subscription.subscriberMethod.threadMode) {
        case PostThread:
            invokeSubscriber(subscription, event);
            break;
        case MainThread:
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            } else {
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case BackgroundThread:
            if (isMainThread) {
                backgroundPoster.enqueue(subscription, event);
            } else {
                invokeSubscriber(subscription, event);
            }
            break;
        case Async:
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
    }
}
```

这里`invokeSubscriber(Subscription subscription, Object event)`方法就是直接通过`Method`, 反射调用, invoke了那个方法.

    `case PostThread`: 直接在当前线程调用这个方法.
    `case MainThread`: 如果当前线程是主线程,则直接调用,否则加入mainThreadPoster的队列.
    `case BackgroundThread`: 如果当前是主线程,加入backgroundPoster队列, 否则直接调用.
    `case Async`: 加入asyncPoster队列.

加入的三个队列类型如下:
```java
private final HandlerPoster mainThreadPoster; 
private final BackgroundPoster backgroundPoster;
private final AsyncPoster asyncPoster;
```

`HandlerPoster`继承自Handler, 内部有一个`PendingPostQueue`.
这三个poster里面都是这个`PendingPostQueue`, 数据结构是`PendingPost`

#### 关于Queue的相关知识
队列Queue: Java中Queue是一个接口, 类文档:
[Queue Java doc] (https://docs.oracle.com/javase/7/docs/api/java/util/Queue.html)
它是继承自Collection这个接口:
[Collection] (https://docs.oracle.com/javase/7/docs/api/java/util/Collection.html)

Queue这个数据结构可以自己定义顺序, 可以用来做FIFO也可以用来做LIFO.
每一种Queue的实现都必须指定要用什么顺序.
不管是什么顺序,head上的那个元素都是`remove()`或`poll()`即将移除的元素.

`offer()`方法将会试图插入一个元素,如果失败了就会返回false.
`remove()`和`poll()`方法都会删除并返回head元素.
`peek()`只查询,不remove.

#### 主线程处理 HandlerPoster
所以这里看看`HandlerPoster`是怎么做的:

1. 它继承自`Handler`, 初始化的时候用的是mainLooper,所以确保了消息处理操作都是在主线程:
`mainThreadPoster = new HandlerPoster(this, Looper.getMainLooper(), 10);`
2. 这个里面写了一个自己的queue: PendingPostQueue里面包含的数据是:`PendingPost`.
`PendingPost`这个类里用了一个pool来实现一个对象池,最大限制是10000.
obtain的时候, 如果池子里有对象,则从池子里拿出来一个, 如果池中没有对象,则new一个新的PendingPost; release的时候放回池子去.

`HandlerPoster`主要做两件事:

1. enqueue一个PendingPost, sendMessage, 
2. 在handleMessage()方法里面处理message.
handleMessage()里面是一个while循环,从队列里面拿出PendingPost然后调用EventBus的invokeSubscriber()方法.
这里调用方法之前就会release该PendingPost.

#### 异步和后台处理 AsyncPoster和BackgroundPoster
`AsyncPoster`和`BackgroundPoster`都是一个`Runnable`.

enqueue的时候把PendingPost加入队列, 然后调用`eventBus.getExecutorService().execute(this);`

`run()`方法里面就是从队列中拿出PendingPost,然后invoke,和上面很像.

默认的对象是: 
`
private final static ExecutorService DEFAULT_EXECUTOR_SERVICE = Executors.newCachedThreadPool();
`
提供了一个线程池,可以异步地执行操作.

那么它们两者有什么不同呢?

AsyncPoster很简单, run里面直接invoke, 没有过多的判断. 即对每一个任务都是直接启动线程执行.
BackgroundPoster比较复杂,有一个boolean来判断是否正在run, run()方法里面是一个while true的循环,当queue全部被执行完之后才return.
如果队列中有任务正在执行,这时候enqueue()操作会加入元素到队列中,等待执行.
即BackgroundPoster只用了一个线程,所有的事件都是按顺序执行的,等到前面的任务执行完了才会进行下一个.


对各个模式的说明可以参见`ThreadMode.java`类.
Async模式下,不管你的post thread是什么,都是会新启线程来执行任务的,所以适用于那些比较耗时的操作.
为了避免并发线程过多, EventBus里面使用了一个线程池来复用线程.

## 事件取消
有一个public的cancel方法:

```java
public void cancelEventDelivery(Object event) {
    PostingThreadState postingState = currentPostingThreadState.get();
    if (!postingState.isPosting) {
        throw new EventBusException(
                "This method may only be called from inside event handling methods on the posting thread");
    } else if (event == null) {
        throw new EventBusException("Event may not be null");
    } else if (postingState.event != event) {
        throw new EventBusException("Only the currently handled event may be aborted");
    } else if (postingState.subscription.subscriberMethod.threadMode != ThreadMode.PostThread) {
        throw new EventBusException(" event handlers may only abort the incoming event");
    }

    postingState.canceled = true;
}
```

这个方法的使用可以从测试代码里面看出来:
1.首先它只能在handler里面调用, 即第一个异常.这里判断的isPosting这个值在post的时候变为true,处理完就变为false.
这里用到的currentPostingState:

```java
private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
    @Override
    protected PostingThreadState initialValue() {
        return new PostingThreadState();
    }
};

```
ThreadLocal类是什么?
[ThreadLocal类] (https://docs.oracle.com/javase/7/docs/api/java/lang/ThreadLocal.html)

```
ThreadLocal instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).
```

主要是用来给每一个线程保存一个不同的状态值.
这个currentPostingThreadState在第一次被调用`get()`方法的时候初始化,也即在`public void post(Object event)` 方法里.
然后修改了它的状态, 之后再在同一个线程里,即可访问到它的状态.

这里cancel的测试也写得很有意思,可以看一下.




## 黏性事件

什么叫Sticky?
字面上看是黏性的.

之前的事件都是非黏性的,即有一个`register()`和`unregister()`方法.
`register()`了subscriber之后, EventBus会扫描该类中的onEventXXX()方法,建立一些map来记录.
`unregister()`即合理地清除了这些数据.

而对于sticky的事件,注册时调用`registerSticky()`, 并没有相应的注销方法.只有一个单独的`removeAllStickyEvents()`方法.

sticky的事件注册的时候, `subscribe()`方法中, 除了重复上面正常的过程之外, 还有一个额外的map:
`private final Map<Class<?>, Object> stickyEvents;`
这个数据类型是: `stickyEvents = new ConcurrentHashMap<Class<?>, Object>();`
存的是event的Class和event对象.

注册时如果发现这个map中相同的event type要处理,***会立即触发***, 通知到它的订阅者.

注意这个sticky event存的是最近的一个事件: **most recent event**.

sticky事件触发的时候调用: 
`public void postSticky(Object event)`


sticky的代码里有一个`cast()`方法:
看文档:
[Class](https://docs.oracle.com/javase/7/docs/api/java/lang/Class.html)

这个`cast()`方法就是用来把对象强转成当前的这个Class类型.


## 结语
EventBus是一个Android上用的消息分发的类库,非常灵活好用,主要的原理是利用了反射注册以及调用. 

本文是在阅读EventBus的源码过程中所记录的东西, 遇到不懂的去查了, 然后留下了链接. 
有点流水账,讲得也不是很深入,如果有错请帮忙指正.









