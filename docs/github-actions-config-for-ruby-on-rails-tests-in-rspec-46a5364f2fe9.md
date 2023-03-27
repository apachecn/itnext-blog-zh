# RSpec 中 Ruby on Rails 测试的 Github 动作配置

> 原文：<https://itnext.io/github-actions-config-for-ruby-on-rails-tests-in-rspec-46a5364f2fe9?source=collection_archive---------3----------------------->

您是否在考虑将 Ruby on Rails 项目 CI 管道迁移到 Github Actions？您将学习如何配置 Rails 应用程序，以使用 Github 操作运行 RSpec 测试。

![](img/006fe8c445c820cc659d6583646ef72e.png)

本文介绍了 Github 操作 YAML 配置的一些内容:

*   如何使用`ruby/setup-ruby`动作用 bundler 安装 Ruby gems 并自动缓存 gems？这样，您就可以从缓存中为您的项目加载 Ruby gems，并快速运行 CI 构建。
*   如何在 Github 动作上使用 Postgres
*   如何在 Github 操作上使用 Redis
*   如何使用 Github Actions build matrix 运行并行作业，并跨多个作业执行 RSpec 测试以节省时间

# Rails 应用程序的 Github 操作 YML 配置

# 拼音/设置-拼音操作

[ruby/setup-ruby](https://github.com/ruby/setup-ruby) 是一个可以用来安装特定 ruby 编程语言版本的动作。它允许你根据你的 Gemfile.lock 开箱即用来缓存 Ruby gems。

建议[用](https://docs.knapsackpro.com/2021/how-to-load-ruby-gems-from-cache-on-github-actions) `[ruby/setup-ruby](https://docs.knapsackpro.com/2021/how-to-load-ruby-gems-from-cache-on-github-actions)` [代替过时的](https://docs.knapsackpro.com/2021/how-to-load-ruby-gems-from-cache-on-github-actions) `[actions/setup-ruby](https://docs.knapsackpro.com/2021/how-to-load-ruby-gems-from-cache-on-github-actions)`。

```
- name: Set up Ruby
  uses: ruby/setup-ruby@v1
  with:
    *# Not needed with a .ruby-version file*
    ruby-version: 2.7
    *# runs 'bundle install' and caches installed gems automatically*
    bundler-cache: true
```

# 如何在 Github 操作上配置 Postgres

要在 Github 操作上使用 Postgres，您需要为 Postgres 设置一个服务。我推荐使用额外的选项来配置 Postgres 使用 RAM 而不是磁盘。这样，您的数据库可以在测试环境中运行得更快。

在下面的配置中，我们还传递了进行健康检查的设置，以确保在开始运行测试之前数据库已经启动并运行。

```
*# If you need DB like PostgreSQL, Redis then define service below.*
*#* [*https://github.com/actions/example-services/tree/master/.github/workflows*](https://github.com/actions/example-services/tree/master/.github/workflows)
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
```

# 如何在 Github 操作上配置 Redis

您可以使用 Redis Docker 容器在 Github 操作上启动 Redis 服务器。看看这有多简单:

```
services:
  redis:
    image: redis
    ports:
      - 6379:6379
    options: --entrypoint redis-server
```

# 如何使用 Github 操作构建矩阵来运行并行作业测试

您可以在 Github Actions 中使用[构建矩阵](https://docs.github.com/en/actions/learn-github-actions/managing-complex-workflows#using-a-build-matrix)来同时运行多个作业。

您将需要在这些并行作业之间分割测试文件。为此，您可以使用带有[队列模式的背包 Pro 在任务](https://docs.knapsackpro.com/2020/how-to-speed-up-ruby-and-javascript-tests-with-ci-parallelisation)之间平均分配测试。通过这种方式，您可以确保在每个作业上执行适当数量的测试，并且在作业之间很好地平衡工作量。简单地说，这种方式可以确保 CI 构建尽可能快—它有最佳的执行时间。

```
- name: Run tests
  env:
    KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC }}
    KNAPSACK_PRO_CI_NODE_TOTAL: ${{ matrix.ci_node_total }}
    KNAPSACK_PRO_CI_NODE_INDEX: ${{ matrix.ci_node_index }}
    KNAPSACK_PRO_FIXED_QUEUE_SPLIT: true
    KNAPSACK_PRO_RSPEC_SPLIT_BY_TEST_EXAMPLES: true
    KNAPSACK_PRO_LOG_LEVEL: info
  run: |
    bundle exec rake knapsack_pro:queue:rspec
```

您可以看到，对于 RSpec，我们还使用了一个`knapsack_pro` Ruby gem 标志`KNAPSACK_PRO_RSPEC_SPLIT_BY_TEST_EXAMPLES`。它允许自动[检测缓慢的测试文件，并在并行作业之间分割它们](https://knapsackpro.com/faq/question/how-to-split-slow-rspec-test-files-by-test-examples-by-individual-it?utm_source=medium&utm_medium=blog_post&utm_campaign=medium-how-to-run-ruby-on-rails-tests-on-github-actions-using-rspec)。

您可以在另一篇文章中了解更多信息，这篇文章解释了[如何通过测试示例](https://docs.knapsackpro.com/2020/how-to-run-slow-rspec-files-on-github-actions-with-parallel-jobs-by-doing-an-auto-split-of-the-spec-file-by-test-examples)对 Spec 文件进行自动分割，在 Github Actions 上用并行作业运行慢速 RSpec 文件。

# Github 动作和 Ruby on Rails 项目的完整 YML 配置

下面是 Github 操作的 CI 管道的完整配置。您可以使用它来运行 Rails 项目的测试。

```
name: Main

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest

    *# If you need DB like PostgreSQL, Redis then define service below.*
    *#* [*https://github.com/actions/example-services/tree/master/.github/workflows*](https://github.com/actions/example-services/tree/master/.github/workflows)
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

    *#* [*https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix*](https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix)
    strategy:
      fail-fast: false
      matrix:
        *# [n] - where the n is a number of parallel jobs you want to run your tests on.*
        *# Use a higher number if you have slow tests to split them between more parallel jobs.*
        *# Remember to update the value of the `ci_node_index` below to (0..n-1).*
        ci_node_total: [8]
        *# Indexes for parallel jobs (starting from zero).*
        *# E.g. use [0, 1] for 2 parallel jobs, [0, 1, 2] for 3 parallel jobs, etc.*
        ci_node_index: [0, 1, 2, 3, 4, 5, 6, 7]

    env:
      RAILS_ENV: test
      GEMFILE_RUBY_VERSION: 2.7.2
      PGHOST: localhost
      PGUSER: postgres
      *# Rails verifies the time zone in DB is the same as the time zone of the Rails app*
      TZ: "Europe/Warsaw"

    steps:
      - uses: actions/checkout@v2

      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          *# Not needed with a .ruby-version file*
          ruby-version: 2.7
          *# runs 'bundle install' and caches installed gems automatically*
          bundler-cache: true

      - name: Create DB
        run: |
          bin/rails db:prepare
      - name: Run tests
        env:
          KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC }}
          KNAPSACK_PRO_CI_NODE_TOTAL: ${{ matrix.ci_node_total }}
          KNAPSACK_PRO_CI_NODE_INDEX: ${{ matrix.ci_node_index }}
          KNAPSACK_PRO_LOG_LEVEL: info
          *# if you use Knapsack Pro Queue Mode you must set below env variable*
          *# to be able to retry CI build and run previously recorded tests*
          *#* [*https://github.com/KnapsackPro/knapsack_pro-ruby#knapsack_pro_fixed_queue_split-remember-queue-split-on-retry-ci-node*](https://github.com/KnapsackPro/knapsack_pro-ruby#knapsack_pro_fixed_queue_split-remember-queue-split-on-retry-ci-node)
          KNAPSACK_PRO_FIXED_QUEUE_SPLIT: true
          *# RSpec split test files by test examples feature - it's optional*
          *#* [*https://knapsackpro.com/faq/question/how-to-split-slow-rspec-test-files-by-test-examples-by-individual-it*](https://knapsackpro.com/faq/question/how-to-split-slow-rspec-test-files-by-test-examples-by-individual-it)
          KNAPSACK_PRO_RSPEC_SPLIT_BY_TEST_EXAMPLES: true
        run: |
          bundle exec rake knapsack_pro:queue:rspec
```

# 摘要

您刚刚学习了如何在 Github Actions 上设置 Rails 应用程序。如果您将项目从不同的 CI 服务器迁移到 Github Actions，我希望这能对您有所帮助。

您可以了解更多关于[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=medium-how-to-run-ruby-on-rails-tests-on-github-actions-using-rspec)的内容，以及它如何帮助您在 CI 上使用并行作业快速运行测试。它可以与 RSpec、Cucumber、Minitest 和其他 Ruby 测试运行程序一起工作。[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=medium-how-to-run-ruby-on-rails-tests-on-github-actions-using-rspec)也可以与 JavaScript 测试运行器一起工作，并具有原生 API 集成。

![](img/836b68a76a3bc730b11c5ca28d82b0a5.png)

*原载于 2021 年 4 月 8 日 https://docs.knapsackpro.com*[](https://docs.knapsackpro.com/2021/how-to-run-ruby-on-rails-tests-on-github-actions-using-rspec)**。**