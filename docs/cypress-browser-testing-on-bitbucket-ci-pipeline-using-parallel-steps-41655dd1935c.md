# 使用并行步骤在 BitBucket CI 管道上测试 Cypress 浏览器

> 原文：<https://itnext.io/cypress-browser-testing-on-bitbucket-ci-pipeline-using-parallel-steps-41655dd1935c?source=collection_archive---------2----------------------->

您是否使用 BitBucket 管道作为 CI 服务器？你在赛普拉斯的缓慢的 E2E 测试中挣扎吗？您知道 BitBucket 管道可以运行并行步骤吗？您可以使用它将您的浏览器测试分布到几个并行的步骤中，以便在短时间内执行端到端的 Cypress 测试。

![](img/802d2d2b56a101598892a9807c40120d.png)

# 如何并行运行测试

跨并行步骤分布测试以分散工作负载并更快地运行测试可能比您想象的更具挑战性。问题是如何在并行作业中划分 Cypress 测试文件，以确保工作均匀分布？但是…平均分配工作是你真正想要的吗？

要获得最短的 CI 构建时间，您需要充分利用可用的 CI 资源。你想避免浪费时间。这意味着您希望确保并行步骤在相似的时间完成工作，因为这意味着 CI 机器利用率没有瓶颈。

很多事情都是未知的，不可预测的。这可能会影响在 BitBucket 管道上执行测试所需的时间。有这样的事情:

*   启动时间—加载 CI docker 容器所花费的时间
*   从缓存加载 npm/yarn 依赖关系
*   运行 Cypress 测试
*   测试可以在不同的浏览器上运行，这会影响测试的执行时间
*   有时测试会失败，它们的执行时间是不同的
*   其他时候，您可能有随机失败的的[脆弱测试，您可以使用 Cypress 中的测试重试来自动重新运行失败的测试用例。这导致运行测试文件的时间更长。](https://docs.knapsackpro.com/2021/fix-intermittently-failing-ci-builds-flaky-tests-rspec)

所有这些都增加了执行时间的不确定性。很难知道如何最好地将测试文件划分到并行的步骤中，以确保这些步骤在相似的时间完成工作。但是有一个解决方案——在运行时拆分动态测试套件。

# 队列模式—动态测试分割

要跨 BitBucket 管道并行步骤分布测试工作，您可以使用带有队列模式的[背包 Pro](https://knapsackpro.com/?utm_source=docs_knapsackpro&utm_medium=blog_post&utm_campaign=how-bitbucket-pipeline-with-parallel-cypress-tests-can-speed-up-ci-build) 。您可以使用`[@knapsack-pro/cypress](https://github.com/KnapsackPro/knapsack-pro-cypress#knapsack-procypress)` [npm 包](https://github.com/KnapsackPro/knapsack-pro-cypress#knapsack-procypress)，它将在 backpack Pro API 端生成一个包含测试文件列表的队列，然后所有并行步骤都可以连接到该队列来使用测试文件并执行它们。这样，并行步骤只有在执行完之前从 backpack Pro API 获取的一组测试之后，才会要求更多的测试。你可以从文章中了解到队列模式的[细节。](https://docs.knapsackpro.com/2020/how-to-speed-up-ruby-and-javascript-tests-with-ci-parallelisation)

# 位桶管道 YML 配置

下面是一个 YML 中的位桶管道配置的例子。如你所见，通过[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=medium-how-bitbucket-pipeline-with-parallel-cypress-tests-can-speed-up-ci-build)运行 Cypress 测试有 3 个并行步骤。如果您想在更多的并行任务上运行您的测试，您只需要添加更多的步骤。

```
image: cypress/base:10
options:
  max-time: 30

*# job definition for running E2E tests in parallel with* [*KnapsackPro.com*](http://knapsackpro.com/)
e2e: &e2e
  name: Run E2E tests with @knapsack-pro/cypress
  caches:
    - node
    - cypress
  script:
    *# run web application in the background*
    - npm run start:ci &
    *# env vars from* [*https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/*](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)
    - export KNAPSACK_PRO_CI_NODE_BUILD_ID=$BITBUCKET_BUILD_NUMBER
    - export KNAPSACK_PRO_COMMIT_HASH=$BITBUCKET_COMMIT
    - export KNAPSACK_PRO_BRANCH=$BITBUCKET_BRANCH
    - export KNAPSACK_PRO_CI_NODE_TOTAL=$BITBUCKET_PARALLEL_STEP
    - export KNAPSACK_PRO_CI_NODE_INDEX=$BITBUCKET_PARALLEL_STEP_COUNT
    *#* [*https://github.com/KnapsackPro/knapsack-pro-cypress#configuration-steps*](https://github.com/KnapsackPro/knapsack-pro-cypress#configuration-steps)
    - export KNAPSACK_PRO_FIXED_QUEUE_SPLIT=true
    - $(npm bin)/knapsack-pro-cypress
  artifacts:
    *# store any generated images and videos as artifacts*
    - cypress/screenshots/**
    - cypress/videos/**

pipelines:
  default:
  - step:
      name: Install dependencies
      caches:
        - npm
        - cypress
        - node
      script:
        - npm ci
  - parallel:
    *# run N steps in parallel*
    - step:
        <<: *e2e
    - step:
        <<: *e2e
    - step:
        <<: *e2e

definitions:
  caches:
    npm: $HOME/.npm
    cypress: $HOME/.cache/Cypress
```

如果你正在寻找一个并行步骤的定制 docker 容器的例子，请参见[这个](https://gist.github.com/ArturT/90b7ec869e3827b580664beb086a8cd6)。

请记住在`KNAPSACK_PRO_TEST_SUITE_TOKEN_CYPRESS`环境变量中添加您的 API 令牌作为[安全变量](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)。

# 摘要

BitBucket Pipeline 是一个 CI 服务器，它允许并行运行脚本。您可以使用并行步骤，通过[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=medium-how-bitbucket-pipeline-with-parallel-cypress-tests-can-speed-up-ci-build)分发您的 Cypress 测试，以节省时间并尽可能快地运行 CI 构建。

![](img/7fc9f0837af680ca0a9a92529b6fc620.png)

*原载于 2021 年 4 月 13 日*[*【https://docs.knapsackpro.com】*](https://docs.knapsackpro.com/2021/how-bitbucket-pipeline-with-parallel-cypress-tests-can-speed-up-ci-build)*。*