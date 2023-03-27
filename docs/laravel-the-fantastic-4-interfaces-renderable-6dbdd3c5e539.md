# Laravel:奇妙的 4 个界面— Htmlable

> 原文：<https://itnext.io/laravel-the-fantastic-4-interfaces-renderable-6dbdd3c5e539?source=collection_archive---------4----------------------->

## 学习响应代码，一次一个实现。

![](img/ae6eda4c67954cc3e512fb77187bac35.png)

照片由 [Pankaj Patel](https://unsplash.com/@pankajpatel?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

该归功于谁。去年我偶然发现了 Josip Crnkovi 的一篇文章，他在文章中介绍了这个框架的一些有用的接口。在其中，他*发现了*一些用于向浏览器发送响应的消息。

在尝试之后，我不得不说这些减轻了很多枯燥的问题，并且将多行代码精简为几行。在这一系列文章中，我将检查它们，因为它们可以帮助您更容易地编写应用程序。

# html 表格

庆幸吧，这是最有用的！推动这个接口的方法叫做`toHtml()`，它负责将类表示为一个 HTML 字符串。

当刀片模板引擎检测到双花括号内的 Htmlable 实例时，就会调用这个方法——你知道，就是那个`{{ }}`的东西。当找到它时，它将继续使用`toHtml()`方法，而不是试图将它转换为字符串。

这对于可以用几行代码呈现为 HTML 的类很有用。例如，[我的 Laralerts 包使用 Htmlable 接口](https://github.com/DarkGhostHunter/Laralerts/blob/master/src/AlertsHtml.php#L37-L53)将每个警报呈现为带有警报消息的 HTML 字符串，将逻辑放在一个地方，而不是在每个警报中调用视图工厂。

这对分页链接也很有用:当你使用类似于`$pagination->links()`的东西时，它会返回一个`AbstractPaginator`的实例，而[会将这个实例转换成一个 HTML 字符串](https://github.com/laravel/framework/blob/5.8/src/Illuminate/Pagination/AbstractPaginator.php#L628-L636)。

# 示例:渲染参与者

假设我们需要将参与者模型的每个实例转换为 HTML，这些实例是播客中出现的人的个人资料。我们可以很容易地返回一个简单的字符串，或者更好地呈现一个视图。

现在我们把它调用到我们的视图中，一旦它被渲染，我们就看着它变魔术。

请注意，这与可渲染不同。如果你单独推一个类，将会抛出一个错误，除非这个类作为最后一个资源有一个`_toString()`方法，它应该镜像`toHtml()`方法。