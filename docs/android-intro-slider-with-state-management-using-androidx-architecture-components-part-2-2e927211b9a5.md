# Android:使用 Arch 组件进行状态管理的介绍滑块。(第二部分)

> 原文：<https://itnext.io/android-intro-slider-with-state-management-using-androidx-architecture-components-part-2-2e927211b9a5?source=collection_archive---------2----------------------->

这是第 1 部分的延续，在第 1 部分中，我们已经熟悉了一些架构组件，并使用`ViewPager`和`Fragments`实现了基本的 intro slider。

如果你想直接进入，你可以简单地在 GitHub 上检查[***part-2-base***分支。](https://github.com/xzhorikx/slider-intro/tree/part-2-base)

![](img/590e5e33e04596d44045751b4a9251aa.png)

简介滑块结构的高级概述

这一部分将使用`[LiveData](https://developer.android.com/topic/libraries/architecture/livedata)`和`[Transformations](https://developer.android.com/reference/androidx/lifecycle/Transformations)`来仔细观察状态管理。让我们看看在这一部分结束时将会实现什么:

简介滑块的最终结果

让我们分析当用户滑动片段时，屏幕上发生了什么变化:

*   背景颜色
*   中央导航点
*   文本元素的可见性

换句话说，每个部分都有一个****状态**，这取决于在任何给定时间的`ViewPager` **转换** **状态**。但是我们如何知道*过渡状态*？幸运的是`ViewPager`有`[OnPageChangeListene](https://developer.android.com/reference/kotlin/androidx/viewpager/widget/ViewPager.OnPageChangeListener)r`允许在每次页面改变时接收回调。每当过渡正在进行时，*onpagesulved(position:Int，positionOffset: Float，positionOffsetPixels: Int)* 被调用，我们对 *position* 和 *positionOffset* 的值感兴趣。**

> ****快速提示:了解页面过渡是如何表现的****
> 
> **如果我们有 N 个片段，那么**位置**的值将是一个在[0，N - 1]范围内的整数。**
> 
> ****位置偏移**始终是一个在[0，1]范围内的浮点数**
> 
> ****位置及其偏移量的总和**代表一个**过渡状态**。**
> 
> **因此，如果位置 1 上的片段滚动 50%到位置 2 上的片段，转换状态等于 1.5000:相对于位置 1 的偏移量是 0.5000。如果位置 1 上的片段被滚动 50%到片段位置 0，则转换状态等于 0.5000:相对于位置 0 的偏移是 0.5000。**

**`ViewPager`转换的状态可以用一个字段来表示:位置和它的偏移量。让我们创建一个新的`ViewHolderScrollState`类来表示滑块中的转换状态:**

**过渡状态的类**

**现在是时候创建包含转换状态的 LiveData 了，这样我们可以在以后将这个状态转换成有用的东西，比如计算的背景颜色或文本元素的可见性。导航到`MainAppActivityViewModel`并添加以下代码:**

**将转换状态保存到视图模型**

> **注意:我们预先知道 mViewHolderScrollStateLiveData 不会在 Activity 中被观察到，这就是为什么它有一个' private '修饰符和一个 public setter。**

**主要活动也需要修改:**

*   **`ViewPager`必须有一个页面更改监听器**
*   **每次转换进行时，页面更改监听器必须更新转换状态**

**将页面更改侦听器添加到视图寻呼机**

**我们已成功将页面更改监听器添加到`ViewPager`！从现在开始，每次转换开始时，其状态(*位置和其偏移量*)保存在`mViewHolderScrollStateLiveData`中。**

# ****让我们动态改变背景颜色****！****

**我们需要保持事情简单，所以我们的活动只需要知道一件事——设置什么背景颜色。它不应该知道这个颜色是如何计算的，也不应该知道它的值取决于什么。`LiveData`非常适合这种情况:**

**向 MainActivity 添加背景色 LiveData**

**注意，出于美观的原因，我们也改变了状态栏的颜色。到目前为止,`ViewModel`还没有`backgroundColorLiveData`,我们将很快创建它，但首先让我们看看它背后的想法:**

*   **每个片段类型都有自己的背景颜色**
*   **我们知道当前的转换状态，这意味着我们知道两个片段的位置，在这两个片段之间发生转换**
*   **根据位置偏移([0，1]范围中的*值)，我们可以在两种颜色之间进行插值，其中一部分由偏移表示***

**[*ArgbEvaluator*](https://developer.android.com/reference/android/animation/ArgbEvaluator)*。evaluate()*派上了用场，因为它能够做到这一点！From docs: *该函数返回给定整数和给定分数的颜色的中间值[…]* 。**

```
ArgbEvaluator.evaluate(offset, color1, color2)
```

**此时，我们一切就绪:*偏移量*从*viewmolderscrollstate 中已知，颜色 1* 和*颜色 2* 可以从片段中获得，这些片段的位置也是从*viewmolderscrollstate*中计算出来的。**

**我们如何在视图模型中获得当前的`ViewHolderScrollState`?答: [**LiveData 转换**](https://developer.android.com/reference/androidx/lifecycle/Transformations) 。简单来说，变换允许观察其他`LiveData`的变化，并将其值映射到所需的对象。**

**视图保持器状态到背景颜色的转换**

**从现在开始，每次`mViewHolderScrollStateLiveData`中的值被改变，它都会将新的`ViewHolderScrollState` 值传递给`backgroundColorLiveData` 转换函数。知道了过渡态，就有可能得到两个片段的位置、偏移量和颜色。**

**检查点:运行应用程序，你将会看到变化的魔力！**

# **是时候创建导航点了！**

**用户在屏幕间导航时的点动画**

**希望此时你已经对`LiveData`和`Transforamtions`如何协同工作有了基本的了解，因为我们将再次使用这种组合！让我们定义一些需要考虑的要点:**

*   **一旦知道需要在`ViewPager`中显示多少片段，就必须动态生成点**
*   **点数等于滑动屏幕的数量**
*   **过渡只发生在两个片段之间。这意味着**只需要在任何给定的时间制作两个点的动画**，所有其他的必须保持不变**
*   **在过渡期间改变的属性: *alpha* 和 *scale***
*   **每个点都有一个定义其在屏幕上外观的状态**

****定义导航点的状态****

**每个点在任何给定时间都有一个状态，在我们的情况下，该状态应该定义*位置*、*α*和*比例*值。在我看来，最好在一个单独的类中显式定义这样的状态，因为这种方法是可伸缩的，并且易于维护。让我们创建`NavigationDotState`类**

**定义导航点视图状态的类**

****动态生成导航点****

**为了生成点，我们需要知道创建视图的数量。我们还需要保存对已创建视图的引用，以便以后操作它们的属性。考虑到这一点，我们可以在`MainActivity`中创建一个助手`initNavigationDots`函数，手动创建视图并将它们添加到`containerIntroDots`线性布局中。**

**将导航点初始化为视图并将它们添加到线性布局容器的函数**

**我们一收到要显示的片段列表就需要调用这个函数，所以最好是调用`fragmentTypeListLiveData`观察者**

**调用导航点初始化**

**就是这样！现在，每当我们收到片段列表时，我们将在主活动中显示相同数量的导航点。**

****检查点:**运行您的应用程序，查看导航点是否正确显示在屏幕底部。**

****向导航点添加动画****

**我们将从顶部开始——为了在任何给定时间显示导航点，activity 需要知道什么？答:列出导航点状态和视图引用。通过在主活动中观察`LiveData`来实现这种方法非常简单——一旦接收到具有新状态的列表，我们就在*位置*获得视图，并对其应用 *alpha* 和 *scale* 值。**

**观察导航点的状态并将其应用于视图**

**就这么简单。现在是时候到`navigationDotStateLiveData`的幕后看看导航点状态是如何定义的了。转到`MainAppActivityViewModel`并创建以下字段**

```
val navigationDotStateLiveData: LiveData<List<NavigationDotState>?>
```

**我们来思考一下，改变导航点的状态需要考虑什么:**

1.  **点的状态取决于`mViewHolderScrollStateLiveData`中的转换状态。如果没有过渡状态，则返回默认导航点状态。**
2.  **在任何给定的过渡时间，只有 2 个点可以被激活。这意味着我们需要知道需要改变状态的点的位置**

**第一步，需要将`Transformation`加到`mViewHolerScrollStateLiveData`上，并检查当前值是否不为空。这将为我们初始化`List<NavigationState>`打下基础**

**将过渡状态转换成具有导航点状态列表**

**至于第二步，需要找到参与转换的两个点的位置。会涉及到一些数学知识，所以要做好准备！**

**首先，我们需要从`offsetSum`开始计算碎片的位置及其偏移值。这样我们就知道哪两个片段参与了转换，以及转换完成的百分比(偏移量)。每个片段可以处于以下状态之一:**

1.  **片段占据整个屏幕，意味着没有过渡。**
2.  **用户正在向片段导航。它对用户来说是完全不可见的，现在开始从屏幕的任何一部分出现。**
3.  **用户正在离开片段。它占据了整个屏幕，现在正被移动到屏幕的任何一部分。**
4.  **片段不参与转换，这意味着它对用户是不可见的**

> ****例题****
> 
> **1.用户从位置 0 的片段向片段 1 滑动了 30%。`*offsetSum*`将会是`*0.30*`。意味着**片段 0** 向**片段 1** 移动了 **30%** 。**片段 2** 不参与转场。**
> 
> **2.用户从位置 2 的片段刷回到位置 1 75%。在这种情况下，`*offsetSum*`将会是`*1.25*`。这意味着**片段 1** 向**片段 2** 移动了 25%。**片段 0** 不参与转场。**

**知道了`offsetSum`的值和片段的位置，我们就可以计算它的导航点的可见性和比例。我们可以迭代`fragmentTypeList`来计算每个片段的状态。**

**计算每个片段的状态**

**从简单的部分开始，我们可以肯定地说，当 fragment 完全可见时，那么它的导航点应该有 max *alpha* 和 *scale* ，当 fragment 不参与过渡时则相反。**

```
when(offsetSum - fragmentIndex){
    0.0f -> {
        // Dot of the fragment that occupies the whole screen  
        newAlpha = NavigationDotState.ALPHA_MAX
        newScale = NavigationDotState.SCALE_MAX
    },
    ...
    else -> {
        // Fragment does not participate in transition
        newAlpha = NavigationDotState.ALPHA_MIN
        newScale = NavigationDotState.SCALE_MIN
    }
}
```

**现在有必要引入两个新的属性:`alphaDelta`和`scaleDelta`。简单来说我们假设`ALPHA_MAX`、`SCALE_MAX`、`ALPHA_MIN`和`SCALE_MIN`不会分别高于`1.0`和低于`0.0`，deltas 就是最大值和最小值之差。**

```
val alphaDelta = ALPHA_MAX - ALPHA_MIN
val scaleDelta = SCALE_MAX - SCALE_MIN
```

**`offset`的范围为`[0.0, 1.0]`，可以作为过渡完成的百分比。因此，现在我们已经做好了计算所需规模和可见性的一切准备。**

**导航点的比例和可见度状态计算的完整代码**

**所有代码就绪后，让我们看看如何计算我们的`navigationStateLiveData`值:**

**航行状态计算的最终代码**

****检查点**:运行你的应用程序来看看导航点的动画效果！**

**导航点动画的最终结果**

# ****最后一章:动态导航文本可见性****

**首先，让我们定义文本元素的可见性规则:**

*   ***跳过*和*下一个*按钮应在除最后一个屏幕之外的所有屏幕上可见**
*   ***完成*按钮应该只在最后一个屏幕上可见**
*   **过渡期间文本元素的可见性取决于*偏移***

**因为我们已经预定义了按钮的数量，所以让我们在单个`NavigationTextState`类中定义它们的可见性**

**保持导航文本元素可见性状态的类**

**主活动应该使用`LiveData`接收状态，并对相应的元素应用可见性**

**根据状态设置导航文本元素的可见性**

**同样，`navigationTextStateLiveData`依赖于`mViewHolderScrollStateLiveData`中的状态，这意味着我们需要应用`Transformations.map(...){ ... }`功能。我们需要检查的事项:**

1.  **如果`viewHolderScrollState`为空，*跳过*和*下一个*按钮应该可见，*完成*应该不可见。**
2.  **如果显示最后一个片段(`floor(offsetSum) == (fragmentTypeList.size - 1)`，那么*跳过*和*下一个*按钮应该不可见，*结束*应该可见。**
3.  **如果`(fragmentTypeList.size - 1) - offsetSum`在`[0.0, 1.0]`的范围内，那么到/从最后一个片段的过渡正在进行。**
4.  **任何其他情况下，*跳过*和*下一个*按钮应该可见，*结束*应该不可见。**

**是时候将我们的知识转化为代码了！**

**导航文本状态的计算**

> ****注意**:视图类中的 getAlpha()和 getVisibility()互不依赖。这意味着即使 getAlpha()可能返回 0.0，如果它的 getVisibility()返回 View.VISIBLE，视图仍然会呈现在屏幕上。**
> 
> **如果视图的 alpha = 0.0，可见性= View。VISIBLE 呈现在任何其他元素之上，那么它将阻止 click listeners 在底层元素中触发。**

**为了防止文本元素彼此“重叠”,有必要改变`navigationTextStateLiveData`观察点——一旦 alpha 低于或高于某个阈值，相应地改变元素的可见性。这就是`navigationTextStateLiveData`观察者现在的样子**

**根据 alpha 值改变文本元素的可见性**

## ****向文本导航元素添加点击监听器****

**一旦在主活动中收到片段列表，就必须初始化文本点击监听器。让我们在主活动中创建`initListeners`函数来初始化点击监听器。`IntroFinishedActivity`此时还没有创建，所以你可以简单地[从 GitHub](https://github.com/xzhorikx/slider-intro/blob/master/app/src/main/java/alexz/sliderintro/activity/IntroFinishedActivity.kt) 获取它。别忘了在`AndroidManifest.xml`申报**

**导航按钮点击监听器的初始化**

**最后，从`fragmentTypeListLiveData`观察器调用`initListeners`**

**从片段实时数据观察器调用点击监听器初始化**

# **结束🎇**

**恭喜你！您已经使用架构组件实现了一个非常棒的可伸缩的 intro slider，并且可能学到了一些关于状态表示和管理的新知识！**

**最终简介滑块结果**

**源代码可在 [GitHub](https://github.com/xzhorikx/slider-intro) 上获得。**