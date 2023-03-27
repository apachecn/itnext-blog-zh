# 处理 mono-repo 应用程序的类型脚本配置。

> 原文：<https://itnext.io/defining-typescript-config-types-for-the-mono-repo-application-a39a5b8c7bc2?source=collection_archive---------3----------------------->

![](img/66fdc967787f25cb074c066ab16819ca.png)

由 [Corinne Kutz](https://unsplash.com/@corinnekutz?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

假设你正在构建一个**单一回购**应用。该结构正在划分一些服务，如`fronted`、`backend`和`docs`。这个应用程序和结构可以由 [Lerna](https://github.com/lerna/lerna) 来处理——一个管理带有多个包的 JavaScript 项目的工具。Lerna 所做的(作为许多特性的一部分)是能够解析本地和全局 *package.json* 文件，以使用一个正确的包列表和依赖关系表示。这意味着你可以在你所有的服务中使用全局包，但是如果你在前端程序中使用一些包，你不需要为后端程序安装它。借助 Lerna `boostrap`特性，您可以使用一个命令和流程来安装和管理所有服务的依赖关系。

好了，看一下回购结构。

```
|root
|--- frontend
|--- backend
|--- docs
```

现在，让我们假设您想要设置一些全局 TypeScript 配置，但是只为一个服务使用它。例如，你正在用 TypeScript 构建一些前端应用程序，但是后端应用程序是用普通的 JavaScript 编写的。然而，在将来，你可能也想使用 TS 作为后端。我们能做什么？

将您的 TypeScript config(*ts config . JSON*)放到根文件夹中。然后定义`rootDir`并在那里放置前端应用程序的文件夹名，就像这样。

```
{
  "compilerOptions": {
    "rootDir": "frontend",
    "types": [
      "@types",
    ]
  },
}
```

现在。当然，你需要一些额外的类型定义。通常，您可以通过在 ***类型*** 对象中添加包名来定义它们。这是你会遇到一些问题的时刻。由于 TypeScript 配置使用您的前端服务的根目录，因此没有符号表明您的类型是全局安装的，TypeScript 正在您的前端服务 *node_modules 中查找它们。*

```
TS2688: Cannot find type definition file for '[@](http://twitter.com/nuxt/image)types'. The file is in the program because: Entry point of type library '[@](http://twitter.com/nuxt/image)types' specified in compilerOptions.
```

如何处理这个问题？超级简单。只需在您的 *tsconfig.json* 文件中定义`typeRoots`属性，并将您的本地 *node_modules* 路径传递到那里。您也可以为别名设置它。就像那样。

```
{
  "compilerOptions": {
    "rootDir": "frontend"
    "paths": {
      "~/*": ["./frontend/*"]
    },
    **"typeRoots": ["./frontend/node_modules/"],**
    "types": [
      "@types",
    ]
    "exclude": ["./*.config.js"]  },
}
```

…或者没有`types`，就这样。

```
{
  "compilerOptions": {
    **"typeRoots": ["./frontend/node_modules/@types"],** }
}
```

这里值得一提的是，TypeScript 可能会寻找一些全局定义的配置，如`commitlint`或`stylelint`。因为它们在大多数情况下是`.js`文件，所以你可以在 *tsconfig.json* 文件中排除它们。

最后，您需要为您的服务定义一些虚拟的 *tsconfig.json* 。为此，创建它并添加这个简单的配置—在`frontend`文件夹/服务中。

```
{
  "extends": "../tsconfig.json",
  "compilerOptions": {}
}
```

**您还可以为所有服务创建一个 TypeScript 配置文件，并在整个 mono-repo 中使用它。通过这种方式，您可以确信您的整个代码库遵循一个标准。**

仅此而已。简单又很有帮助。享受吧。