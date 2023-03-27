# 使用 cURL 探索 REST APIs

> 原文：<https://itnext.io/explore-rest-apis-with-curl-e58ebaaabf7a?source=collection_archive---------1----------------------->

## 对使用 Web 服务一无所知？

![](img/6529222701219c479048c96c135ecb49.png)

照片由[乔丹·哈里森](https://unsplash.com/@jordanharrison?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/network?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

没有什么比和网络技术公司一起工作更让我感到不自在的了。这是关于如何主要使用 cURL Unix 命令使用 REST APIs 的概述，对您和我都有好处。我还将看看如何从一些不同的编程语言中做到这一点。

阅读更多信息:

*   [在 Julia 中使用 HTTP 协议和表单](https://erik-engheim.medium.com/working-with-the-http-protocol-in-julia-5697d531f0c7) —设置 HTTP 请求，模拟响应。如何传输图像数据？HTML 表单呢？如何在请求处理器中处理它们？
*   [不同 Web 技术指南](https://erik-engheim.medium.com/a-guide-to-different-web-technologies-and-how-they-are-related-a0951c7b43f4)—Web sockets 与 Unix sockets 有何不同。AJAX vs WebSockets，jQuery，React 等等，它们是如何联系和重叠的？

为了避免你不得不到处跳来跳去，我会尽量把所有的关键信息都写在这篇文章里。

## 什么是 REST API？

当你使用 C/C++，Python 等时，它不像一个编程 API。相反，它是作为一组 URL 公开的 API。这意味着除了 Web 浏览器之外，您可以与 REST API 进行交互。REST 只是使用 Web 技术(如 HTTP 协议)与服务器上运行的应用程序进行交互的一种简单方式。

您主要通过使用以下两种 HTTP 请求之一来实现这一点:

*   当你在网络浏览器中写一个网址时，就会发生这种情况。一个 GET 请求被发送到 web 服务器，一些数据被返回，通常是 web 浏览器显示的 HTML 格式的网页。
*   `POST`用于发送数据。它还包括指定一个 URL，但也包括提供一些要发送的数据。

我所说的*请求*是指你通过 TCP/IP 连接向服务器发送一些数据，目的是取回一些特定的数据。

例如，您可以通过在网络浏览器中输入 URL 地址`http://blog.translusion.com`来访问我的主页，或者您可以使用 Unix `curl`命令来下载 HTML 页面:

```
$ curl [http://blog.translusion.com](http://blog.translusion.com)
```

如果您也添加了`-i`开关，您还会得到类似如下的 HTTP 报头:

```
$ curl -i http://blog.translusion.com

HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Server: GitHub.com
Last-Modified: Thu, 15 Oct 2020 22:47:13 GMT
Content-Length: 9879
Accept-Ranges: bytes
Age: 146
Connection: keep-alive
X-Served-By: cache-osl6530-OSL
```

实际上，您可以使用一个更加“原始”的工具 NetCat 来完成同样的任务，这个工具通过 Unix 命令`nc`来调用。使用这个命令，您基本上只需发送原始数据。好处是您可以更好地理解 HTTP 协议是如何工作的。

```
$ nc blog.translusion.com 80
GET / HTTP/1.1
Host: blog.translusion.com
```

您在命令下面看到的内容不会返回。确切地说，这是我在终端中写的内容。你点击输入一个额外的时间，你会得到与`curl`相同的内容。点击`Ctrl+D`退出。

要执行一个 HTTP 请求，应该在`nc`中写什么可能并不明显。但是你可以通过在服务器模式下运行`nc`并使用`curl`连接到它来作弊。然后你就可以知道`curl`到底是做什么的了。在单独的窗口中，您可以运行以下命令来侦听端口 1234 上的连接:

```
$ nc -l 1234
```

然后在您原来的终端窗口中，您可以使用`curl`向您的本地`nc`服务器发出请求

```
$ curl -i [http://localhost:1234/foo/bar.html](http://localhost:1234/foo/bar.html)
```

在假服务器端，您将看到以下输出:

```
$ nc -l 1234
GET /foo/bar.html HTTP/1.1
Host: localhost:1234
User-Agent: curl/7.64.1
Accept: */*
```

对于 HTTP 响应，您可以在头部获得一系列信息。最重要的部分是这样的部分:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
```

它告诉您使用的 HTTP 协议版本，请求是否成功，您得到的是什么类型的数据以及它是如何格式化的。以下是一些常见的 HTTP 状态代码，您可能已经在某个时候见过了:

*   `200 OK`成功
*   `400 Bad Request`请求中的语法错误
*   `401 Unauthorized`认证失败
*   `402 Not found`网页不在那里

## 贸易工具

好的，希望这能给你一个模糊的概念，当我说 REST API 的时候，我在说什么。稍后我们将会看到更多的例子。了解一些可能用于 REST API 的工具是很有用的。

*   使用 HTTP 协议获取网站或任何其他数据。
*   [curlish](https://pythonhosted.org/curlish/) 类似 curl 但支持 [OAuth 认证](https://en.wikipedia.org/wiki/OAuth)。例如被脸书使用。
*   [jq](https://stedolan.github.io/jq/) Like *sed* 为 JSON 数据
*   nc 是 TCP/IP 的“瑞士军刀”。您可以将它用作服务器或客户端。您可以读取和发送原始 HTTP 请求和回复。
*   [ifconfig](https://en.wikipedia.org/wiki/Ifconfig) 列出网络接口及其 IP 地址
*   [Charles Proxy](https://www.charlesproxy.com) 让你看到你和互联网之间的所有流量。*响应*和*请求*可以被记录和重放。非常通用的工具。

## 阅读 REST API 文档

让我们看看如何使用`curl`来探索一个文档化的 REST API。这里有一个来自 JFrog 的例子，是我最近正在做的。JFrog 提供了许多服务，其中之一是为二进制文件提供一个存储库，他们称之为 Artifactory。

## 文件信息

其中一个 API 调用是 [File Info](https://www.jfrog.com/confluence/display/JFROG/Artifactory+REST+API#ArtifactoryRESTAPI-FileInfo) ，它用于获取存储在您在 JFrog artifactory 中创建的存储库中的文件的信息。

文档的重要部分提供了以下信息:

*   **用法:**
*   **出产:**

*用法*告诉我们，我们需要将它指定为一个`GET`请求，而*产生的*告诉我们，我们得到的响应将是一个 JSON 数据。

花括号表示占位符。因此,`{repoKey}`是您的存储库的名称,`{filePath}`是您想要了解的 repo 中某个文件的路径。

在我下面的例子中，我通过将`stuff`指定为我的 repo 的名称并将文件路径指定为`myfile.txt`来调用这个 API。

```
$ curl -u erik:qwerty -X GET "https://mickeymouse.jfrog.io/artifactory/api/storage/stuff/myfile.txt"
```

如果您将`nc`设置为一个服务器，并连接到一个本地`nc`服务器，那么这个`curl`调用将会导致这个文本被传输到服务器。

```
GET /artifactory/api/storage/stuff/myfile.txt HTTPS/1.1
Host: mickeymouse.jfrog.io:80
Authorization: Basic ZXJpazpxd2VydHkK
```

请注意添加`-u erik:qwerty`来指定登录名和密码是如何导致以下字段被添加到请求中的:

```
Authorization: Basic ZXJpazpxd2VydHkK
```

我们如何得到它？事实上，您可以自己创建这个字符串。`ZXJpazpxd2VydHkK`其实只是`erik:qwerty`的一个 [Base64 编码](https://en.wikipedia.org/wiki/Base64)。编码只是以不同的格式存储一些信息的一种方式。Base64 涉及将每个 6 位集合转换成从 0 到 63 的可见字符。

我们可以用`base64` Unix 命令来执行这个操作:

```
echo "erik:qwerty" | base64
ZXJpazpxd2VydHkK
```

进行这个*文件信息*调用将给出 JSON 数据作为反馈。我们通常希望以某种方式解析它。

```
{
  "repo" : "stuff",
  "path" : "/myfile.txt",
  "created" : "2020-10-16T12:51:33.916Z",
  "createdBy" : "erik",
  "lastModified" : "2020-10-16T12:51:33.699Z",
  "modifiedBy" : "erik",
  "lastUpdated" : "2020-10-16T12:51:33.917Z",
  "downloadUri" : "https://mickeymouse.jfrog.io/artifactory/stuff/myfile.txt",
  "mimeType" : "text/plain",
  "size" : "42",
  "checksums" : {
    "sha1" : "d0d8980bdbe9622acff4c41614d84b9692dd13be",
    "md5" : "db98b69f101495872bda4805e2803742",
    "sha256" : "3bc6b50993e609908d2946c748b8eee664201c9bbb0a45b9648a6bc9f64c1b15"
  },
  "originalChecksums" : {
    "sha256" : "3bc6b50993e609908d2946c748b8eee664201c9bbb0a45b9648a6bc9f64c1b15"
  },
  "uri" : "https://mickeymouse.jfrog.io/artifactory/api/storage/stuff/myfile.txt"
}
```

我们可以像这样使用`jq`取出 MD5 校验和:

```
$ curl -u erik:qwerty -X GET "https://mickeymouse.jfrog.io/artifactory/api/storage/stuff/myfile.txt" | jq '.["checksums"]["md5"]'
"db98b69f101495872bda4805e2803742"
```

## 使用 Julia 处理 REST APIs

我们需要一些不同的包来使用 Julia 和 REST APIs。

```
(@v1.5) pkg> add HTTP
(@v1.5) pkg> add JSON
```

使用`HTTP`包，我们可以执行 REST 请求:

```
julia> using HTTP

julia> r = HTTP.request(:GET, "https://erik:qwerty@mickeymouse.jfrog.io/artifactory/api/storage/stuff/myfile.txt");

julia> julia> fieldnames(typeof(r))
(:version, :status, :headers, :body, :request)
```

使用 JSON 包，我们可以从请求响应的有效负载体中提取数据:

```
julia> using JSON
julia> s = String(r.body);

julia> dict = JSON.parse(s);

julia> dict["checksums"]
Dict{String,Any} with 3 entries:
  "sha256" => "3bc6b50993e609908d2946c748b8eee664201c9bbb0a45b9648a6bc9f64c1b15"
  "md5"    => "db98b69f101495872bda4805e2803742"
  "sha1"   => "d0d8980bdbe9622acff4c41614d84b9692dd13be"

julia> dict["checksums"]["md5"]
"db98b69f101495872bda4805e2803742"
```

## 使用 Python 处理 REST APIs

使用 Julia 或`jq`来处理 REST APIs 的问题是，这两个软件通常都不包含在随机的 Linux 安装中。这有什么关系？你经常在 docker 容器中运行这些东西。例如，我正在使用位桶管道，这是一种在源代码提交时触发代码运行的方式。

在我的例子中，我想使用 REST API 调用将生成的二进制代码从构建发送到 JFrog 二进制存储库。这意味着通常使用一些标准的 docker 容器。在这些容器中，通常会安装以下内容:

*   `curl`
*   `python` 2.7

但是`jq`、`julia`、`ruby`和许多其他潜在有用的东西将不在那里。其次，你也没有安装很多 Python 包。因此，我们在这里探索的不是使用某种终极 Python 包来使用 REST，而是使用 Python 自带的东西。

我们通常可以访问 python [JSON 包](https://docs.python.org/3/library/json.html)。在此阅读用法示例[。](https://docs.python.org/3/library/json.html)

下面是一个基于标准安装库的 Python 2.7 解决方案。我们使用`urllib2`包来创建一个 URL 请求。

```
import urllib2
import base64
import json

url = "https://mickeymouse.jfrog.io/artifactory/api/storage/stuff/myfile.txt"
login = base64.encodestring("erik:qwerty")[:-1]
authheader =  "Basic %s" % login

req = urllib2.Request(url)
req.add_header("Authorization", authheader)
io = urllib2.urlopen(req)

s = io.read()
dict = json.loads(s)
dict[u'checksums'][u'md5']
```

这个解决方案需要一些注释。例如，为什么有一个用于创建`login`变量的`[:-1]`？因为`base64.encodestring`在末尾添加了一个我们不想包含的新行。它不是 Base-64 编码的一部分

还要注意，用`urllib2`你不能写:

```
url = "https://erik:qwerty@mickeymouse.jfrog.io"
```

软件包会误解，认为你试图提供一个端口号，而不是提供登录名和密码。由于这个原因，我们必须用代码手动构造基本身份验证的头条目:

```
login = base64.encodestring("erik:qwerty")[:-1]
authheader =  "Basic %s" % login
req.add_header("Authorization", authheader)
```

## 结束语

这绝不是一个详尽的处理。我真的刚刚介绍了如何用不同的方式做一个适度复杂的 REST 调用。