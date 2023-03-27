# Azure 事件中心“基于角色的访问控制”在起作用

> 原文：<https://itnext.io/azure-event-hubs-role-based-access-control-in-action-2addd2dc31a3?source=collection_archive---------5----------------------->

[Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/?WT.mc_id=medium-blog-abhishgu) 是流媒体平台和事件摄取服务，每秒可以接收和处理数百万个事件。在这篇博客中，我们将讨论与 Azure 事件中心相关的一个安全方面。

![](img/1b3765d445d1f7f51449b0fbcf07035f.png)

约翰·萨尔维诺在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

`Shared Access Signature (SAS)`是 Azure Event Hubs 的一种常用认证机制，可用于对您想要授予的访问类型实施粒度控制——它通过在事件中心资源(名称空间或主题)上配置规则来工作。然而，建议您尽可能使用 Azure AD 凭证(通过`SAS`)，因为它提供了类似的功能，无需管理`SAS`令牌或担心撤销受损的`SAS`。

> *要了解有关此主题的更多信息，请查看“* [*使用共享访问签名(SAS)验证对事件中心资源的访问*](https://docs.microsoft.com/azure/event-hubs/authenticate-shared-access-signature?WT.mc_id=medium-blog-abhishgu) *”。*

有几种方法可以做到这一点:

*   使用[托管身份](https://docs.microsoft.com/azure/event-hubs/authenticate-managed-identity?tabs=latest&WT.mc_id=medium-blog-abhishgu)或
*   通过在您的客户端应用程序中使用[服务主体](https://docs.microsoft.com/azure/event-hubs/authenticate-application?WT.mc_id=medium-blog-abhishgu)

我们将探讨第二个选项，即如何在您的 Azure Event Hubs 客户端应用程序中使用基于 Azure Active Directory 的身份验证。通过一个实例，您将了解到:

*   Azure 事件中心角色概述
*   如何使用 Azure CLI 为事件中心配置服务主体和 RBAC 策略
*   Event Hubs 通过特定的 SDK 支持多种编程语言，我们将看到如何将 RBAC 用于 T4 和 T5 客户端

> *该代码在本次 GitHub 回购中可用—*[](https://github.com/abhirockzz/azure-eventhubs-rbac-example)

# *概观*

*本节提供了一个高层次的概述，帮助您理解关键术语:角色、安全原则和基于角色的访问控制(`RBAC`)。*

***角色** : Azure Event Hubs 定义了特定的角色，每个角色都允许你对其资源采取特定的动作——`Data Owner`、`Data Sender`、`Data Receiver`。它们是不言自明的——发送者和接收者角色*仅*分别允许发送和接收，而所有者角色就像一个管理员权限，允许您完成访问。*

> **有关这些角色的详细信息，请参考其文档链接—* [*Azure Event Hubs 数据所有者*](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles?WT.mc_id=medium-blog-abhishgu#azure-event-hubs-data-owner) *，* [*Azure Event Hubs 数据发送者*](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles?WT.mc_id=medium-blog-abhishgu#azure-event-hubs-data-sender) *，* [*Azure Event Hubs 数据接收者*](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles?WT.mc_id=medium-blog-abhishgu#azure-event-hubs-data-receiver)*

***RBAC，服务主体**:服务主体是被授予这些角色的实体——这是“基于角色的访问控制”,因为您授予服务主体的角色定义了*他们可以执行哪些操作*—在这种情况下，这些操作是:发送、接收或一切！*

## *…场景*

*这是我们将用来学习的例子。有两个事件中心客户端应用程序——一个 Java 生产者和一个 Go 消费者。我们将配置细粒度的规则(即实施 RBAC ),以便:*

*   *Java 应用程序只能向事件中心主题发送消息，*
*   *Go 应用程序只能从事件中心主题接收消息*

*以下是相关的代码片段:*

*在 Java producer 客户端中，使用了`[DefaultAzureCredentialBuilder](https://docs.microsoft.com/java/api/com.azure.identity.defaultazurecredentialbuilder.defaultazurecredentialbuilder?view=azure-java-stable&WT.mc_id=medium-blog-abhishgu)`。*

```
*String eventhubsNamespace = System.getenv("EVENTHUBS_NAMESPACE");
String eventhubName = System.getenv("EVENTHUB_NAME");EventHubProducerClient producer = new EventHubClientBuilder().credential(eventhubsNamespace, eventhubName, **new DefaultAzureCredentialBuilder()**.build()).buildProducerClient();*
```

*按照惯例(默认)，Java SDK 尝试读取服务主体信息的预定义环境变量，并基于此进行身份验证。如果这不起作用，它就退回到尝试`SAS` auth 机制，并寻找另一组环境变量*

*类似地，在 Go consumer 客户端中，事件中心客户端的创建过程也大大简化了*

```
*ehNamespace := os.Getenv("EVENTHUBS_NAMESPACE")
ehName := os.Getenv("EVENTHUB_NAME")hub, err := eventhub.**NewHubWithNamespaceNameAndEnvironment**(ehNamespace, ehName)*
```

> **开发人员的体验在 SDK 之间是一致的，* `[*NewHubWithNamespaceNameAndEnvironment*](https://godoc.org/github.com/Azure/azure-event-hubs-go#NewHubWithNamespaceNameAndEnvironment)` *与* `*DefaultAzureCredentialBuilder*` *在遵循固定惯例方面做了相同的事情，首先尝试使用 Azure Active Directory (AAD)支持的服务主体进行身份验证，然后是 SAS 机制**

*也就是说，您确实需要一些东西来完成本教程:*

# *先决条件*

*你需要一个[微软 Azure 账户](https://docs.microsoft.com/azure/?WT.mc_id=medium-blog-abhishgu)。去报名参加一个免费的吧！*

*`Azure CLI`或者`Azure Cloud Shell`——你可以选择安装 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu) 如果你还没有的话(应该很快！)或者直接从你的浏览器使用 [Azure 云壳](https://azure.microsoft.com/features/cloud-shell/?WT.mc_id=medium-blog-abhishgu)。*

*我们走吧..从建立 Azure 活动中心开始*

# *设置 Azure 事件中心*

*这些都是通过 Azure CLI 完成的，但是如果你愿意，你也可以使用 Azure Portal。最终目标是拥有一个 [Azure 事件中心名称空间](https://docs.microsoft.com/azure/event-hubs/event-hubs-features?WT.mc_id=medium-blog-abhishgu#namespace)以及一个事件中心(又名主题)*

*如果还没有 Azure 资源组，请创建一个*

```
*AZURE_SUBSCRIPTION=[to be filled]
AZURE_RESOURCE_GROUP=[to be filled]
AZURE_LOCATION=[to be filled]az account set --subscription $AZURE_SUBSCRIPTION
az group create --name $AZURE_RESOURCE_GROUP --location $AZURE_LOCATION*
```

*使用`[az eventhubs namespace create](https://docs.microsoft.com/cli/azure/eventhubs/namespace?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-eventhubs-namespace-create)`创建一个活动中心名称空间*

```
*EVENT_HUBS_NAMESPACE=[to be filled]az eventhubs namespace create --name $EVENT_HUBS_NAMESPACE --resource-group $AZURE_RESOURCE_GROUP --location $AZURE_LOCATION  --enable-auto-inflate false*
```

*然后使用`[az eventhubs eventhub create](https://docs.microsoft.com/cli/azure/eventhubs/eventhub?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-eventhubs-eventhub-create)`创建一个活动中心*

```
*EVENT_HUB_NAME=[to be filled]az eventhubs eventhub create --name $EVENT_HUB_NAME --resource-group $AZURE_RESOURCE_GROUP --namespace-name $EVENT_HUBS_NAMESPACE --partition-count 3*
```

*..现在是关键部分…*

# *…安全配置*

*我们将使用 Azure CLI 通过`[az ad sp create-for-rbac](https://docs.microsoft.com/cli/azure/ad/sp?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-ad-sp-create-for-rbac)`创建服务主体*

*对于发送者应用程序(我们将其命名为`eh-sender-sp`)*

```
*az ad sp create-for-rbac -n "eh-sender-sp"*
```

*您将得到这样一个 JSON 响应——请记下`appId`、`password`和`tenant`*

```
*{
  "appId": "fe7280c7-5705-4789-b17f-71a472340429",
  "displayName": "eh-sender-sp",
  "name": "http://eh-sender-sp",
  "password": "29c719dd-f2b3-46de-b71c-4004fb6116ee",
  "tenant": "42f988bf-86f1-42af-91ab-2d7cd011db42"
}*
```

*对于接收器应用程序(我们将其命名为`eh-receiver-sp`)*

```
*az ad sp create-for-rbac -n "eh-receiver-sp"*
```

*您将得到一个 JSON 响应——请记下`appId`、`password`和`tenant`*

> **您也可以使用 Azure Portal 来* [*创建服务主体*](https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app?WT.mc_id=medium-blog-abhishgu) *和* [*生成客户端秘密*](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal?WT.mc_id=medium-blog-abhishgu#create-a-new-application-secret)*

## *强制 RBAC*

*使用 CLI 完成这一过程需要几个步骤，但从长远来看是有益的，例如对自动化而言。也可以使用 Azure 门户网站来执行*

*使用`[az role definition list](https://docs.microsoft.com/cli/azure/role/definition?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-role-definition-list)`获取两个角色的 id*

```
*EH_SENDER_ROLE_ID=$(az role definition list -n "Azure Event Hubs Data Sender" -o tsv --query '[0].id')EH_RECEIVER_ROLE_ID=$(az role definition list -n "Azure Event Hubs Data Receiver" -o tsv --query '[0].id')*
```

> *`*Azure Event Hubs Data Sender*`*`*Azure Event Hubs Data Receiver*`*是角色名***

**使用`[az eventhubs namespace show](https://docs.microsoft.com/cli/azure/eventhubs/namespace?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-eventhubs-namespace-show)`获取 Azure Event Hubs 名称空间的订阅 ID**

```
**export RESOURCE_GROUP=[replace with resource group name]
export EVENTHUBS_NAMESPACE=[replace with namespace]EVENTHUBS_ID=$(az eventhubs namespace show --resource-group $RESOURCE_GROUP --name $EVENTHUBS_NAMESPACE -o tsv --query 'id')**
```

**使用`[az role assignment create](https://docs.microsoft.com/cli/azure/role/assignment?view=azure-cli-latest&WT.mc_id=medium-blog-abhishgu#az-role-assignment-create)`分配角色**

```
**export SENDER_SP_ID=[replace with Service Principal "appId" for the sender SP]export RECEIVER_SP_ID=[replace with Service Principal "appId" for the receiver SP]az role assignment create --assignee $SENDER_SP_ID --role $EH_SENDER_ROLE_ID --scope $EVENTHUBS_IDaz role assignment create --assignee $RECEIVER_SP_ID --role $EH_RECEIVER_ROLE_ID --scope $EVENTHUBS_ID**
```

**好了，你都准备好了！**

# **让我们试一试**

**从 GitHub 克隆示例应用程序**

```
**git clone [https://github.com/abhirockzz/azure-eventhubs-rbac-example.git](https://github.com/abhirockzz/azure-eventhubs-rbac-example.git)**
```

**启动 Go 消费者应用程序:**

```
**export EVENTHUBS_NAMESPACE=[replace with namespace]
export EVENTHUB_NAME=[replace with name of the Event Hub]export AZURE_TENANT_ID=[replace with Service Principal "tenant" for the receiver SP]export AZURE_CLIENT_ID=[replace with Service Principal "appId" for the receiver SP]export AZURE_CLIENT_SECRET=[replace with Service Principal "password" for the receiver SP]cd azure-eventhubs-rbac-example/consumer-gogo run main.go**
```

**程序将阻塞，等待事件…**

**在另一个终端中，启动 producer 应用程序—这将只发送 10 个事件，然后退出(如果您想发送更多事件，请重新运行)**

```
**cd azure-eventhubs-rbac-example/producer-java//build the Java app - it uses Maven
mvn clean installexport EVENTHUBS_NAMESPACE=[replace with namespace]
export EVENTHUB_NAME=[replace with name of the Event Hub]export AZURE_TENANT_ID=[replace with Service Principal "tenant" for the producer SP]export AZURE_CLIENT_ID=[replace with Service Principal "appId" for the producer SP]export AZURE_CLIENT_SECRET=[replace with Service Principal "password" for the producer SP]java -jar target/eventhubs-java-producer-jar-with-dependencies.jar**
```

**您应该会在消费者应用程序终端中看到收到的事件！**

> ***作为一个练习，为了模拟一个错误场景，您可以交换发送者和消费者应用程序的客户端详细信息，以查看它们的行为。***

# **结论**

**希望这有助于演示如何使用 Azure Active Directory 将 RBAC 用于事件中心应用程序。在以后的博客文章中，我将尝试讨论托管身份。在那之前，敬请期待！**