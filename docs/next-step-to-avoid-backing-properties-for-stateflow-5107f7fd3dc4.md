# 避免支持 StateFlow 属性的下一步

> 原文：<https://itnext.io/next-step-to-avoid-backing-properties-for-stateflow-5107f7fd3dc4?source=collection_archive---------2----------------------->

![](img/c524a9e0fd6d3b6128e17d81e6cbeb2b.png)

[https://unsplash.com/photos/Zj8JwP3M3Do](https://unsplash.com/photos/Zj8JwP3M3Do)

本文灵感来源于[这篇原帖](https://medium.com/google-developer-experts/avoid-backing-properties-for-livedata-and-stateflow-706006c9867e)和[这篇回复](/re-avoid-backing-properties-for-livedata-and-stateflow-2160cab96b56)。这里我要讲一下我对提到的问题的观察。

我们可以把问题重新表述为“*让值 getter 在所有地方可见，而值 setter 只在* `*ViewModel*`内部可见”。让我们从这句话开始，尝试使用 Kotlin 的可见性结构来解决它。

# **保护**流的可变性

让 *value getter 在所有地方*可见很容易——它只是`public`(在 Kotlin 中默认)。为了让*让值设置器只在* `*ViewModel*`内部可见，我们需要让它`private`并放在`ViewModel`内部。但是`ViewModel`很多。因此，我们可以创建`class BaseViewModel`，将值 setter 放入该类，并使其成为`protected`。有趣的事情从这里开始。

`MutableStateFlow`中的值设置者是`public`，我们对此无能为力。这就是为什么我们定义`ViewModelStateFlow`来包装“真实的”`StateFlow<T>`:

```
sealed class ViewModelStateFlow<T>(stateFlow: StateFlow<T>) : StateFlow<T> by stateFlow
```

这个类是`public`并且打算在所有地方使用。但是`ViewModelStateFlow`的值设置器将是`class BaseViewModel`中的`protected`，所以它只能在`BaseViewModel`的子类中使用。

现在只剩下为`ViewModelStateFlow`提供子类了。这个实现将在`BaseViewModel.kt`内部`private`以避免其泄露。

```
private class ViewModelStateFlowImpl<T>(
        initial: T,
        val wrapped: MutableStateFlow<T> = *MutableStateFlow*(initial)
) : ViewModelStateFlow<T>(wrapped)
```

在`ViewModelStateFlowImpl`中，我们创建了用于`ViewModelStateFlow`中委托的`MutableStateFlow`。

此外，我们需要在`class BaseViewModel`中提供一个工厂方法来创建`ViewModelStateFlow`。我们不能只使用`ViewModelStateFlowImpl`的构造函数，否则`private`类型的`ViewModelStateFlowImpl`会在`ViewModel`中通过`public val someFlow`暴露出来。

```
protected fun <T> createViewModelStateFlow(initial: T): ViewModelStateFlow<T> = ViewModelStateFlowImpl(initial)
```

下一步是`ViewModelStateFlow`的值设置器。是`class BaseViewModel`内部的`protected`函数知道`ViewModelStateFlowImpl`的实现细节。

```
protected fun <T> ViewModelStateFlow<T>.setValue(value: T): Unit = when (this) {
    is ViewModelStateFlowImpl -> wrapped.value = value
}
```

# 使用视图模型状态流

有了这些定义，我们可以轻松地在任何视图模型中创建状态流，并在那里访问它的值 getter 和 setter:

但是在“视图”类(`SomeActivity`、`SomeFragment`等)中，我们只能访问值 getter:

我们实现了我们想要的。但是总有改进的空间。例如，我们需要使用`setValue`方法，而不是仅仅给 flow 分配一个新值(正如在`MutableStateFlow`中所做的)。到目前为止，我还没有找到为`ViewModelStateFlow`定义属性 setter 的方法。

Btw，C++里有`friend`关键字，在这里会很有帮助。但是，Kotlin 中的这个关键字可能不合适，并会给 Kotlin 的可见性结构带来更多的复杂性。

完整代码概括如下: