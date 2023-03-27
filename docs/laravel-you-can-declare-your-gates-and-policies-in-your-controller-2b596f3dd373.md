# 你可以在你的控制器中声明你的门

> 原文：<https://itnext.io/laravel-you-can-declare-your-gates-and-policies-in-your-controller-2b596f3dd373?source=collection_archive---------3----------------------->

## 是时候精简您的 AuthServiceProvider 了

![](img/7b6dbf843875d215ef03cacab06b2aa2.png)

[天枢刘](https://unsplash.com/@tianshu?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍照

如果您不知道的话， [Laravel 包含了授权机制](https://laravel.com/docs/5.8/authorization#writing-gates)来优雅地授权一个经过身份验证的用户执行特定的操作或资源。

提醒一下，这里有一些基础知识: [**盖茨**](https://laravel.com/docs/5.8/authorization#gates) 可以操作任何逻辑，比如授权一个用户查看管理仪表板，如果他是“管理员”的话。同时， [**策略**](https://laravel.com/docs/5.8/authorization#creating-policies) 专门作用于资源的 CRUD 操作:创建、检索、更新和删除。例如，只允许文章作者更新自己的出版物，而拒绝其他用户。

要声明它们，你通常要去你的`AuthServiceProvider`并定义你的应用程序使用的每个门和策略。您可以从数组(对于策略)中完成，如果您有很多数组，甚至可以为门创建自己的数组。否则，如文档所述，手动注册它们是安全的。

在这个例子中，我只是做了一些方便的事情，因为我有很多门和策略。

好的，我们已经定义了我们的政策和入口。如果你看到`$gates`数组，有很多被定义为`see-something`。根据用户的“角色”，他可以看到**仪表板**、**图库**和**系统配置**。这些仅用于特定的控制器。

Welp，我们不需要定义`AuthServiceProvider`中的每个门。虽然建议这样做，因为它允许您集中授权，使其在全球范围内可用，但有时最好在您的控制器中声明它，然后就到此为止，特别是如果它只是一个简单的基于 Closure 的 gate，您只使用一次。

# 是的，把它放在构造函数里就行了

以同样的方式在控制器中声明一个中间件，您可以使用 Gate facade 来定义一个门——是的，听起来很奇怪。当控制器被实例化时，`__construct`方法将在其他任何事情之前运行，并且由于我们正在使用[自动注入](https://laravel.com/docs/5.8/container#automatic-injection)，门将被自动解析，因此我们可以在方法内部使用它并定义我们的门。

然后，在你的方法中，你只是照常进行。

这只是一个简单的例子，但是如果你有多个门，或者你只在一个控制器中使用一个特定的门，就像我在这个例子中所做的那样，它可以让你的`AuthServiceProvider`变得更小。

此外，您可以将授权中间件设置为只处理某些方法，或者排除它们。

```
$this->middleware('can:see-dashboard')->except('otherMethod');
```

如果 Gate 需要可选参数，这种排除就非常方便。由于我们不能在触及方法逻辑之前添加参数，从中间件中排除该方法将允许您在直接使用授权助手时放置参数。

```
public function changeOwner(Request $request, Article $article)
{
    $this->authorize('can-change-owner', $article); // ... Change ownerage of the Article
}
```

# 何时*不*使用这种方法

正如您所猜测的，门位于控制器内部。这意味着门**将只在控制器**内部工作，完全分散。

如果您希望从多个点调用相同的授权逻辑，比如在 vie 中，您可能希望使用经典的方法来定义门，因为它们将在引导时可用，并且在应用程序内部全局可用:控制器、控制台、内部作业、侦听器，只要您能想到的。

这意味着，您也可以使用`Gate::authorizeForUser()`，并手动发布应该(或不应该)被授权执行该特定操作的用户。

在你的应用中不需要过多考虑关卡。