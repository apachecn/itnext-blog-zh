# 如何使用构建矩阵特性在 Travis CI 上运行并行测试

> 原文：<https://itnext.io/how-to-run-parallel-tests-on-travis-ci-with-build-matrix-feature-4771a9e1af63?source=collection_archive---------3----------------------->

Travis CI 允许您在同一个 CI 构建中运行多个作业。它们甚至允许多达 200 个开源项目的并行作业(私有存储库也是如此)。您可以利用 Travis build matrix 特性，通过将测试分成许多较小的作业来运行您的测试套件的子集，从而更快地运行您的项目。

![](img/1563d9ae199e58c65ab997288ab8601c.png)

# 构建矩阵功能如何工作？

[构建矩阵功能](https://docs.travis-ci.com/user/build-matrix/)允许自动创建语言和环境相关配置选项集的所有可能组合的矩阵。例如，当您想要在两种不同的编程语言版本和两种不同的浏览器上测试您的项目时，Travis 会为每种编程语言和浏览器生成 4 个并行作业来运行您的测试。

# 跨并行作业拆分测试

我们可以通过向您的项目[背包 Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-travis-ci-parallel-jobs-with-build-matrix-feature-fast) 中添加构建矩阵特性来拆分测试，这将在并行作业之间自动拆分您的 **Ruby** 或 **Javascript** 测试，所有并行作业将运行测试套件的子集并在相似的时间内完成工作，因此您将尽可能快地获得测试套件通过或不通过的结果，而无需等待最慢的作业。

如何使用[backpack _ pro](https://github.com/KnapsackPro/knapsack_pro-ruby)Ruby gem 在并行作业上运行 Ruby 测试:

```
*# .travis.yml*
script:
  *# Step for RSpec*
  - "bundle exec rake knapsack_pro:rspec"

  *# Step for Cucumber*
  - "bundle exec rake knapsack_pro:cucumber"

  *# Step for Minitest*
  - "bundle exec rake knapsack_pro:minitest"

  *# Step for test-unit*
  - "bundle exec rake knapsack_pro:test_unit"

  *# Step for Spinach*
  - "bundle exec rake knapsack_pro:spinach"

env:
  global:
    *# tokens should be set in travis settings in web interface to avoid expose tokens in build logs*
    - KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC=rspec-token
    - KNAPSACK_PRO_TEST_SUITE_TOKEN_CUCUMBER=cucumber-token
    - KNAPSACK_PRO_TEST_SUITE_TOKEN_MINITEST=minitest-token
    - KNAPSACK_PRO_TEST_SUITE_TOKEN_TEST_UNIT=test-unit-token
    - KNAPSACK_PRO_TEST_SUITE_TOKEN_SPINACH=spinach-token

    - KNAPSACK_PRO_CI_NODE_TOTAL=2
  matrix:
    - KNAPSACK_PRO_CI_NODE_INDEX=0
    - KNAPSACK_PRO_CI_NODE_INDEX=1
```

如何用 JavaScript 在并行作业上用[@ backpackage-pro/Cypress](https://github.com/KnapsackPro/knapsack-pro-cypress)运行 Cypress 测试:

```
*# .travis.yml*
script:
  - "$(npm bin)/knapsack-pro-cypress"

env:
  global:
    - KNAPSACK_PRO_CI_NODE_TOTAL=2
    *# allows to be able to retry failed tests on one of parallel job (CI node)*
    - KNAPSACK_PRO_FIXED_QUEUE_SPLIT=true

  matrix:
    - KNAPSACK_PRO_CI_NODE_INDEX=0
    - KNAPSACK_PRO_CI_NODE_INDEX=1
```

通过跨 Travis 并行作业以动态的方式进行测试套件分割，我们节省了更多的时间，并保持了 CI 构建的快速性。我也将并行作业称为 CI 节点，因为它们是单个 CI 构建的一部分。在这个视频中，我描述了几个可以通过动态测试套件分割解决的问题。

# Travis CI 构建行动矩阵

如果你想看看[backpack Pro 如何帮助在并行任务中分割测试](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-travis-ci-parallel-jobs-with-build-matrix-feature-fast)你可以查看开源项目 Consul——开放政府和电子参与网络软件。有一个[配置项构建列表供咨询](https://travis-ci.org/consul/consul)使用。

如果你想了解更多关于使用[背包 Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-travis-ci-parallel-jobs-with-build-matrix-feature-fast) 进行测试的信息，你可以查看我们[博客](https://docs.knapsackpro.com/)上的其他文章，比如[使用 Cypress test runner](https://docs.knapsackpro.com/2018/run-javascript-e2e-tests-faster-with-cypress-on-parallel-ci-nodes) 进行测试。

*原载于*[*docs.knapsackpro.com*](https://docs.knapsackpro.com/2018/how-to-run-travis-ci-parallel-jobs-with-build-matrix-feature-fast)*。*