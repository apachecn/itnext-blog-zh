# Ruby on Rails 应用程序的 Postgres 和 Redis 数据库连接池大小配置

> 原文：<https://itnext.io/postgres-and-redis-database-connection-pool-size-config-for-ruby-on-rails-app-3725239710fa?source=collection_archive---------7----------------------->

对于许多程序员来说，为 Rails 应用程序配置数据库连接池可能不是一项简单的任务。数据库级别有最大打开连接数的限制。您的服务器环境配置可能会随时间变化，并影响到所需的数据库连接数。例如，当您根据 web 流量自动调整服务器数量时，您使用的服务器数量可能会发生变化。这意味着为 Puma 或 Unicorn 服务器运行的 web 进程/线程的数量可能会改变。所有这些都增加了额外的复杂性。当你使用两个数据库时(例如 Postgres + Redis)，一切都变得更加复杂。在本文中，我们将解决这个问题。您将学习如何估计 Ruby on Rails 生产应用程序所需的数据库连接。

![](img/196392d4311ca9b0d085c631fdeed1e5.png)

# 为什么可用的数据库连接很重要？

第一个问题是，为什么需要关心可用的数据库连接？答案很简单。假设您将 Ruby 应用程序配置为打开太多的数据库连接。在这种情况下，当数据库无法处理更多来自 Rails 应用程序的新连接时，可能会出现来自应用程序的`ActiveRecord::ConnectionTimeoutError`异常。这可能会导致您的 web 应用程序用户看到 500 个错误。

这个问题可能不会立即显现出来。通常你会在生产中发现它。您的应用程序可能工作得很好，直到特定的环境导致 Rails 应用程序需要更多的数据库连接，这可能会触发异常泛滥。我们来看看如何避免。

# RoR 应用程序逐步配置

让我们将一个典型的 Ruby on Rails 应用程序分解成使用数据库的更小的组件。

*   我们有一个 Rails 应用程序，它使用 Postgres 数据库来使用 ActiveRecord。
*   我们也使用 Redis 数据库为 Sidekiq 这样的后台工作人员服务。

看起来很简单，不是吗？让我们从这开始，稍后，我们将增加更多的复杂性:)

# Postgres 数据库连接—如何检查限制？

如何检查 Postgres 有多少可用连接？

*   如果您使用安装了 Postgres 的专用服务器，那么很可能您有一个默认的`max_connections`,通常是 100 个连接。
*   如果您在 AWS 上使用 Postgres 实例，那么您需要查看 AWS 文档，以了解到您的数据库实例的最大允许连接数是多少(这取决于您是否使用 Amazon RDS 或 Aurora 以及什么是服务器实例类)
*   如果你使用 Heroku，你可以检查`Connection Limit`的 [Postgres Heroku 附加](https://elements.heroku.com/addons/heroku-postgresql#pricing)来检查最大可接受的连接。

# ActiveRecord 连接池

在您的 Rails 应用程序中，`config/database.yml`文件包含了`pool`选项。如 [Rails 文档](https://edgeguides.rubyonrails.org/configuring.html#database-pooling)中所述:

> *活动记录数据库连接由* `*ActiveRecord::ConnectionAdapters::ConnectionPool*` *管理，确保连接池同步有限数量的数据库连接的线程访问量。*
> 
> *由于缺省情况下连接池是在活动记录中处理的，所以所有应用服务器(瘦、Puma、Unicorn 等。)应该表现的一样。数据库连接池最初是空的。随着连接需求的增加，它将创建连接，直到达到连接池的限制。*
> 
> *任何一个请求都会在第一次需要访问数据库时检查一个连接。在请求结束时，它将重新检查连接。这意味着队列中的下一个请求可以再次使用额外的连接槽。*

`pool`可以这样定义:

```
production:
  adapter: postgresql
  database: blog_production
  pool: 5
```

或者作为数据库 URL 的一部分:

```
development:
  url: postgresql://localhost/blog_production?pool=5
```

当您在 Amazon RDS 这样的外部服务器上托管数据库时，URL 选项很受欢迎。那么您可以这样定义 URL:

```
production:
  url: postgres://blog_production:PASSWORD@blog-production.abcdefgh.eu-west-1.rds.amazonaws.com/blog_production?sslca=config/rds-combined-ca-bundle.pem&pool=5
```

请注意，对于生产，您不应该在`config/database.yml`文件中提交凭证。相反，将它存储在环境变量中，然后在 Rails 应用程序运行时读取该值。

```
production:
  url: <%= ENV['DB_URL'] %>
```

# ActiveRecord 连接池如何影响 Postgres max 连接数？

让我们从一个简单的例子开始。您的应用程序可以使用 Puma 或 Unicorn 之类的应用服务器。让我们关注 Puma，因为它更复杂，因为它为几个进程(Puma 术语中称为 workers)和线程提供了单独的配置。Unicorn 只在单线程中运行。它的工作方式与单线程设置的 Puma 完全一样。

![](img/34398eed51e2130fae7c152f906f3622.png)

# Puma 配置:1 个进程和 1 个线程

假设您使用 Puma 服务器运行 Rails 应用程序。Puma 被配置为运行一个进程(worker ),它只有一个线程。

puma 进程最多可以打开 5 个到数据库的连接，因为`pool`选项在`config/database.yml`中被定义为 5。通常情况下，连接会比这个数目少，因为当您运行一个进程和一个线程时，只需要一个到 Postgres 数据库的连接来进行数据库查询。

有时数据库连接可能会中断。在这种情况下，ActiveRecord 可以打开一个新连接，然后您可能会有两个活动连接。在最糟糕的情况下，当 4 个连接失效时，Rails 最多可以打开 5 个连接。

# Puma 配置:1 个进程和 2 个线程

如果在一个 Puma 进程(worker)中使用两个线程，那么这意味着这两个线程可以在 Puma 进程中使用同一个数据库连接池。

这意味着 5 个可能的连接中有 2 DB 将被打开。如果有任何连接失效，那么可以打开更多的连接，直到达到 5 个连接池的限制。

# Puma 配置:2 个进程和每个进程 2 个线程

如果您运行 2 个 Puma 进程(workers ),并且每个进程有 2 个线程，那么这意味着每个进程将打开 2 DB 连接，因为每个进程有 2 个线程。您有 2 个进程，因此这意味着在应用程序开始时，可能有 4 个 DB 连接打开。每个进程都有自己的池，所以有两个池。每个池最多可以打开 5 DB 连接。这意味着在最坏的情况下，甚至可以创建 10 个到数据库的连接。

假设每个 Puma 进程使用 2 个线程，那么最好将`pool`选项设置为 2 +一些备用连接。它允许 ActiveRecord 在一个数据库连接失效时打开一个新的连接。

# Puma 配置:Heroku 上有 2 个进程和 2 个线程，以及 2 个 web dynos

如果您使用 Heroku 托管您的应用程序，它允许通过添加更多服务器(dynos)来水平扩展您的 web 应用程序。假设您在 2 个服务器(2 个 Heroku dynos)上运行您的应用程序，每个 dyno 运行 2 个 Puma 进程，每个进程有 2 个线程。这意味着在开始时，您的应用程序可能会打开 6 个数据库连接。原因如下:

2 个 dynos X 2 个 Puma 进程 X 2 个 Puma 线程= 6 个 DB 连接

2 个 dynos X 2 个 Puma 进程 X 池大小(5) =总池大小 20

这意味着在最坏的情况下，您的应用程序可能会打开 20 DB 的连接。

## 自动缩放 web 应用程序

如果您在 web 流量高峰期通过添加更多服务器来自动扩展 web 服务器，您需要小心。确保您的应用程序保持在 Postgres 最大连接数限制内。上面的例子向您展示了如何计算预期的开放数据库连接和最坏的情况。请调整您的池大小，以确保在最坏的情况下，您将低于数据库引擎的最大连接数限制。

# 还有什么可以打开 DB 连接？

我们刚刚谈到了像 Puma 这样的 web 服务器，它可以打开连接并消耗您的最大数据库连接数限制。但是其他非网络流程也可以做到这一点:

*   您在 Heroku dyno `heroku run bin/rails console --app=my-app-name`中运行 Rails 控制台。它运行 Rails 应用程序的一个实例，1 DB 连接将打开。在最坏的情况下，在`pool`中定义的连接数可以被打开。但是您的数据库连接不太可能会中断。因此不应使用整个池限制。
*   您通过 Heroku Scheduler(类似 cron 的工具)运行预定的 rake 任务。如果定期执行 rake 任务，它们需要打开到 DB 的连接，以便每个 rake 任务至少使用池中的 1 DB 连接。假设你有 10 个每小时开始的 rake 任务。这意味着每小时需要 10 个可用的数据库连接。如果你的评估仅仅基于网络连接，很容易忽略这一点。
*   您使用 Sidekiq 这样的后台工作器来执行异步作业。您的作业可能会打开数据库连接。我们以后再谈。

# 后台工作程序— Sidekiq 和 ActiveRecord 池

Sidekiq 进程将像 Puma 一样使用在`config/database.yml`中定义的池。Sidekiq 进程中的所有 Sidekiq 线程都可以使用一个公共连接池。

如果您运行多个服务器(Heroku dynos)，那么它的工作方式类似于 Puma 示例。

2 个服务器(dynos) X 1 个 Sidekiq 进程 X 10 个 Sidekiq 线程= 20 DB 连接将打开。

在`config/database.yml`中，您需要至少 10 个池大小，因为 Sidekiq 默认使用 10 个线程。

如果使用小于 10 的池大小，那么 Sidekiq 线程将会争用池中有限的连接。在一段时间内这可能是好的，但是您应该意识到这会增加您的作业的处理时间，因为不是所有的 Sidekiq 线程都会并行使用 DB 连接。这也会导致[这里描述的一个问题](https://github.com/mperham/sidekiq/wiki/Problems-and-Troubleshooting#cannot-get-database-connection-within-500-seconds)。

# Sidekiq 和 Redis 数据库连接

![](img/e1bd0a8cdcab1214485c24d059208f69.png)

雷迪斯

Sidekiq 使用 Redis 数据库来存储异步作业。如果您计算到 Redis 的 DB 连接以及 Postgres 连接，那将是最好的。Sidekiq 服务器进程至少需要(并发+ 5)个连接。`concurrency`选项是每个 Sidekiq 进程的 Sidekiq 线程数。

使用前面的例子:

2 台服务器(dynos) X 1 个 Sidekiq 进程 X 10 个 Sidekiq 线程= 2 台服务器(dynos) X 1 个 Sidekiq 进程 X (10 + 5) =需要 30 个 Redis 连接。

更多信息请见 [Sidekiq 文档](https://github.com/mperham/sidekiq/wiki/Using-Redis#complete-control)。

# Redis 数据库连接

如果您使用 Redis 来处理后台作业，那么不仅仅是 Sidekiq 进程在使用 Redis 连接。Puma 进程和线程也可以使用 Redis 向 Sidekiq 队列添加新的作业。通常每个 Puma 线程有一个 Redis 连接。

如果您使用`Redis.new`显式地打开一个新的 Redis 连接，这也可以为每个 Puma 线程创建一个新的连接。

# 摘要

我们讨论了几个 Postgres 和 Redis 的例子，计算 Rails 应用程序所需的数据库连接。我希望这将使您更好地理解如何估计您的数据库级需要多少 DB 连接来适当地满足您的应用程序的需求。

如果你想改进你的 Rails 应用程序工作流程，请考虑检查如何在你的 CI 服务器上用[背包 Pro](https://knapsackpro.com/?utm_source=medium&utm_medium=blog_post&utm_campaign=medium-estimate-database-connections-pool-size-for-rails-application) 并行运行[自动化测试。](https://docs.knapsackpro.com/2020/how-to-speed-up-ruby-and-javascript-tests-with-ci-parallelisation)

![](img/c3714556e8ce3f65da78bd4da43a0338.png)

*原载于 2021 年 4 月 26 日 https://docs.knapsackpro.com*[](https://docs.knapsackpro.com/2021/estimate-database-connections-pool-size-for-rails-application)**。**