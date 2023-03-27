# 关于:避免支持 LiveData 和 StateFlow 的属性

> 原文：<https://itnext.io/re-avoid-backing-properties-for-livedata-and-stateflow-2160cab96b56?source=collection_archive---------2----------------------->

最近我偶然发现了 Danny Preussler 的这篇文章，这让我思考，如果利用 Kotlin 的特性，是否有更好的方法来避免背景字段。

我想到了一个主意，虽然我不认为后台字段是如此邪恶，以至于需要不惜一切代价避免它们，但对我来说，围绕这个主题玩一会儿还是很有趣的。虽然我不一定推荐使用这种避免支持字段的方式，但展示这种想法并在 Kotlin 中展示一些鲜为人知的技巧可能会很有趣。

读完这篇文章后，我的第一个想法是，是否有任何方法可以利用 Kotlin 中的`Delegate`模式来避免后台字段。我受到了 Jetpack Compose 中的`mutableStateOf()`的启发，它在`ViewModel`中启用了这种语法:

```
var someState by mutableStateOf(1)
  private set
```

这很棒，因为它只允许在`ViewModel`中改变状态，视图层没有修改状态的选项，只能看到最新的值。

我们能不能利用这种方法为`LiveData<T>`或`StateFlow<T>`避免写后台字段？

事实证明这并不容易。为了实现这种功能，Jetpack compose 在自己的定制类型`MutableState<T>`和`State<T>`上使用扩展函数，如下所示:

```
@Suppress(**"NOTHING_TO_INLINE"**)
**inline operator fun** <T> State<T>.getValue(thisObj: Any?, property: KProperty<*>): T = **value**@Suppress(**"NOTHING_TO_INLINE"**)
**inline operator fun** <T> MutableState<T>.setValue(thisObj: Any?, property: KProperty<*>, value: T) {
    **this**.**value** = value
}
```

*注意:我不知道在 Kotlin 中可以用这种方式覆盖 setter 和 getter！这是通过浏览 Jetpack Compose 代码库学到的很好的一课。*

然而问题是，访问`someState`值直接返回值，而不是`MutableState<T>`的实例，也不是`State`。在我们的例子中，如果我们想这样包装`LiveData<T>`,无论如何都不可能观察到它，因为它是不可访问的。

那么这种语法是不可能存在的吗？

嗯，有一些变通办法。

可以创建一个委托，为`LiveData<T>`设置一个 setter 和 getter，这将允许在 ViewModel 中限制`set`为`private`。

乍一看，这可能不是很有帮助，因为我们不想一直覆盖保存状态的`LiveData`实例。但是无论如何我们可以利用它:

```
fun <T> liveDataDelegate(defaultValue: T): ReadWriteProperty<Any, LiveData<T>> = object : ReadWriteProperty<Any, LiveData<T>> {
    private val liveData: MutableLiveData<T> = MutableLiveData(defaultValue)

    override fun getValue(thisRef: Any, property: KProperty<*>): LiveData<T> = liveData

    override fun setValue(thisRef: Any, property: KProperty<*>, value: LiveData<T>) {
        liveData.postValue(value.value!!)
    }
}class MyViewModel : ViewModel() {
    var state by *liveDataDelegate*(1)
        private set fun updateState(newValue: Int) {
        state = MutableLiveData(newValue)
    }
}
```

我们使用`ReadWriteProperty<>`委托，它使用`MutableLiveData`的实例作为后台字段，当我们想要向它提交新值时，我们将新值包装到`MutableLiveData`的新实例中，而`ReadWriteProperty`委托打开该值并将其提交到其底层的`MutableLiveData`属性中。

这个委托的 Getter 返回`MutableLiveData`的底层实例，但是只将其公开为`LiveData`。这样仍然可以观察到`LiveData`，但仅限于对`MyViewModel`进行修改。

好的，但是这条线`state = MutableLiveData(newValue)`有点难看，所以我们能做得稍微好一点吗？

我们可以利用扩展函数来获得稍微简洁的语法，最终得到这个最终解决方案:

```
fun <T> liveDataDelegate(defaultValue: T): ReadWriteProperty<Any, LiveData<T>> = object : ReadWriteProperty<Any, LiveData<T>> {
    private val liveData: MutableLiveData<T> = MutableLiveData(defaultValue)

    override fun getValue(thisRef: Any, property: KProperty<*>): LiveData<T> = liveData

    override fun setValue(thisRef: Any, property: KProperty<*>, value: LiveData<T>) {
        liveData.postValue((value as WrapLiveData<T>).startValue)
    }
}

private class WrapLiveData<T>(val startValue: T): LiveData<T>(startValue)
fun <T> T.post(): LiveData<T> = WrapLiveData(this)class MyViewModel : ViewModel() {
    var state by *liveDataDelegate*(1)
        private setfun updateState(newValue: Int) {
        state = newValue.post()
    }
}
```

## 结束语

我不太相信到处都写`state = newValue.post()`比有一个支持字段更方便。然而，至少在我看来，这可能比实现一个`ViewModel`的接口要稍微方便一些。也许这一点我也错了。:-)我只是想提出解决这个问题的另一种方法。与此同时，就我个人而言，我很高兴使用 backing field，但我很高兴有这个机会与代表们一起玩，作为解决这个问题的另一种方法，并在这个过程中学习一些新的东西。