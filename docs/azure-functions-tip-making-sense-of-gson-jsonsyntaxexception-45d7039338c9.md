# Azure 函数提示:理解 GSON JsonSyntaxException

> 原文：<https://itnext.io/azure-functions-tip-making-sense-of-gson-jsonsyntaxexception-45d7039338c9?source=collection_archive---------7----------------------->

![](img/be6d9eccd3f739cf2e9e8216780902e9.png)

我正在使用 [Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/?WT.mc_id=medium-blog-abhishgu) trigger 开发 [Azure 函数](https://docs.microsoft.com/azure/azure-functions/?WT.mc_id=medium-blog-abhishgu)，突然出现了这个错误:

```
Result: Failure
Exception: IllegalStateException: Expected BEGIN_OBJECT but was BEGIN_ARRAY at line 1 column 2 path $
Stack: **com.google.gson.JsonSyntaxException**: java.lang.IllegalStateException: Expected BEGIN_OBJECT but was BEGIN_ARRAY at line 1 column 2 path $
	at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read(ReflectiveTypeAdapterFactory.java:226)
	at com.google.gson.Gson.fromJson(Gson.java:927)
	at com.google.gson.Gson.fromJson(Gson.java:892)
	at com.google.gson.Gson.fromJson(Gson.java:841).... //omitted for brevity
```

# 会出什么问题呢？

## **TL；博士**

当使用 Azure 函数的[事件中心触发器时，请不要忘记使用正确的基数，如果你这样做了，希望你偶然看到这篇博文，如果你`Google it on Bing!`；-)](https://docs.microsoft.com/azure/azure-functions/functions-triggers-bindings?WT.mc_id=medium-blog-abhishgu)

## (稍微)长一点的版本…

这个错误对我来说是新的。经过进一步反省，我意识到这个问题与 Azure Functions 平台不能调用函数方法本身有关(也就是说，它与函数逻辑本身无关)

> 顺便说一句，有很多特性可以让 Azure 函数的故障排除变得更容易
> 
> [内置监控](https://docs.microsoft.com/azure/azure-functions/functions-monitoring?WT.mc_id=medium-blog-abhishgu)和[日志分析](https://docs.microsoft.com/azure/azure-functions/functions-monitor-log-analytics?tabs=csharp&WT.mc_id=medium-blog-abhishgu)
> 
> [诊断](https://docs.microsoft.com/azure/azure-functions/functions-diagnostics?WT.mc_id=medium-blog-abhishgu)
> 
> 使用 VS 代码进行远程调试

Azure Functions 使用`[gson](https://docs.microsoft.com/azure/azure-functions/functions-reference-java?WT.mc_id=medium-blog-abhishgu#pojos)` [库来序列化/反序列化有效负载](https://docs.microsoft.com/azure/azure-functions/functions-reference-java?WT.mc_id=medium-blog-abhishgu#pojos)

下面是该方法的一个片段:

```
@FunctionName("processData")
public void process(
    @EventHubTrigger(name = "event", eventHubName = "", connection = "EventHubConnectionString") 
    Data data, final ExecutionContext context) {
        //implementation
    }
```

在这种情况下，它无法将 Event Hubs 有效负载转换为该方法所期望的`Data` POJO。

我查看了 Azure Functions Java API 文档，以确认我是否正确地使用了`[@EventHubTrigger](https://docs.microsoft.com/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger?view=azure-java-stable&WT.mc_id=medium-blog-abhishgu)`。而我偶然发现了`[cardinality](https://docs.microsoft.com/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger.cardinality?view=azure-java-stable&WT.mc_id=medium-blog-abhishgu#com_microsoft_azure_functions_annotation_EventHubTrigger_cardinality__)` [属性](https://docs.microsoft.com/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger.cardinality?view=azure-java-stable&WT.mc_id=medium-blog-abhishgu#com_microsoft_azure_functions_annotation_EventHubTrigger_cardinality__)

定义如下:

> 触发器输入的基数。如果输入是一条消息，请选择“一”；如果输入是一组消息，请选择“多”。如果未指定，则默认为“多”..它的默认值是`Cardinality.MANY`

啊哈时刻！现在理解错误消息`Expected BEGIN_OBJECT but was BEGIN_ARRAY`并不太难——因为`Cardinality`，平台试图序列化一个`Data`POJO 的列表/数组，但是方法只期望一个。显式包含`cardinality`(`Cardinality.ONE`)是解决方案，即

```
@EventHubTrigger(
    name = "event", 
    eventHubName = "", 
    connection = "EventHubConnectionString",
    **cardinality = Cardinality.ONE**)
```

> *另一种方法是将签名改为接受* `*List<Data>*` *并选择列表中的第一项，例如* `*data.get(0)*`

目前就这些。如果你觉得这很有帮助，展示一些:心:并继续关注更多🙌！