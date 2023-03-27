# 如何在 GitLab CI 管道上运行 RSpec 中的并行测试并加速 Ruby & JavaScript 测试

> 原文：<https://itnext.io/how-to-run-parallel-tests-in-rspec-on-gitlab-ci-pipeline-and-speed-up-ruby-javascript-testing-6fca26817693?source=collection_archive---------0----------------------->

由于 CI 并行化特性，GitLab CI 允许您更快地运行测试。您可以在多个 GitLab 运行程序上运行并行作业。为了做到这一点，您将学习如何在并行任务之间以动态的方式分割测试，以确保 GitLab 管道中没有瓶颈。由于 CI 构建可以尽可能快地运行，所以您的 **Ruby & JS 测试也可以非常快**。

![](img/222ccda752dce145cb0871791b9a8524.png)

GitLab CI

# GitLab CI 并行化

当您想要并行运行测试以在几分钟内而不是等待几小时内完成您的 1 小时测试套件时，常见的问题是找到一种如何在并行作业上分割测试的方法。对于每个测试文件，一些 Ruby 或 JavaScript 测试可能需要几毫秒甚至几分钟的时间(例如在 RSpec 特性测试中使用 Capybara)。当使用 [Cypress test runner 作为浏览器测试](https://docs.knapsackpro.com/2019/cypress-parallel-testing-with-jenkins-pipeline-stages)可能需要相当长的时间来执行时，在 E2E 也会出现测试缓慢的问题(端到端测试)。

如果您添加更多的并行 GitLab 运行程序，您可能还会注意到一些运行程序可以稍后开始工作，或者不是所有的作业都可以同时开始(例如，当您在自己的基础设施上运行 GitLab 运行程序，而其他 CI 构建占用了一些运行程序)。

# 动态测试套件拆分消除 CI 构建瓶颈

跨并行作业(并行 CI 节点)优化测试分布的一个解决方案是跨并行 GitLab 运行程序以较小的块分布测试文件。这确保了活跃的运行者可以消耗和执行你的测试，而过于忙碌的运行者和缓慢的测试会运行较少的测试用例。重要的是确保所有并行的跑步者在相同的时间完成工作，这样你就不会看到有太多工作要处理的 GitLab 跑步者停滞不前。

为了以动态的方式为 Ruby & JavaScript 测试划分测试，你可以在 backpack Pro 中使用队列模式。下面我将解释队列模式是如何工作的，它解决了什么问题。

# 并行测试的 GitLab YAML 配置

在这里你可以找到一个 Ruby on Rails 项目的示例配置`.gitlab-ci.yml`，它使用[背包 Pro 队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-parallel-jobs-for-rspec-tests-on-gitlab-ci-pipeline-and-speed-up-ruby-javascript-testing)在 2 个并行任务上执行了 RSpec 测试。类似的配置将用于带有 Jest 或 Cypress 测试的 JavaScript 项目(背包 Pro 中支持的测试运行程序的完整列表可以在这里找到)。

请记住在 GitLab 设置->CI/CD->Variables(Expand)中将[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-parallel-jobs-for-rspec-tests-on-gitlab-ci-pipeline-and-speed-up-ruby-javascript-testing)的 API token 设置为环境变量名`KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC`作为屏蔽变量。

```
*# .gitlab-ci.yml*
image: "ruby:2.6"

services:
  - postgres

variables:
  RAILS_ENV: test
  POSTGRES_DB: database_name
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: ""
  DATABASE_URL: "postgresql://postgres:postgres@postgres:5432/database_name"
  *# KNAPSACK_PRO_TEST_SUITE_TOKEN_RSPEC: it is set in Settings -> CI/CD -> Variables (Expand) as masked variable*

before_script:
  - apt-get update -qq && apt-get install -y -qq nodejs
  - ruby -v
  - which ruby
  - gem install bundler --no-document
  - bundle install --jobs $(nproc)  "${FLAGS[@]}"

  *# Database setup*
  - bin/rails db:setup

rspec:
  parallel: 2
  script:
    - bundle exec rake knapsack_pro:queue:rspec
```

注意，通过改变`parallel`选项，你可以运行几十个并行任务，这使得你可以在几分钟内运行非常长的测试套件，而不是等待几个小时。

# 摘要

由于测试的并行化，GitLab 及其 CI/CD 工具允许运行快速 CI 构建。通过使用[backpack Pro 队列模式](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=how-to-run-parallel-jobs-for-rspec-tests-on-gitlab-ci-pipeline-and-speed-up-ruby-javascript-testing)，你可以确保你的测试以一种最佳的方式被分割到并行任务中，这样你的团队就能尽快得到测试结果。

— — —

最初发布于:[https://docs . knapsackpro . com/2019/how-to-run-parallel-jobs-for-rspec-tests-on-git lab-ci-pipeline-and-speed-up-ruby-JavaScript-testing](https://docs.knapsackpro.com/2019/how-to-run-parallel-jobs-for-rspec-tests-on-gitlab-ci-pipeline-and-speed-up-ruby-javascript-testing)