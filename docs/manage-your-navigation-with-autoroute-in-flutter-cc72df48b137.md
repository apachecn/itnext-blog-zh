# 自动路由颤振导航

> 原文：<https://itnext.io/manage-your-navigation-with-autoroute-in-flutter-cc72df48b137?source=collection_archive---------0----------------------->

## 在 Flutter 中使用 AutoRoute 管理您的导航

![](img/6a72639fec640aa2ecb77c99984cd040.png)

导航对于任何应用都是非常重要的，在社区的帮助下，Flutter 有很多解决方案。随着 Navigator 2.0 的推出，导航解决方案也发生了变化。今天，我将谈论其中一个解决方案 AutoRoute…

软件包的创建者是这样向我们解释 [auto_route](https://pub.dev/packages/auto_route) 的；

> 什么是自动路由？
> 
> 这是一个 Flutter 导航包，它允许强类型参数传递，轻松的深度链接，并使用代码生成来简化路线设置，也就是说，它需要最少的代码来生成应用程序内导航所需的一切。
> 
> 为什么选择自动路由？
> 
> 如果您的应用程序需要深度链接或保护路由，或者只是一个干净的路由设置，您将需要使用命名/生成的路由，并且您将最终为中介参数类编写大量样板代码，检查所需的参数，提取参数，以及一堆其他东西。AutoRoute 为您做了所有这些，甚至更多。

在本文中，我们将看到如何在我们的项目中实现[自动路线](https://pub.dev/packages/auto_route)。最后，我将给出一个工作示例。事不宜迟，我们开始吧。

## 入门指南

*   让我们从添加 [auto_route](https://pub.dev/packages/auto_route) 到我们的项目开始。

```
dependencies:        
  auto_route: [latest-version]        

dev_dependencies:        
  auto_route_generator: [latest-version]        
  build_runner: [latest-version]
```

*   然后，让我们像这样为我们的路线创建一个路线页面。

*   您可以使用 replaceInRouteName 参数将自动生成的路由名称从 ProducDetailsPageRoute 缩短为 ProductDetailsRoute。
*   现在我们只需要运行发电机。您可以使用[watch]标志来监视文件系统的编辑和必要的重建。

```
flutter packages pub run build_runner watch
```

*   如果您只想运行一次并退出，请在项目终端中运行这段代码。

```
flutter packages pub run build_runner build
```

*   运行我们的发电机后，只剩下一件事要做。让我们转到我们的`main.dart`文件，像这样修改我们的 MaterialApp。

这样，我们成功地创建了我们的路线系统。对于导航，我们有不同的方法来导航到我们想要的页面。这里有一些如何根据我们的需要导航的例子。

# 奖金

## 传递参数

那是最好的部分！！AutoRoute 自动检测并处理您的页面参数，生成的 Route 对象将提供您的页面需要的所有参数，包括路径/查询参数。

```
**class** **ProductDetailsPage** **extends** **StatelessWidget** {        
 **const** ProductDetailsPage({**required** **this**.product});        

  **final** Product product;     
  ...
```

在导航时，你可以像这样简单地添加。

```
router.push(ProductDetailsRoute(product: product));
```

## 返回结果

您可以使用 pop 完成器返回结果。

```
**var** result = **await** router.push(LoginRoute());
```

然后在你的`LoginPage`里面弹出结果。

```
router.pop(**true**);
```

但是你可以看到，这种方法是动态的，未来可能会有风险。为了避免使用动态值，我们可以使用这个方法来指定。

```
AutoRoute<bool>(page: LoginPage),
```

然后当我们导航时，我们可以指定我们期望的结果类型。

```
**var** result = **await** router.push<bool>(LoginRoute());
```

最后，我们用相同的类型。

```
router.pop<bool>(**true**);
```

## 无上下文导航

要在没有上下文的情况下导航，您可以简单地将生成的路由器赋给一个全局变量。但是不建议使用全局变量，这被认为是一种不好的做法，大多数时候应该使用依赖注入。可以使用 [get_it](https://pub.dev/packages/get_it) 进行依赖注入。下面是怎么做的。

## 路径参数(动态线段)

您可以通过在动态段前面加上冒号来定义动态段。

```
AutoRoute(path: '/product/:id', page: ProductDetailsPage),
```

## 重定向路径

可以使用`redirectTo`重定向路径。当`/`匹配时，以下设置将引导我们到`/books`。

## 路线守卫

路由守卫是中间件或拦截器，如果不经过指定的守卫，就不能将路由添加到堆栈中，守卫对于限制对特定路由的访问非常有用。我们通过从 auto_route 包中扩展`AutoRouteGuard`并在`onNavigation`方法中实现我们的逻辑来创建一个路由守卫。

*   在那之后，我们把我们的警卫分配到我们想要保护的路线上。

```
AutoRoute(page: ProfileScreen, guards: [AuthGuard]);
```

*   最后，我们将 AuthGuard 传递给生成的路由器。

```
**final** _appRouter = AppRouter(authGuard: AuthGuard());
```

## 导航观察员

你可以观察你的导航动作，比如推送、替换、弹出等。我们通过扩展一个`AutoRouterObserver`实现了一个自动路由观察器，它只是一个支持 tab 路由的`NavigatorObserver`。

*   我们简单地像这样添加我们的导航器。

本文到此为止。🎉🎉

要了解更多信息，您可以查看这里的官方文档👇👇

 [## AutoRoute *颤振导航变得简单

### 根路由器是应用程序的顶级路由器，充当应用程序的导航入口点。使用一个…

autoroute.vercel.app](https://autoroute.vercel.app/basics/root_router) 

**感谢您的阅读！**👏👏

如果你喜欢这篇文章，请点击👏按钮(你知道你可以升到 50 吗？)

另外，别忘了关注我，在你的社交网站上分享这篇文章！也让你的朋友知道吧！！