# 如何使用 Jenkins 并行管道拆分 RSpec 测试以运行更快的 CI 构建

> 原文：<https://itnext.io/how-to-split-rspec-tests-with-jenkins-parallel-pipeline-to-run-faster-ci-builds-17955692c16e?source=collection_archive---------6----------------------->

Jenkins CI 服务器有一个声明性管道，允许您设置 Jenkins 并行阶段。您可以使用阶段来同时运行它们(并行运行),以在几个更小更快的块中执行您的 RSpec 测试套件，而不是一个长时间的测试套件运行。

![](img/5b72206d4ae33d4e546cca4142c68fa0.png)

RSpec + Jenkins

# 与 Jenkins 工作流管道并行运行阶段

您将使用 Jenkinsfile 和管道语法来并行执行任务。RSpec 测试需要在各个阶段中以相等的时间分割，要做到这一点，您需要确保每个 RSpec 规范文件的时间不会在其中一个阶段上混合，因为这可能会导致瓶颈——一个比其他阶段花费更多时间来运行测试的阶段。

为了平均分割 RSpec 测试，您可以使用[backpack _ pro gem](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=split-rspec-tests-with-jenkins-parallel-pipeline-to-run-specs-faster)及其队列模式，该模式将动态分割并行 Jenkins 阶段的规格，以确保每个阶段花费相似的时间。你可以从下面的视频中了解更多关于[背包 _ 专业队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=split-rspec-tests-with-jenkins-parallel-pipeline-to-run-specs-faster)以及当你在 CI 服务器上拆分测试时它解决了什么样的边缘情况。

# 声明性 Jenkins 管道中的阶段

在 Jenkins 中，管道作为代码是定义配置的一种非常流行的方式。这里有一个用[背包 _pro ruby gem](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=split-rspec-tests-with-jenkins-parallel-pipeline-to-run-specs-faster) 运行 RSpec 测试的例子，以确保你的 Ruby on Rails 测试在并行阶段中以最佳方式分割。

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

        // Example how to run RSpec tests in Knapsack Pro Queue Mode
        // Queue Mode should be a last stage if you have other stages in your pipeline
        // thanks to that it can autobalance CI build time if other tests were not perfectly distributed
        stage('Run rspec') {
          sh """KNAPSACK_PRO_CI_NODE_BUILD_ID=${env.BUILD_TAG} ${knapsack_options} bundle exec rake knapsack_pro:queue:rspec"""
        }
      }
    }
  }

  parallel nodes // run CI nodes in parallel
}
```

# 摘要

自动拆分测试以加速测试阶段是确保您的 Ruby on Rails 项目 CI 构建最终尽可能快地完成的一种方式。您可以了解其他 Ruby 或 JavaScript 测试运行程序的 [Jenkins 并行管道，如 Cypress 或 Jest](https://docs.knapsackpro.com/2018/jenkins-pipeline-how-to-run-parallel-tests-in-your-workflow-stages) ，以及 CI 并行化如何通过更快的测试节省时间。

— — —

最初发布于:[https://docs . knapsackpro . com/2019/split-rspec-tests-with-Jenkins-parallel-pipeline-to-run-specs-faster](https://docs.knapsackpro.com/2019/split-rspec-tests-with-jenkins-parallel-pipeline-to-run-specs-faster)