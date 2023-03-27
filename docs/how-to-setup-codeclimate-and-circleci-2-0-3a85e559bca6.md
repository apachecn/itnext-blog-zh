# 如何设置 CodeClimate 和 CircleCI 2.0 并行版本以使用 SimpleCov 和 JUnit 格式化程序运行 RSpec

> 原文：<https://itnext.io/how-to-setup-codeclimate-and-circleci-2-0-3a85e559bca6?source=collection_archive---------7----------------------->

如何为你的 **RSpec** 测试套件合并 **CodeClimate** 报告，该测试套件在 CircleCI 2.0 上执行并行构建？您将学习如何使用 CircleCI 为您的 for **Ruby on Rails** 项目运行 RSpec 并行测试，以及如何将并行作业合并的测试覆盖发送到 CodeClimate。我们将介绍以下配置示例:

![](img/6686cf507a4e3afb041211cade572999.png)

*   如何使用 CodeClimate Test Reporter 所需的 **SimpleCov** 来准备每个并行作业上的 RSpec 测试覆盖摘要，然后如何合并它，以便您能够将其发送到 CodeClimate dashboard。
*   如何使用并行作业上运行的 RSpec 的 **JUnit** 格式化程序。多亏了 JUnit formatter，您可以在 CircleCI UI 视图中看到漂亮的测试结果。例如，当您的测试失败时，CircleCI 会在 CI 构建步骤的顶部显示失败的测试。
*   如何[使用 backpack Pro 队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=codeclimate-and-circleci-2-0-parallel-builds-config-for-rspec-with-simplecov-and-junit-formatter)将您的 RSpec 测试拆分到并行作业中。在文章的最后，您将看到一个视频，解释 backpackage _ pro ruby gem 中的队列模式如何在并行作业之间动态分配规格，以确保每个作业花费相似的时间，从而确保 CI 构建尽可能快(以获得最佳的 CI 构建时间)。

# 基于 CircleCI 2.0 的 CodeClimate 和并行构建

让我们从 CircleCI 2.0 的完整 YAML 配置开始。您会发现每个重要步骤的注释及其作用。您可能已经在项目中配置了一些东西，所以查看下面的完整示例可能对您更熟悉。如果你是第一次将 CodeClimate 添加到你的项目中，那么除了下面的配置之外，你需要设置 simplecov gem，这将在下一节中介绍。

下面是使用 RSpec 和 CodeClimate 进行并行构建的完整 CircleCI 2.0 示例。

```
*# .circleci/config.yml*
version: 2
jobs:
  build:
    *# set here how many parallel jobs you want to run.*
    *# more parallel jobs the faster is your CI build*
    parallelism: 2
    docker:
      *# specify the version you desire here*
      - image: circleci/ruby:2.6.3-node-browsers
        environment:
          PGHOST: 127.0.0.1
          PGUSER: rails-app-with-knapsack_pro
          RAILS_ENV: test
          RACK_ENV: test

          *# API token should be set in CircleCI environment variables settings instead of here*
          *# KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC: rspec-token*

      *# Specify service dependencies here if necessary*
      *# CircleCI maintains a library of pre-built images*
      *# documented at https://circleci.com/docs/2.0/circleci-images/*
      - image: circleci/postgres:9.4.12-alpine
        environment:
          POSTGRES_DB: rails-app-with-knapsack_pro_test
          POSTGRES_PASSWORD: ""
          POSTGRES_USER: rails-app-with-knapsack_pro

    working_directory: ~/repo

    steps:
      - checkout

      *# create directory for xml reports created by junit formatter*
      - run: mkdir -p tmp/test-reports/rspec/queue_mode/

      - run:
          name: Install Code Climate Test Reporter
          command: |
            curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./cc-test-reporter
            chmod +x ./cc-test-reporter
      - run: ./cc-test-reporter before-build

      *# Download and cache dependencies*
      - restore_cache:
          keys:
          - v2-dependencies-bundler-{{ checksum "Gemfile.lock" }}
          *# fallback to using the latest cache if no exact match is found*
          - v2-dependencies-bundler-

      - run:
          name: install ruby dependencies
          command: |
            bundle install --jobs=4 --retry=3 --path vendor/bundle

      - save_cache:
          paths:
            - ./vendor/bundle
          key: v2-dependencies-bundler-{{ checksum "Gemfile.lock" }}

      *# wait for postgres to be available*
      - run: dockerize -wait tcp://localhost:5432 -timeout 1m
      *# Database setup*
      - run: bin/rails db:setup

      *# Run RSpec tests with knapsack_pro Queue Mode and use junit formatter*
      *# junit formatter must be configured as described in FAQ for knapsack_pro Queue Mode*
      *# this is also described in this article later*
      *# https://github.com/KnapsackPro/knapsack_pro-ruby#how-to-use-junit-formatter-with-knapsack_pro-queue-mode*
      - run: bundle exec rake "knapsack_pro:queue:rspec[--format documentation --format RspecJunitFormatter --out tmp/test-reports/rspec/queue_mode/rspec.xml]"

      - run:
          name: Code Climate Test Coverage
          command: |
            ./cc-test-reporter format-coverage -t simplecov -o "coverage/codeclimate.$CIRCLE_NODE_INDEX.json"

      *# store coverage directory with CodeClimate reports prepared based on simplecov reports*
      *# it's special step used to persist a temporary file to be used by another job in the workflow*
      - persist_to_workspace:
          root: coverage
          paths:
            - codeclimate.*.json

      *# store test reports created with junit formatter in order to allow CircleCI* 
      *# show info about executed tests in UI on top of CI build steps*
      - store_test_results:
          path: tmp/test-reports

      *# store test reports created with junit formatter in order to allow CircleCI* 
      *# let you browse recorded xml files in Artifacts tab*
      - store_artifacts:
          path: tmp/test-reports

  upload-coverage:
    docker:
      - image: circleci/ruby:2.6.3-node
    environment:
      *# you can add your CodeClimate test report ID here or in CircleCI*
      *# settings for environment variables*
      CC_TEST_REPORTER_ID: use-here-your-codeclimate-test-report-id
    working_directory: ~/repo

    steps:
      *# This will restore files from persist_to_workspace step*
      *# Thanks to it we will have access to CodeClimate test coverage files from*
      *# each parallel job. We need them in order to merge it into one file in next step.*
      - attach_workspace:
          at: ~/repo
      - run:
          name: Install Code Climate Test Reporter
          command: |
            curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./cc-test-reporter
            chmod +x ./cc-test-reporter
      - run:
          *# merge CodeClimate files from each parallel job into sum coverage*
          *# and then upload it to CodeClimate dashboard*
          command: |
            ./cc-test-reporter sum-coverage --output - codeclimate.*.json | ./cc-test-reporter upload-coverage --debug --input -

workflows:
  version: 2

  commit:
    jobs:
      *# run our CI build with tests*
      - build
      *# once CI build is completed then we merge CodeClimate reports*
      *# from each parallel job and upload summary coverage to CodeClimate*
      - upload-coverage:
          requires:
             - build
```

# RSpec 的 SimpleCov 配置

当您使用 [simplecov](https://github.com/colszowka/simplecov) gem 来为 RSpec 创建测试覆盖时，当您想要在许多 CircleCI 作业上并行运行测试时，您需要记住另外一件事情。使用`SimpleCov.command_name`为 simplecov 报告设置一个唯一的名称。

```
*# spec/rails_helper.rb or spec/spec_helper.rb*
require 'simplecov'
SimpleCov.**start**

*# this is needed when you use knapsack_pro Queue Mode*
KnapsackPro**::**Hooks**::**Queue.**before_queue** **do**
  SimpleCov.**command_name**("rspec_ci_node_#{KnapsackPro**::**Config**::**Env.**ci_node_index**}")
**end**
```

# RSpec 的 JUnit 格式化程序

为了在 CircleCI UI 中显示关于您的测试套件的信息，比如失败的测试，您需要使用 JUnit formatter 为您的 RSpec 测试套件生成 xml 报告。

由于 gem[RSpec _ JUnit _ formatter](https://github.com/sj26/rspec_junit_formatter)，您可以为 RSpec 使用 junit formatter。

```
*# knapsack_pro Queue Mode*
bundle exec rake "knapsack_pro:queue:rspec[--format documentation --format RspecJunitFormatter --out tmp/test-reports/rspec/queue_mode/rspec.xml]"
```

xml 报告将包含在基于工作队列的中间测试子集运行中执行的所有测试。您需要在子集队列钩子之后添加一个钩子来将`rspec.xml`重命名为`rspec_final_results.xml`,因为最终的结果文件将只包含一个 xml 标签，所有的测试都在 CI 节点上执行。这与队列模式的工作方式有关。详细解释在[刊](https://github.com/KnapsackPro/knapsack_pro-ruby/issues/40)中。

```
*# spec_helper.rb or rails_helper.rb*

*# TODO This must be the same path as value for rspec --out argument*
*# Note the path should not contain sign ~, for instance path ~/project/tmp/rspec.xml may not work. Please use full path instead.*
TMP_RSPEC_XML_REPORT **=** 'tmp/test-reports/rspec/queue_mode/rspec.xml'
*# move results to FINAL_RSPEC_XML_REPORT so the results won't accumulate with duplicated xml tags in TMP_RSPEC_XML_REPORT*
FINAL_RSPEC_XML_REPORT **=** 'tmp/test-reports/rspec/queue_mode/rspec_final_results.xml'

KnapsackPro**::**Hooks**::**Queue.**after_subset_queue** **do** **|**queue_id, subset_queue_id**|**
  **if** File.**exist?**(TMP_RSPEC_XML_REPORT)
    FileUtils.**mv**(TMP_RSPEC_XML_REPORT, FINAL_RSPEC_XML_REPORT)
  **end**
**end**
```

这个例子基于[常见问题](https://github.com/KnapsackPro/knapsack_pro-ruby#how-to-use-junit-formatter)。

# 摘要和队列模式来做动态测试套件拆分

由于利用了 Circle CI 2.0 上的并行作业和任何 CI 提供者上的 CI 并行化，CI 构建可以快得多([查看更多 CI 提供者的并行化示例](https://docs.knapsackpro.com/))。您可以查看[背包 Pro CI 并行化工具](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=codeclimate-and-circleci-2-0-parallel-builds-config-for-rspec-with-simplecov-and-junit-formatter)，并在下面的视频中了解更多关于队列模式及其解决的问题。

[可以在这里找到 backpack _ pro gem](https://docs.knapsackpro.com/knapsack_pro-ruby/guide/)的安装指南，以便设置您的 RSpec 测试套件。

— — —

最初发布于:[https://docs . knapsackpro . com/2019/code climate-and-circle ci-2-0-parallel-builds-config-for-rspec-with-simple cov-and-JUnit-formatter](https://docs.knapsackpro.com/2019/codeclimate-and-circleci-2-0-parallel-builds-config-for-rspec-with-simplecov-and-junit-formatter)