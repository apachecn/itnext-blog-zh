# 连续累计

> 原文：<https://itnext.io/continuous-integration-6a8914a5edc8?source=collection_archive---------6----------------------->

## 它的想法和如何使用它

[饭桶周刊](https://medium.com/@aerabi/list/git-weekly-9fe103e35b4b) #25
等级:初学者🥉

持续集成是一种软件开发实践，在这种实践中，开发人员定期将他们的代码更改合并到一个中央存储库中，从而允许他们快速检测和修复错误。这种做法旨在帮助团队避免当多个开发人员处理一个项目并对同一代码库进行更改时可能出现的问题。

![](img/19769ac7e983d62a0bdb1ed6bd1b9b97.png)

要使用持续集成，开发人员需要有一个共享的存储库，在那里他们可以提交他们的代码更改。这里最常见的实践是拥有一个 git 存储库并围绕它构建 CI 管道，尽管这不是唯一可能的实践。

一旦建立了存储库，开发人员就可以配置他们的开发环境，以便在他们提交对代码库的更改时自动运行测试和检查。这可以包括运行静态分析工具来检查代码错误，或者运行自动化单元测试来确保代码按照预期运行。

如果这些检查中的任何一项失败，持续集成系统将警告开发人员并停止集成过程。这允许开发人员修复问题并重新提交他们的代码进行集成，防止错误进入主代码库。

除了帮助开发人员避免错误，持续集成还可以通过允许他们更频繁地集成代码更改来帮助团队更有效地工作。这可以加快开发过程，并使团队更容易在相同的代码基础上协作和工作。

例如，我们将使用 GitHub 操作为 Node.js 项目创建一个简单的 CI 管道。

# Node.js 项目的 GitHub 操作

GitHub Actions 是一个强大的工具，允许开发人员自动化他们的软件开发工作流程。GitHub 动作的一个常见用途是在项目上运行自动化测试，在这一节中，我们将看看如何使用 GitHub 动作来测试 Node.js 项目。

首先，您需要为您的项目创建一个新的 GitHub 存储库，如果您还没有创建的话。一旦你建立了你的存储库，你就可以创建一个新的 GitHub 动作，方法是转到存储库中的`Actions`标签并点击`Set up a workflow yourself`按钮。

这将打开 GitHub 动作编辑器，在这里你可以通过点击`Start a new workflow`按钮创建一个新的工作流文件。您可以为工作流文件命名，并为其在存储库中选择一个位置。

接下来，您需要向您的工作流添加一个`Node.js`环境。通过将以下代码添加到工作流文件中，可以做到这一点:

```
name: Node.js CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x]

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}
```

这段代码建立了一个新的运行在最新版本 Ubuntu 上的`Node.js`环境。它还指定了工作流应该在所有推送事件上运行，这意味着无论何时您将变更推送到您的存储库，工作流都将被触发。

随着`Node.js`环境的建立，您现在可以向您的工作流添加步骤来运行您的测试。例如，您可以添加以下代码来运行项目中的`npm test`命令:

```
 - name: Install dependencies
      run: npm install
    - name: Run tests
      run: npm test
```

这段代码为您的项目安装依赖项，然后运行`npm test`命令来执行您的测试。您还可以在工作流程中添加其他步骤来运行其他命令或执行其他操作。

将所有必要的步骤添加到工作流后，您可以保存您的更改并将它们提交到存储库中。这将触发工作流并自动运行您的测试。您可以在存储库中的`Actions`选项卡上查看工作流程的进度和结果。

# 最后的话

GitHub Actions 是一个强大的工具，它可以帮助你自动化你的软件开发工作流程，并使在你的项目上运行测试变得更加容易。通过遵循这篇博文中概述的步骤，您可以轻松地设置 GitHub 操作来测试 Node.js 项目。

总的来说，对于任何希望改进工作流程并避免代码错误的软件开发团队来说，持续集成都是一项重要的实践。通过定期集成代码变更和运行自动化检查，团队可以确保他们的代码总是处于良好的状态，并准备好交付给用户。

我每周都会在 git 上写一篇博文。

*   [订阅](https://medium.com/subscribe/@aerabi)my Medium publishes，以便在新的 Git 周刊发布时获得通知。
*   在 Twitter 上关注[我，了解其他平台上发布的更多更新和文章。](https://twitter.com/MohammadAliEN)