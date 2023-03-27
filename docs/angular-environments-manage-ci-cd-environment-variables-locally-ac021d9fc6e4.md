# 角度环境:本地管理 CI/CD 环境变量

> 原文：<https://itnext.io/angular-environments-manage-ci-cd-environment-variables-locally-ac021d9fc6e4?source=collection_archive---------2----------------------->

## 有角的

## …或者如何使用可变环境文件。

![](img/d50a38d870e32cf97abddb494dc2cf8b.png)

照片由[孟](https://unsplash.com/@ideasboom?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

运行 angular 应用程序时定义环境非常简单。你可以遵循角度指南，它有很好的解释:

 [## 有角的

### Angular 是一个构建移动和桌面 web 应用程序的平台。加入数百万开发者的社区…

angular.io](https://angular.io/guide/build#configure-target-specific-file-replacements) 

然而，在一些 CI/CD 环境中，env 变量存储在一个字典中，并在创建一个版本时注入到文件中。

因此，保持干净的文件并使用*可变的* env 文件会变得复杂。但是让我们从头开始，一个*环境.产品. ts* :

这里，我们使用分隔符`%#<key>#%`。该文件必须以这种方式进行版本控制，以便 CI/CD 任务运行程序可以用相应的值替换每个键。

为了在本地管理每个环境，我们将生成专用的*环境文件*。

# 1.将 env 变量存储在专用文件中

很简单，我们需要创建一个 json 文件来存储一个键值对列表。这个文件必须满足一个命名约定，才能被脚本获取(在本文后面解释)。这里，我们使用`variables.{env}.json`模式。

# 2.编写文件生成脚本

为了创建这个脚本，我们需要两个输入:

*   `variables.{env}.json`文件
*   `environment.prod.ts`文件

在脚本中，语句会将第一个文件中包含的所有变量注入到第二个文件的副本中。

**注意:**可以检查生成的文件是否存在。为了降低可读性，我没有实现它。

# 3.更新 angular.json 以处理多种环境

你可以注意到我们有两种生产配置(`production`和`local-production`)。第一个在运行`ng serve — prod`快捷命令时使用。

# 4.创建 npm 命令

最后，我们需要添加一些 npm 命令来(a)生成文件，(b)为我们的应用程序提供服务。当然，这意味着更新我们的 *package.json* :

瞧！现在，您只需要运行适当的 npm 命令:`npm run start:dev`、`npm run start:staging`或`npm run start:prod`；或者不管你的环境是什么；要处理好几种环境，保存好你的 *environment.prod.ts* 文件。

# 技巧

不要忘记将生成的文件添加到您的*中。gitignore* 因为在你的回购上没有用😉！