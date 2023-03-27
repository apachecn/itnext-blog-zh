# 如何将 git hash 添加到 Angular 应用程序中

> 原文：<https://itnext.io/how-to-add-git-hash-into-your-angular-application-bec20b702bb1?source=collection_archive---------2----------------------->

假设您想要将当前的 git 提交散列添加到您的 angular UI(我目前使用的是 angular 6 ),这样以后就可以更容易地调试您部署的版本——这可以通过使用如下的 **git-describe** lib 来轻松实现:

1.  安装**git——将**描述为开发依赖项
2.  在根项目上创建一个 grab-git-info.js

grab-git-info.js 脚本的输出将是/src/下的‘git-version . JSON’文件，它将包含我们的应用程序所需的所有 git 信息。

为了能够导入 json 文件(或任何其他 json 文件),我们需要添加一个定义文件来声明所添加的模块，以便 Typescript 编译器能够识别它。

3.在你的/src 下创建 typing ings . d . ts 文件(在这里阅读更多关于 typing ings . d . ts 的内容:[https://angular . io/guide/typescript-configuration # typescript-typing ings](https://angular.io/guide/typescript-configuration#typescript-typings)

从现在开始，您可以将位于/src 下的任何 json 文件作为一个模块导入！

```
**import** gitInfo **from '../../../git-version.json'**;
```

享受:)