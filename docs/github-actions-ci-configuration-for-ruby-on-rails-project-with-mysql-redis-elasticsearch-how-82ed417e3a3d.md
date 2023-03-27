# 使用 MySQL、Redis、Elasticsearch 为 Ruby on Rails 项目配置 GitHub Actions CI——如何运行并行测试

> 原文：<https://itnext.io/github-actions-ci-configuration-for-ruby-on-rails-project-with-mysql-redis-elasticsearch-how-82ed417e3a3d?source=collection_archive---------6----------------------->

您将学习如何在 GitHub Actions 上配置 Ruby on Rails 项目。这个特定的 Rails 项目有 MySQL 和 Redis 数据库。CI 上还运行着 Elasticsearch 服务。如果您的项目接近下面的设置，那么 yaml 配置将允许您在 GitHub CI 服务器上运行测试。

如果你碰巧使用 PostgreSQL，你可以查看我们之前的一篇关于在 GitHub 上使用 Postgres 配置 [Rails 应用程序的文章](https://docs.knapsackpro.com/2019/how-to-run-rspec-on-github-actions-for-ruby-on-rails-app-using-parallel-jobs)。

![](img/5065110ca43528a45e93198a924320fb.png)

# Rails 的 GitHub 操作配置

在您的存储库中，您需要创建文件`.github/workflows/main.yaml`,由于它 GitHub Actions 将运行您的 CI 构建。您可以在 GitHub 存储库页面上的 Actions 选项卡中找到已执行 CI 构建的结果。

在本例中， **Rails** 应用有 **MySQL** 、 **Redis** 和 **Elasticsearch** 数据库。您需要用 docker 容器设置服务来运行每个服务。在下面的配置中，还有一个对 MySQL 和 Elasticsearch 进行健康检查的步骤，以确保在开始运行测试之前两者都已启动并运行。

由于 GitHub Actions 中的矩阵特性和[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=github-actions-ci-config-for-ruby-on-rails-project-with-mysql-redis-elasticsearch-how-to-run-parallel-tests)ruby gem 可以自动平衡任务间的测试分布，所以测试可以跨并行任务执行。使用背包 Pro 队列模式的自动平衡测试将确保每个并行作业在相似的时间完成工作。由于没有瓶颈(没有运行太多测试的缓慢工作),您可以享受快速的 CI 构建时间，因为您可以在并行任务中获得最佳的测试划分。

```
# .github/workflows/main.yaml
name: Mainon: [push]jobs:
  vm-job:
    name: CI
    runs-on: ubuntu-latest# If you need DB like MySQL then define service below.
    # Example for PostgreSQL and Redis can be found here:
    # [https://github.com/actions/example-services/tree/master/.github/workflows](https://github.com/actions/example-services/tree/master/.github/workflows)
    services:
      # How to use MySQL
      mysql:
        image: mysql:5.7
        env:
          MYSQL_ROOT_PASSWORD: root
        ports:
        - 3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3
      # How to use Redis
      redis:
        image: redis
        ports:
        - 6379/tcp
      # How to use Elasticsearch
      elasticsearch:
        image: elasticsearch:6.8.3
        ports:
        - 9200/tcp
        options: -e="discovery.type=single-node" --health-cmd="curl [http://localhost:9200/_cluster/health](http://localhost:9200/_cluster/health)" --health-interval=10s --health-timeout=5s --health-retries=10# [https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix](https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix)
    strategy:
      fail-fast: false
      matrix:
        # Set N number of parallel jobs you want to run tests on.
        # Use higher number if you have slow tests to split them on more parallel jobs.
        # Remember to update ci_node_index below to 0..N-1
        ci_node_total: [5]
        # set N-1 indexes for parallel jobs
        # When you run 2 parallel jobs then first job will have index 0, the second job will have index 1 etc
        ci_node_index: [0, 1, 2, 3, 4]steps:
    - name: Set up Ruby 2.6
      uses: actions/setup-ruby@v1
      with:
        ruby-version: 2.6.3- name: Checkout
      uses: actions/checkout@v1
      with:
        fetch-depth: 1- name: Verify MySQL connection from host
      run: |
        sudo apt-get install -y mysql-client libmysqlclient-dev
        mysql --host 127.0.0.1 --port ${{ job.services.mysql.ports[3306] }} -uroot -proot -e "SHOW GRANTS FOR 'root'@'localhost'"
        mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql --host 127.0.0.1 --port ${{ job.services.mysql.ports[3306] }} -uroot -proot mysql
    - name: Verify Elasticsearch connection from host
      env:
        ELASTIC_SEARCH_URL: [http://localhost:${{](http://localhost:${{) job.services.elasticsearch.ports[9200] }}
      run: |
        echo $ELASTIC_SEARCH_URL
        curl -fsSL "$ELASTIC_SEARCH_URL/_cat/health?h=status"
    - name: Bundle Install and Create DB
      env:
        RAILS_ENV: test
        DB_PASSWORD: root
        # tell Rails to use proper port for MySQL
        DB_PORT: ${{ job.services.mysql.ports[3306] }}
      run: |
        cp .env.sample .env
        cp config/database.yml.ci config/database.yml
        gem install bundler --version 1.17.3 --no-ri --no-rdoc
        bundle install --jobs 4 --retry 3 --path vendor/bundle
        # create DB for Rails 5.x
        bin/rails db:setup
        # create DB for Rails 6.x
        bin/rails db:prepare
    - name: Run tests
      env:
        DB_PASSWORD: root
        DB_PORT: ${{ job.services.mysql.ports[3306] }} # get randomly assigned published port
        REDIS_PORT: ${{ job.services.redis.ports[6379] }}
        ELASTIC_SEARCH_URL: [http://localhost:${{](http://localhost:${{) job.services.elasticsearch.ports[9200] }}
        RAILS_ENV: test
        # You need to set API tokens in GitHub repository Settings -> Secrets
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

# 背包专业队列模式如何工作

在本视频中，您将了解跨并行作业(并行 CI 节点)拆分的动态测试套件如何工作。

# 背包专业常规模式如何工作

[backpack Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog&utm_campaign=github-actions-ci-config-for-ruby-on-rails-project-with-mysql-redis-elasticsearch-how-to-run-parallel-tests)也有一种确定性的拆分测试方式。在运行测试之前，测试只被拆分一次。在切换到队列模式之前，这是您第一次尝试记录 CI 构建的最简单的方法。

你可以在[安装指南](https://docs.knapsackpro.com/integration/)中了解更多关于背包 Pro 的信息。

# 摘要

如果你想了解更多关于 GitHub 动作的信息，请查看我们之前的文章[在 GitHub 动作上用 PostgreSQL 测试 Rails 应用](https://docs.knapsackpro.com/2019/how-to-run-rspec-on-github-actions-for-ruby-on-rails-app-using-parallel-jobs)。还有一个视频展示了它的作用。

*原载于 2019 年 9 月 27 日*[*https://docs.knapsackpro.com*](https://docs.knapsackpro.com/2019/github-actions-ci-config-for-ruby-on-rails-project-with-mysql-redis-elasticsearch-how-to-run-parallel-tests)*。*