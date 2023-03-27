# 如何高效运行 CodeShip 并行测试管道——什么是最佳 CI 并行化？

> 原文：<https://itnext.io/how-to-run-codeship-parallel-test-pipelines-efficiently-what-is-optimal-ci-parallelization-4afd36dece89?source=collection_archive---------4----------------------->

当您使用 CodeShip 作为 CI 服务器时，您可以通过并行测试管道显著提高 CI 构建的速度。管道允许您同时运行多个命令，例如，您可以将测试套件划分到几个管道中，从而更快地完成 CI 构建。

![](img/73220e0d1261a40c91f3e2e71d605bb9.png)

代码船

# 1.如何在 CodeShip 上运行并行命令

## 通过 CodeShip 接口设置

一种方法是通过[代码接口](https://documentation.codeship.com/basic/builds-and-configuration/parallel-tests/#using-parallel-test-pipelines)定义命令。一旦启用了并行测试管道，每个项目就可以拥有多个并行运行的测试管道。

为了尽可能快地运行 CI 构建，我们需要确保并行命令将以所有命令同时完成的方式运行测试套件的子集，以避免缓慢的管道瓶颈。为了分割测试套件，我们将使用带有队列模式的[backpack Pro，它为 Ruby 和 JavaScript 测试进行跨管道的动态测试套件分割](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-codeship-parallel-test-pipelines-efficiently-optimal-ci-parallelization)，以保持在并行管道(也称为 CI 节点)上以最佳方式运行我们的测试。

## Ruby on Rails 项目中的测试套件示例

配置测试管道(使用 1/2)

```
*# first CI node running in parallel*

*# RSpec tests in Knapsack Pro Queue Mode (dynamic test suite split)*
KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=0 bundle exec rake knapsack_pro:queue:rspec
```

配置测试管道(使用 2/2)

```
*# second CI node running in parallel*

*# RSpec tests in Knapsack Pro Queue Mode (dynamic test suite split)*
KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=1 bundle exec rake knapsack_pro:queue:rspec
```

## JavaScript 中 Cypress 测试运行器的测试套件分割示例

配置测试管道(使用 1/2)

```
*# first CI node running in parallel*
KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=0 $(npm bin)/knapsack-pro-cypress
```

配置测试管道(使用 2/2)

```
*# second CI node running in parallel*
KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=1 $(npm bin)/knapsack-pro-cypress
```

在本文中，您可以了解更多关于 JavaScript 中用于 E2E 测试的 [Cypress test runner。](https://docs.knapsackpro.com/2018/run-javascript-e2e-tests-faster-with-cypress-on-parallel-ci-nodes)

# 2.通过 codeship-services.yml 进行设置

如果您使用 CodeShip Pro，则通过使用`type: parallel`头定义并行步骤组，然后嵌套您想要并行化的所有步骤，如下例所示:

```
- type: parallel
  steps:
  - service: app
    command: KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=0 bundle exec rake knapsack_pro:queue:rspec
  - service: app
    command: KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=1 bundle exec rake knapsack_pro:queue:rspec
```

如何配置 [CodeShip Pro 并行度](https://documentation.codeship.com/pro/builds-and-configuration/steps/#parallelizing-steps-and-tests)的更多示例

# 3.动态测试套件分割是如何工作的

如果您想更好地了解动态测试套件拆分的工作原理以及它可以解决哪些问题，请查看视频:

# 摘要

并行运行测试是缩短 CI 构建时间的快速方法。为了提高效率，我们可以使用[backpackage Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-codeship-parallel-test-pipelines-efficiently-optimal-ci-parallelization)在 CI 节点之间以最佳方式分割测试套件，以保持 CI 节点自动平衡，并尽可能快地运行 CI 构建。

*最初发表于*[T5【docs.knapsackpro.com】](https://docs.knapsackpro.com/2018/how-to-run-codeship-parallel-test-pipelines-efficiently-optimal-ci-parallelization)*。*