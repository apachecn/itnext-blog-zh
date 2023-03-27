# 使用 Cassandra API 为 Azure Cosmos DB 构建容错应用程序

> 原文：<https://itnext.io/build-fault-tolerant-applications-with-cassandra-api-for-azure-cosmos-db-d83f5f1fffb7?source=collection_archive---------2----------------------->

![](img/0bb0dd84ae9dac79e435e1d5af29dffa.png)

Azure Cosmos DB 是一个资源管理系统，它允许你根据你配置的[提供的吞吐量](https://docs.microsoft.com/azure/cosmos-db/set-throughput)每秒执行一定数量的操作。如果客户端超过了这个限制，消耗的[请求单元](https://docs.microsoft.com/azure/cosmos-db/request-units)比供应的多，就会导致后续请求的速率限制和抛出异常——它们也被称为 [429 错误](https://docs.microsoft.com/rest/api/cosmos-db/http-status-codes-for-cosmosdb?WT.mc_id=medium-blog-abhishgu)。

借助一个实际的例子，我将演示如何通过处理和重试受这些速率限制错误影响的操作，在您的 [Go](https://golang.org/) 应用程序中引入容错。为了帮助你理解，这个博客的示例应用程序代码可以在 GitHub 上找到[——它使用了 Apache Cassandra](https://github.com/abhirockzz/cosmos-go-rate-limiting) 的 [gocql 驱动程序。](https://github.com/gocql/gocql)

在本帖中，我们将介绍:

*   运行示例应用程序之前的初始设置和配置
*   执行各种负载测试场景并分析结果
*   重试策略实施的快速概述。

解决速率限制的一种方法是调整供应的吞吐量，以满足您的应用需求。有多种方法可以做到这一点，包括使用 Azure portal、Azure CLI 和 CQL (Cassandra Query Language)命令。

## 但是，如果您想在应用程序本身中处理这些错误呢？

好在 Azure Cosmos DB 的 Cassandra API 将速率限制异常转化为 Cassandra 本地协议上的过载错误。由于`gocql`驱动程序允许你插入自己的 [RetryPolicy](https://pkg.go.dev/github.com/gocql/gocql?tab=doc#RetryPolicy) ，你可以编写一个定制的实现来拦截这些错误，并在某个(冷却)时间段后重试它们。然后这个策略可以[应用于每个查询](https://pkg.go.dev/github.com/gocql/gocql?tab=doc#Query.RetryPolicy)或者使用 [ClusterConfig](https://pkg.go.dev/github.com/gocql/gocql?tab=doc#ClusterConfig) 在全局级别应用。

Azure Cosmos DB 扩展库使得在 Java 应用程序中使用重试策略变得非常容易。GitHub 上有一个等价的 Go 版本[,并且已经在这篇博文的示例应用程序中使用。](https://github.com/abhirockzz/cosmos-cassandra-go-extension)

# 重试策略在起作用

正如所承诺的，您将使用一个简单而实用的示例来完成整个过程。用于演示概念的示例应用程序是一个服务，它将 REST 端点暴露给保存在 Azure Cosmos DB 的 Cassandra 表中的`POST`订单数据。

您将在这个 API 服务上运行一些负载测试，以查看速率限制是如何表现的以及如何处理的。

## 先决条件

首先安装[嘿](https://github.com/rakyll/hey)，一个负载测试程序。您可以为 [Linux](https://storage.googleapis.com/hey-release/hey_linux_amd64) 、 [Mac](https://storage.googleapis.com/hey-release/hey_darwin_amd64) 和 [Windows](https://storage.googleapis.com/hey-release/hey_windows_amd64) 下载特定于操作系统的二进制文件(64 位)(如果您在下载实用程序时遇到问题，请参考[GitHub repo](https://github.com/rakyll/hey#installation)获取最新信息)

> *您可以使用任何其他允许您在 HTTP 端点上生成负载的工具*

克隆这个 GitHub repo 并切换到正确的目录:

```
git clone github.com/abhirockzz/cosmos-go-rate-limiting 
cd cosmos-go-rate-limiting
```

## 设置 Azure Cosmos DB

选择 Cassandra API 选项，创建一个 Azure [Cosmos DB 帐户](https://docs.microsoft.com/azure/cosmos-db/create-cassandra-api-account-java?WT.mc_id=medium-blog-abhishgu#create-a-database-account)

要创建一个键空间和表，使用下面的`CQL`:

```
CREATE KEYSPACE ordersapp WITH REPLICATION = {'class' : 'SimpleStrategy'};

    CREATE TABLE ordersapp.orders (
        id uuid PRIMARY KEY,
        amount int,
        state text,
        time timestamp
    );
```

## 启动应用程序

打开终端并为应用程序设置环境变量:

```
export COSMOSDB_CASSANDRA_CONTACT_POINT=.cassandra.cosmos.azure.com 
export COSMOSDB_CASSANDRA_PORT=10350 
export COSMOSDB_CASSANDRA_USER= 
export COSMOSDB_CASSANDRA_PASSWORD= 
#optional (default: 5) 
#export MAX_RETRIES=
```

要启动应用程序:

```
go run main.go 

//wait for this output 
Connected to Azure Cosmos DB
```

为了测试应用程序是否按预期工作，通过从不同的终端调用 REST 端点(每个订单一次)插入几个订单:

```
curl [http://localhost:8080/orders](http://localhost:8080/orders)
```

应用程序会生成随机数据，因此您不必在调用端点时输入数据

确认订单已成功存储。你可以在 Azure 门户中使用[托管的 CQL 外壳，并执行下面的查询:](https://docs.microsoft.com/azure/cosmos-db/cassandra-support?WT.mc_id=medium-blog-abhishgu#hosted-cql-shell-preview)

```
select count(*) from ordersapp.orders;

    // you should see this output
    system.count(*) 
    ----------------- 
        1 
    (1 rows)
```

你都准备好了。

# 让负载测试开始吧！

用 300 个请求调用 REST 端点。这足以使系统过载，因为默认情况下只分配了 400 RU/s。

若要启动负载测试:

```
hey -t 0 -n 300 [http://localhost:8080/orders](http://localhost:8080/orders)
```

注意应用程序终端中的日志。开始时，您将看到订单被成功创建。例如:

```
Added order ID 25a8cec1-e67a-11ea-9c17-7f242c2eeac0
Added order ID 25a8f5ef-e67a-11ea-9c17-7f242c2eeac0
Added order ID 25a8f5ea-e67a-11ea-9c17-7f242c2eeac0
...
```

一段时间后，随着吞吐量下降并最终超过供应的限制，Azure Cosmos DB 将对应用程序请求进行速率限制。这将以如下所示的错误形式出现:

```
Request rate is large: ActivityID=ac78fac3-5c36-4a20-8ad7-4b2d0768ffe4, **RetryAfterMs=112**, Additional details='Response status code does not indicate success: **TooManyRequests (429)**; Substatus: 3200; ActivityId: ac78fac3-5c36-4a20-8ad7-4b2d0768ffe4; Reason: ({
      "Errors": [
        "Request rate is large. More Request Units may be needed, so no changes were made. Please retry this request later. Learn more: http://aka.ms/cosmosdb-error-429"
      ]
    });
```

> *在上面的错误消息中，请注意以下内容:TooManyRequests (429)和 RetryAfterMs=112*

**观察查询错误**

为了简单起见，我们将使用日志输出进行测试/诊断。在查询执行过程中遇到的任何错误(在这种情况下与速率限制相关)都会被一个 [gocql 截获。QueryObserver](https://pkg.go.dev/github.com/gocql/gocql?tab=doc#QueryObserver) 。随机生成的订单 ID 也会记录在每个错误消息中，这样您就可以检查日志以确认失败的订单是否已经重试并(最终)存储在 Azure Cosmos DB 中。

以下是代码片段:

```
....
    type OrderInsertErrorLogger struct {
       orderID string
    }

    // implements gocql.QueryObserver
    func (l OrderInsertErrorLogger) ObserveQuery(ctx context.Context, oq gocql.ObservedQuery) {
      err := oq.Err
      if err != nil {
         log.Printf("Query error for order ID %sn%v", l.orderID, err)
      }
    }

    ....

    // the Observer is associated with each query
    rid, _ := uuid.GenerateUUID()
    err := cs.Query(insertQuery).Bind(rid, rand.Intn(200)+50, fixedLocation, time.Now()).Observer(OrderInsertErrorLogger{orderID: rid}).Exec()
    ....
```

**完成了多少订单？**

切换回负载测试终端，检查一些统计数据(为了简洁起见，输出已经被编辑)

```
Summary: 

      Total:        2.8507 secs 
      Slowest:      1.3437 secs 
      Fastest:      0.2428 secs 
      Average:      0.5389 secs 
      Requests/sec: 70.1592 
    .... 

    Status code distribution: 
      [200] 300 responses
```

> *根据多种因素，具体情况下的数字会有所不同。*

这不是一个原始的基准测试，我们也没有生产级应用，所以你可以忽略`Requests/sec`等。但是请注意`Status code distribution`属性，它表明我们的应用程序对所有请求都用`HTTP` `200`来响应。

让我们确认一下最终的数字。在 Azure Cosmos DB 门户中打开 **Cassandra Shell** 并执行相同的查询:

```
select count(*) from ordersapp.orders;

    //output

    system.count(*)
    -----------------
        301
```

您应该看到已经插入了 300 个额外的*行(订单)。关键的一点是，所有的订单都成功地存储在 Azure Cosmos DB 中——尽管存在速率限制错误，因为我们的应用程序代码基于我们配置的重试策略透明地重试了它们(只用一行代码！)*

```
clusterConfig.RetryPolicy = retry.NewCosmosRetryPolicy(numRetries)
```

# 关于动态吞吐量管理的一点注记

如果您的应用大部分时间都以大约 60–70%的吞吐量运行，那么使用[自动扩展配置的吞吐量](https://docs.microsoft.com/azure/cosmos-db/provision-throughput-autoscale?WT.mc_id=medium-blog-abhishgu)可以通过在不使用时按比例缩减来帮助优化您的 RU/s 和成本使用，您只需按小时支付工作负载所需的资源。

那么，如果没有重试策略，会发生什么呢？

# 停用策略以查看差异

停止应用程序(在终端中按`control+c`，设置一个环境变量并重新启动应用程序:

```
export USE_RETRY_POLICY=false 
go run main.go
```

在再次运行负载测试之前，使用`select count(*) from ordersapp.orders;`记下订单表中的行数

```
hey -t 0 -n 300 [http://localhost:8080/orders](http://localhost:8080/orders)
```

在应用程序日志中，您会注意到相同的速率限制错误。在运行负载测试的终端中，在输出摘要的末尾，您将看到一些请求未能成功完成，即它们返回了除`HTTP 200`之外的响应

```
...
Status code distribution: 
  [200] 240 responses 
  [429] 60 responses
```

由于没有强制实施重试策略，应用程序不再重试由于速率限制而失败的请求。

# 增加调配的吞吐量

您可以使用 Azure 门户增加请求单元(例如，加倍到`800 RU/s`)并运行相同的负载测试

```
hey -t 0 -n 300 [http://localhost:8080/orders](http://localhost:8080/orders)
```

您将**而不是**看到速率限制(`HTTP 429`)错误，以及相对较低的延迟、每秒请求数等。

尝试增加请求的数量(使用`-n`标志),看看应用程序何时突破了吞吐量阈值，从而达到速率限制。正如预期的那样，所有订单都将成功持久化(没有任何错误或重试)

下一节简要介绍了[定制重试策略](https://github.com/abhirockzz/cosmos-cassandra-go-extension)的工作原理。

> *这是一个实验性的实现，您应该编写自定义策略来满足应用程序的容错和性能要求。*

# 在幕后

`CosmosRetryPolicy`遵守 [gocql。通过实现`Attempt`和`GetRetry`函数来返回策略](https://pkg.go.dev/github.com/gocql/gocql?tab=doc#RetryPolicy)接口。

```
type CosmosRetryPolicy struct {
        MaxRetryCount         int
        FixedBackOffTimeMs    int
        GrowingBackOffTimeMs  int
        numAttempts           int
}
```

仅当该查询的重试尝试次数小于或等于最大重试配置或将最大重试配置设置为-1(无限次重试)时，才会启动重试

```
func (crp *CosmosRetryPolicy) Attempt(rq gocql.RetryableQuery) bool { 
    crp.numAttempts = rq.Attempts() 
    return rq.Attempts() <= crp.MaxRetryCount || crp.MaxRetryCount == -1
}
```

`GetRetryType`函数检测错误类型，如果是速率受限错误(`HTTP 429`)，它会尝试提取`RetryAfterMs`字段的值(从错误消息中)，并在重试查询前使用该值休眠。

```
func (crp *CosmosRetryPolicy) GetRetryType(err error) gocql.RetryType {

       switch err.(type) {
       default:
             retryAfterMs := crp.getRetryAfterMs(err.Error())
             if retryAfterMs == -1 {
                 return gocql.Rethrow
             }
            time.Sleep(retryAfterMs)
            return gocql.Retry

    //other case statements have been omitted for brevity
}
```

Azure Cosmos DB 不仅为您提供了使用各种方式配置和调整吞吐量需求的灵活性，还提供了允许应用程序处理速率限制错误的基本原语，从而使它们变得健壮和容错。这篇博客文章演示了如何为 Go 应用程序实现这一点，但是这些概念适用于任何语言及其各自的 [CQL 兼容](https://docs.microsoft.com/azure/cosmos-db/cassandra-support?WT.mc_id=medium-blog-abhishgu#cassandra-protocol)驱动程序，您可以选择这些驱动程序来使用 Cassandra API for Azure Cosmos DB。

# 要了解更多信息:

请查阅官方文档中的一些资源:

*   [自动扩展调配吞吐量的使用案例和优势](https://docs.microsoft.com/azure/cosmos-db/provision-throughput-autoscale?WT.mc_id=medium-blog-abhishgu)
*   [Azure Cosmos DB 中 Cassandra API 支持的详细信息](https://docs.microsoft.com/azure/cosmos-db/cassandra-support?WT.mc_id=medium-blog-abhishgu)
*   [使用 Go 应用程序和 Cassandra API for Azure Cosmos DB 开始运行](https://docs.microsoft.com/azure/cosmos-db/create-cassandra-go?WT.mc_id=medium-blog-abhishgu)
*   [关于 Azure Cosmos DB 中 Cassandra API 的常见问题](https://docs.microsoft.com/azure/cosmos-db/create-cassandra-go?WT.mc_id=medium-blog-abhishgu)
*   [请求单位概念](https://docs.microsoft.com/azure/cosmos-db/request-units?WT.mc_id=medium-blog-abhishgu)