# 使用 Kotlin 去除片段/活动样板代码

> 原文：<https://itnext.io/get-rid-of-fragment-activity-boilerplate-code-using-kotlin-1b103763baf8?source=collection_archive---------4----------------------->

今天，我将向您展示一种方法来删除片段和活动中最常见的样板代码，尤其是在使用 MVVM 架构时。

![](img/303760cd4178898fa9e3c0168bf0f75b.png)

声明片段/活动的当前状态:充斥着如下内容:oncreate view | binding = DatabindingUtil…| inflater . inflate()| binding . life cycle owner =…| viewModel =…

# 为什么？

Android 团队最近引入了一个变化，允许在片段的构造函数中直接传递布局资源。这消除了重写 onCreateView 方法的需要，在不使用数据绑定时节省了大约 3 行代码。这也给了我灵感，让我创作出一些去除更多样板文件的东西。忘记每次都做同样的数据绑定和膨胀。

# 我为什么要在乎？

当您使用 MVVM 和数据绑定时，该解决方案将从每个活动/片段中删除大约 10 行代码，但是即使您不像我一样做每件事，您仍然会有很多好处。
10 行可能看起来不多，但相信我，这真的很重要。有了 10 个片段，就节省了 100 行代码，在非常简单的片段中，你实际上只会看到重要的东西，而不是 2/3 的样板文件。

# 我们将去掉哪些部分？

*   膨胀布局资源
*   声明 ViewModel 成员变量并设置它
*   声明绑定变量并设置它
*   将绑定与视图模型连接起来

# 示例(类似于活动)

这门课:

用样板文件上课

会是这样的:

类，您可以随意调用它。

# BaseFragment 类看起来怎么样？

# **注意事项:**

1.  **绑定**和**视图模型**是继承的，不需要在子类中声明它们，但是**绑定**可能为空(不过这并不重要，因为**绑定**主要用于 BaseFragment 中的内容，你根本不可能在子类中使用该变量)。另外:你不需要像以前一样知道绑定的类别，甚至可以在 Android Studio 中修复一些偶然的红色标记。
2.  不幸的是，目前在 Kotlin 中无法将 ViewModel 作为泛型和类来传递。[kot Lin 论坛上的问题](https://discuss.kotlinlang.org/t/avoid-passing-generic-and-kclass-of-same-class-when-inheriting/16217/2)
    如果没有通用 VM，我们将无法访问子类中 ViewModel 的特定字段&方法。获取 ViewModel 实例需要该类。**这里我们使用 Koin 的 getViewModel()方法，但是如果您愿意，也可以使用 ViewModelProviders。**
3.  我们不能在第 19 行使用 binding.setViewModel，因为我们需要一个通用的方法，但是 setVariable 也可以。唯一的限制是变量**在 xml 中必须精确地命名为 viewModel，**，但是您可以选择任何您喜欢的名称。对于不同的 xml 文件，不需要有不同的视图模型名称，所以我只留下视图模型或者类似 vm 的东西。对于扩展该类的片段/活动，该名称将是相同的。
4.  这个课程对于活动来说是非常相似的，所以我将省略它
5.  您可以立即在构造函数中看到布局资源和视图模型，无需在类中搜索它们。

# 结论

在 xml 中使用相同的 ViewModel 名称来交换数百行代码，并且在继承时必须指定 ViewModel 类两次……对我来说似乎不是一件坏事。

请告诉我你是否认为有任何可以改进的地方，无论是代码还是这篇文章。
这是我的第一个媒体帖子，我相信在:D 之后我会继续编辑它

祝你愉快