# Transformation使用

`distinctUtilChanged`

`public static LiveData<X> distinctUntilChanged(LiveData<X> source)`

此函数会创建一个新的`LiveData`对象，直到`LiveData`对象的值发生变化前，这个对象不会发出任何通知。**变化的依据是`equals()`函数返回`false`**。

```kotlin
// Repo
object EasyRepo {
    var counterModel = CounterModel(0, "")
}

// ViewModel
class EasyViewModel(val repo: EasyRepo) : ViewModel() {
    val counterLd = MutableLiveData<CounterModel>()
    val counterCopyLd = counterLd.distinctUntilChanged()
    // Transformations.distinctUntilChanged(counterLd)
}

// UI
class EasyFragemnt : Fragment() {
    ...
    fun onCreateView(...) {
        viewModel.counterCopyLd.observe(viewLifecycleOwner) {
            // may udpate UI
        }
    }
}
```



`map`

`public static LiveData<Y> map(LiveData<X> source, Function<X, Y> mapFunc)`

此函数会返回一个`LiveData`，通过对源上设置的每个值应用mapFunction并返回一个对应的`LiveData`。

此函数与`Observable.map(Function)`类似。`transform`将在**主线程**上执行。

```kotlin
// ViewModel
class EasyViewModel(val repo: EasyRepo) : ViewModel() {
   	val userLd = MutableLiveData<User>()
    val userNameLd = Transformations.map(userLd) { itUser ->
        "${itUser.firstName} ${itUser.secondName}"
    }
}

// UI
class EasyFragemnt : Fragment() {
    ...
    fun onCreateView(...) {
        viewModel.userNameLd.observe(viewLifecycleOwner) {
            Log.i(TAG, "al-fragment-onUserNameChanged() value=$it")
        }
    }
}

```

问题：`userLd`的值发生了变化，`userNameLd`是如何感知到变化并且处理回调的？

> 1. 当`LiveData`的Observer所关联的LifecycleOwner从非活跃状态转为活跃状态的时候，会检查自己绑定`LiveData`，当对应的`LiveData`发现有对应的Owner激活，就会触发`LiveData`的`onActive()`函数。
> 2. `onActive`开始执行激活对应的操作，之前添加过的Sources会开始遍历自己保存的`LiveData`，并执行对应Source中的`plug()`函数。
> 3. `plug`函数中会调用`observeForever`永久观察source的变化，如果有变化发生，会调用自己的`onChanged()`函数。



`swtichMap`

`public static LiveData<Y> switchMap(LiveData<X> source, Function<X, LiveData<T>> switchMapFunction)`

通过对源上设置的每个值应用`switchMapFunction`，返回从输入源`LiveData`映射的`LiveData`。

返回的`LiveData`委托给通过调用`switchMapFunction`创建的最近的`LiveData`，并将最近的值设置为`source`，而不改变引用。通过这种方式，`switchMapFunction`可以对任何注册到`switchMap（）`返回的`LiveData`的观察者透明的更改后备的`LiveData`。

注意：当后备的`LiveData`被切换，后续从前面的`LiveData`更新的值不会被设置到输出的`LiveData`上。此方法和`Observable.switchMap(Function)`比较类似。

`switchMapFunction`将会在主线程上执行。

> 暂时还没有太弄懂

```kotlin
val cakeEaterLd = MutableLiveData<User>()
val cakeEaterMapLd = Transformations.switchMap(cakeEaterLd) {
    input -> MutableLiveData("${input?.firstName} ${input?.secondName}")
}
```



### 补充：什么是`MediatorLiveData`

`LiveData`的子类，它可能会监听其他的`LiveData`对象并且对从这些`LiveData`对象通知过来的`OnChanged`事件做出响应。该类将正确地将其活动/非活动状态向下传播到源`LiveData`对象。

考虑如下的场景：假设有两个`LiveData`的实例，将它们命名为`LiveDta1`和`LiveData2`，然后我们希望将它们的触发更新合并到一个`liveDataMerger`对象中去。然后，`liveData1`和`liveData2`将会变成`MediatorLiveData liveDataMerger`的源，每次`LiveData1 liveData2` `onChanged`回调被调用，我们将会设置一个新的值到`liveDataMerger`里。

```java
LiveData liveData1 = ...;
LiveData liveData2 = ...;

MediatorLiveData liveDataMerger = new MediatorLiveData<>();
liveDataMerger.addSource(liveData1, value -> liveDataMerger.setValue(value));
liveDataMerger.addSource(liveData2, value -> liveDataMerger.setValue(value));
```

简单的来说就是把几个单独的`LiveData`合并到同一个`MediatorLiveData`中，当单独的`LiveData`收到更新后，`MediatorLiveData`同样会收到更新的通知。