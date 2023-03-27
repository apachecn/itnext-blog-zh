# 使用 Jenkins Pipeline 在您的工作流程阶段运行并行测试

> 原文：<https://itnext.io/run-parallel-tests-in-your-workflow-stages-with-jenkins-pipeline-38003cf01075?source=collection_archive---------1----------------------->

Jenkins Pipeline 是一套插件，允许为 CI 上的测试环境创建从简单到复杂的构建阶段。我们可以使用 Jenkins Pipeline 同时运行几个阶段，这得益于跨几个阶段的并行测试套件，可以更快地完成测试。

![](img/1f601fd4c75d2367067a3876c70fa44f.png)

詹金斯

为了使用 [Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/) 运行并行阶段，我们将需要一个适当的 Jenkinsfile，它通过管道领域特定语言(DSL)语法将我们的交付管道表示为代码。

我们需要解决的另一个问题是，如何将我们的测试套件划分到并行的阶段，使得在所有阶段执行的测试套件的每个子集都能同时完成工作。在相似的时间内完成所有阶段的测试很重要，这样才能尽快运行我们的 CI 构建并消除瓶颈阶段。

# 如何在并行的 Jenkins 阶段中平均划分测试套件

为了将我们的测试划分到并行阶段，我们可以使用[背包 Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=jenkins-pipeline-how-to-run-parallel-tests-in-your-workflow-stages) ，它允许跨阶段(也称为 CI 节点)动态分配测试。这样，我们将在最佳时间内运行我们的并行测试。

在这里，您可以了解更多动态测试套件分配是如何工作的，以及它可以帮助解决哪些问题。

# 并行管道的 Jenkinsfile

在这个例子中，我将向您展示如何在并行的 Jenkins 阶段中分割您的 Ruby 和 JavaScript 测试。

这是 Ruby 的例子，如何分割 Cucumber 和 RSpec 测试套件。在[背包专业版](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=jenkins-pipeline-how-to-run-parallel-tests-in-your-workflow-stages)上还可以找到其他测试跑步者，如 Minitest、Test::Unit 等:

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

        // example how to run cucumber tests in Knapsack Pro Regular Mode
        stage('Run cucumber') {
          sh """${knapsack_options} bundle exec rake knapsack_pro:cucumber"""
        }

        // example how to run rspec tests in Knapsack Pro Queue Mode
        // Queue Mode should be as a last stage so it can autobalance build if tests in regular mode were not perfectly distributed
        stage('Run rspec') {
          sh """KNAPSACK_PRO_CI_NODE_BUILD_ID=${env.BUILD_TAG} ${knapsack_options} bundle exec rake knapsack_pro:queue:rspec"""
        }
      }
    }
  }

  parallel nodes // run CI nodes in parallel
}
```

下面是如何拆分 Cypress 测试的 JavaScript 示例。你可以在这里了解更多关于赛普拉斯测试赛跑者的[分裂 E2E 测试。](https://docs.knapsackpro.com/2018/run-javascript-e2e-tests-faster-with-cypress-on-parallel-ci-nodes)

```
timeout(time: 60, unit: 'MINUTES') {
  node() {
    stage('Checkout') {
      checkout([/* checkout code from git */])

      // determine git commit hash because we need to pass it to Knapsack Pro
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
            KNAPSACK_PRO_CI_NODE_BUILD_ID=${env.BUILD_TAG}\
        """

        // example how to run tests with Knapsack Pro
        stage('Run tests') {
          sh """${knapsack_options} $(npm bin)/knapsack-pro-cypress"""
        }
      }
    }
  }

  parallel nodes // run CI nodes in parallel
}
```

Jenkins Pipeline 为您提供了一个连续交付(CD)管道，它是您将软件从版本控制直接交付给用户的过程的自动化表达。通过快速运行 CI 构建来节省工程团队的时间非常重要。一种方法是[测试套件并行化，这可以用 backpack Pro 的 Ruby 和 JavaScript 测试](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=jenkins-pipeline-how-to-run-parallel-tests-in-your-workflow-stages)以最佳方式完成。

*最初发表于*[T5【docs.knapsackpro.com】](https://docs.knapsackpro.com/2018/jenkins-pipeline-how-to-run-parallel-tests-in-your-workflow-stages)*。*