# 如何使用 Codefresh CI 并行作业将 RoR 项目的 RSpec 运行速度提高几倍

> 原文：<https://itnext.io/how-to-use-codefresh-ci-parallel-jobs-to-run-rspec-a-few-times-faster-for-ror-project-de4caeb8a83e?source=collection_archive---------2----------------------->

如果你是一个 Ruby 开发者，Codefresh.io 似乎是一个非常好的 CI 解决方案。我已经在 Codefresh 上测试了我的一个项目，看看它如何允许运行快速测试。Codefresh 有一个矩阵特性，允许您为 CI 构建运行并行步骤。在本文中，您将看到如何利用 Codefresh matrix 配置和 backpack Pro 客户端库，通过 RSpec 测试套件并行测试您的 Ruby on Rails 项目。

![](img/8e8bdcab365bb5e982751c1dd80a71a2.png)

# 在 Codefresh.io 上配置 Rails 项目

为了在 Codefresh 上运行针对 Rails 项目的 CI 构建，您需要 YAML 配置文件和 Docker 映像，它们将用作运行测试的基本 Docker 容器。

我们还需要 CI 服务器上的 DB。在我的 Ruby on Rails 项目中，我使用 PostgreSQL，所以我必须配置 Codefresh 服务来运行 Postgres。

让我们从创建需要添加到存储库中的`.codefresh/codefresh.yml`文件和`Test.Dockerfile`开始。

如你所见，我使用[backpack _ pro ruby gem](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=how-to-use-codefresh-ci-parallel-steps-to-run-rspec-a-few-times-faster-for-rails-project)作为运行我的 RSpec 测试的命令。它将知道基于`KNAPSACK_PRO_CI_NODE_INDEX`值，在一个特定的并行步骤上应该执行哪组测试。基于为矩阵定义的`KNAPSACK_PRO_CI_NODE_INDEX`变量列表生成节点索引。

backpack Pro 将自动在并行任务(步骤)之间分割测试，以确保每个步骤花费相似的时间。这是可能的，感谢背包专业队列模式，它可以跨并行步骤进行动态测试套件分割。您可以从本文末尾的视频中了解更多关于队列模式如何工作的技术细节，但现在让我们来看看 Codefresh 是如何工作的。您可以在下面的视频中看到我在 Codefresh web dashboard 中的 CI 构建。

如果你忘记了视频中的步骤，那么我列出了你需要在 Codefresh 仪表板和 YAML 配置文件中设置的东西。

*   您需要在 Codefresh dashboard 中设置一个类似`KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC`的 API 令牌，请参见左侧菜单管道- >设置(管道旁边的 cog 图标)- >变量选项卡(参见右侧的垂直菜单)。根据您使用的测试运行程序添加新的 API 令牌:
*   `KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC`
*   `KNAPSACK_PRO_TEST_SUITE_TOKEN_CUCUMBER`
*   `KNAPSACK_PRO_TEST_SUITE_TOKEN_MINITEST`
*   `KNAPSACK_PRO_TEST_SUITE_TEST_UNIT`
*   `KNAPSACK_PRO_TEST_SUITE_TOKEN_SPINACH`
*   设置在哪里可以找到 Codefresh YAML 文件。在 Codefresh dashboard 中，请参见左侧菜单管道->设置(管道旁边的 cog 图标)->工作流选项卡(顶部的水平菜单)-> YAML 路径(在此设置`./.codefresh/codefresh.yml`)。
*   使用`.codefresh/codefresh.yml`文件中的`KNAPSACK_PRO_CI_NODE_TOTAL`环境变量设置您想要运行多少个并行作业(并行 CI 节点)。
*   确保在矩阵部分列出了值从`0`到`KNAPSACK_PRO_CI_NODE_TOTAL-1`的所有`KNAPSACK_PRO_CI_NODE_INDEX`环境变量。Codefresh 将生成一个并行作业矩阵，其中每个作业都有不同的`KNAPSACK_PRO_CI_NODE_INDEX`值。多亏了这一点，backpackage Pro 知道应该对每个并行任务运行什么样的测试。

# 队列模式如何工作

在这个视频中，我解释了动态测试套件分割的技术细节，以及它解决了什么问题来帮助您在最佳时间内运行快速 CI 构建。

Ruby 中有更多的测试运行程序，如 Cucumber、Minitest 等，这些都是由[背包 Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=how-to-use-codefresh-ci-parallel-steps-to-run-rspec-a-few-times-faster-for-rails-project) 支持的。甚至像 Cypress.io 或 Jest 这样的 JavaScript 工具也可以用 backpack Pro wrapper 启动([参见安装指南](https://docs.knapsackpro.com/integration/))。

我希望本教程对您有用，并且由于在 Codefresh.io 上运行并行步骤，您可以从 Rails 项目的更快 CI 构建中受益

*原载于 2019 年 10 月 18 日*[*https://docs.knapsackpro.com*](https://docs.knapsackpro.com/2019/how-to-use-codefresh-ci-parallel-steps-to-run-rspec-a-few-times-faster-for-rails-project)*。*