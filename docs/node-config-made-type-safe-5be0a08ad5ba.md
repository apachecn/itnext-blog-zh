# “节点配置”成为类型安全的

> 原文：<https://itnext.io/node-config-made-type-safe-5be0a08ad5ba?source=collection_archive---------6----------------------->

![](img/e1f707ca97c0f3caaa107fb8e8d3ba5c.png)

>我现在有了一个闪亮的新博客。阅读这篇文章，了解 https://blog.goncharov.page/node-config-made-type-safe 的最新动态

[node-config](https://github.com/lorenwest/node-config) 多年来一直是 Node.js 社区的默认配置解决方案。它简单而强大的设计帮助它像病毒一样在多个 JS 库中传播。然而，这些设计选择并不总是受到新的严格类型儿童的欢迎。比如打字稿。我们如何继续使用我们最喜欢的配置工具，并保持类型安全呢？

# 步骤 1:为您的配置创建一个界面

假设您的项目中有一个`config`文件夹，在最简单的情况下，其结构如下:

*   `default.ts`
*   `production.ts`

> *是的，*[*node-config*](https://github.com/lorenwest/node-config)*说出现成的打字稿！*

让我们考虑一个为应用程序编写配置的例子，它创建了一个只有猫居住的新世界。

我们的默认配置如下所示:

我们的生产配置可能是这样的:

基本上，我们的生产配置总是默认配置的子集。因此，我们可以为默认配置创建一个接口，并将该接口的一部分( [DeepPartial](https://stackoverflow.com/questions/45372227/how-to-implement-typescript-deep-partial-mapped-type-not-breaking-array-properti/49936686#49936686) 为真)用于我们的生产配置。

让我们用界面添加`constraint.ts`文件:

然后我们可以在我们的`default.ts`中使用它:

而在我们的`production.ts`:

# 步骤 2:使`config.get`成为类型安全的

到目前为止，我们解决了各种配置之间的任何不一致。但是`config.get`还是返回`any`。

为了解决这个问题，让我们添加另一个类型化版本的`config.get`到它的原型中。

假设您的项目的根目录中有文件夹`config`，文件夹`src`中有您的应用程序的代码，让我们在`src/config.service.ts`创建一个新文件

现在我们可以在应用程序的任何地方使用`config.getTyped`，从`src/config.service`导入它。

> *要知道，我们再也做不到* `*caonfig.getTyped('cat.pawsNum')*` *。不可能使它成为类型安全的。*
> 
> *使* `*import config from 'config'*` *工作集*`*"esModuleInterop":true*`*`*compilerOptions*`*于你的* `*tsconfig.json*` *。**

*在我们的`src/app.ts`中可能是这样的:*

*[**现场演示**](https://repl.it/@aigoncharov/node-config-made-type-safe-demo)*

> **你可能想要使用*[*ts config-paths*](https://github.com/dividab/tsconfig-paths)*来避免深度相对导入。**

*希望你已经找到了对你的项目有用的东西。请随时将您的反馈传达给我！我非常感谢任何批评和问题。*