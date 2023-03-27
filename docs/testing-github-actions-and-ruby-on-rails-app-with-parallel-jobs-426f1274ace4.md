# 使用并行作业测试 GitHub 操作和 Ruby on Rails 应用程序

> 原文：<https://itnext.io/testing-github-actions-and-ruby-on-rails-app-with-parallel-jobs-426f1274ace4?source=collection_archive---------9----------------------->

GitHub 推出了他们自己的 CI 服务器解决方案，名为 GitHub Actions。您将学习如何使用 YAML 配置文件在 GitHub Actions 上设置 Ruby on Rails 应用程序。为了更快地运行 RSpec 测试套件，您将在 GitHub 操作上使用矩阵策略配置并行作业。

![](img/054668fba51942e3663aa7039fda2a18.png)

# 自动化 GitHub 操作的工作流程

GitHub Actions 利用世界一流的 CI/CD，轻松实现所有软件工作流程的自动化。通过简单的 YAML 配置，就可以直接从 GitHub 构建、测试和部署您的代码。

您甚至可以创建几个 YAML 配置文件，在您的配置项上运行一组不同的规则，如计划每日配置项构建。但是让我们严格关注如何在 GitHub 操作上运行 Rails 应用程序的测试。

# 用 YAML 配置在 GitHub 上设置 Ruby on Rails 动作

在您的项目存储库中，您需要创建文件`.github/workflows/main.yaml`,感谢它 GitHub 将运行您的 CI 构建。您可以在 GitHub 存储库的 *Actions* 选项卡中找到 CI 构建的结果。

在我们的例子中，Rails 应用程序有 Postgres 数据库，所以您需要用 docker 容器设置服务来运行 Postgres DB。

```
*# If you need DB like PostgreSQL then define service below.*
*# Example for Redis can be found here:*
*# https://github.com/actions/example-services/tree/master/.github/workflows*
services:
  postgres:
    image: postgres:10.8
    env:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ""
      POSTGRES_DB: postgres
    ports:
    *# will assign a random free host port*
    - 5432/tcp
    *# needed because the postgres container does not provide a healthcheck*
    options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
```

为了能够安装来自项目`Gemfile`的`pg` ruby gem，你将需要 Ubuntu 系统中的`libpq-dev`库，因此需要安装它。`libpq`是一组库函数，允许客户端程序向 PostgreSQL 后端服务器传递查询并接收这些查询的结果。我们需要它来编译`pg`宝石。下一步将是安装我们的红宝石宝石。

```
*# required to compile pg ruby gem*
- name: install PostgreSQL client
  run: sudo apt-get install libpq-dev

- name: Build and create DB
  env:
    *# use localhost for the host here because we have specified a container for the job.*
    *# If we were running the job on the VM this would be postgres*
    PGHOST: localhost
    PGUSER: postgres
    PGPORT: ${{ job.services.postgres.ports[5432] }} *# get randomly assigned published port*
    RAILS_ENV: test
  run: |
    gem install bundler
    bundle install --jobs 4 --retry 3
    bin/rails db:setup
```

要跨并行作业运行 RSpec 测试，您需要设置矩阵特性，并因此运行跨作业分布的整个测试套件。

# 配置构建矩阵

构建矩阵为要测试的虚拟环境提供了不同的配置。例如，一个工作流可以为一种语言、操作系统等的多个支持版本运行一个作业。对于每个配置，都会有一个作业副本运行并报告状态。

在运行并行测试的情况下，您希望在相同的 Ruby 版本和 Ubuntu 系统上运行 Rails 应用程序。但是您希望将 RSpec 测试套件分成 2 组，这样一半的测试将进行到第一个并行作业，另一半将进行到另一个作业。

为了分割测试，你可以使用 Ruby gem[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=how-to-run-rspec-on-github-actions-for-ruby-on-rails-app-using-parallel-jobs)来动态分割并行 GitHub 任务中的测试。由于每个并行作业将消耗一组从 backpack Pro API 队列获取的测试，以确保每个并行作业在相似的时间完成工作。这允许均匀分布的测试，并且在并行作业中没有瓶颈(没有缓慢的作业)。您的配置项构建将尽可能快。

在我们的例子中，您将测试分成两个并行的任务，因此您需要将 2 设置为`matrix.ci_node_total`。那么每个并行作业应该从 0 开始为`matrix.ci_node_index`分配索引。第一个并行作业的索引为 0，第二个作业的索引为 1。这允许[backpackage Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=how-to-run-rspec-on-github-actions-for-ruby-on-rails-app-using-parallel-jobs)知道在一个特定的任务上应该执行什么样的测试。

```
*# https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix*
strategy:
  fail-fast: false
  matrix:
    *# Set N number of parallel jobs you want to run tests on.*
    *# Use higher number if you have slow tests to split them on more parallel jobs.*
    *# Remember to update ci_node_index below to 0..N-1*
    ci_node_total: [2]
    *# set N-1 indexes for parallel jobs*
    *# When you run 2 parallel jobs then first job will have index 0,*
    *# the second job will have index 1 etc*
    ci_node_index: [0, 1]
```

您还需要为[背包 Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=how-to-run-rspec-on-github-actions-for-ruby-on-rails-app-using-parallel-jobs) 指定 API 令牌，对于 RSpec，它将是`KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC`。

然后，您可以使用 backpack Pro 在队列模式下为 RSpec 运行测试:

`bundle exec rake knapsack_pro:queue:rspec`

# Rails 测试的完整 GitHub 动作配置文件

在这里你可以找到 GitHub 动作和 Ruby on Rails 项目的完整 YAML 配置文件。

我还录制了一段视频，展示了它是如何工作的，以及如何在 GitHub Actions 上配置 CI 构建并行作业。

GitHub Actions + Rails app 并行测试

```
 # .github/workflows/main.yaml
name: Main

on: [push]

jobs:
  vm-job:
    runs-on: ubuntu-latest

    # If you need DB like PostgreSQL then define service below.
    # Example for Redis can be found here:
    # https://github.com/actions/example-services/tree/master/.github/workflows
    services:
      postgres:
        image: postgres:10.8
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: ""
          POSTGRES_DB: postgres
        ports:
        # will assign a random free host port
        - 5432/tcp
        # needed because the postgres container does not provide a healthcheck
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5

    # https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix
    strategy:
      fail-fast: false
      matrix:
        # Set N number of parallel jobs you want to run tests on.
        # Use higher number if you have slow tests to split them on more parallel jobs.
        # Remember to update ci_node_index below to 0..N-1
        ci_node_total: [2]
        # set N-1 indexes for parallel jobs
        # When you run 2 parallel jobs then first job will have index 0, the second job will have index 1 etc
        ci_node_index: [0, 1]

    steps:
    - uses: actions/checkout@v1

    - name: Set up Ruby 2.6
      uses: actions/setup-ruby@v1
      with:
        ruby-version: 2.6.3

    # required to compile pg ruby gem
    - name: install PostgreSQL client
      run: sudo apt-get install libpq-dev

    - name: Build and create DB
      env:
        # use localhost for the host here because we have specified a container for the job.
        # If we were running the job on the VM this would be postgres
        PGHOST: localhost
        PGUSER: postgres
        PGPORT: ${{ job.services.postgres.ports[5432] }} # get randomly assigned published port
        RAILS_ENV: test
      run: |
        gem install bundler
        bundle install --jobs 4 --retry 3
        bin/rails db:setup

    - name: Run tests
      env:
        PGHOST: localhost
        PGUSER: postgres
        PGPORT: ${{ job.services.postgres.ports[5432] }} # get randomly assigned published port
        RAILS_ENV: test
        KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC }}
        KNAPSACK_PRO_TEST_SUITE_TOKEN_CUCUMBER: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_CUCUMBER }}
        KNAPSACK_PRO_TEST_SUITE_TOKEN_MINITEST: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_MINITEST }}
        KNAPSACK_PRO_TEST_SUITE_TEST_UNIT: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_TEST_UNIT }}
        KNAPSACK_PRO_TEST_SUITE_TOKEN_SPINACH: ${{ secrets.KNAPSACK_PRO_TEST_SUITE_TOKEN_SPINACH }}
        KNAPSACK_PRO_CI_NODE_TOTAL: ${{ matrix.ci_node_total }}
        KNAPSACK_PRO_CI_NODE_INDEX: ${{ matrix.ci_node_index }}
      run: |
        # run tests in Knapsack Pro Regular Mode
        bundle exec rake knapsack_pro:rspec
        bundle exec rake knapsack_pro:cucumber
        bundle exec rake knapsack_pro:minitest
        bundle exec rake knapsack_pro:test_unit
        bundle exec rake knapsack_pro:spinach

        # you can use Knapsack Pro in Queue Mode once recorded first CI build with Regular Mode
        bundle exec rake knapsack_pro:queue:rspec
        bundle exec rake knapsack_pro:queue:cucumber
        bundle exec rake knapsack_pro:queue:minitest
```

源代码:[https://gist . github . com/ArturT/a 35284 f 34 c 2d c0b 61 a0ad 2 B4 DD 4 bacae](https://gist.github.com/ArturT/a35284f34c2dc0b61a0ad2b4dd4bacae)

# 队列模式下的动态测试套件拆分

如果你想更好地理解背包 Pro 中的队列模式是如何工作的，以及它还能解决什么问题，你可以在下面的视频中找到一些有用的信息。

我希望这对你有所帮助。

*原载于 2019 年 9 月 16 日*[*https://docs.knapsackpro.com*](https://docs.knapsackpro.com/2019/how-to-run-rspec-on-github-actions-for-ruby-on-rails-app-using-parallel-jobs)*。*