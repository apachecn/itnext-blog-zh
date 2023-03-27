# 将 React 本机应用程序迁移到 TypeScript

> 原文：<https://itnext.io/migrating-a-react-native-app-to-typescript-21f3a3d5ff57?source=collection_archive---------3----------------------->

![](img/c2785f60bf1061f315d19031e4c954f0.png)

照片由 [Artem Sapegin](https://unsplash.com/photos/b18TRXc8UPQ?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/search/photos/react?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

最近开始写 React 原生 app。我创建了一个新项目，写了一些不错的组件，介绍了林挺和一些库，然后用道具类型写了一些道具。但我还是觉得少了点什么。ES6 本身很棒，但我真的感觉到缺乏一个全面的打字系统。所以我把我的项目迁移到 TypeScript。以下是如何做到这一点:

我们需要的是:

*   使用 Expo 的现有 React 本机项目，使用 create-react-native-app 脚本创建
*   react-本机类型脚本-转换器
*   以打字打的文件
*   一大堆打字

首先，让我们安装新工具:

`npm i --save-dev react-native-typescript-transformer typescript tslib ts-jest`

ts-jest 只有在您打算使用 jest 进行测试时才是必要的。如果没有，可以跳过。

我要用 tslint 替换 eslint，这样现在就可以卸载了:

`npm uninstall --save-dev babel-eslint eslint eslint-plugin-react`

如果您有. eslintrc.js 文件，您可以删除它，稍后我们将在 tslint.json 文件中配置林挺规则。

因为我将使用 TypeScript 来键入道具，所以我也不再需要道具类型:

`npm uninstall --save prop-types`

最后，我为我正在使用的所有库安装了类型:

`npm install --save-dev @types/jest @types/react @types/react-native @types/react-navigation @types/react-redux @types/react-test-renderer`

以下是所有这些更改后我的 package.json diff 的样子:

react-native-typescript-transformer 是关键元素。这是我们在构建过程中用来传输代码的。为此，我修改了我的 app.json 文件，配置 Expo 在打包时使用 react-native-typescript-transformer，如下所示:

最后，我创建了一个包含所有编译器选项的 tsconfig.json 文件:

之后，您可以测试打包程序。运行`npm start`并按下`R`清除缓存。现在，在您的移动设备上打开 Expo 中的应用程序。即使我们实际上还没有写任何类型，它也应该正确地构建。这可能看起来很奇怪——TypeScript 应该返回一大堆错误。但这实际上是 react-native-typescript-transformer 做出的设计决定，以在开发期间实现代码的增量构建和快速重载，即使它没有被正确地键入。对于实际的打字检查，你应该依靠`tsc`和`tslint`。如果您还没有这些软件，您可以全局安装它们:

`npm i -g typescript tslint`

在您的项目目录中执行`tsc`现在将通过 transpiler 运行您的代码，显示您需要修复的任何错误。然而，要让林挺发挥作用，你首先需要有一些规则。您可以使用`tslint --init`创建一个新的 tslint.json 文件，并根据您的喜好进行配置。这是我的最后的样子:

为了便于比较，下面是我以前的. eslintrc.js 文件:

现在，从你的根目录使用`tslint -p tsconfig.json`将检查你的代码并打印所有错误的列表。您还可以在 IDE 中安装 TSLint，它可以在您编写新代码时使用相同的配置文件来显示编辑器中的所有错误。

经过这一切，一切都应该设置正确。现在是困难的部分:使用`tsc`和`tslint -p tsconfig.json`你可以确定你需要在代码中做的所有改变，所有缺少的类型和分号，所有你需要创建的接口。这需要一段时间。但是，恐怕这部分迁移我帮不了你。但是真的，这是有趣的部分，所以享受吧，祝你好运！