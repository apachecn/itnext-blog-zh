# 使用 Go SDK 开始使用 Azure Data Explorer

> 原文：<https://itnext.io/getting-started-with-azure-data-explorer-using-the-go-sdk-807c50bc8105?source=collection_archive---------3----------------------->

在一个例子的帮助下，这篇博文将带你了解如何使用 [Azure Data explorer Go SDK](https://docs.microsoft.com/azure/data-explorer/kusto/api/golang/kusto-golang-client-library?WT.mc_id=medium-blog-abhishgu) 从 [Azure Blob storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-introduction?WT.mc_id=medium-blog-abhishgu) 容器中获取数据，并使用 SDK 以编程方式查询数据。在快速概述了如何设置 Azure Data Explorer 集群(和一个数据库)之后，我们将研究代码以了解发生了什么(以及如何发生),并最终使用一个简单的 CLI 界面测试应用程序

![](img/3281168afb46ac0a08fcf96a0c90c417.png)

使用 Go SDK 的 Azure Data Explorer

样本数据是一个 CSV 文件，可以从这里下载

> *代码在 GitHub*[*https://github.com/abhirockzz/azure-dataexplorer-go*](https://github.com/abhirockzz/azure-dataexplorer-go)上有

# 什么是 Azure Data Explorer？

[Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/?WT.mc_id=medium-blog-abhishgu) (也称为 **Kusto** )是一种快速、可扩展的数据探索服务，用于分析来自任何数据源的大量不同数据，如网站、应用程序、物联网设备等。这些数据可用于诊断、监控、报告、机器学习和其他分析功能。

它支持[几种摄取方法](https://docs.microsoft.com/azure/data-explorer/ingest-data-overview?WT.mc_id=medium-blog-abhishgu)，包括到通用服务的连接器，如[事件中心](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-hub?WT.mc_id=medium-blog-abhishgu)，使用 SDK 的编程摄取，如[。NET](https://docs.microsoft.com/azure/data-explorer/net-sdk-ingest-data?WT.mc_id=medium-blog-abhishgu) 和 [Python](https://docs.microsoft.com/azure/data-explorer/python-ingest-data?WT.mc_id=medium-blog-abhishgu) ，并直接访问引擎进行探索。它还集成了分析和建模服务，使用 [Power BI](https://docs.microsoft.com/azure/data-explorer/power-bi-best-practices?WT.mc_id=medium-blog-abhishgu) 等工具进行额外的数据分析和可视化

![](img/6651c2e6f98d08324ba59405399db90e.png)

[Azure 数据浏览器流程](https://docs.microsoft.com/azure/data-explorer/data-explorer-overview?WT.mc_id=medium-blog-abhishgu#azure-data-explorer-flow)

## Go SDK for Azure Data Explorer

[Go 客户端 SDK](https://github.com/Azure/azure-kusto-go) 允许您使用 [Go](https://golang.org/) 查询、控制和接收 Azure Data Explorer 集群。请注意，这是为了与 Azure Data Explorer 集群(以及相关组件，如表等)进行交互。).创建 Azure Data Explorer 集群、数据库等。你应该使用[管理组件(控制平面)SDK](https://github.com/Azure/azure-sdk-for-go/tree/master/services/kusto/mgmt) ，它是更大的 [Azure SDK for Go](https://docs.microsoft.com/azure/developer/go/?WT.mc_id=medium-blog-abhishgu) 的一部分

> *API 文档—【https://godoc.org/github.com/Azure/azure-kusto-go】[](https://godoc.org/github.com/Azure/azure-kusto-go)*

*在开始之前，下面是您试用示例应用程序所需的内容*

# *先决条件*

*[安装 Go 1.13 或以上](https://golang.org/dl/)*

*你需要一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)。去吧[注册一个免费的！](https://azure.microsoft.com/free/?WT.mc_id=medium-blog-abhishgu)*

*安装 Azure CLI 如果你还没有的话(应该很快！)*

# *设置 Azure Data Explorer 群集，创建数据库并配置安全性*

*首先使用 [az kusto 集群创建](https://docs.microsoft.com/cli/azure/kusto/cluster?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-kusto-cluster-create)创建一个集群。完成后，使用 [az kusto database create](https://docs.microsoft.com/cli/azure/kusto/database?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-kusto-database-create) 创建一个数据库，例如*

```
*az kusto cluster create -l "Central US" -n MyADXCluster -g MyADXResGrp --sku Standard_D11_v2 --capacity 2az kusto database create --cluster-name MyADXCluster -g MyADXResGrp -n MyADXdbaz kusto database show --cluster-name MyADXCluster --name MyADXdb --resource-group MyADXResGrp*
```

*使用 [az ad sp create-for-rbac](https://docs.microsoft.com/cli/azure/ad/sp?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-ad-sp-create-for-rbac) 创建服务主体*

```
*az ad sp create-for-rbac -n "test-datax-sp"*
```

*您将得到一个 JSON 响应——请记下`appId`、`password`和`tenant`,因为您将在后续步骤中使用它们*

```
*{
  "appId": "fe7280c7-5705-4789-b17f-71a472340429",
  "displayName": "test-datax-sp",
  "name": "http://test-datax-sp",
  "password": "29c719dd-f2b3-46de-b71c-4004fb6116ee",
  "tenant": "42f988bf-86f1-42af-91ab-2d7cd011db42"
}*
```

*您需要为[服务主体](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals?WT.mc_id=medium-blog-abhishgu#service-principal-object)分配角色，以便它可以访问您刚刚创建的数据库。要使用 Azure 门户实现这一点，请打开 Azure Data Explorer 集群，导航到`Data > Databases`并选择数据库。从左侧菜单中选择`Permissions`并点击`Add`继续。*

*![](img/9e4c7a158ae948dd18ead4e7c86ea6f4.png)*

> **更多信息，请参考* [*安全 Azure 中的 Azure Data Explorer 集群*](https://docs.microsoft.com/azure/data-explorer/security?WT.mc_id=medium-blog-abhishgu#role-based-access-control)*

# *代码遍历*

*概括地说，示例代码是这样做的:*

*   *连接到一个 Azure Data Explorer 集群(当然！)*
*   *创建一个表格(并列出它们以确保万无一失)*
*   *创建数据映射*
*   *从 Azure Blob 存储中的 CSV 文件接收/加载现有数据*
*   *对您刚刚摄取的数据运行查询*

*让我们来看看每一个步骤*

## *连接到 Azure 数据浏览器群集*

*我们使用服务主体对 Azure Data Explorer 进行身份验证，并提供 Azure 租户 ID、客户端 ID 和客户端机密(在使用`az ad sp create-for-rbac`创建主体后获得)*

```
*auth := kusto.Authorization{Config: auth.NewClientCredentialsConfig(clientID, clientSecret, tenantID)}
kc, err := kusto.New(kustoEndpoint, auth)
if err != nil {
    log.Fatal("failed to create kusto client", err)
}*
```

> **你可以在这里* [*查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/cli/util/util.go#L12)*

## *创建表和数据映射*

*要创建一个表，我们只需执行`create table`*

```
*func CreateTable(kc *kusto.Client, kustoDB string) {
    _, err := kc.Mgmt(context.Background(), kustoDB, kusto.NewStmt(createTableCommand))
    if err != nil {
        log.Fatal("failed to create table", err)
    } log.Printf("table %s created\n", kustoTable)
}*
```

> **您可以在这里* [*查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/setup/setup.go#L22)*

*注意我们如何使用[客户端。Mgmt](https://godoc.org/github.com/Azure/azure-kusto-go/kusto#Client.Mgmt) 来执行这个操作，因为这是一个`management`查询。稍后，您将看到如何执行查询从 Azure Data Explorer 读取数据。*

*为了确认，我们运行一个查询来检查数据库中的表，即`show tables`*

```
*func FindTable(kc *kusto.Client, kustoDB string) []TableInfo {
    var tables []TableInfo
    ri, err := kc.Mgmt(context.Background(), kustoDB, kusto.NewStmt(testQuery))
    if err != nil {
        log.Fatalf("failed to execute query %s - %s", testQuery, err)
    }
    var t TableInfo
    for {
        row, err := ri.Next()
        if err != nil {
            if err == io.EOF {
                break
            } else {
                log.Println("error", err)
            }
        }
        row.ToStruct(&t)
        tables = append(tables, t)
    }
    return tables
}
...
type TableInfo struct {
    Name string `kusto:"TableName"`
    DB   string `kusto:"DatabaseName"`
}*
```

> **你可以在这里* [*查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/setup/setup.go#L32)*

*执行查询后，`[ToStruct](https://github.com/Azure/azure-kusto-go/blob/master/kusto/data/table/table.go#L120)`用于将结果保存到一个用户定义的`TableInfo`结构的实例中*

*一旦创建了表，我们就可以[配置数据映射](https://docs.microsoft.com/azure/data-explorer/kusto/management/mappings?WT.mc_id=medium-blog-abhishgu),在摄取过程中使用这些映射将输入数据映射到 Kusto 表中的列*

```
*func CreateMapping(kc *kusto.Client, kustoDB string) {
    _, err := kc.Mgmt(context.Background(), kustoDB, kusto.NewStmt(createMappingCommand))
    if err != nil {
        log.Fatal("failed to create mapping", err)
    }
    log.Printf("mapping %s created\n", kustoMappingRefName)
}*
```

> **你可以在这里* [*查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/setup/setup.go#L61)*

## *从 Azure Blob 存储中接收数据*

*为了获取数据，我们使用了`[Ingestion](https://godoc.org/github.com/Azure/azure-kusto-go/kusto/ingest#Ingestion)`客户端*

```
*const blobStorePathFormat = "https://%s.blob.core.windows.net/%s/%s%s"func CSVFromBlob(kc *kusto.Client, blobStoreAccountName, blobStoreContainer, blobStoreToken, blobStoreFileName, kustoMappingRefName, kustoDB, kustoTable string) {
    kIngest, err := ingest.New(kc, kustoDB, kustoTable)
    if err != nil {
        log.Fatal("failed to create ingestion client", err)
    }
    blobStorePath := fmt.Sprintf(blobStorePathFormat, blobStoreAccountName, blobStoreContainer, blobStoreFileName, blobStoreToken)
    err = kIngest.FromFile(context.Background(), blobStorePath, ingest.FileFormat(ingest.CSV), ingest.IngestionMappingRef(kustoMappingRefName, ingest.CSV)) if err != nil {
        log.Fatal("failed to ingest file", err)
    }
    log.Println("Ingested file from -", blobStorePath)
}*
```

> **你可以在这里* [*查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/load/ingest.go#L15)*

*我们在 Azure Blob 存储中有文件的路径，我们在`[FromFile](https://godoc.org/github.com/Azure/azure-kusto-go/kusto/ingest#Ingestion.FromFile)`函数中引用它以及文件类型(本例中为`CSV`)和我们刚刚创建的数据映射(`StormEvents_CSV_Mapping`)*

## *查询数据*

*我们使用以下查询从`StormEvents`表中获取一些数据:*

```
*StormEvents | where EventType == 'Flood' and State == 'WASHINGTON' | sort by DamageProperty desc | project StartTime, EndTime, Source, DamageProperty*
```

*这一次，我们使用`client.Query`(不是`Mgmt`)从表中读取数据。*

```
*func Get(kc *kusto.Client, kustoDB string) []StormDetails {
    var events []StormDetail
    ri, err := kc.Query(context.Background(), kustoDB, kusto.NewStmt(query)) if err != nil {
        log.Fatalf("failed to execute query %s - %s", query, err)
    }
    for {
        row, err := ri.Next()
        if err != nil {
            if err == io.EOF {
                break
            } else {
                log.Println("error", err)
            }
        }
        var event StormDetail
        row.ToStruct(&event)
        events = append(events, event)
    }
    return events
...
type StormDetail struct {
    Start  time.Time `kusto:"StartTime"`
    End    time.Time `kusto:"EndTime"`
    From   string    `kusto:"Source"`
    Damage int32     `kusto:"DamageProperty"`
}*
```

*使用`ToStruct`将结果中的每一行转换成一个`StormDetail`结构*

> **您可以在这里* [*查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/query/query.go#L15)*

*然后，列表以用户友好的表格格式显示到标准输出中*

```
*...
data := [][]string{}
for _, detail := range details {
    data = append(data, []string{detail.Start.String(), detail.End.String(), detail.From, strconv.Itoa(int(detail.Damage))})
}
log.Println("StormEvents data....")
table := tablewriter.NewWriter(os.Stdout)
table.SetHeader([]string{"Start Time", "End Time", "From", "Damage"})for _, v := range data {
    table.Append(v)
}
table.Render()
...*
```

> **您可以在这里* [*查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/cli/query.go#L23)*

*最后，为了删除表格，我们使用`.drop table StormEvents`*

```
*const dropTableQ = ".drop table StormEvents"func dropTable(kc *kusto.Client) {
    _, err := kc.Mgmt(context.Background(), kustoDB, kusto.NewStmt(dropTableQ))
    if err != nil {
        log.Fatal("failed to drop table - ", err)
    }
}*
```

> **您可以在* [*这里查看代码*](https://github.com/abhirockzz/azure-dataexplorer-go/blob/master/setup/setup.go#L72)*

# *运行示例*

*现在您已经知道发生了什么，让我们使用 CLI 来尝试一下*

*设置所需的环境变量*

```
*export AZURE_SP_CLIENT_ID="service principal client id"
export AZURE_SP_CLIENT_SECRET="<service principal client secret>"
export AZURE_SP_TENANT_ID="<tenant ID>"#e.g. https://mykusto.southeastasia.kusto.windows.net
export KUSTO_ENDPOINT="https://<cluster name>.<azure region>.kusto.windows.net"*
```

*获取代码并构建它*

```
*git clone https://github.com/abhirockzz/azure-dataexplorer-go
cd azure-dataexplorer-go
go build -o azdatax//to confirm
chmod a+x azdatax && ./azdatax//output
CLI to test sample program for Azure Data ExplorerUsage:
  azdatax [command]Available Commands:
  create-mapping creates a mapping named StormEvents_CSV_Mapping
  create-table   creates a table named StormEvents
  drop-table     drops the StormEvents table
  get            queries data from StormEvents
  help           Help about any command
  ingest         ingests CSV file from blob store
  list-tables    lists tablesFlags:
  -h, --help   help for azdataxUse "azdatax [command] --help" for more information about a command.*
```

*让我们首先创建一个表:*

```
*./azdatax create-table --dbname <name of the database you created initially>//output
Connected to Azure Data Explorer
table StormEvents created*
```

*要列出表格:*

```
*./azdatax list-tables --dbname <name of the database you created initially>//output
Connected to Azure Data Explorer
Table name: StormEvents, Database name: testkustodb*
```

*要创建数据映射:*

```
*./azdatax create-mapping --dbname <name of the database you created initially>//output
Connected to Azure Data Explorer
mapping StormEvents_CSV_Mapping created*
```

*要摄取数据:*

```
*./azdatax ingest --dbname <name of the database you created initially>//outputConnected to Azure Data Explorer
Ingested file from - [https://kustosamplefiles.blob.core.windows.net/samplefiles/StormEvents.csv?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D](https://kustosamplefiles.blob.core.windows.net/samplefiles/StormEvents.csv?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D)*
```

*在尝试查询数据(下一步)之前，请等待一段时间，等待接收完成*

*要查询数据:*

```
*./azdatax get --dbname <name of the database you created initially>//output
Connected to Azure Data Explorer
StormEvents data....
+-------------------------------+-------------------------------+---------------------------+----------+
|          START TIME           |           END TIME            |           FROM            |  DAMAGE  |
+-------------------------------+-------------------------------+---------------------------+----------+
| 2007-12-02 23:00:00 +0000 UTC | 2007-12-05 23:00:00 +0000 UTC | Official NWS Observations | 50000000 |
| 2007-01-03 00:00:00 +0000 UTC | 2007-01-03 22:00:00 +0000 UTC | Newspaper                 |    20000 |
| 2007-12-03 03:00:00 +0000 UTC | 2007-12-05 19:00:00 +0000 UTC | Official NWS Observations |    12000 |
| 2007-01-03 00:00:00 +0000 UTC | 2007-01-03 22:00:00 +0000 UTC | Newspaper                 |     5000 |
| 2007-03-12 00:00:00 +0000 UTC | 2007-03-13 23:00:00 +0000 UTC | Public                    |        0 |
| 2007-03-12 00:00:00 +0000 UTC | 2007-03-14 23:00:00 +0000 UTC | Other Federal             |        0 |
+-------------------------------+-------------------------------+---------------------------+----------+*
```

*最后，删除 StormEvents 表:*

```
*./azdatax drop-table --dbname <name of the database you created initially>//output
Connected to Azure Data Explorer
Table StormEvents dropped*
```

# *结论*

*希望这有助于演示如何使用 Go SDK 与 Azure Data Explorer 交互。这显然只是其中的一种方式！更多信息，请阅读文档，祝您探索愉快！*