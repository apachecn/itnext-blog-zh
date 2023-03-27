# 在 Swift 中发送异步 HTTP 请求

> 原文：<https://itnext.io/sending-asynchronous-http-requests-in-swift-bfc619a1a2a?source=collection_archive---------1----------------------->

## 任务和等待的实用介绍

![](img/401e66a3c5104408d7fba4253d4b9e73.png)

照片由[杰斯·贝利](https://unsplash.com/@jessbaileydesigns?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

这篇文章介绍了如何在 Swift 中异步执行代码。首先，将提供关于这一主题的总体看法。之后，两个具体的代码示例将展示如何在实际的例子中使用异步代码。

代码示例是为 Xcode Playgrounds 创建的，可以直接复制、粘贴和运行。

这两个请求都将在 reqbin.com 上运行，这是一个在线 REST 和 SOAP API 测试工具。

在第一个例子中，将使用 HTTP GET 请求，这也可以在他们的网站上在线试用。

然后，作为第二个例子，HTTP POST 请求[将与 JSON](https://reqbin.com/req/4rwevrqh/post-json-example) 有效负载一起使用。

这篇文章还将介绍如何将数据编码和解码成 JSON。

## 异步代码执行

在我们进入如何发送 HTTP 请求的细节之前，让我们看一下异步代码执行。

Swift [文档](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html)对“异步代码”做了如下描述:

> 异步代码可以被挂起，稍后再恢复，尽管一次只能执行一部分程序。暂停和恢复程序中的代码可以让它继续在短期操作上取得进展，如更新其 UI，同时继续处理长期运行的操作，如通过网络获取数据或解析文件。

要使一个长期运行的函数异步，我们需要用`async`-关键字来标记它。

```
func myLongRunningFunction() async {
    ...
  // do something that takes lots of time
  ...
}
```

然后，为了调用这个函数，我们需要使用`await` -关键字。

```
await myLongRunningFunction()
```

让我们举一个`Task.sleep()`的具体例子。在下面的代码中，我们希望等待 5 秒钟。

```
print("Before")
try Task.sleep(nanoseconds: 5 * 1_000_000_000)
print("Done")
print("After")
```

此代码无法编译。我们得到错误消息:“在不支持并发的函数中进行了‘async’调用”。

因为我们的 Xcode 操场的主线程是同步的，所以我们需要提前调用`async`。

```
print("Before")
async {
    try await Task.sleep(nanoseconds: 5 * 1_000_000_000)
    print("Done")
}
print("After")
```

虽然这段代码运行得很好，但它不再编译只是时间问题。

原因是，`async`已被弃用，将被`Task.init`取代。

让我们迅速改变这一点。

```
print("Before")
Task {
    try await Task.sleep(nanoseconds: 5 * 1_000_000_000)
    print("Done")
}
print("After")
```

运行该命令将产生以下输出。

```
Before
After
Done
```

而“之前”和“之后”将立即打印出来。打印“完成”需要 5 秒钟。

现在，我们已经了解了异步代码执行的基本情况，让我们看看如何使用它来发送 HTTP 请求。

## 发送 HTTP GET 请求

下面是发送 HTTP GET 请求的示例代码。

基本上我们需要做三个步骤:

*   使用 URL 创建请求
*   发送请求
*   评估响应

```
import PlaygroundSupport
import UIKit

struct SuccessInfo: Decodable {
    let success: String
}

guard let url = URL(string: "https://reqbin.com/echo/get/json") else { fatalError("Missing URL") }

var urlRequest = URLRequest(url: url)
urlRequest.addValue("application/json", forHTTPHeaderField: "Content-Type")

Task {
    let (data, response) = try await URLSession.shared.data(for: urlRequest)
    guard (response as? HTTPURLResponse)?.statusCode == 200 else { fatalError("Error while fetching data") }

    let successInfo = try JSONDecoder().decode(SuccessInfo.self, from: data)

    print(String(data: data, encoding: .utf8) ?? "default value")
    print("Success: \(successInfo.success)")
}

print("I am here already!")

PlaygroundPage.current.needsIndefiniteExecution = true
```

重要的类别有:

*   [网址](https://developer.apple.com/documentation/foundation/url)

> 标识资源位置的值，如远程服务器上的项目或本地文件的路径

[URLRequest](https://developer.apple.com/documentation/foundation/urlrequest/)

> "独立于协议或 URL 方案的 URL 加载请求."

*   [URLSession](https://developer.apple.com/documentation/foundation/urlsession/)

> "协调一组相关网络数据传输任务的对象."

*   [HTTPURLResponse](https://developer.apple.com/documentation/foundation/httpurlresponse/)

> "与 HTTP 协议 URL 加载请求的响应相关联的元数据."

## 发送 HTTP POST 请求

本节展示了带有 JSON 有效负载的 HTTP POST 请求。同样，我们需要执行与第一个示例相同的三个步骤。

这个例子中最大的不同是，我们向服务器发送一些数据。

我们的数据被编码在`MyJsonObject`中。你可以使用任何你想要的有效负载，后端会忽略它。

大多数 REST 服务用相同的资源类型响应，因此我们可以使用`MyJsonObject`作为唯一的类型，并让它从`Codable`继承。

```
import PlaygroundSupport
import UIKit

struct MyJsonObject: Encodable {
    let id: Int
    let foo: String
    let bar: String
}

struct SuccessInfo: Decodable {
    let success: String
}

let myJsonObject = MyJsonObject(id: 1, foo: "Hello", bar: "World")
let payload = try JSONEncoder().encode(myJsonObject)

guard let url = URL(string: "https://reqbin.com/echo/post/json") else { fatalError("Missing URL") }

var urlRequest = URLRequest(url: url)
urlRequest.addValue("application/json", forHTTPHeaderField: "Content-Type")
urlRequest.httpMethod = "POST"

Task {
    let (data, response) = try await URLSession.shared.upload(for: urlRequest, from: payload)

    guard (response as? HTTPURLResponse)?.statusCode == 200 else { fatalError("Error while fetching data") }

    let successInfo = try JSONDecoder().decode(SuccessInfo.self, from: data)

    print(String(data: data, encoding: .utf8) ?? "default value")
    print("Success: \(successInfo.success)")
}

print("I am here already!")

PlaygroundPage.current.needsIndefiniteExecution = true
```

让我们来看看，找出可编码、可解码和可编码之间的区别。

*   [JSONDecoder](https://developer.apple.com/documentation/foundation/jsondecoder/)

> "一个从 JSON 对象中解码数据类型实例的对象."

*   [可编码](https://developer.apple.com/documentation/swift/encodable/)

> "一种可以将自身编码成外部表示的类型."

*   [可解码](https://developer.apple.com/documentation/swift/decodable/)

> "一种能从外部表示中自我解码的类型."

*   [Codable](https://developer.apple.com/documentation/swift/codable/)

> "一种可以将自身转换成外部表示或从外部表示转换出来的类型."

## 运行代码

当执行示例中的代码时，您可能会看到“我已经在这里了！”在 HTTP-response 正文之前打印。

## 结论

在对如何使用`async`和`await`在 Swift 中编写异步代码进行了一般性介绍之后，提供了两个关于如何通过 HTTP GET 和 POST 请求异步发送和接收数据的实例。

请随意将代码复制并粘贴到您自己的 Xcode-Playground 中，并进行试验。

感谢您的阅读！

*   如果你喜欢这个，请[在媒体](https://twissmueller.medium.com/)上跟随我
*   给我买杯咖啡让我继续前进
*   通过[在这里注册](https://twissmueller.medium.com/membership)来支持我和其他媒体作者

[](https://twissmueller.medium.com/membership) [## 通过我的推荐链接加入媒体

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

twissmueller.medium.com](https://twissmueller.medium.com/membership) 

## 资源

*   [满足 Swift 并发](https://developer.apple.com/news/?id=2o3euotz)
*   [并发](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html)
*   [在 Swift 中异步会面/等待](https://developer.apple.com/videos/play/wwdc2021/10132/)
*   [对 URLSession 使用 async/await](https://developer.apple.com/videos/play/wwdc2021/10095/)