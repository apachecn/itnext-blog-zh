# 使用 PostgreSQL pgoutput 插件在 Azure 上通过 Debezium 捕获变更数据

> 原文：<https://itnext.io/using-postgresql-pgoutput-plugin-for-change-data-capture-with-debezium-on-azure-845d3bb2787a?source=collection_archive---------2----------------------->

[使用 Debezium、Postgres 和 Kafka 在 Azure 上建立变更数据捕获架构](https://dev.to/azure/tutorial-set-up-a-change-data-capture-architecture-on-azure-using-debezium-postgres-and-kafka-49h6)是一个关于如何使用 Debezium 从 Azure PostgreSQL 捕获变更数据并将它们发送到 Azure Event Hubs for Kafka 的教程——它使用了`wal2json`输出插件。

## 那`pgoutput`插件呢？

这个博客将提供一个如何使用`pgoutput`插件的快速浏览。为了简单起见，我不会重复很多细节，而是使用 Kafka connect、Kafka(和 Zookeeper)的容器化版本(使用 Docker Compose)。所以，你唯一需要的是 Azure PostgreSQL，你可以使用各种选项来设置它，包括 [Azure 门户](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal?WT.mc_id=medium-blog-abhishgu)、 [Azure CLI](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-azure-cli?WT.mc_id=medium-blog-abhishgu) 、 [Azure PowerShell](https://docs.microsoft.com/azure/postgresql/quickstart-create-postgresql-server-database-using-azure-powershell?WT.mc_id=medium-blog-abhishgu) 、 [ARM 模板](https://docs.microsoft.com/azure/postgresql/quickstart-create-postgresql-server-database-using-arm-template?tabs=azure-portal&WT.mc_id=medium-blog-abhishgu)。

> *GitHub 上有资源—*[*https://github.com/abhirockzz/debezium-postgres-pgoutput*](https://github.com/abhirockzz/debezium-postgres-pgoutput)

## 使用右边的`*publication.autocreate.mode*`

使用`pgoutput`插件，为[publication . auto create . mode](https://debezium.io/documentation/reference/connectors/postgresql.html#postgresql-publication-autocreate-mode)使用合适的值是很重要的。如果您正在使用 **all_tables** (这是默认设置)，*您需要确保预先为您想要为变更数据捕获配置的特定表创建发布。如果找不到发布，连接器将尝试使用* `*CREATE PUBLICATION <publication_name> FOR ALL TABLES;*` *创建一个，这将由于缺少权限而失败。*

其他两个选项按预期工作:

*   **禁用**:您需要确保出版物是预先创建的。如果在启动时没有发现发布，连接器将*而不是*尝试创建发布——它将抛出一个异常并停止。
*   **过滤**:您可以(可选)选择预先创建出版物。如果找不到发布，连接器将为所有匹配当前筛选器配置的表创建一个新的发布。

> *这一点在 docs*[*https://debezium . io/documentation/reference/1.3/connectors/PostgreSQL . html # PostgreSQL-on-azure*](https://debezium.io/documentation/reference/1.3/connectors/postgresql.html#postgresql-on-azure)中已经强调过了

## 让我们尝试不同的场景..

在此之前:

```
git clone https://github.com/abhirockzz/debezium-postgres-pgoutput && cd debezium-postgres-pgoutput
```

启动 Kafka、Zookeeper 和 Kafka 连接容器:

```
export DEBEZIUM_VERSION=1.2
docker-compose up
```

> 第一次拉集装箱可能需要一段时间

所有容器启动并运行后，连接到 Azure PostgreSQL，创建一个表并插入一些数据:

```
psql -h <DBNAME>.postgres.database.azure.com -p 5432 -U <DBUSER>@<DBNAME> -W -d postgres --set=sslmode=requirepsql -h abhishgu-pg.postgres.database.azure.com -p 5432 -U abhishgu@abhishgu-pg -W -d postgres --set=sslmode=requireCREATE TABLE inventory (id SERIAL, item VARCHAR(30), qty INT, PRIMARY KEY(id));
```

**当 publication.autocreate.mode 设置为过滤时**

这与 Azure PostgreSQL 配合得很好——它不需要超级用户权限，因为连接器基于 filter/*list 值为特定于*的*表创建发布

用 Azure PostgreSQL 实例的详细信息更新连接器配置文件(`pg-source-connector.json`)，然后创建连接器

要创建连接器，请执行以下操作:

```
curl -X POST -H "Content-Type: application/json" --data @pg-source-connector.json [http://localhost:8083/connectors](http://localhost:8083/connectors)
```

注意日志(在 docker 编写终端中):

```
Creating new publication 'mytestpub' for plugin 'PGOUTPUT'   [io.debezium.connector.postgresql.connection.PostgresReplicationConnection]
```

连接器启动后，检查 PostgreSQL 中的发布:

```
pubname  | schemaname | tablename 
-----------+------------+-----------
 mytestpub | public     | inventory
```

有用吗？

在`inventory`表中插入几条记录

```
psql -h <DBNAME>.postgres.database.azure.com -p 5432 -U <DBUSER>@<DBNAME> -W -d postgres --set=sslmode=requireINSERT INTO inventory (item, qty) VALUES ('apples', '100');
INSERT INTO inventory (item, qty) VALUES ('oranges', '42');select * from inventory;
```

连接器应该将变更事件从 PostgreSQL WAL(预写日志)推送到 Kafka。查看相应 Kafka 主题中的消息:

```
//exec into the kafka docker container
docker exec -it debezium-postgres-pgoutput_kafka_1 bashcd bin && ./kafka-console-consumer.sh --topic myserver.public.inventory --bootstrap-server kafka:9092 --from-beginnin
```

您应该看到几个变更日志事件有效负载(对应于两个`INSERT`)

> *是的，它们是冗长的，因为模式包含在有效负载中*

**将 publication.autocreate.mode 更改为禁用**

对于这种模式，我们需要预先创建一个出版物。既然我们已经有一个(`mytestpub`)，那就用吧。你所需要做的就是将`pg-source-connector.json`中的`publication.autocreate.mode`更新为`disabled`。

重新创建连接器:

```
//delete
curl -X DELETE localhost:8083/connectors/inventory-connector//create
curl -X POST -H "Content-Type: application/json" --data @pg-source-connector.json [http://localhost:8083/connectors](http://localhost:8083/connectors)
```

使用与上一节中相同的步骤进行端到端测试——一切都应该正常工作！

> *确认一下，将连接器配置中的* `*publication.name*` *更新为一个不存在的。由于缺少发布，连接器将无法启动(如预期的那样)*

**试试 publication . auto create . mode = all _ tables**

将`publication.autocreate.mode`设置为`all_tables`，`publication.name`设置为不存在的(如`testpub1`)并创建连接器:

```
curl -X POST -H "Content-Type: application/json" --data @pg-source-connector.json [http://localhost:8083/connectors](http://localhost:8083/connectors)
```

(不出所料)它将失败，并出现类似如下的错误:

```
....
INFO Creating new publication 'testpub1' for plugin 'PGOUTPUT' (io.debezium.connector.postgresql.connection.PostgresReplicationConnection:127)
ERROR WorkerSourceTask{id=inventory-connector-0} Task threw an uncaught and unrecoverable exception (org.apache.kafka.connect.runtime.WorkerTask:179)
io.debezium.jdbc.JdbcConnectionException: ERROR: must be superuser to create FOR ALL TABLES publication
....
```

注意部分`must be superuser to create FOR ALL TABLES publication`——如前所述，`CREATE PUBLICATION <publication_name> FOR ALL TABLES;`由于缺乏超级用户权限而失败。

> *正如我前面提到的，你需要通过手工为*特定的*表*创建发布来解决这个问题

## 打扫

为了清理，使用 [az postgres server delete](https://docs.microsoft.com/cli/azure/postgres/server?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-postgres-server-delete) 删除 Azure PostgreSQL 实例并移除容器

```
az postgres server delete -g <resource group> -n <server name>docker-compose down -v
```

这篇简短的博文就到这里。敬请关注更多内容！