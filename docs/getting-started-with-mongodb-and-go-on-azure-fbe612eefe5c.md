# 开始使用 MongoDB 并继续使用 Azure

> 原文：<https://itnext.io/getting-started-with-mongodb-and-go-on-azure-fbe612eefe5c?source=collection_archive---------5----------------------->

作为一个围棋爱好者，很高兴看到以官方 MongoDB Go 驱动程序的形式为 MongoDB 提供一流的支持[。我们将通过为一个好的老式 CRUD 风格的应用程序构建一个简单的 REST API 来学习如何使用它！](https://github.com/mongodb/mongo-go-driver)

在这篇博客中，将涵盖以下主题:

*   演练应用程序:CRUD 操作
*   为 MongoDB API 设置 Azure Cosmos DB
*   设置 Azure 应用服务并将应用程序部署到云
*   带着 REST API 兜一圈

接下来，您可以自由选择使用 MongoDB 集群( [Docker 可能是最快/最简单的选择](https://hub.docker.com/_/mongo))。我将使用 [Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/?WT.mc_id=medium-blog-abhishgu) ，它是微软的全球分布式多模型数据库服务，支持文档、键值、图表和列数据模型。它实现了 MongoDB 的 wire 协议，使得任何理解该协议版本的 MongoDB 客户端驱动程序(包括 Go 驱动程序)都可以本地连接到 Cosmos DB。

> *您可以在没有 Azure 订阅的情况下免费试用*[*Azure Cosmos DB*](https://azure.microsoft.com/try/cosmosdb/?WT.mc_id=medium-blog-abhishgu)*(在有限的时间内)，或者使用* [*Azure Cosmos DB 免费层*](https://docs.microsoft.com/azure/cosmos-db/optimize-dev-test?WT.mc_id=medium-blog-abhishgu#azure-cosmos-db-free-tier) *来获得一个前 400 RU/s 和 5 GB 免费存储空间的帐户。*

应用程序(API)将被部署到 [Azure 应用程序服务](https://docs.microsoft.com/azure/app-service/?WT.mc_id=medium-blog-abhishgu)。它使您能够以自己选择的编程语言构建和托管 web 应用、移动后端和 RESTful APIs，而无需管理基础设施。[您可以在 Linux 上使用它](https://docs.microsoft.com/azure/app-service/containers/app-service-linux-intro?WT.mc_id=medium-blog-abhishgu)在 Linux 上本地托管 web 应用程序，以获得支持的应用程序堆栈，并支持多种语言的内置 Docker 映像[，如 Node.js、Java、Python](https://docs.microsoft.com/azure/app-service/containers/app-service-linux-intro?WT.mc_id=medium-blog-abhishgu#languages) 等。

> *虽然我们在本例中有一个 Go 应用程序，但我们仍然可以在应用程序服务上托管它，因为它也支持自定义 Docker 图像！*

# 概观

这个应用程序很简单，它公开了一个 REST API 来创建、读取、更新和删除带有 GitHub ID、博客和技能的开发人员档案。

像往常一样，代码可以在 GitHub 上获得，但是让我们快速浏览一下！

下面是代码布局:

```
.
├── Dockerfile
├── api
│   ├── crud.go
│   └── crud_test.go
├── go.mod
├── go.sum
├── main.go
├── model
│   └── developer.go
└── test.sh
```

主要的 CRUD 操作在`api`包中的`crud.go`文件中。它包含创建、读取、更新和删除操作的实现。

`Create`函数获取 MongoDB 集合的句柄，并将请求体(JSON 有效负载)整理成一个结构(`model.Developer`)。然后使用`[InsertOne](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.InsertOne)` [函数](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.InsertOne)插入该结构，处理错误(上面未显示)并在成功的情况下返回一个`HTTP 201`。

```
coll := a.Connection.Database(a.DBName).Collection(a.CollectionName)
...
var dev model.Developer
json.NewDecoder(req.Body).Decode(&dev)
...
res, err := coll.InsertOne(ctx, dev)
...
rw.WriteHeader(http.StatusCreated)
```

下面是读取操作的工作原理:

```
vars := mux.Vars(req)
githubID := vars["github"]
...
coll := a.Connection.Database(a.DBName).Collection(a.CollectionName)
r := coll.FindOne(context.Background(), bson.M{githubIDAttribute: githubID})
...
var p model.Developer
r.Decode(&p)
json.NewEncoder(rw).Encode(&p)
```

`[FindOne](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.FindOne)` [函数](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.FindOne)与过滤标准`github_id`一起使用，过滤标准是集合`bson.M{githubIDAttribute: githubID}`的分区键。如果找到，结果将被转换为结构并返回给调用方。

获取所有开发人员配置文件是类似的

```
r, err := coll.Find(context.Background(), bson.D{})
...
devs := []model.Developer{}
err = r.All(context.Background(), &devs)
...
```

`[Find](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.Find)`使用一个空的`bson`文档作为过滤器，列出集合中的所有文档，结果以 JSON 数组的形式发送回调用者

`[FindOneAndReplace](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.FindOneAndReplace)`用于更新一条特定的记录。`github_id`用作过滤标准，传入的 JSON 有效负载是更新后的记录。

```
var dev model.Developer
json.NewDecoder(req.Body).Decode(&dev)
...
r := coll.FindOneAndReplace(context.Background(), bson.M{githubIDAttribute: githubID}, &dev)
...
rw.WriteHeader(http.StatusNoContent)
```

使用`[FindOneAndDelete](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.FindOneAndDelete)`完成删除，它接受`github_id`作为要删除的记录的过滤标准

```
vars := mux.Vars(req)
githubID := vars["github"]
...
r := coll.FindOneAndDelete(context.Background(), bson.M{githubIDAttribute: githubID})
...
rw.WriteHeader(http.StatusNoContent)
```

在`main.go`一切都联系在一起。它将 CRUD 操作处理程序与 HTTP 路由相关联，启动服务器并设置一个优雅的退出机制

```
.....
func main() {
	r := mux.NewRouter() 
        r.Methods(http.MethodPost).Path("/developers")
                                  .HandlerFunc(crudAPI.Create)
	r.Methods(http.MethodGet).Path("/developers/{github}")
                                 .HandlerFunc(crudAPI.Read)
        r.Methods(http.MethodGet).Path("/developers")
                                 .HandlerFunc(crudAPI.ReadAll)
        r.Methods(http.MethodPut).Path("/developers")
                                 .HandlerFunc(crudAPI.Update)
        r.Methods(http.MethodDelete).Path("/developers/{github}")
                                    .HandlerFunc(crudAPI.Delete) server := http.Server{Addr: ":" + port, Handler: r} go func() {
		err := server.ListenAndServe()
		if err != nil && err != http.ErrServerClosed {
			log.Fatalf("could not start server %v", err)
		}
	}()
....
	exit := make(chan os.Signal)
	signal.Notify(exit, syscall.SIGTERM, syscall.SIGINT)
	<-exit
....
	c, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer func() {
		crudAPI.Close()
		cancel()
	}()
	err := server.Shutdown(c)
....
```

好了，除此之外，是时候提供所需的服务并部署应用程序了。在此之前，我们先来看看一些先决条件

# 先决条件

*   一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)—[注册一个免费账户吧！](https://azure.microsoft.com/free/?WT.mc_id=medium-blog-abhishgu)
*   `Azure CLI`或者`Azure Cloud Shell`——你可以选择安装 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu) 如果你还没有的话(应该很快！)或者直接从浏览器使用 [Azure 云壳](https://azure.microsoft.com/features/cloud-shell/?WT.mc_id=medium-blog-abhishgu)。

# 设置 Azure Cosmos DB

您需要创建一个启用了 MongoDB API 支持的 Azure Cosmos DB 帐户以及一个数据库和集合。您可以使用 Azure 门户或 Azure CLI

> *了解如何在 Azure Cosmos DB* 中使用数据库、容器和项目的更多信息

## 使用 Azure 门户网站

请遵循以下步骤:

*   [创建一个 Azure Cosmos DB 帐户](https://docs.microsoft.com/azure/cosmos-db/create-mongodb-java?WT.mc_id=medium-blog-abhishgu#create-a-database-account)
*   [添加一个数据库和集合](https://docs.microsoft.com/azure/cosmos-db/create-mongodb-java?WT.mc_id=medium-blog-abhishgu#add-a-collection)并获取连接字符串

## 使用 Azure CLI

(相同的命令可用于 Azure CLI 或 Azure Cloud Shell)

导出以下环境变量:

```
export RESOURCE_GROUP=[to be filled]
export LOCATION=[to be filled]
export COSMOS_DB_ACCOUNT=[to be filled]
export COSMOS_DB_NAME=[to be filled]
export COSMOS_COLLECTION_NAME=[to be filled]
export SHARDING_KEY_PATH='[to be filled]'
```

首先创建一个[新的资源组](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-group-create)，在它下面你可以放置你所有的资源。完成后，您可以删除资源组，这将依次删除所有服务

```
az group create --name $RESOURCE_GROUP --location $LOCATION
```

[创建账户](https://docs.microsoft.com/cli/azure/cosmosdb?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-cosmosdb-create)(通知`--kind MongoDB`)

```
az cosmosdb create --resource-group $RESOURCE_GROUP --name abhishgu-mongodb --kind MongoDB
```

[创建数据库](https://docs.microsoft.com/cli/azure/cosmosdb/mongodb/database?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-cosmosdb-mongodb-database-create)

```
az cosmosdb mongodb database create --account-name $COSMOS_DB_ACCOUNT --name $COSMOS_DB_NAME --resource-group $RESOURCE_GROUP
```

最后，[在数据库中创建一个集合](https://docs.microsoft.com/cli/azure/cosmosdb/mongodb/collection?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-cosmosdb-mongodb-collection-create)

```
az cosmosdb mongo collection create --account-name $COSMOS_DB_ACCOUNT --database-name $COSMOS_DB_NAME --name $COSMOS_COLLECTION_NAME --resource-group-name $RESOURCE_GROUP --shard $SHARDING_KEY_PATH
```

获取连接字符串并保存它。你以后会用到它

```
az cosmosdb list-connection-strings --name $COSMOS_DB_ACCOUNT --resource-group $RESOURCE_GROUP -o tsv --query connectionStrings[0].connectionString
```

# 部署到 Azure 应用服务

## 为应用程序构建 Docker 映像

> *这一步是可选的。你可以使用我在*[*docker hub*](https://hub.docker.com/)上提供的预建图像 `[*abhirockzz/mongodb-go-app*](https://hub.docker.com/r/abhirockzz/mongodb-go-app)`

*您可以使用`Dockerfile`来构建您自己的映像，并将其推送到您选择的 Docker 注册表(公共/私有)*

> *[*这里有一个教程*](https://docs.microsoft.com/azure/app-service/containers/tutorial-custom-docker-image?WT.mc_id=medium-blog-abhishgu) *提供了一个如何使用* [*Azure 容器注册表*](https://azure.microsoft.com/services/container-registry/?WT.mc_id=medium-blog-abhishgu) *和 Azure Web App 服务*的例子*

```
*docker build -t [REPOSITORY_NAME/YOUR_DOCKER_IMAGE:TAG] .
//e.g.
docker build -t [abhirockzz/mongodb-go-app] .//login to your registry
docker login//push image to registry
docker push [REPOSITORY_NAME/YOUR_DOCKER_IMAGE:TAG]*
```

## *创建 Azure 应用服务*

*是时候将我们的应用部署到云上了——让我们使用 Azure 应用服务快速完成这项工作。首先，创建一个应用服务计划，为我们的 web 应用运行定义一组计算资源。*

> **关于 App 服务计划*的详细信息，请参见 [*文档*](https://docs.microsoft.com/azure/app-service/overview-hosting-plans?toc=/azure/app-service/containers/toc.json&WT.mc_id=medium-blog-abhishgu)*

**该计划与一个`SKU`相关联——就像认知服务一样，你也需要为应用服务选择一个`SKU`(定价层)。对于这个例子，我们将使用一个小层(`B1`)。**

> ***接受的值是* `*B1, B2, B3, D1, F1, FREE, P1V2, P2V2, P3V2, PC2, PC3, PC4, S1, S2, S3, SHARED*`**

```
**export APP_SERVICE_PLAN_NAME=[to be filled]
export APP_SERVICE_SKU=B1az appservice plan create --name $APP_SERVICE_PLAN_NAME --resource-group $RESOURCE_GROUP --sku $APP_SERVICE_SKU --is-linux**
```

> ***文档为* `[*az appservice plan create*](https://docs.microsoft.com/cli/azure/appservice/plan?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-appservice-plan-create)`**

**设置环境变量**

```
**export APP_NAME=[to be filled]
export DOCKER_IMAGE=[to be filled]
export MONGODB_CONNECTION_STRING="[to be filled]"
export DATABASE_NAME=[to be filled]
export COLLECTION_NAME=[to be filled]**
```

> ***请确保在连接字符串值*周围使用双引号( `*""*` *)***

**[部署应用](https://docs.microsoft.com/cli/azure/webapp?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-webapp-create)**

```
**az webapp create --resource-group $RESOURCE_GROUP --plan $APP_SERVICE_PLAN_NAME --name $APP_NAME --deployment-container-image-name $DOCKER_IMAGE**
```

**[添加环境变量作为应用所需的配置设置](https://docs.microsoft.com/cli/azure/webapp/config/appsettings?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-webapp-config-appsettings-set)**

```
**az webapp config appsettings set --resource-group $RESOURCE_GROUP --name $APP_NAME --settings MONGODB_CONNECTION_STRING=$MONGODB_CONNECTION_STRING DATABASE_NAME=$DATABASE_NAME COLLECTION_NAME=$COLLECTION_NAME**
```

**API 应该在某个时候部署。当它完成时，就去试一试吧！**

# **测试 API**

> **我已经使用了 `*curl*` *进行演示，但是你也可以使用你选择的任何其他工具***

**[获取已部署应用的端点/主机名](https://docs.microsoft.com/cli/azure/webapp?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-webapp-show)**

```
**APP_URL=$(az webapp show --name $APP_NAME --resource-group $RESOURCE_GROUP -o tsv --query 'defaultHostName')**
```

**创建一些开发人员档案**

```
**curl -X POST -d '{"github_id":"abhirockzz", "blog":"dev.to/abhirockzz", "skills":["java","azure"]}' $APP_URL/developerscurl -X POST -d '{"github_id":"abhishek", "blog":"medium.com/@abhishek1987/", "skills":["go","mongodb"]}' $APP_URL/developers**
```

**使用 GitHub ID 查找单个开发人员配置文件**

```
**curl $APP_URL/developers/abhirockzz**
```

**如果您搜索不存在的配置文件**

```
**curl $APP_URL/developers/foo//developer profile with GitHub ID foo does not exist**
```

**获取所有开发配置文件**

```
**curl $APP_URL/developers**
```

**删除开发人员档案**

```
**curl -X DELETE $APP_URL/developers/abhishek**
```

**再次确认**

```
**curl $APP_URL/developers**
```

**完成后，您可以删除资源组来删除所有服务**

```
**az group delete --name $RESOURCE_GROUP --no-wait**
```

**这个博客到此为止。我希望得到您的反馈和建议！简单地发推文或发表评论。如果你觉得这篇文章有用，请点赞并关注😃😃**