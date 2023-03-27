# 如何使用并行测试在 Heroku CI 上更快地运行 RSpec、Jest、Cypress 测试

> 原文：<https://itnext.io/how-to-run-rspec-jest-cypress-tests-faster-on-heroku-ci-with-parallel-testing-77d182bf469?source=collection_archive---------5----------------------->

Heroku 为团队提供了现成的 CI 解决方案。Heroku CI 可以在 dyno 实例中为您的项目运行测试。更有趣的是，您可以将并行 dynos 作为 CI 构建的一部分来运行。这允许您在并行 dynos 上分割测试，以更快地完成 CI 构建并节省时间。

![](img/fee777406f087469fb8c90b7d18bc65e.png)

Heroku 对每个 dyno 的运行时间收费。这意味着，您可以将测试套件拆分到多个 dyno 上，而不是在单个 dyno 上运行您的慢速测试套件，并支付或多或少相同的费用，从而显著减少项目的 CI 构建时间。

# 如何从 Heroku CI 开始

> 如果您在 Heroku dashboard 中已经有一个团队，那么您可以使用 Heroku CI 并为您的项目运行测试。

在 Heroku，你可以为你的一个项目打开你的团队和特定的管道。您可以在那里找到一个*测试*选项卡，在那里您可以启用 Heroku CI。

您还需要在项目的存储库中有一个 *app.json* 文件。该文件包含在 Heroku 上运行项目所需的信息。您可以向 *app.json* 文件添加 Heroku CI 所需的附加配置。

为了使用 Heroku CI 并行测试运行，我们需要启用它。您可以请求 Heroku 支持人员为您的项目激活它。这个特性允许为您的 CI 构建运行多达 32 个并行 dynos。

您还可以观看更详细的视频中的所有步骤，或者从本文中复制一些示例。

下面是如何在 Heroku CI 上为 Ruby 和 JavaScript 项目分割测试的例子。

# 如何在 Ruby on Rails 项目中为测试套件运行并行 dynos

您可以将用 **RSpec** 、 **Minitest** 、 **Cucumber** 或其他测试程序编写的 Ruby 测试以动态方式拆分到并行的 dyno 上，所有的 dyno 将在相似的时间完成工作，这样您就可以尽快得到测试套件是绿色还是红色的结果。为此，您可以使用[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-tests-faster-on-heroku-ci-with-parallel-dynos)工具及其[队列模式进行动态测试套件分割](https://youtu.be/hUEB1XDKEFY)。使用 *quantity* 键，您可以设置想要使用多少个并行 dynos 来运行您的*脚本测试*命令。

```
{
  "environments": {
    "test": {
      "formation": {
        "test": {
          "quantity": 2
        }
      },
      "addons": [
        "heroku-postgresql"
      ],
      "scripts": {
        "test": "bundle exec rake knapsack_pro:queue:rspec"
      },
      "env": {
        "KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC": "rspec-token"
      }
    }
  }
}
```

您还可以更改 CI 构建的动态大小。如果您运行需要更多 CPU 或内存的复杂测试，那么您可以向 *app.json* 添加 *size* 参数来定义 [dyno type](https://devcenter.heroku.com/articles/dyno-types) 。

```
{
  "environments": {
    "test": {
      "formation": {
        "test": {
          "quantity": 1,
          "size": "performance-l"
        }
      }
   }
}
```

# 针对赛普拉斯 E2E 测试套件，在并行 Heroku CI dynos 上运行 JavaScript 测试

端到端测试(E2E)通常需要很多时间，因为点击网站的多个场景非常耗时。在多个 dynos 上拆分 **Cypress** 测试套件将帮助我们节省大量时间，并保持 CI 快速构建。

我们可以使用[@ backpackage-pro/cypress](https://github.com/KnapsackPro/knapsack-pro-cypress)项目。它使用[背包 Pro 队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-tests-faster-on-heroku-ci-with-parallel-dynos)。

这是你的 *app.json* 的配置

```
{
  "environments": {
    "test": {
      "formation": {
        "test": {
          "quantity": 2
        }
      },
      "addons": [
        "heroku-postgresql"
      ],
      "scripts": {
        "test": "$(npm bin)/knapsack-pro-cypress"
      },
      "env": {
        "KNAPSACK_PRO_TEST_SUITE_TOKEN_CYPRESS": "example"
      }
    }
  }
}
```

如果你想了解更多关于 Cypress 的知识，那么查看一篇文章中的视频[在并行 CI 节点上用 Cypress 运行 javascript E2E 测试](https://docs.knapsackpro.com/2018/run-javascript-e2e-tests-faster-with-cypress-on-parallel-ci-nodes)。

# 在并行 Heroku CI dynos 上用 JavaScript 运行 Jest 测试

我们可以使用[@ backpackage-pro/Jest](https://github.com/KnapsackPro/knapsack-pro-jest)客户端库来拆分你的 **Jest** 测试。它使用[背包 Pro 队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-tests-faster-on-heroku-ci-with-parallel-dynos)。

这是你的 *app.json* 的配置

```
{
  "environments": {
    "test": {
      "formation": {
        "test": {
          "quantity": 2
        }
      },
      "addons": [
        "heroku-postgresql"
      ],
      "scripts": {
        "test": "$(npm bin)/knapsack-pro-jest"
      },
      "env": {
        "KNAPSACK_PRO_TEST_SUITE_TOKEN_JEST": "example"
      }
    }
  }
}
```

# 摘要

> 您可以在安装指南中找到各种测试运行器的所有集成。
> [参见背包 Pro](https://docs.knapsackpro.com/integration/) 中支持的测试跑者。

赫罗库词是赫罗库的一大补充。如果您已经使用 Heroku，您可以轻松地利用并行 dyno 并保持 CI 的低成本，因为您只需为 dyno 的使用付费。最棒的是，您可以使用测试套件的子集运行许多并行 CI dynos，从而为您的工程团队节省大量时间。你可以了解更多关于[背包专业版如何通过更快的 CI 构建帮助你节省时间](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-tests-faster-on-heroku-ci-with-parallel-dynos)。

在下面的视频中了解更多关于队列模式及其解决的问题。

— — —

最初发布于:[https://docs . knapsackpro . com/2019/how-to-run-tests-faster-on-heroku-ci-with-parallel-dynos](https://docs.knapsackpro.com/2019/how-to-run-tests-faster-on-heroku-ci-with-parallel-dynos)