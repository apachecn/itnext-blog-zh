# 通过测试示例使用 Spec 文件的自动分割，在带有并行作业的 Github 操作上运行缓慢的 RSpec 文件

> 原文：<https://itnext.io/run-slow-rspec-files-on-github-actions-with-parallel-jobs-using-an-auto-split-of-the-spec-file-by-6bee53e31fb4?source=collection_archive---------7----------------------->

将您的 CI 构建任务在多台并行运行的机器之间进行分割是一个让您的 CI 测试更快的好方法，这将为构建特性带来更多的时间。Github Actions 允许轻松运行并行作业。在上一篇文章中，我们解释了如何使用 backpack Pro 在 GitHub Actions 上的并行作业之间[有效地分割 RSpec 测试文件。今天，我们将展示如何解决缓慢的测试文件对整个构建时间产生负面影响的问题。](https://docs.knapsackpro.com/2019/how-to-run-rspec-on-github-actions-for-ruby-on-rails-app-using-parallel-jobs)

![](img/ce3f03555cf9eb52efb4c50ffda699bd.png)

# 考虑一下分裂

假设您有一个包含 30 个 RSpec 规范文件的项目。每个文件包含多个测试实例。大多数文件都是相当健壮、快速的单元测试。假设还有一些较慢的文件，比如特性规格。也许一个这样的特征规格文件需要大约 5 分钟来执行。

当我们在不同的并行机器上运行不同的 spec 文件时，我们努力在所有机器上获得相似的执行时间。在一个描述的场景中，即使我们运行 30 个并行的任务(每个任务只运行一个测试文件)，5 分钟的特性规范也会成为整个构建的瓶颈。29 台机器可能在几秒钟内完成它们的工作，但是直到剩下的 1 个节点执行完它的文件，整个构建才算完成。

# 分步解决

为了解决测试文件速度慢的问题，我们需要拆分文件中的内容。我们可以重构它，并确保测试实例存在于单独的、更小的测试文件中。但是这有两个问题:

首先，它需要工作。尽管在所描述的场景中确实很合理，但在现实生活中，通常不仅仅是一个文件引起问题。通常有许多缓慢而复杂的测试文件，它们有自己复杂的设置，比如嵌套的`before`块、`let`块等等。我们都见过他们(可能是他们以这种方式结束的原因)，不是吗？；-)像这样重构文件一点也不好玩，而且似乎总是有更多更重要的工作要做，至少从我们的经验来看是这样。

其次，我们认为代码组织应该基于其他考虑。如何创建文件和类很可能是遵循项目中商定的一些方法的结果。将类分成更小的类，以便 CI 构建可以运行得更快，这违反了您的惯例。对一些人来说，这可能比其他人更令人不安，但我们认为最好避免这种情况是公平的——如果有更好的方法来达到同样的目的…

# 输入按测试实例划分

如您所知，RSpec 允许我们运行单个示例，而不是整个文件。我们决定利用这一点，通过从速度较慢的文件中收集关于单个例子的信息来解决瓶颈测试文件的问题。然后，这样的例子被动态地分布在并行节点之间，并单独运行，因此没有一个单独的文件会成为整个构建的瓶颈。重要的是，不需要额外的工作——这是由`knapsack_pro` gem 自动完成的。每个示例都在正确的上下文中运行，就像运行整个文件一样。

如果你已经在队列模式下使用[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-slow-rspec-files-on-github-actions-with-parallel-jobs-by-doing-an-auto-split-of-the-spec-file-by-test-examples)，你可以通过添加一个 ENV 变量到你的 GitHub Actions 工作流配置中来启用这个特性:`KNAPSACK_PRO_RSPEC_SPLIT_BY_TEST_EXAMPLES: true`(请确保你运行的是最新版本的`knapsack_pro` gem)。经过几次运行后，背包专业版将开始自动分裂你最慢的测试文件的个别例子。

下面是使用 RSpec 的 Rails 项目的 GitHub Actions 工作流配置的完整示例:

```
*# .github/workflows/main.yaml*

name: Main

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest

    *# If you need DB like PostgreSQL, Redis then define service below.*
    *# https://github.com/actions/example-services/tree/master/.github/workflows*
    services:
      postgres:
        image: postgres:10.8
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: ""
          POSTGRES_DB: postgres
        ports:
          - 5432:5432
        *# needed because the postgres container does not provide a healthcheck*
        *# tmpfs makes DB faster by using RAM*
        options: >-
          --mount type=tmpfs,destination=/var/lib/postgresql/data
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis
        ports:
          - 6379:6379
        options: --entrypoint redis-server

    *# https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix*
    strategy:
      fail-fast: false
      matrix:
        *# Set N number of parallel jobs you want to run tests on.*
        *# Use higher number if you have slow tests to split them on more parallel jobs.*
        *# Remember to update ci_node_index below to 0..N-1*
        ci_node_total: [8]
        *# set N-1 indexes for parallel jobs*
        *# When you run 2 parallel jobs then first job will have index 0, the second job will have index 1 etc*
        ci_node_index: [0, 1, 2, 3, 4, 5, 6, 7]

    steps:
      - uses: actions/checkout@v2

      - name: Set up Ruby
        uses: actions/setup-ruby@v1
        with:
          ruby-version: 2.6

      - uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      - name: Bundle install
        env:
          RAILS_ENV: test
        run: |
          bundle config path vendor/bundle
          bundle install --jobs 4 --retry 3
      - name: Create DB
        env:
          *# use localhost for the host here because we have specified a container for the job.*
          *# If we were running the job on the VM this would be postgres*
          PGHOST: localhost
          PGUSER: postgres
          RAILS_ENV: test
        run: |
          bin/rails db:prepare
      - name: Run tests
        env:
          PGHOST: localhost
          PGUSER: postgres
          RAILS_ENV: test
          KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC }}
          KNAPSACK_PRO_CI_NODE_TOTAL: ${{ matrix.ci_node_total }}
          KNAPSACK_PRO_CI_NODE_INDEX: ${{ matrix.ci_node_index }}
          KNAPSACK_PRO_FIXED_QUEUE_SPLIT: true
          KNAPSACK_PRO_RSPEC_SPLIT_BY_TEST_EXAMPLES: true
          KNAPSACK_PRO_LOG_LEVEL: info
        run: |
          bundle exec rake knapsack_pro:queue:rspec
```

你可以在下面的视频中找到更多细节:

请在评论中告诉我们你对这个解决方案的看法。如果你想试试这个设置，你也可以参考我们的 FAQ 条目，解释[如何分割慢速 RSpec 测试文件](https://knapsackpro.com/faq/question/how-to-split-slow-rspec-test-files-by-test-examples-by-individual-it?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-slow-rspec-files-on-github-actions-with-parallel-jobs-by-doing-an-auto-split-of-the-spec-file-by-test-examples)。

一如既往，如果您在项目中遇到任何配置 GitHub 操作的问题，请不要犹豫，我们很乐意帮助您！

*原载于 2020 年 6 月 15 日 https://docs.knapsackpro.com**[*。*](https://docs.knapsackpro.com/2020/how-to-run-slow-rspec-files-on-github-actions-with-parallel-jobs-by-doing-an-auto-split-of-the-spec-file-by-test-examples)*