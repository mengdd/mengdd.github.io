---
title: Android MVP模式 谷歌官方代码解读
date: 2016-10-22 19:11:27
tags: [MVP, MVC, MVVM, Clean Architecture, Data Binding, Loader, Content Provider, Dagger, RxJava]
catetories: [Android, Architecture]
---

# Google官方MVP Sample代码解读
关于Android程序的构架, 当前(2016.10)最流行的模式即为MVP模式, Google官方提供了Sample代码来展示这种模式的用法.
Repo地址: [android-architecture](https://github.com/googlesamples/android-architecture).
本文为阅读官方sample代码的阅读笔记和分析.

<!-- more -->

官方Android Architecture Blueprints [beta]:
Android在如何组织和构架一个app方面提供了很大的灵活性, 但是同时这种自由也可能会导致app在测试, 维护, 扩展方面变得困难.

Android Architecture Blueprints展示了可能的解决方案. 在这个项目里, 我们用各种不同的构架概念和工具实现了同一个应用([To Do App](https://github.com/googlesamples/android-architecture/wiki/To-do-app-specification)). 主要的关注点在于代码结构, 构架, 测试和维护性. 
但是请记住, 用这些模式构架app的方式有很多种, 要根据你的需要, 不要把这些当做绝对的典范.

## MVP模式 概念
之前有一个MVC模式: Model-View-Controller.
MVC模式 有两个主要的缺点: 首先, View持有Controller和Model的引用; 第二, 它没有把对UI逻辑的操作限制在单一的类里, 这个职能被Controller和View或者Model共享.
所以后来提出了MVP模式来克服这些缺点.

MVP(Model-View-Presenter)模式:
- Model: 数据层. 负责与网络层和数据库层的逻辑交互.
- View: UI层. 显示数据, 并向Presenter报告用户行为.
- Presenter: 从Model拿数据, 应用到UI层, 管理UI的状态, 决定要显示什么, 响应用户的行为.
MVP模式的最主要优势就是耦合降低, Presenter变为纯Java的代码逻辑, 不再与Android Framework中的类如Activity, Fragment等关联, 便于写单元测试.


## todo-mvp 基本的Model-View-Presenter架构
app中有四个功能:
- Tasks
- TaskDetail
- AddEditTask
- Statistics

每个功能都有:
- 一个定义View和Presenter接口的`Contract`接口;
- 一个Activity用来管理fragment和presenter的创建;
- 一个实现了View接口的Fragment;
- 一个实现了Presenter接口的presenter.

![mvp](/images/mvp.png)

### 基类
Presenter基类:
```java
public interface BasePresenter {
    void start();
}
```
例子中这个`start()`方法都在Fragment的`onResume()`中调用.

View基类:
```java
public interface BaseView<T> {
    void setPresenter(T presenter);
}
```

### View实现
- Fragment作为每一个View接口的实现, 主要负责数据显示和在用户交互时调用Presenter, 但是例子代码中也是有一些直接操作的部分, 比如点击开启另一个Activity, 点击弹出菜单(菜单项的点击仍然是调用presenter的方法).
- View接口中定义的方法多为`showXXX()`方法. 

- Fragment作为View实现, 接口中定义了方法:
```java
@Override
public boolean isActive() {
    return isAdded();
}
```

在Presenter中数据回调的方法中, 先检查View.isActive()是否为true, 来保证对Fragment的操作安全.

### Presenter实现
- Presenter的`start()`方法在`onResume()`的时候调用, 这时候取初始数据; 其他方法均对应于用户在UI上的交互操作. 
- New Presenter的操作是在每一个Activity的`onCreate()`里做的: 先添加了Fragment(View), 然后把它作为参数传给了Presenter. 这里并没有存Presenter的引用.
- Presenter的构造函数有两个参数, 一个是Model(Model类一般叫XXXRepository), 一个是View. 构造中先用guava的`checkNotNull()`
检查两个参数是否为null, 然后赋值到字段; 之后再调用View的`setPresenter()`方法把Presenter传回View中引用.

### Model实现细节
- Model只有一个类, 即`TasksRepository`. 它还是一个单例. 因为在这个应用的例子中, 我们操作的数据就这一份.

它由手动实现的注入类`Injection`类提供:
```java
public class Injection {

    public static TasksRepository provideTasksRepository(@NonNull Context context) {
        checkNotNull(context);
        return TasksRepository.getInstance(FakeTasksRemoteDataSource.getInstance(),
                TasksLocalDataSource.getInstance(context));
    }
}
```

构造如下:
```java
private TasksRepository(@NonNull TasksDataSource tasksRemoteDataSource,
                        @NonNull TasksDataSource tasksLocalDataSource) {
    mTasksRemoteDataSource = checkNotNull(tasksRemoteDataSource);
    mTasksLocalDataSource = checkNotNull(tasksLocalDataSource);
}
```

- 数据分为local和remote两大部分. local部分负责数据库的操作, remote部分负责网络. Model类中还有一个内存缓存.
- `TasksDataSource`是一个接口. 接口中定义了Presenter查询数据的回调接口, 还有一些增删改查的方法.

### 单元测试
MVP模式的主要优势就是便于为业务逻辑加上单元测试.
本例子中的单元测试是给`TasksRepository`和四个feature的Presenter加的.
Presenter的单元测试, Mock了View和Model, 测试调用逻辑, 如:
```java
public class AddEditTaskPresenterTest {

    @Mock
    private TasksRepository mTasksRepository;
    @Mock
    private AddEditTaskContract.View mAddEditTaskView;
    private AddEditTaskPresenter mAddEditTaskPresenter;

    @Before
    public void setupMocksAndView() {
        MockitoAnnotations.initMocks(this);
        when(mAddEditTaskView.isActive()).thenReturn(true);
    }

    @Test
    public void saveNewTaskToRepository_showsSuccessMessageUi() {
        mAddEditTaskPresenter = new AddEditTaskPresenter("1", mTasksRepository, mAddEditTaskView);

        mAddEditTaskPresenter.saveTask("New Task Title", "Some Task Description");

        verify(mTasksRepository).saveTask(any(Task.class)); // saved to the model
        verify(mAddEditTaskView).showTasksList(); // shown in the UI
    }
    ...
}
```

## todo-mvp-loaders 用Loader取数据的MVP
基于上一个例子todo-mvp, 只不过这里改为用Loader来从Repository得到数据.
![todo-mvp-loaders](/images/mvp-loaders.png)

使用Loader的优势:
- 去掉了回调, 自动实现数据的异步加载;
- 当内容改变时回调出新数据;
- 当应用因为configuration变化而重建loader时, 自动重连到上一个loader.

### Diff with todo-mvp
既然是基于todo-mvp, 那么之前说过的那些就不再重复, 我们来看一下都有什么改动:
`git difftool -d todo-mvp`

添加了两个类:
`TaskLoader`和`TasksLoader`.

在Activity中new Loader类, 然后传入Presenter的构造方法.

`Contract`中View接口删掉了`isActive()`方法, Presenter删掉了`populateTask()`方法.


### 数据获取
添加的两个新类是`TaskLoader`和`TasksLoader`, 都继承于`AsyncTaskLoader`, 只不过数据的类型一个是单数, 一个是复数.

`AsyncTaskLoader`是基于`ModernAsyncTask`, 类似于`AsyncTask`, 
把load数据的操作放在`loadInBackground()`里即可, `deliverResult()`方法会将结果返回到主线程, 我们在listener的`onLoadFinished()`里面就可以接到返回的数据了, (在这个例子中是几个Presenter实现了这个接口).

`TasksDataSource`接口的这两个方法:
```java
List<Task> getTasks();
Task getTask(@NonNull String taskId);
```
都变成了同步方法, 因为它们是在`loadInBackground()`方法里被调用.

Presenter中保存了`Loader`和`LoaderManager`, 在`start()`方法里`initLoader`, 然后`onCreateLoader`返回构造传入的那个loader. 
`onLoadFinished()`里面调用View的方法. 此时Presenter实现`LoaderManager.LoaderCallbacks`.

### 数据改变监听
`TasksRepository`类中定义了observer的接口, 保存了一个listener的list:
```java
private List<TasksRepositoryObserver> mObservers = new ArrayList<TasksRepositoryObserver>();

public interface TasksRepositoryObserver {
    void onTasksChanged();
}
```
每次有数据改动需要刷新UI时就调用:
```java
private void notifyContentObserver() {
    for (TasksRepositoryObserver observer : mObservers) {
        observer.onTasksChanged();
    }
}
```
在两个Loader里注册和注销自己为`TasksRepository`的listener: 在`onStartLoading()`里add, `onReset()`里面remove方法.
这样每次`TasksRepository`有数据变化, 作为listener的两个Loader都会收到通知, 然后force load:
```java
@Override
public void onTasksChanged() {
    if (isStarted()) {
        forceLoad();
    }
}
```
这样`onLoadFinished()`方法就会被调用.

## todo-databinding
基于todo-mvp, 使用[Data Binding library](http://developer.android.com/tools/data-binding/guide.html#data_objects)来显示数据, 把UI和动作绑定起来.

说到ViewModel, 还有一种模式叫MVVM(Model-View-ViewModel)模式.
这个例子并没有严格地遵循`Model-View-ViewModel`模式或者`Model-View-Presenter`模式, 因为它既用了ViewModel又用了Presenter.

![todo-databinding](/images/mvp-databinding.png)

Data Binding Library让UI元素和数据模型绑定:
- layout文件用来绑定数据和UI元素;
- 事件和action handler绑定;
- 数据变为可观察的, 需要的时候可以自动更新.


### Diff with todo-mvp
添加了几个类:
- `StatisticsViewModel`;
- `SwipeRefreshLayoutDataBinding`;
- `TasksItemActionHandler`;
- `TasksViewModel`;

从几个View的接口可以看出方法数减少了, 原来需要多个showXXX()方法, 现在只需要一两个方法就可以了.

### 数据绑定
以`TasksDetailFragment`为例:
以前在todo-mvp里需要这样:
```java
public void onCreateView(...) {
    ...
    mDetailDescription = (TextView)
root.findViewById(R.id.task_detail_description);
}

@Override
public void showDescription(String description) {
    mDetailDescription.setVisibility(View.VISIBLE);
    mDetailDescription.setText(description);
}
```

现在只需要这样:
```java
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.taskdetail_frag, container, false);
    mViewDataBinding = TaskdetailFragBinding.bind(view);
    ...
}

@Override
public void showTask(Task task) {
    mViewDataBinding.setTask(task);
}
```

因为所有数据绑定的操作都写在了xml里:
```xml
<TextView
    android:id="@+id/task_detail_description"
    ...
    android:text="@{task.description}" />
```

### 事件绑定
数据绑定省去了`findViewById()`和`setText()`, 事件绑定则是省去了`setOnClickListener()`.

比如`taskdetail_frag.xml`中的
```xml
<CheckBox
    android:id="@+id/task_detail_complete"
    ...
    android:checked="@{task.completed}"
    android:onCheckedChanged="@{(cb, isChecked) ->
    presenter.completeChanged(task, isChecked)}" />
```

其中Presenter是这时候传入的:
```java
@Override
public void onActivityCreated(Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    mViewDataBinding.setPresenter(mPresenter);
}
```

### 数据监听
在显示List数据的界面`TasksFragment`, 仅需要知道数据是否为空, 所以它使用了`TasksViewModel`来给layout提供信息, 当尺寸设定的时候, 只有一些相关的属性被通知, 和这些属性绑定的UI元素被更新.
```java
public void setTaskListSize(int taskListSize) {
    mTaskListSize = taskListSize;
    notifyPropertyChanged(BR.noTaskIconRes);
    notifyPropertyChanged(BR.noTasksLabel);
    notifyPropertyChanged(BR.currentFilteringLabel);
    notifyPropertyChanged(BR.notEmpty);
    notifyPropertyChanged(BR.tasksAddViewVisible);
}
```

### 其他实现细节
- Adapter中的Data Binding, 见`TasksFragment`中的`TasksAdapter`.
```java
@Override
public View getView(int i, View view, ViewGroup viewGroup) {
    Task task = getItem(i);
    TaskItemBinding binding;
    if (view == null) {
        // Inflate
        LayoutInflater inflater = LayoutInflater.from(viewGroup.getContext());

        // Create the binding
        binding = TaskItemBinding.inflate(inflater, viewGroup, false);
    } else {
        binding = DataBindingUtil.getBinding(view);
    }

    // We might be recycling the binding for another task, so update it.
    // Create the action handler for the view
    TasksItemActionHandler itemActionHandler =
            new TasksItemActionHandler(mUserActionsListener);
    binding.setActionHandler(itemActionHandler);
    binding.setTask(task);
    binding.executePendingBindings();
    return binding.getRoot();
}
```
- Presenter可能会被包在ActionHandler中, 比如`TasksItemActionHandler`.
- ViewModel也可以作为View接口的实现, 比如`StatisticsViewModel`.
- `SwipeRefreshLayoutDataBinding`类定义的`onRefresh()`动作绑定.


## todo-mvp-clean
这个例子是基于Clean Architecture的原则: 
[The Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html).
关于Clean Architecture, 还可以看这个Sample App: [Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture).

这个例子在todo-mvp的基础上, 加了一层domain层, 把应用分为了三层:
![mvp-clean](/images/mvp-clean.png)

Domain: 盛放了业务逻辑, domain层包含use cases或者interactors, 被应用的presenters使用. 这些use cases代表了所有从presentation层可能进行的行为.

**关键概念**
和基本的mvp sample最大的不同就是domain层和use cases. 从presenters中抽离出来的domain层有助于避免presenter中的代码重复.

Use cases定义了app需要的操作, 这样增加了代码的可读性, 因为类名反映了目的.

Use cases对于操作的复用来说也很好. 比如`CompleteTask`在两个Presenter中都用到了.

Use cases的执行是在后台线程, 使用[command pattern](http://www.oodesign.com/command-pattern.html). 这样domain层对于Android SDK和其他第三方库来说都是完全解耦的.


### Diff with todo-mvp
每一个feature的包下都新增了domain层, 里面包含了子目录model和usecase等.

`UseCase`是一个抽象类, 定义了domain层的基础接口点.
`UseCaseHandler`用于执行use cases, 是一个单例, 实现了command pattern.
`UseCaseThreadPoolScheduler`实现了`UseCaseScheduler`接口, 定义了use cases执行的线程池, 在后台线程异步执行, 最后把结果返回给主线程.
`UseCaseScheduler`通过构造传给`UseCaseHandler`.
测试中用了`UseCaseScheduler`的另一个实现`TestUseCaseScheduler`, 所有的执行变为同步的.

`Injection`类中提供了多个Use cases的依赖注入, 还有`UseCaseHandler`用来执行use cases.

Presenter的实现中, 多个use cases和`UsseCaseHandler`都由构造传入, 执行动作, 比如更新一个task:
```java
private void updateTask(String title, String description) {
    if (mTaskId == null) {
        throw new RuntimeException("updateTask() was called but task is new.");
    }
    Task newTask = new Task(title, description, mTaskId);
    mUseCaseHandler.execute(mSaveTask, new SaveTask.RequestValues(newTask),
            new UseCase.UseCaseCallback<SaveTask.ResponseValue>() {
                @Override
                public void onSuccess(SaveTask.ResponseValue response) {
                    // After an edit, go back to the list.
                    mAddTaskView.showTasksList();
                }

                @Override
                public void onError() {
                    showSaveError();
                }
            });
}
```


## todo-mvp-dagger
**关键概念**: 
[dagger2](http://google.github.io/dagger/) 是一个静态的编译期依赖注入框架.
这个例子中改用dagger2实现依赖注入. 这样做的主要好处就是在测试的时候我们可以用替代的modules. 这在编译期间通过flavors就可以完成, 或者在运行期间使用一些调试面板来设置.

### Diff with todo-mvp
`Injection`类被删除了.
添加了5个Component, 四个feature各有一个, 另外数据对应一个: `TasksRepositoryComponent`, 这个Component被保存在Application里.

数据的module: `TasksRepositoryModule`在`mock`和`prod`目录下各有一个.

对于每一个feature的Presenter的注入是这样实现的:
首先, 把Presenter的构造函数标记为@Inject, 然后在Activity中构造component并注入到字段:
```java
@Inject AddEditTaskPresenter mAddEditTasksPresenter;
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.addtask_act);
    .....
    
    // Create the presenter
    DaggerAddEditTaskComponent.builder()
            .addEditTaskPresenterModule(
                    new AddEditTaskPresenterModule(addEditTaskFragment, taskId))
            .tasksRepositoryComponent(
                    ((ToDoApplication) getApplication()).getTasksRepositoryComponent()).build()
            .inject(this);
}
```
这个module里provide了view和taskId:
```java
@Module
public class AddEditTaskPresenterModule {

    private final AddEditTaskContract.View mView;

    private String mTaskId;

    public AddEditTaskPresenterModule(AddEditTaskContract.View view, @Nullable String taskId) {
        mView = view;
        mTaskId = taskId;
    }

    @Provides
    AddEditTaskContract.View provideAddEditTaskContractView() {
        return mView;
    }

    @Provides
    @Nullable
    String provideTaskId() {
        return mTaskId;
    }
}
```

注意原来构造方法里调用的setPresenter方法改为用方法注入实现:
```java
/**
 * Method injection is used here to safely reference {@code this} after the object is created.
 * For more information, see Java Concurrency in Practice.
 */
@Inject
void setupListeners() {
    mAddTaskView.setPresenter(this);
}
```

## todo-mvp-contentproviders
这个例子是基于todo-mvp-loaders的, 用content provider来获取repository中的数据.
![mvp-contentproviders](/images/mvp-contentproviders.png)
使用[Content Provider](https://developer.android.com/guide/topics/providers/content-providers.html)的优势是:
- 管理了结构化数据的访问;
- Content Provider是跨进程访问数据的标准接口.

### Diff with todo-mvp-loaders
注意这个例子是唯一一个不基于最基本的todo-mvp, 而是基于todo-mvp-loaders. (但是我觉得也可以认为是直接从todo-mvp转化的.)
看diff: `git difftool -d todo-mvp-loaders`.

去掉了`TaskLoader`和`TasksLoader`. (回归到了基本的todo-mvp).

`TasksRepository`中的方法不是同步方法, 而是异步加callback的形式. (回归到了基本的todo-mvp). 

`TasksLocalDataSource`中的读方法都变成了空实现, 因为Presenter现在可以自动收到数据更新.

新增`LoaderProvider`用来创建Cursor Loaders, 有两个方法:
```java
// 返回特定fiter下或全部的数据
public Loader<Cursor> createFilteredTasksLoader(TaskFilter taskFilter)

// 返回特定id的数据
public Loader<Cursor> createTaskLoader(String taskId) 
```
其中第一个方法的参数`TaskFilter`, 用来指定过滤的selection条件, 也是新增类.


`LoaderManager`和`LoaderProvider`都是由构造传入Presenter, 在回调`onTaskLoaded()`和`onTasksLoaded()`中init loader. 

在`TasksPresenter`中还做了判断, 是init loader还是restart loader:
```java
@Override
public void onTasksLoaded(List<Task> tasks) {
    // we don't care about the result since the CursorLoader will load the data for us
    if (mLoaderManager.getLoader(TASKS_LOADER) == null) {
        mLoaderManager.initLoader(TASKS_LOADER, mCurrentFiltering.getFilterExtras(), this);
    } else {
        mLoaderManager.restartLoader(TASKS_LOADER, mCurrentFiltering.getFilterExtras(), this);
    }
}
``` 
其中initLoader()和restartLoader()时传入的第二个参数是一个bundle, 用来指明过滤类型, 即是带selection条件的数据库查询.

同样是在onLoadFinshed()的时候做View处理, 以`TaskDetailPresenter`为例:
```java
@Override
public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
    if (data != null) {
        if (data.moveToLast()) {
            onDataLoaded(data);
        } else {
            onDataEmpty();
        }
    } else {
        onDataNotAvailable();
    }
}
```
数据类Task中新增了静态方法从Cursor转为Task, 这个方法在Presenter的`onLoadFinished()`和测试中都用到了.
```java
public static Task from(Cursor cursor) {
    String entryId = cursor.getString(cursor.getColumnIndexOrThrow(
            TasksPersistenceContract.TaskEntry.COLUMN_NAME_ENTRY_ID));
    String title = cursor.getString(cursor.getColumnIndexOrThrow(
            TasksPersistenceContract.TaskEntry.COLUMN_NAME_TITLE));
    String description = cursor.getString(cursor.getColumnIndexOrThrow(
            TasksPersistenceContract.TaskEntry.COLUMN_NAME_DESCRIPTION));
    boolean completed = cursor.getInt(cursor.getColumnIndexOrThrow(
            TasksPersistenceContract.TaskEntry.COLUMN_NAME_COMPLETED)) == 1;
    return new Task(title, description, entryId, completed);
}
```

另外一些细节: 
数据库中的内存cache被删了.
Adapter改为继承于`CursorAdapter`.

### 单元测试
新增了`MockCursorProvider`类, 用于在单元测试中提供数据.
其内部类`TaskMockCursor` mock了Cursor数据. 
Presenter的测试中仍然mock了所有构造传入的参数, 然后准备了mock数据, 测试的逻辑主要还是拿到数据后的view操作, 比如:
```java
@Test
public void loadAllTasksFromRepositoryAndLoadIntoView() {
    // When the loader finishes with tasks and filter is set to all
    when(mBundle.getSerializable(TaskFilter.KEY_TASK_FILTER)).thenReturn(TasksFilterType.ALL_TASKS);
    TaskFilter taskFilter = new TaskFilter(mBundle);

    mTasksPresenter.setFiltering(taskFilter);

    mTasksPresenter.onLoadFinished(mock(Loader.class), mAllTasksCursor);

    // Then progress indicator is hidden and all tasks are shown in UI
    verify(mTasksView).setLoadingIndicator(false);
    verify(mTasksView).showTasks(mShowTasksArgumentCaptor.capture());
}
```


## todo-mvp-rxjava
关于这个例子, 之前看过作者的文章: [Android Architecture Patterns Part 2:
Model-View-Presenter](https://upday.github.io/blog/model-view-presenter/), 
这个文章上过Android Weekly Issue #226.

这个例子也是基于todo-mvp, 使用RxJava处理了presenter和数据层之间的通信.

### MVP基本接口改变
BasePresenter接口改为:
```java
public interface BasePresenter {
    void subscribe();
    void unsubscribe();
}
```
View在`onResume()`的时候调用Presenter的`subscribe()`; 在onPause()的时候调用presenter的`unsubscribe()`.

如果View接口的实现不是Fragment或Activity, 而是Android的自定义View, 那么在Android View的`onAttachedToWindow()`和`onDetachedFromWindow()`方法里分别调用这两个方法.

Presenter中保存了:
```java
private CompositeSubscription mSubscriptions;
```
在`subscribe()`的时候, `mSubscriptions.add(subscription);`;
在`unsubscribe()的时候, `mSubscriptions.clear();`.

### Diff with todo-mvp
数据层暴露了RxJava的`Observable`流作为获取数据的方式, `TasksDataSource`接口中的方法变成了这样:
```java
Observable<List<Task>> getTasks();

Observable<Task> getTask(@NonNull String taskId);
```
callback接口被删了, 因为不需要了.

`TasksLocalDataSource`中的实现用了[SqlBrite](https://github.com/square/sqlbrite), 从数据库中查询出来的结果很容易地变成了流:
```java
@Override
public Observable<List<Task>> getTasks() {
    ...
    return mDatabaseHelper.createQuery(TaskEntry.TABLE_NAME, sql)
            .mapToList(mTaskMapperFunction);
}
```

`TasksRepository`中整合了local和remote的data, 最后把`Observable`返回给消费者(Presenters和Unit Tests). 这里用了`.concat()`和`.first()`操作符.


Presenter订阅TasksRepository的Observable, 然后决定View的操作, 而且Presenter也负责线程的调度.
简单的比如`AddEditTaskPresenter`中:
```java
@Override
public void populateTask() {
    if (mTaskId == null) {
        throw new RuntimeException("populateTask() was called but task is new.");
    }
    Subscription subscription = mTasksRepository
            .getTask(mTaskId)
            .subscribeOn(mSchedulerProvider.computation())
            .observeOn(mSchedulerProvider.ui())
            .subscribe(new Observer<Task>() {
                @Override
                public void onCompleted() {

                }

                @Override
                public void onError(Throwable e) {
                    if (mAddTaskView.isActive()) {
                        mAddTaskView.showEmptyTaskError();
                    }
                }

                @Override
                public void onNext(Task task) {
                    if (mAddTaskView.isActive()) {
                        mAddTaskView.setTitle(task.getTitle());
                        mAddTaskView.setDescription(task.getDescription());
                    }
                }
            });

    mSubscriptions.add(subscription);
}
```
`StatisticsPresenter`负责统计数据的显示, `TasksPresenter`负责过滤显示所有数据, 里面的RxJava操作符运用比较多, 可以看到链式操作的特点.

关于线程调度, 定义了`BaseSchedulerProvider`接口, 通过构造函数传给Presenter, 然后实现用`SchedulerProvider`, 测试用`ImmediateSchedulerProvider`. 这样方便测试.




