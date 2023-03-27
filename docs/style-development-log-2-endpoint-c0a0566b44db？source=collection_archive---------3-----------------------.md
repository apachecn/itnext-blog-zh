# 风格发展日志 2 —终点

> 原文：<https://itnext.io/style-development-log-2-endpoint-c0a0566b44db?source=collection_archive---------3----------------------->

> *加入我们的不和谐社区*[*https://discord.gg/bPcscvBM*](https://discord.gg/hdCdYfwS4C)*获取消息。*

我将在一个系列中分享与风格后端框架相关的发展，我已经在[文章](/style-backend-framework-d544bdb78a36)中宣布了。如果你对
风格没有概念或者不理解下面例子中的组件，请看看主文章。

众所周知，服务器响应客户端的点称为`endpoint`。请求中的所有有效路径都以服务器上的端点结束。此外，在运行良好的服务器上，所有无效路径都应该被重定向到有效路径(即使只是一个发送错误代码的端点。).

换句话说，端点是服务器中最重要、最丰富的组件。在这种情况下，除了用“样式”尽可能容易地编码之外，我更喜欢不损失性能的方法。

# **端点组件**

像所有其他东西一样，`Endpoint`是一个`Component`。所以它可以放在任何你想放的地方。但是，端点不接收子实例。所有其他组件都有一个/多个子实例。因此，端点构成了组件树的端点。

基本上，创建端点的方法是创建一个`Endpoint`接口。

Enpoint 接口应该覆盖`onCall`方法。该方法在中途处理请求。

```
class MyEndpoint extends Endpoint { @override
  FutureOr<Object> onCall(Request request) {
    return "Hello World!"**;** }}
```

## **路线上的地点**

```
class MyServer extends StatelessComponent {
  const MyServer({Key? key}) : super(key: key)**;** @override
  Component build(BuildContext context) {
    return Server(children: [ /// [http://localhost/hello](http://host:port/hello)
      Route("hello"**,** root: MyEndpoint()
      ])**;** }
}
```

# 状态端点

StatefulEndpoint 仅在必要时可用于保持状态。

```
class MyStatefulEndpoint extends StatefulEndpoint {
  @override
  EndpointState<StatefulEndpoint> createState() => _MyStatefulEndpointState()**;** }

class _MyStatefulEndpointState extends EndpointState<MyStatefulEndpoint> {

  int counter = **0;** @override
  FutureOr<Object> onCall(Request request) {
    counter++**;** return counter**;** }
}
```

> 为避免冲突，使用`CallQueue(MyStatefulEndpoint())`

## OnCall 返回类型

如您所见，onCall 函数应该返回一个对象实例。

但是在某些情况下，您可能会得到一个错误。因为支持的类型只有以下几种:

> 未来支持。

1.可编码的(Json 可编码的。即使内容类型不是 json。也叫 Uint8List)

```
return "any"**;**
```

2.`Response`实例(使用 request.response(..).这种方式可以设置自定义标题，状态码等。)

```
return request.response(Body("message"),statusCode: 230)**;**
```

3.AccessEvent(使用当前数据执行事件访问并使用数据进行响应)

```
return Read(collection: "greeters"**,** identifier: "john")**;**
```

4.DbResult(使用自定义数据访问执行事件，并使用数据进行响应)

```
final db = context
  .findAncestorStateOfType<MyTopLevelDataAccess>()!
  .context.dataAccess**;**return db.read(Read(collection: "greeters"**,** identifier: "john"))**;**
```

5.车身仪表

```
return Body("data")**; // auto content-type** return JsonBody({"hello" : "world"})**;** return BinaryBody(Uint8List.fromList([]))**;** return StringBody("data")**;**
```

> 有缓存控制端点。跟随我进行缓存控制演示。
> 
> 小型高速缓存控制端点示例:

```
class MyLastModifiedEndpoint extends LastModifiedEndpoint { @override
  FutureOr<ResponseWithCacheControl<DateTime>> onRequest(
      ValidationRequest<DateTime> request) { return ResponseWithLastModified("Hello"**,** request: request**,** lastModified: DateTime.now())**;**}}
```

## 提高绩效

通过预先指定端点的返回类型，可以将性能提高 3 倍。

```
class AnyEndpoint extends Endpoint {
  AnyEndpoint() : super()**;** **/// Prefer type** @override
  EndpointPreferredType? get preferredType =>
      EndpointPreferredType.anyEncodable**;** @override
  FutureOr<Object> onCall(Request request) {
    return "any"**;** }
}
```

可用值有:`body` 、`response` 、`anyEncodable` 、`dbResult` 、`accessEvent .`

> 为什么绩效提高了？不是检查每个请求的返回值类型，而是在返回类型已知的地方创建 EndpointCalling 实例。

# **简单端点**

在核心风格包中有许多简单的组件。与端点相关的简易组件将在下面列出。

## 简单端点

您可以只定义 onCall 函数，而不是定义一个接口。

```
SimpleEndpoint((request**,**context) => 
"Hello ${request.arguments["user"]}!"
)**,**
```

对于静态响应。

```
SimpleEndpoint.static("Hello World!")**,**
```

## 接入点

接入点使用当前数据访问执行由 eventBuilder 创建的访问事件，并将数据发送到客户端。

```
AccessPoint((req**,** _) => Read(
    request: req**,** collection: "greeters"**,** identifier: req.arguments["user"]!))**,**
```

## RestAccessPoint

根据 Rest Api 标准，它是一个 AccessPoint 定义。

查看剩余的[文档](https://github.com/Mehmetyaz/style/blob/main/packages/style/documentation/data_access/simple_access_point.md)。

```
RestAccessPoint("route")**,**
```

## 内容交付

它为你的文件服务。支持缓存控制和监视更改。

```
ContentDelivery("/directory/to/contents")
```

## 扔

始终投掷终点

```
Throw(ServiceUnavailable("coming_soon"))**,**
```

## 再直接的

将请求重定向到内部或外部路径。

```
Redirect("../hello")**,** Redirect("https://www.google.com")**,**
```

或者由请求产生

```
GeneratedRedirect(
  generate: (r) => "../hello?u=${r.arguments["user"]}"
)**,**
```