# 如何在 Jenkins 管道阶段上运行黄瓜并行测试

> 原文：<https://itnext.io/how-to-run-cucumber-parallel-tests-on-jenkins-pipeline-stages-6382bbe4518f?source=collection_archive---------4----------------------->

Cucumber 是一个流行的用于行为驱动开发(BDD)的自动化测试工具，但是当你在你的工作项目中使用它一段时间后，自动化测试的数量就会增加，你需要花费几十分钟来运行你的 Cucumber 测试套件。有时候，复杂的项目可能需要几个小时的执行时间来进行黄瓜测试。为了节省时间并加快您在 CI(持续集成)上构建 Cucumber 的速度，您可以使用 CI 并行化。在本文中，您将看到如何使用 Jenkins 并行管道为 Jenkins 实现这一点。

![](img/7084faa73a7909894687625c9839c10f.png)

黄瓜+詹金斯

# 如何平行拆分黄瓜测试

Jenkins 允许您将管道配置为代码，并使用 Jenkins 管道阶段来定义将并行(同时)执行的任务。

在这个例子中，您将使用持续集成工具，如 cucumber ruby gem 和[backpack _ pro gem，来跨并行的 Jenkins 阶段](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=cucumber-testing-with-jenkins-parallel-pipeline-to-get-down-ci-build-time)分割测试。

当您使用 Cucumber 测试 Ruby on Rails 项目时，您希望确保并行执行的测试将在相似的时间完成工作，以充分利用您的 CI 服务器资源。正如你所知道的，一些测试文件可以有很少的测试，而其他的测试用例要多得多。根据测试文件的内容，每个测试文件的执行时间可能不同，这使得在并行任务之间以最佳方式分配测试变得更加困难。

将测试套件中的测试文件分割到每个 Jenkins 并行阶段是很重要的，这样可以消除瓶颈(一个比其他阶段运行测试时间更长的缓慢阶段)。要做到这一点，你可以[使用 Cucumber](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=cucumber-testing-with-jenkins-parallel-pipeline-to-get-down-ci-build-time) test runner 的背包 Pro 队列模式，跨并行阶段动态分割测试套件。在下面的视频中，您将了解队列模式是如何工作的:

# 詹金斯流水线并行黄瓜测试

这是 Jenkinsfile 的配置，其中使用[backpack _ pro 插件](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=cucumber-testing-with-jenkins-parallel-pipeline-to-get-down-ci-build-time)运行黄瓜测试的 4 个并行阶段。

```
timeout(time: 60, unit: 'MINUTES') {
  node() {
    stage('Checkout') {
      checkout([/* checkout code from git */])

      // determine git commit hash because we need to pass it to knapsack_pro
      COMMIT_HASH = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()

      stash 'source'
    }
  }

  def num_nodes = 4; // define your total number of CI nodes (how many parallel jobs will be executed)
  def nodes = [:]

  for (int i = 0; i < num_nodes; i++) {
    def index = i;
    nodes["ci_node_${i}"] = {
      node() {
        stage('Setup') {
          unstash 'source'
          // other setup steps
        }

        def knapsack_options = """\
            KNAPSACK_PRO_CI_NODE_TOTAL=${num_nodes}\
            KNAPSACK_PRO_CI_NODE_INDEX=${index}\
            KNAPSACK_PRO_COMMIT_HASH=${COMMIT_HASH}\
            KNAPSACK_PRO_BRANCH=${env.BRANCH_NAME}\
        """

        // Example how to run Cucumber tests in Knapsack Pro Queue Mode
        // Queue Mode should be a last stage if you have other stages in your pipeline
        // thanks to that it can autobalance CI build time if other tests were not perfectly distributed
        stage('Run Cucumber tests') {
          sh """KNAPSACK_PRO_CI_NODE_BUILD_ID=${env.BUILD_TAG} ${knapsack_options} bundle exec rake knapsack_pro:queue:cucumber"""
        }
      }
    }
  }

  parallel nodes // run CI nodes in parallel
}
```

# 摘要

测试大型项目是一项耗费精力的任务。用 Cucumber 进行自动化测试会有所帮助。当您可以利用 Jenkins stages 的并行测试执行，并通过使用[背包 Pro 队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=cucumber-testing-with-jenkins-parallel-pipeline-to-get-down-ci-build-time)进行最佳测试套件分割时，您的 CI 服务器(如 Jenkins)可以提供更多帮助。队列模式也适用于其他 [Ruby 或 JavaScript 测试运行者](https://docs.knapsackpro.com/integration/)。

— — —

最初发布于:[https://docs . knapsackpro . com/2019/cucumber-testing-with-Jenkins-parallel-pipeline-to-get-down-ci-build-time](https://docs.knapsackpro.com/2019/cucumber-testing-with-jenkins-parallel-pipeline-to-get-down-ci-build-time)