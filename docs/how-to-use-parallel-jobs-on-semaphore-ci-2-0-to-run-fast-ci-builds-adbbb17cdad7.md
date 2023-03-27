# 如何在 Semaphore CI 2.0 上使用并行作业来运行快速 CI 构建

> 原文：<https://itnext.io/how-to-use-parallel-jobs-on-semaphore-ci-2-0-to-run-fast-ci-builds-adbbb17cdad7?source=collection_archive---------7----------------------->

Semaphore CI 2.0 允许用并行作业配置您的 CI 构建任务。通过这种方式，您可以同时运行几个互不依赖的不同命令。但是我们也可以使用并行作业将您的测试套件分割成几个作业，这样可以节省时间。我将向您展示如何为 Ruby 或 JavaScript 项目(Rails / Node 项目)加速 CI 构建。

![](img/8c169eb4ed8d65c9afdb1a488e68d725.png)

信号量 CI 2.0

使用 Semaphore CI 2.0 ，您不必像在其他 CI 提供者中那样为可以并行运行的容器数量付费。相反，他们计算运行容器所花费的工作时间。这就产生了运行更多并行任务的动机，以便快速执行我们的测试，并且仍然将 bill 保持在一个相似的水平，就好像我们只是在一个容器中运行所有的测试，浪费了我们自己的时间。

# 让我们通过并行作业节省时间

为了用我们的测试以最佳方式运行并行作业，我们需要确保每个作业在相似的时间完成工作。这样就不会出现像作业执行太多测试或太慢测试这样的瓶颈。缓慢的工作可能会影响并使我们的整个 CI 构建更慢。尤其是端到端测试(E2E)可能会非常慢，并且它们的执行时间可能会有所不同。

您可以使用[背包 Pro 队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=run-parallel-jobs-on-semaphore-ci-2-0-to-get-faster-ci-build-time)，以动态的方式在并行任务之间分割测试，以确保所有任务在相似的时间完成工作。您可以在本文最后的视频中了解更多关于队列模式可以解决的其他问题，但是现在让我们跳到信号量 CI 2.0 演示示例和我们可以使用的配置示例。

在这里，您可以找到用于项目的信号量 CI 2.0 配置，使用:

*   [Ruby on Rails](https://docs.knapsackpro.com/2019/run-parallel-jobs-on-semaphore-ci-2-0-to-get-faster-ci-build-time#ruby-on-rails-config-for-semaphore-20) (RSpec，也支持其他测试运行程序，如 Minitest、Cucumber 等等)
*   Cypress.io 中的 JavaScript 测试端到端测试运行程序
*   [玩笑式的 JavaScript 测试](https://docs.knapsackpro.com/2019/run-parallel-jobs-on-semaphore-ci-2-0-to-get-faster-ci-build-time#jest-config-for-semaphore-20)

# Semaphore 2.0 的 Ruby on Rails 配置

gem 支持 Semaphore CI 2.0 提供的环境变量来运行您的测试。您必须在`.semaphore/semaphore.yml`配置文件中定义一些东西。

*   你需要设置`KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC`。如果你不想在 yml 文件中提交秘密，那么你可以[遵循这个指南](https://docs.semaphoreci.com/article/66-environment-variables-and-secrets)。
*   您需要创建尽可能多的具有唯一名称的作业(节点 0 —背包专业版、节点 1 —背包专业版等),以运行尽可能多的并行作业。如果您的测试套件很长，您应该使用更多的并行作业。
*   如果您有 2 个并行任务，您需要为每个任务设置`KNAPSACK_PRO_CI_NODE_TOTAL=2`。
*   您需要为节点 0 设置从 0 开始的作业索引，如`KNAPSACK_PRO_CI_NODE_INDEX=0`。

下面你可以找到 Rails 项目的完整信号量 CI 2.0 配置。

```
*# .semaphore/semaphore.yml*
*# Use the latest stable version of Semaphore 2.0 YML syntax:*
version: v1.0

*# Name your pipeline. In case you connect multiple pipelines with promotions,*
*# the name will help you differentiate between, for example, a CI build phase*
*# and delivery phases.*
name: Demo Rails 5 app

*# An agent defines the environment in which your code runs.*
*# It is a combination of one of available machine types and operating*
*# system images.*
*# See https://docs.semaphoreci.com/article/20-machine-types*
*# and https://docs.semaphoreci.com/article/32-ubuntu-1804-image*
agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu1804

*# Blocks are the heart of a pipeline and are executed sequentially.*
*# Each block has a task that defines one or more jobs. Jobs define the*
*# commands to execute.*
*# See https://docs.semaphoreci.com/article/62-concepts*
blocks:
  - name: Setup
    task:
      env_vars:
        - name: RAILS_ENV
          value: test
      jobs:
        - name: bundle
          commands:
          *# Checkout code from Git repository. This step is mandatory if the*
          *# job is to work with your code.*
          *# Optionally you may use --use-cache flag to avoid roundtrip to*
          *# remote repository.*
          *# See https://docs.semaphoreci.com/article/54-toolbox-reference#libcheckout*
          - checkout
          *# Restore dependencies from cache.*
          *# Read about caching: https://docs.semaphoreci.com/article/54-toolbox-reference#cache*
          - cache restore gems-$SEMAPHORE_GIT_BRANCH-$(checksum Gemfile.lock),gems-$SEMAPHORE_GIT_BRANCH-,gems-master-
          *# Set Ruby version:*
          - sem-version ruby 2.6.1
          - bundle install --jobs=4 --retry=3 --path vendor/bundle
          *# Store the latest version of dependencies in cache,*
          *# to be used in next blocks and future workflows:*
          - cache store gems-$SEMAPHORE_GIT_BRANCH-$(checksum Gemfile.lock) vendor/bundle

  - name: RSpec tests
    task:
      env_vars:
        - name: RAILS_ENV
          value: test
        - name: PGHOST
          value: 127.0.0.1
        - name: PGUSER
          value: postgres
        - name: KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC
          value: your_api_token_here
      *# This block runs two jobs in parallel and they both share common*
      *# setup steps. We can group them in a prologue.*
      *# See https://docs.semaphoreci.com/article/50-pipeline-yaml#prologue*
      prologue:
        commands:
          - checkout
          - cache restore gems-$SEMAPHORE_GIT_BRANCH-$(checksum Gemfile.lock),gems-$SEMAPHORE_GIT_BRANCH-,gems-master-
          *# Start Postgres database service.*
          *# See https://docs.semaphoreci.com/article/54-toolbox-reference#sem-service*
          - sem-service start postgres
          - sem-version ruby 2.6.1
          - bundle install --jobs=4 --retry=3 --path vendor/bundle
          - bundle exec rake db:setup

      jobs:
      - name: Node 0 - Knapsack Pro
        commands:
          - KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=0 bundle exec rake knapsack_pro:queue:rspec

      - name: Node 1 - Knapsack Pro
        commands:
          - KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=1 bundle exec rake knapsack_pro:queue:rspec
```

# Semaphore 2.0 的 Cypress.io 配置

`@knapsack-pro/cypress`支持 Semaphore CI 2.0 提供的环境变量来运行您的测试。你必须在`.semaphore/semaphore.yml`配置文件中定义一些东西。

*   你需要设置`KNAPSACK_PRO_TEST_SUITE_TOKEN_CYPRESS`。如果你不想在 yml 文件中提交秘密，那么你可以[遵循这个指南](https://docs.semaphoreci.com/article/66-environment-variables-and-secrets)。
*   您需要创建尽可能多的具有唯一名称的作业(节点 0 —背包专业版、节点 1 —背包专业版等),以运行尽可能多的并行作业。如果您的测试套件很长，您应该使用更多的并行作业。
*   如果您有 2 个并行任务，您需要为每个任务设置`KNAPSACK_PRO_CI_NODE_TOTAL=2`。
*   您需要像`KNAPSACK_PRO_CI_NODE_INDEX=0`一样为节点 0 设置从 0 开始的作业索引。

下面你可以找到信号量 CI 2.0 配置的示例部分。

```
blocks:
  - name: Cypress tests
    task:
      env_vars:
        - name: KNAPSACK_PRO_TEST_SUITE_TOKEN_CYPRESS
          value: your_api_token_here
      prologue:
        commands:
          - checkout
          - nvm install --lts carbon
          - sem-version node --lts carbon

      jobs:
        - name: Node 0 - Knapsack Pro
          commands:
            - KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=0 $(npm bin)/knapsack-pro-cypress

        - name: Node 1 - Knapsack Pro
          commands:
            - KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=1 $(npm bin)/knapsack-pro-cypress
```

# 信号量 2.0 的 Jest 配置

`@knapsack-pro/jest`支持 Semaphore CI 2.0 提供的环境变量来运行您的测试。您必须在`.semaphore/semaphore.yml`配置文件中定义一些东西。

*   你需要设置`KNAPSACK_PRO_TEST_SUITE_TOKEN_JEST`。如果你不想在 yml 文件中提交秘密，那么你可以[遵循这个指南](https://docs.semaphoreci.com/article/66-environment-variables-and-secrets)。
*   您需要创建尽可能多的具有唯一名称的作业(节点 0 —背包专业版、节点 1 —背包专业版等),以运行尽可能多的并行作业。如果您的测试套件很长，您应该使用更多的并行作业。
*   如果您有 2 个并行作业，您需要为每个作业设置`KNAPSACK_PRO_CI_NODE_TOTAL=2`。
*   您需要为节点 0 设置从 0 开始的作业索引，就像`KNAPSACK_PRO_CI_NODE_INDEX=0`一样。

下面你可以找到信号量 CI 2.0 配置的示例部分。

```
blocks:
  - name: Cypress tests
    task:
      env_vars:
        - name: KNAPSACK_PRO_TEST_SUITE_TOKEN_JEST
          value: your_api_token_here
      prologue:
        commands:
          - checkout
          - nvm install --lts carbon
          - sem-version node --lts carbon

      jobs:
        - name: Node 0 - Knapsack Pro
          commands:
            - KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=0 $(npm bin)/knapsack-pro-jest

        - name: Node 1 - Knapsack Pro
          commands:
            - KNAPSACK_PRO_CI_NODE_TOTAL=2 KNAPSACK_PRO_CI_NODE_INDEX=1 $(npm bin)/knapsack-pro-jest
```

如您所见，由于利用了 Semaphore CI 2.0 上的并行作业，您的 CI 构建可以快得多。您可以查看[背包 Pro CI 并行化工具](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=run-parallel-jobs-on-semaphore-ci-2-0-to-get-faster-ci-build-time)，并在下面的视频中了解更多关于队列模式及其解决的问题。

【docs.knapsackpro.com】最初发表于[](https://docs.knapsackpro.com/2019/run-parallel-jobs-on-semaphore-ci-2-0-to-get-faster-ci-build-time)**。**