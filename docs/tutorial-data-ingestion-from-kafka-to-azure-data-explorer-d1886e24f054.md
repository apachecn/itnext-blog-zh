# 教程:从 Kafka 到 Azure Data Explorer 的数据摄取

> 原文：<https://itnext.io/tutorial-data-ingestion-from-kafka-to-azure-data-explorer-d1886e24f054?source=collection_archive---------3----------------------->

这篇博客将涵盖使用 [Kafka Connect](https://kafka.apache.org/documentation/#connect) 从 [Kafka](https://kafka.apache.org/) 到 [Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/data-explorer-overview?WT.mc_id=medium-blog-abhishgu) (Kusto)的数据摄取。

Azure Data Explorer 是一种快速、可扩展的数据探索服务，允许您收集、存储和分析来自任何不同来源的大量数据，如网站、应用程序、物联网设备等。Kafka Connect platform 允许您以可伸缩和可靠的方式在 Apache Kafka 和外部系统之间传输数据。用于 Azure Data Explorer 的 Kafka Connect Sink 连接器允许您将 Kafka 主题中的数据移动到 Azure Data Explorer 表中，您可以稍后查询和分析这些表。

> *下面是本博客的 GitHub repo—*[*https://GitHub . com/abhirockzz/Kafka-kusto-ingestion-tutorial*](https://github.com/abhirockzz/kafka-kusto-ingestion-tutorial)

我们的目标是快速开始*，所以我们将保持事情的简单性，并将所有事情归档！这包括 Kafka、Zookeeper、Kafka Connect worker 和事件生成器应用程序——在 [docker-compose.yaml](https://github.com/abhirockzz/kafka-kusto-ingestion-tutorial/blob/master/docker-compose.yaml) 中定义*

*![](img/990f8939cc3eb485613ef441468674e8.png)*

*在本教程中，您将:*

*   *获得单个组件的概述*
*   *配置和设置 Azure 数据浏览器并安装连接器*
*   *运行端到端演示*

# *先决条件*

*   *你需要一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)。也许试试免费的？*
*   *安装 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu) 如果你还没有的话(应该很快！)或者直接从你的浏览器使用 [Azure 云壳](https://azure.microsoft.com/features/cloud-shell/?WT.mc_id=medium-blog-abhishgu)。*
*   *[对接器](https://docs.docker.com/get-docker/)和[对接器组成](https://docs.docker.com/compose/install)安装*

# *概观*

*如前所述，所有组件都在`docker-compose.yaml`文件中定义。让我们一点一点地检查一下:*

*卡夫卡和动物园管理员的部分非常简单——使用 [debezium](https://hub.docker.com/r/debezium/kafka/) 图片*

```
*zookeeper:
    image: debezium/zookeeper:1.2
    ports:
      - 2181:2181
kafka:
    image: debezium/kafka:1.2
    ports:
      - 9092:9092
    links:
      - zookeeper
    depends_on:
      - zookeeper
    environment:
      - ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092*
```

*`events-producer`服务是一个[简单应用程序](https://github.com/abhirockzz/kafka-kusto-ingestion-tutorial/tree/master/storm-events-producer),它向 Kafka 主题发送风暴事件数据。Storm Events data 是 Azure Data Explorer 文档中使用的一个典型示例(例如， [check this Quickstart](https://docs.microsoft.com/azure/data-explorer/ingest-sample-data?WT.mc_id=medium-blog-abhishgu#ingest-data) 和 [the complete CSV file](https://kustosamplefiles.blob.core.windows.net/samplefiles/StormEvents.csv?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D) )。producer 应用程序使用原始 CSV，但只包括选定的字段(如开始和结束时间、状态、来源等。)而不是整行(超过 20 列)。以下是示例数据:*

```
*2007-01-01 00:00:00.0000000,2007-01-01 05:00:00.0000000,23357,WISCONSIN,Winter Storm,COOP Observer
2007-01-01 00:00:00.0000000,2007-01-01 06:00:00.0000000,9488,NEW YORK,Winter Weather,Department of Highways
2007-01-01 00:00:00.0000000,2007-01-01 06:00:00.0000000,9487,NEW YORK,Winter Weather,Department of Highways
...*
```

*Docker Compose 中的服务组件是这样定义的:*

```
***events-producer:**
    build:
      **context: ./storm-events-producer**
    links:
      - kafka
    depends_on:
      - kafka
    environment:
      - KAFKA_BOOTSTRAP_SERVER=kafka:9092
      - KAFKA_TOPIC=storm-events
      - **SOURCE_FILE=StormEvents.csv***
```

*接收器连接器是许多奇迹发生的地方！让我们来探索一下:*

## *用于 Azure 数据浏览器的 Kafka 接收器连接器*

*以下是 docker 编写文件中的`kusto-connect`服务:*

```
***kusto-connect:**
    build:
      **context: ./connector**
    ports:
      - 8083:8083
    links:
      - kafka
    depends_on:
      - kafka
    environment:
      - BOOTSTRAP_SERVERS=kafka:9092
      - GROUP_ID=adx
      - CONFIG_STORAGE_TOPIC=my_connect_configs
      - OFFSET_STORAGE_TOPIC=my_connect_offsets
      - STATUS_STORAGE_TOPIC=my_connect_statuses*
```

*容器是从一个`Dockerfile`构建的——这使得您可以更容易地在本地运行它，而不是从外部 Docker 注册表中获取它*

```
*FROM debezium/connect:1.2WORKDIR $KAFKA_HOME/connectARG KUSTO_KAFKA_SINK_VERSIONRUN curl -L -O [https://github.com/Azure/kafka-sink-azure-kusto/releases/download/v$KUSTO_KAFKA_SINK_VERSION/kafka-sink-azure-kusto-$KUSTO_KAFKA_SINK_VERSION-jar-with-dependencies.jar](https://github.com/Azure/kafka-sink-azure-kusto/releases/download/v$KUSTO_KAFKA_SINK_VERSION/kafka-sink-azure-kusto-$KUSTO_KAFKA_SINK_VERSION-jar-with-dependencies.jar)*
```

*它基于 [Debezium Kafka Connect](https://hub.docker.com/r/debezium/connect/) 图像之上。只需下载 Kusto 连接器 JAR ( [版本 1.0.1](https://github.com/Azure/kafka-sink-azure-kusto/releases/tag/v1.0.1) 在撰写本文时)并将其放在 Kafka Connect 插件目录中。就是这样！*

*接收器连接器配置文件如下所示:*

```
*{
    "name": "storm",
    "config": {
        "connector.class": "com.microsoft.azure.kusto.kafka.connect.sink.KustoSinkConnector",
        "flush.size.bytes": 10000,
        "flush.interval.ms": 50000,
        "tasks.max": 1,
        "topics": "storm-events",
        "kusto.tables.topics.mapping": "[{'topic': 'storm-events','db': '<enter database name>', 'table': 'Storms','format': 'csv', 'mapping':'Storms_CSV_Mapping'}]",
        "aad.auth.authority": "<enter tenant ID>",
        "aad.auth.appid": "<enter application ID>",
        "aad.auth.appkey": "<enter client secret>",
        "kusto.url": "https://ingest-<name of cluster>.<region>.kusto.windows.net",
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "value.converter": "org.apache.kafka.connect.storage.StringConverter"
    }
}*
```

*在 Azure Data Explorer 中将数据加载/导入到表中的过程被称为摄取。这也是连接器的工作方式。*

*![](img/a8d5723fc44921873a368f5853816b2c.png)*

*[https://docs . Microsoft . com/en-us/azure/data-explorer/ingest-data-overview](https://docs.microsoft.com/en-us/azure/data-explorer/ingest-data-overview)*

*在幕后，它使用了用于 Azure Data Explorer 的 [Java SDK 中的以下模块](https://docs.microsoft.com/azure/data-explorer/kusto/api/java/kusto-java-client-library?WT.mc_id=medium-blog-abhishgu)*

*   *`data`:连接，发布(控制)命令，查询数据*
*   *`ingest`:摄取数据*

*编写时，连接器支持的数据格式有:`csv`、`json`、`txt`、`avro`、`apacheAvro`、`tsv`、`scsv`、`sohsv`、`psv`。Kafka 主题中的数据被写到磁盘上的文件中。然后，基于以下连接器配置，这些文件被发送到 Azure Data Explorer 当文件到达`flush.size.bytes` **或**时，`flush.interval.ms`间隔已过。*

> **上述机制的唯一例外是作为字节数组*处理的 `*avro*` *和* `*apacheAvro*` *数据类型**

*通过“发送到 Azure Data Explorer”，我真正的意思是文件被排队等待接收(使用[ingest client . ingestfromfile](https://github.com/Azure/azure-kusto-java/blob/35d682a89b6b76be5196d73ad10a52fd47ef9ad5/ingest/src/main/java/com/microsoft/azure/kusto/ingest/IngestClient.java#L31))*

*好了，到目前为止有很多理论…*

# *..我们来试试吧！*

*克隆此回购:*

```
*git clone [https://github.com/abhirockzz/kafka-kusto-ingestion-tutorial](https://github.com/abhirockzz/kafka-kusto-ingestion-tutorial)cd kafka-kusto-ingestion-tutorial*
```

*使用 Azure Portal 、 [Azure CLI](https://docs.microsoft.com/azure/data-explorer/create-cluster-database-cli?WT.mc_id=medium-blog-abhishgu) 或任何客户端 SDK(如 [Python](https://docs.microsoft.com/azure/data-explorer/create-cluster-database-python?WT.mc_id=medium-blog-abhishgu) )开始创建 Azure Data Explorer 集群和数据库[。](https://docs.microsoft.com/azure/data-explorer/create-cluster-database-portal?WT.mc_id=medium-blog-abhishgu)*

*完成后，创建一个表(`Storms`)和相应的映射(`Storms_CSV_Mapping`):*

```
*.create table Storms (StartTime: datetime, EndTime: datetime, EventId: int, State: string, EventType: string, Source: string).create table Storms ingestion csv mapping 'Storms_CSV_Mapping' '[{"Name":"StartTime","datatype":"datetime","Ordinal":0}, {"Name":"EndTime","datatype":"datetime","Ordinal":1},{"Name":"EventId","datatype":"int","Ordinal":2},{"Name":"State","datatype":"string","Ordinal":3},{"Name":"EventType","datatype":"string","Ordinal":4},{"Name":"Source","datatype":"string","Ordinal":5}]'*
```

## *启动容器并安装连接器*

*在安装连接器之前，我们需要创建一个服务主体，以便连接器进行身份验证并连接到 Azure Data Explorer 服务。*

*使用 [az ad sp create-for-rbac](https://docs.microsoft.com/cli/azure/ad/sp?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-ad-sp-create-for-rbac) 命令:*

```
*az ad sp create-for-rbac -n "kusto-sp"*
```

*您将得到一个 JSON 响应——请记下`appId`、`password`和`tenant`,因为您将在后续步骤中使用它们*

```
*{
  "appId": "fe7280c7-5705-4789-b17f-71a472340429",
  "displayName": "kusto-sp",
  "name": "http://kusto-sp",
  "password": "29c719dd-f2b3-46de-b71c-4004fb6116ee",
  "tenant": "42f988bf-86f1-42af-91ab-2d7cd011db42"
}*
```

*启动容器:*

```
*docker-compose up*
```

*生产者应用程序将开始向`storm-events`主题发送事件。您应该会看到类似于以下内容的日志:*

```
*....
events-producer_1  | sent message to partition 0 offset 0
events-producer_1  | event  2007-01-01 00:00:00.0000000,2007-01-01 00:00:00.0000000,13208,NORTH CAROLINA,Thunderstorm Wind,Public
events-producer_1  | 
events-producer_1  | sent message to partition 0 offset 1
events-producer_1  | event  2007-01-01 00:00:00.0000000,2007-01-01 05:00:00.0000000,23358,WISCONSIN,Winter Storm,COOP Observer
events-producer_1  | 
events-producer_1  | sent message to partition 0 offset 2
events-producer_1  | event  2007-01-01 00:00:00.0000000,2007-01-01 05:00:00.0000000,23357,WISCONSIN,Winter Storm,COOP Observer
events-producer_1  | 
events-producer_1  | sent message to partition 0 offset 3
events-producer_1  | event  2007-01-01 00:00:00.0000000,2007-01-01 06:00:00.0000000,9494,NEW YORK,Winter Weather,Department of Highways
events-producer_1  | 
events-producer_1  | sent message to partition 0 offset 4
events-producer_1  | 2020/08/20 16:51:35 event  2007-01-01 00:00:00.0000000,2007-01-01 06:00:00.0000000,9488,NEW YORK,Winter Weather,Department of Highways
....*
```

*我们现在可以安装接收器连接器来消费这些事件，并将它们接收到 Azure Data Explorer 中*

*替换`adx-sink-config.json`中下列属性的值:`aad.auth.authority`、`aad.auth.appid`、`aad.auth.appkey`、`kusto.tables.topics.mapping`(数据库名称)和`kusto.url`*

```
*{
    "name": "storm",
    "config": {
        "connector.class": "com.microsoft.azure.kusto.kafka.connect.sink.KustoSinkConnector",
        "flush.size.bytes": 10000,
        "flush.interval.ms": 50000,
        "tasks.max": 1,
        "topics": "storm-events",
        "kusto.tables.topics.mapping": "[{'topic': 'storm-events',**'db': '<enter database name>**', 'table': 'Storms','format': 'csv', 'mapping':'Storms_CSV_Mapping'}]",
        **"aad.auth.authority": "<enter tenant ID>"**,
        **"aad.auth.appid": "<enter application ID>**",
        "**aad.auth.appkey": "<enter client secret>**",
        **"kusto.url": "https://ingest-<name of cluster>.<region>.kusto.windows.net"**,
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "value.converter": "org.apache.kafka.connect.storage.StringConverter"
    }
}*
```

*在不同的终端中，记录连接器服务日志:*

```
*docker-compose logs -f | grep kusto-connect*
```

*安装连接器:*

```
*curl -X POST -H "Content-Type: application/json" --data @adx-sink-config.json http://localhost:8083/connectors//check status
curl [http://localhost:8083/connectors/storm/status](http://localhost:8083/connectors/storm/status)*
```

*连接器应该弹起动作。同时，在另一个终端中，您应该会看到类似于以下内容的日志:*

```
*kusto-connect_1    | INFO   ||  Refreshing Ingestion Resources   [com.microsoft.azure.kusto.ingest.ResourceManager]kusto-connect_1    | INFO   ||  Kusto ingestion: file (/tmp/kusto-sink-connector-0a8a9fa2-9e4b-414d-bae1-5d01f3969522/kafka_storm-events_0_0.csv.gz) of size (9192) at current offset (93)   [com.microsoft.azure.kusto.kafka.connect.sink.TopicPartitionWriter]kusto-connect_1    | INFO   ||  WorkerSinkTask{id=storm-0} Committing offsets asynchronously using sequence number 1: {storm-events-0=OffsetAndMetadata{offset=94, leaderEpoch=null, metadata=''}}   [org.apache.kafka.connect.runtime.WorkerSinkTask]
ct.runtime.WorkerSinkTask]kusto-connect_1    | INFO   ||  Kusto ingestion: file (/tmp/kusto-sink-connector-0a8a9fa2-9e4b-414d-bae1-5d01f3969522/kafka_storm-events_0_94.csv.gz) of size (1864) at current offset (111)   [com.microsoft.azure.kusto.kafka.connect.sink.TopicPartitionWriter]kusto-connect_1    | INFO   ||  WorkerSinkTask{id=storm-0} Committing offsets asynchronously using sequence number 2: {storm-events-0=OffsetAndMetadata{offset=112, leaderEpoch=null, metadata=''}}   [org.apache.kafka.connect.runtime.WorkerSinkTask]
....*
```

*等待一段时间，直到数据在`Storms`表中结束。要进行确认，请检查行数并确认摄取过程中没有失败:*

```
*Storms | count. show ingestion failures*
```

*一旦有了一些数据，尝试几个查询。要查看所有记录:*

```
*Storms*
```

*使用`where`和`project`过滤特定数据*

```
*Storms
| where EventType == 'Drought' and State == 'TEXAS'
| project StartTime, EndTime, Source, EventId*
```

*使用`[summarize](https://docs.microsoft.com/azure/data-explorer/write-queries?WT.mc_id=medium-blog-abhishgu#summarize)` [运算符](https://docs.microsoft.com/azure/data-explorer/write-queries?WT.mc_id=medium-blog-abhishgu#summarize)*

```
*Storms
| summarize event_count=count() by State
| where event_count > 10
| project State, event_count
| render columnchart*
```

*您应该会看到类似如下的图表:*

*![](img/a4d1aa80adb740968660858685000cef.png)*

*这些只是几个例子。深入研究 [Kusto 查询语言文档](https://docs.microsoft.com/azure/data-explorer/kusto/query/?WT.mc_id=medium-blog-abhishgu)或探索关于[如何使用](https://docs.microsoft.com/azure/data-explorer/ingest-json-formats?tabs=kusto-query-language&WT.mc_id=medium-blog-abhishgu)[标量操作符](https://docs.microsoft.com/azure/data-explorer/write-queries?WT.mc_id=medium-blog-abhishgu#scalar-operators)、[时间表](https://docs.microsoft.com/azure/data-explorer/kusto/query/tutorial?pivots=azuredataexplorer&WT.mc_id=medium-blog-abhishgu#timecharts)等将 JSON 格式的样本数据摄取到 Azure Data Explorer 中的教程。*

> **如果您想从头开始，只需停止容器(* `*docker-compose down -v*` *)、删除(* `*drop table Storms*` *)并重新创建* `*Storms*` *表(以及映射)并重新启动容器(* `*docker-compose up*` *)**

## *打扫*

*要删除 Azure Data Explorer 集群/数据库，请使用 [az 集群删除](https://docs.microsoft.com/cli/azure/kusto/cluster?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-kusto-cluster-delete)或 [az kusto 数据库删除](https://docs.microsoft.com/cli/azure/kusto/database?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-kusto-database-delete)*

```
*az kusto cluster delete -n <cluster name> -g <resource group name>az kusto database delete -n <database name> --cluster-name <cluster name> -g <resource group name>*
```

# *这是一个总结！*

*我希望这有助于您开始使用 Kafka Connect sink 连接器构建从 Kafka 到 Azure Data Explorer 的数据接收管道。这不是将数据摄取到 Azure Data Explorer 的唯一方式(当然！).欢迎您浏览文档并探索其他技术，如[一键摄取](https://docs.microsoft.com/azure/data-explorer/ingest-data-one-click?WT.mc_id=medium-blog-abhishgu)，使用[事件网格](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-grid-overview?WT.mc_id=medium-blog-abhishgu)，[物联网中心](https://docs.microsoft.com/azure/data-explorer/ingest-data-iot-hub-overview?WT.mc_id=medium-blog-abhishgu)等等！*

*下次再见，愉快的探索！*