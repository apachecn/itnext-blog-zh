# 提示:使用 MongoDB Go 驱动程序连接到 Azure Cosmos DB MongoDB API

> 原文：<https://itnext.io/tip-connecting-to-azure-cosmos-db-mongodb-api-using-the-mongodb-go-driver-a9ca1230a1c7?source=collection_archive---------6----------------------->

[Azure Cosmos DB 提供 MongoDB API 支持](https://docs.microsoft.com/azure/cosmos-db/mongodb-introduction?WT.mc_id=medium-blog-abhishgu)，这意味着您可以使用特定语言的驱动程序来本地连接到 Cosmos DB。

![](img/df66a03c2426c934ebb8fb5a3eab8be8.png)

> *文档中有很多资源，例如* [*一个 Java 快速入门*](https://docs.microsoft.com/azure/cosmos-db/create-mongodb-java?WT.mc_id=medium-blog-abhishgu) *，* [*Node.js + React 教程*](https://docs.microsoft.com/azure/cosmos-db/tutorial-develop-mongodb-react?WT.mc_id=medium-blog-abhishgu) *等。*

但是 [Golang](http://golang.org/) 呢，具体来说，官方 [MongoDB 驱动](https://github.com/mongodb/mongo-go-driver)呢？我期望它“正常工作”，但是我面临一个小问题。在糟糕的一天，我可能会花上几个小时，然后(可能)放弃。幸运的是，事实并非如此！希望这篇文章能帮你节省一些时间，以防你遇到这种情况😉

我首先为 MongoDB 版本`3.6` ( [另一个支持的版本是](https://docs.microsoft.com/azure/cosmos-db/mongodb-feature-support?WT.mc_id=medium-blog-abhishgu) `[3.2](https://docs.microsoft.com/azure/cosmos-db/mongodb-feature-support?WT.mc_id=medium-blog-abhishgu)`)创建了一个 Azure Cosmos DB 帐户，并从门户获取了[连接字符串。格式如下:](https://docs.microsoft.com/azure/cosmos-db/connect-mongodb-account?WT.mc_id=medium-blog-abhishgu)

```
mongodb://[COSMOSDB_ACCOUNT_NAME]:[PRIMARY_PASSWORD]@[COSMOSDB_ACCOUNT_NAME].mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb&retrywrites=false&maxIdleTimeMS=120000&appName=@[COSMOSDB_ACCOUNT_NAME]@
```

> *注意，在编写的时候，你只能使用 Azure portal*[*创建 MongoDB 版本*](https://docs.microsoft.com/azure/cosmos-db/cli-samples-mongodb?WT.mc_id=medium-blog-abhishgu) `[*3.6*](https://docs.microsoft.com/azure/cosmos-db/cli-samples-mongodb?WT.mc_id=medium-blog-abhishgu)` [*(不是 Azure CLI)*](https://docs.microsoft.com/azure/cosmos-db/cli-samples-mongodb?WT.mc_id=medium-blog-abhishgu)

我使用这个简单的程序只是为了测试连通性

```
package mainimport (
	"context"
	"fmt"
	"log"
	"os"
	"time" "go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)var connectionStr stringfunc init() {
	connectionStr = os.Getenv("MONGODB_CONNECTION_STRING")
	if connectionStr == "" {
		log.Fatal("Missing enviornment variable MONGODB_CONNECTION_STRING")
	}
}func main() { ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
	defer cancel() fmt.Println("connecting...")
	clientOptions := options.Client().ApplyURI(connectionStr)
	c, err := mongo.NewClient(clientOptions) err = c.Connect(ctx) if err != nil {
		log.Fatalf("unable to connect %v", err)
	}
	err = c.Ping(ctx, nil)
	if err != nil {
		log.Fatalf("unable to ping Cosmos DB %v", err)
	} fmt.Println("connected!")
}
```

要进行测试，只需设置连接字符串并运行程序

```
export MONGODB_CONNECTION_STRING=<copy-paste from azure portal>go run main.go
```

我本以为会看到`connected!`，但结果却是这样:

```
unable to Ping: connection(cdb-ms-prod-southeastasia1-cm1.documents.azure.com:10255[-5]) connection is closed
```

我根据“有根据的猜测”开始调试。Azure Cosmos DB 通过[有线协议兼容性](https://docs.microsoft.com/azure/cosmos-db/mongodb-introduction?WT.mc_id=medium-blog-abhishgu#wire-protocol-compatibility)提供 MongoDB 支持，并允许您使用单个端点进行连接(参见连接字符串)。这意味着您不必担心底层的数据库集群拓扑。

也许司机想在这点上“聪明一点”？我开始更仔细地研究 Mongo 驱动程序`[ClientOptions](https://godoc.org/go.mongodb.org/mongo-driver/mongo/options#ClientOptions)`及其可用方法，结果发现:`[SetDirect](https://godoc.org/go.mongodb.org/mongo-driver/mongo/options#ClientOptions.SetDirect)`方法！

这是 Godoc:

```
SetDirect specifies whether or not a direct connect should be made. To use this option, a URI with a single host must be specified through ApplyURI. If set to true, the driver will only connect to the host provided in the URI and will not discover other hosts in the cluster. This can also be set through the "connect" URI option with the following values:1\. "connect=direct" for direct connections2\. "connect=automatic" for automatic discovery.The default is false ("automatic" in the connection string).
```

我所要做的就是更新`ClientOptions`

```
clientOptions := options.Client().ApplyURI(connectionStr).**SetDirect(true)**
```

> *你也可以通过添加* `*connect=direct*` *(根据 Godoc)将它添加到连接字符串本身，我可以确认它也能工作*

就是这样。现在我能够连接到 Azure Cosmos DB MongoDB API 了🙌请继续关注即将发布的博客文章，在这篇文章中，我将借助一个实际的例子深入研究 Azure Cosmos DB + Go 上的 MongoDB！