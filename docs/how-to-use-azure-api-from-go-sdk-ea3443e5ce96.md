# 如何使用 Go SDK 中的 Azure API

> 原文：<https://itnext.io/how-to-use-azure-api-from-go-sdk-ea3443e5ce96?source=collection_archive---------4----------------------->

![](img/9eb7fd2baed624a011b4449e62d7f1b6.png)

在这篇文章中，我们将通过一些基本的例子来说明如何在 go 中使用 Azure SDK。我们将要经历的示例程序非常简单。首先，它获取 Azure 订阅中所有资源组的列表，然后遍历每个资源组中的所有虚拟机。你猜怎么着..它使用 Go 令人敬畏的并发性(go go goroutines)完成所有这些工作。听起来很简单，但实际上我们在例子中所做的操作应该给你一个如何使用 Azure 的 SDK 和 API 的很好的概述。

该程序位于 [github 库](https://github.com/nordcloud/azure-go-example)中，你可以自由地克隆它、派生它并对其进行操作。您只需要有一个正常工作的 go 和 dep 安装。

开始吧！然而，在我们继续之前，我们首先需要以某种方式验证 Azure API。对于 Azure，这意味着创建一个服务主体帐户，我们的程序将使用该帐户进行身份验证，并承担具有执行 API 操作所需权限的角色。

要创建服务主体，让我们使用 Azure CLI，如下所示。该命令将输出一个身份验证文件，其中包含客户端 id、客户端机密和连接 Azure 所需的一系列信息。记得保管好！

```
*az ad sp create-for-rbac —sdk-auth > my.auth*
```

现在我们已经创建了服务主体，克隆示例的 repo

```
*git clone* [*git@github.com*](mailto:git@github.com)*:nordcloud/azure-go-example.git*
```

这个程序非常简单，只包含一个文件— `*main.go*`。在我们继续之前，我们需要运行`*dep ensure*`,在我们的程序目录中出售 golang SDK 依赖项。

**代码**

让我们打开 main.go 文件，看看`*main()*`方法。

你能看到的是示例程序的一般流程。首先，它使用我们在前面步骤中创建的服务提供商身份进行授权，然后获取订阅中所有资源组的列表，最后，对于每个资源组，它列出所有虚拟机。

好，让我们看看代码中发生了什么。再来看`*newSessionFromFile*` 法。有趣的是我们用 SDK 的方法`*NewAuthorizerFromFile*.`得到授权者的那一行

```
func newSessionFromFile() (*AzureSession, error) {authorizer, err := auth.NewAuthorizerFromFile(azure.PublicCloud.ResourceManagerEndpoint)if err != nil {
    return nil, errors.Wrap(err, "Can't initialize authorizer")
}authInfo, err := readJSON(os.Getenv("AZURE_AUTH_LOCATION"))if err != nil {
    return nil, errors.Wrap(err, "Can't get authinfo")
}sess := AzureSession{
    SubscriptionID: (*authInfo)["subscriptionId"].(string),
    Authorizer:     authorizer,
}return &sess, nil
}
```

这个方法假设`*AZURE_AUTH_LOCATION*` env 变量包含我们之前创建的服务主体 auth 文件的路径。它读取文件并返回一个授权者，该授权者随后被传递给资源 API 客户端。我们将授权者和订阅 id 一起打包到`*AzureSession*`结构中。我们使用`*readJSON()*`方法从同一个 auth file 文件中读取订阅的 id。

让我们回到`*main()*`的方法。现在，我们有了一个工作会话，我们需要获得一个资源组列表。为此我让我们看看`*getGroups*` 的方法。它将一个会话作为参数，并为 groups API 创建一个新的客户机。客户端被传递给一个我们在上一步中创建的授权者。

```
*grClient := resources.NewGroupsClient(sess.SubscriptionID)*
```

> 创建客户端并执行其方法的模式(通常是`*List*`、`*Get*`、`*CreateOrUpdate*`)对于 SDK 中的所有资源都是相同的。一旦你掌握了诀窍，你就可以在 Go 中使用 Azure API 而不用查看 doc。

为了获得资源组列表，我们遍历由资源组客户机的`*ListComplete*`方法返回的所有资源组的列表，并将它们添加到一个列表中。

> Azure SDKs 是自动生成的，或多或少非常 RESTful。每个资源类型都有相同的方法，比如`*CreateOrUpdate*`、`*List*`等等。您可以在这里看到 API 描述，这里描述的方法将映射到 SDK 方法，并将类型返回到 Go 结构。

```
for list, err := grClient.ListComplete(context.Background(), "", nil); list.NotDone(); err = list.Next() {if err != nil {
        return nil, errors.Wrap(err, "error traversing RG list")
    }
    rgName := *list.Value().Name
    tab = append(tab, rgName)
}
```

到目前为止，我们所做的就是授权我们，并获得一个资源组列表。现在，对于每个组，我们将列出该组中的所有虚拟机。此外，我们将使用 go 例程同时完成这一任务！`*main()*`方法中的 For 循环遍历由`*getGroups*`方法返回的资源组，并且对于每个返回的`*rg*`，它使用`*go*`关键字运行一个并发的 goroutine。

```
*go getVM(sess, group, &wg)*
```

goroutine 在`getVM`方法中实现。该方法做与`*getGroups*`类似的事情。它创建一个虚拟机客户端(`*NewVirtualMachinesClient*`)并遍历所有打印它们的虚拟机。

```
for vm, err := vmClient.ListComplete(context.Background(), rg); vm.NotDone(); err = vm.Next() {
    if err != nil {
        log.Print("got error while traverising RG list: ", err)
    }
    i := vm.Value()
    fmt.Printf("(%s) VM %s\n", rg, *i.Name)
}
```

主线程使用`*WaitGroup*`原语及其`*Wait()*`方法等待所有并发 go routines 的终止，该方法在这里用于实现一个等待所有 go 例程完成的屏障。你可以在这里阅读更多关于`*WaitGroups*`和同步[的内容。](https://golang.org/pkg/sync/#WaitGroup)

在我们运行程序之前，我们首先需要导出带有服务主体信息的`*my.auth*`文件路径的`*AZURE_AUTH_LOCATION*`变量。

```
*export AZURE_AUTH_LOCATION=/path/to/my.auth go run main.go*
```

您应该会看到类似这样的输出:

```
(rg1) VM ubuntu
(rg1) VM ubuntutest
(rg2) VM ubuntu2
(rg2) VM ubuntu3
(rg2) VM ubuntu5
(rg2) VM ubuntu44
(rg2) VM ubuntu333
(rg2) VM ubuntu3
(myGroup) VM windows
(myGroup) VM Windas
(testGroup233) VM vm-1-west
(testGroup233) VM vm-2-west
(testGroup233) VM vm-3-west
(testGroup233) VM vm-4-west
...
```

在下一篇文章中，我们将向你展示如何使用 Go 中的 GCP SDK🙂

*原载于 2018 年 10 月 8 日*[*nordcloud.com*](https://nordcloud.com/how-to-use-azure-api-from-go-sdk/)*。*