# 使用 InversifyJS 在 React 中进行依赖注入

> 原文：<https://itnext.io/dependency-injection-in-react-using-inversifyjs-a38ff0c6601?source=collection_archive---------1----------------------->

![](img/29cc4470f50e8f0663aba031c2b4f769.png)

InversifyJS 是一个强大的、轻量级的、易于使用的 JavaScript 依赖注入库。不幸的是，由于 React 的性质，将它用作组件特性并不简单。这是因为 InversifyJS 中的依赖注入依赖于构造函数注入，而 React 不希望用户扩展其组件的构造函数。

然而，有一些方法允许我们在 React 代码中使用控制反转。通过创建一个示例项目，我将向您展示在处理项目时如何绕过这个问题。

# 示例项目

我已经为本文创建了一个简单的 React 项目。这里有趣的地方是:

*   IoC 初始化
*   *NameProvider* 类(它将提供一个由 React 组件显示的名称)

*清单 1 IoC 初始化和域名提供商*

这是 InversifyJS 的典型用法，创建一个新的 IoC 容器，我们将一个类绑定到该容器的给定标识符下(在本例中是一个*字符串*，但它也可以是一个*符号*或一个类型)。这是一个瞬态范围声明，这意味着对于每一次注入，它都将创建该类的一个新实例。还有单例作用域(每次注入总是相同的实例)和请求作用域(对 *container.get* 的一次调用相同的实例)。

每个可以通过 Inversify 注入的类都必须用*可注入的*来修饰，通常是通过在构造函数内部或在属性上使用注入修饰器。然而，这在 React 中是不可能的，所以我们必须用不同的方法来做。

项目的其余部分是一个简单的 React 应用程序。 *Hello* 是一个使用 provider 在 header 中显示名称的组件。在*索引*中，我们将其呈现在 DOM 中。在我们注入我们的 *NameProvider* 实例之前，代码不会工作。

# 使用 inversify-inject-decorator

[](https://www.npmjs.com/package/inversify-inject-decorators) [## 投资注入装饰者

### 用于 InversifyJS 的惰性求值属性注入装饰器

www.npmjs.com](https://www.npmjs.com/package/inversify-inject-decorators) 

当我们不能进行构造函数注入时，这是使用 Inversify 的推荐方法。这个库提供了四个额外的装饰器在项目中使用: *lazyInject* ， *lazyInjectNamed* ， *lazyInjectTagged* ， *lazyMultiInject* 。顾名思义，它们提供了一个延迟求值的注入，这意味着在初始化对象时，不像通常那样提供依赖关系。相反，它是在第一次使用时提供的，然后缓存起来供以后使用。可以通过在初始化期间提供一个适当的布尔值来关闭缓存。

为此，我们首先执行 *getDecorators* 函数，该函数将返回上面提到的 Decorators。在这种情况下，我们将只使用 *lazyInject* ，因为我们不使用名称或标签，也不在一个标识符下绑定多个类来使用多重注入。然后我们用 *lazyInject* 来装饰我们的 *nameProvider* 属性。

*清单 2 lazyInject 导出和带有注入提供者的组件*

在大多数 React 和 InversifyJS 项目中使用它相当简单。不幸的是，使用这种方法获得循环依赖是很常见的，尤其是在将容器模块放在单独的文件中，然后将它们合并到一个容器中，类似于 ioc.ts 文件。这可以通过将模块加载和导出 *lazyInject* 分成单独的文件来避免。你可以在这里看到整个例子:

[](https://github.com/tswistak/react-inversify-inject-decorators-example) [## tswistak/react-inversify-inject-decorators-示例

### react+InversifyJS+inversify-inject-decorators。贡献给 tswistak/react-inversify-inject-decorators-example…

github.com](https://github.com/tswistak/react-inversify-inject-decorators-example) 

# 使用 inversify-react

[](https://www.npmjs.com/package/inversify-react) [## 反作用

### 要连接的组件和装饰器使用 inversify 进行反应。

www.npmjs.com](https://www.npmjs.com/package/inversify-react) 

inversify-react 是一个以稍微不同的方式执行依赖注入的库。在我们的使用中，我们得到一个 React 组件提供者，它保存 IoC 容器并向下传递给 React 树。我们还得到四个装饰器——*提供*、*提供. singleton* 、*提供. transient* 和*解决*。在我们的例子中，我们只需要解析获得依赖关系和*提供者*来将它传递给容器。 *resolve* 的工作方式与前面方法中的 *lazyInject* 类似，但它不需要任何初始化。

要使用它，我们只需将我们的应用程序放在我们要向其传递 IoC 容器的 *Provider* 组件中。在 *Hello* 中，我们用 *resolve* 来修饰 *nameProvider* ，并为其提供一个合适的标识符。ioc.ts 应保持不变(与之前的方法相反)。您可以在此处检查所有更改:

*清单 3 提供者的用法和解析*

这种简单的方法意味着我们不需要在 IoC 中做任何改变，我们只是沿着 React 树向下传递一个容器，并使用适当的装饰器获得依赖关系。然而，该库目前处于开发的早期阶段，API 甚至可能在没有警告的情况下随时更改。另一个缺点是我们只能得到最简单的依赖注入。它将涵盖大多数情况，但我在一些项目中需要多次注入，这是我无法用这个库完成的。你可以在这里看到整个例子:

[](https://github.com/tswistak/inversify-react-example) [## tswistak/inversify-react-example

### reaction+InversifyJS+inversify-reaction。通过创建一个……为 tsw stak/inversify-reaction-example 开发做出贡献

github.com](https://github.com/tswistak/inversify-react-example) 

# 使用反应反转

[](https://www.npmjs.com/package/react-inversify) [## 反应-反转

### 反应的依赖注射

www.npmjs.com](https://www.npmjs.com/package/react-inversify) 

这个名字类似于以前的库，但是这个有一个完全不同的方法。我们有一个*提供者*组件，类似于逆向反应中的组件，还有一个更高阶的*连接*组件，它将依赖注入道具。

它比以前的方法更难使用，因为我们没有在组件中使用装饰器。我们需要创建一个类，在那里我们以一种典型的方式注入依赖项，使用*注入*。然后，我们使用 *connect* HOC 将这个类的一个实例提供给我们的组件的属性，如下所示:

*列表 4 提供者和连接的用法*

用法有点复杂，因为我们需要创建一个额外的类并用 HOC 包装我们的组件。在这个例子中，我把它作为一个装饰器，但是你也可以用传统的方式来使用它。一般来说，这个方法最符合 React，因为对象是作为属性传递的，而不是在组件中实例化的。另一个优点是充分利用了 InversifyJS，这意味着我们不需要依赖于库提供的东西——但是它确实比其他解决方案需要编写更多的代码。您可以在这里看到整个示例:

[](https://github.com/tswistak/react-inversify-example) [## t wistak/reactor-inversify-示例

### reactor+InversifyJS+reactor-inversify。通过创建一个……为 tsw stak/react-inversify-示例开发做出贡献

github.com](https://github.com/tswistak/react-inversify-example) 

# 摘要

因此，我们有三种方法在 React 项目中使用 InversifyJS。每一种都有优点和缺点，也就是说没有完美的解决方案，但是我希望这能让您更容易地选择最适合您的项目的库。

*帖子也发表在* [*协同代码博客*](https://www.synergycodes.com/blog/dependency-injection-in-react-using-inversifyjs) *上。*

您对创建数据可视化应用程序感兴趣吗？[点击此处了解 GoJS 库](https://synergycodes.com/gojs-ebook/)！