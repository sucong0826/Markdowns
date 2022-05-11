# ViewModel指南

`ViewModel`类旨在以注重生命周期的方式存储和管理界面相关的数据。`ViewModel`类让数据可在发生屏幕旋转等配置更改后继续保留。



### 实现ViewModel

架构组件为界面控制器提供了`ViewModel`辅助程序类，<u>该类负责为界面准备数据</u>。在配置更改期间会自动保留`ViewModel`对象，以便它们存储的数据立即可供下一个activity或fragment实例使用。

```kotlin
class MyViewModel : ViewModel() {
	private val users: MutableLiveData<List<User>> by lazy {
        MutableLiveData<List<User>>().also {
            loadUsers()
        }
    }
    
    fun getUsers(): LiveData<List<User>> {
        return users
    }
    
    private fun loadUsers() {
        // do an asynchronous operation to fetch users
    }
}
```

如果重新创建了Activity，它接收的`MyViewModel`实例与第一个Activity创建的实例相同。当所有者Activity完成时，框架会调用`ViewModel`对象的`onCleared()`，以便它可以清理资源。

```kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val model: MyViewModel by viewModels()
        model.getUsers().observe(this, Observer<List<User>> { users->
       		// update UI     
        })
    }
}
```

> ViewModel绝不能引用视图、Lifecycle或可能存储对Activity上下文的引用的任何类。同时，`ViewModel`对象绝对不能观察对生命周期感知型可观察对象（如`LiveData`对象）的更改。



### 在Fragment之间共享数据

`Activity`中的两个或更多`Fragment`需要相互通讯是一种很常见的现象。可以使用`ViewModel`对象解决这一常见的难点。这两个fragment可以使用其activity范围共享`ViewModel`来处理此类通讯。

```kotlin
class SharedViewModel : ViewModel() {
    val selected = MutableLiveData<Item>()
    
    fun select(item: Item) {
        selected.value = item
    }
}

class ListFragment : Fragment() {
    private lateinit var itemSelector: Selector
    private val model: SharedViewModel by activityViewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        itemSelector.setOnClickListener { item ->
            // update the UI
        }
    }
}

class DetailFragment : Fragment() {
    private val mode: SharedViewModel by activityViewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        model.selected.observe(viewLifecycleOwner, Observer<Item> {
            // update UI
        })
    }
}
```

请注意，这两个Fragment都会检索包含它们的Activity。这样，当这两个Fragment各自获取`ViewModelProvider`时，它们会收到相同的`SharedViewModel`实例（其范围限定为该Activity）。

此方式具有以下优势：

1. Activity不需要执行任何操作，也不需要对此通信有任何了解。
2. 除了`SharedViewModel`约定之外，`Fragment`不需要相互了解。如果其中一个Fragment消失，另一个Fragment将继续照样工作。
3. 每个Fragment都有自己的生命周期，而不受另一个Fragment的生命周期的影响。如果一个Fragment替换另一个Fragment，界面将继续工作而没有任何问题。