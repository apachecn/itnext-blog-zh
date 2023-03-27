# 使用 CATKeys 通过客户端认证的 TLS 保护节点服务

> 原文：<https://itnext.io/securing-node-services-using-client-authenticated-tls-with-catkeys-938709726fbc?source=collection_archive---------2----------------------->

![](img/ca1afaaf5b83a67dc56a4ceb4f18f894.png)

有些服务是公开的，每个人都应该可以使用。有些服务是私有的，应该只允许一组特定的客户端访问。

本指南介绍了使用 CATKeys 通过客户端认证 TLS 来保护基于节点的 web 服务的步骤，以便只有经过授权的客户端才能访问私有 web 服务。

# 客户端验证的 TLS

[客户端认证 TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security#Client-authenticated_TLS_handshake) 是 TLS 握手的一个版本，它使用客户端证书在客户端和服务器之间提供[相互认证](https://en.wikipedia.org/wiki/Mutual_authentication)(也称为双向认证)。

相互身份验证意味着客户端将只连接到有效的服务器(普通 TLS 就是这种情况)，而且服务器将只允许有效的客户端连接。

这使得它在只有特权客户端能够访问 web 服务或 RPC 端点的情况下非常有用。例如，您可能有一个使用私有 web 服务的公共 web 服务。

我将使用一个名为 CATKeys 的库(我是该库的作者)演示一种简单的方法来保护带有客户端认证 TLS 的节点服务器。

# 猫钥匙

[CATKeys](https://github.com/93million/catkeys) 是在阅读了 Anders Brownworth 的[帖子后创建的，该帖子描述了如何使用 OpenSSL 来生成使用客户端证书保护 HTTPS 所需的 ca、证书和密钥。这是一篇写得很好，内容广泛的帖子，如果你想了解幕后发生的事情，值得一读。](https://engineering.circle.com/https-authorized-certs-with-node-js-315e548354a2)

CATKeys 是一个简单的库，只需很少的努力就能提供相互认证。它支持 HTTPS 以及 TLS 通信。使用简单的命令生成密钥，然后在您使用`https.createServer()`、`https.request()`、`tls.createServer()`或`tls.connect()`的任何地方使用 CATKeys 作为替换。

在本指南中，我们将创建一个简单的 web 服务，它将使用包含用于连接的客户端密钥名称的 JSON 对象进行响应。

## 创建简单节点 HTTP 服务器

让我们从一个简单的 http 服务器开始，我们将把它迁移到 CATKeys。它将返回一个 JSON 对象，描述服务器是否安全。

创建一个服务器并另存为`serve.js`:

```
const http = require('http')
const serve = () => {
  http.createServer(
    (req, res) => {
      res.writeHead(200)
      res.write(JSON.stringify({
        isSecure: req.socket.authorized === true
      }))
      res.end()
    }
  ).listen(8080)
}
serve()
```

现在让我们创建一个客户端并将其保存为`request.js`:

```
const http = require('http')
const request = () => {
  const req = http.request(
    'http://localhost:8080',
    (res) => {
      const data = []
      res.on('data', (chunk) => { data.push(chunk) })
      res.on('end', () => { console.log(data.join('')) })
      res.on('error', console.error)
    }
  )
  req.end()
}
request()
```

测试文件是否有效。在一个终端中启动服务器:

```
node serve.js
```

在另一个示例中，运行请求:

```
node request.js
```

您应该会在终端上看到服务器的响应输出:

```
$ node request.js
{"isSecure":false}
```

## 迁移到猫键

以下命令需要从项目根目录运行。

安装来自 NPM 的 CATKeys:

```
npm i --save catkeys
```

生成服务器和客户端密钥:

```
npx catkeys create-key --keydir catkeys --server
npx catkeys create-key --keydir catkeys
```

`--keydir` arg 指定存储密钥的目录的位置。`--server` arg 创建一个服务器密钥。服务器密钥总是需要首先创建，因为客户端密钥是从服务器密钥创建的。

名为`catkeys`的目录现在存在于您的项目根目录中，它包含两个键:

```
$ ls -l catkeys
total 32
-rw-r--r--  1 pommy  staff  5372  4 Sep 21:10 client.catkey
-rw-r--r--  1 pommy  staff  7857  4 Sep 21:09 server.catkey
```

CATKeys 是用于创建服务器和发出请求的`https`和`tls`方法的包装器。CATKeys 方法的签名与 Node 提供的签名相同。唯一的区别是 CATKeys 方法是异步的。这意味着需要进行以下更改:

*   `https`应该从`catkeys`库中导入
*   `await`应与 CATKeys 方法一起使用
*   封闭函数应该使用`async`

做出这些更改后，`serve.js`现在看起来像这样:

```
const { https } = require('catkeys')
const serve = async () => {
  (await https.createServer(
    (req, res) => {
      res.writeHead(200)
      res.write(JSON.stringify({
        isSecure: req.socket.authorized === true
      }))
      res.end()
    }
  )).listen(8080)
}serve()
```

在`request.js`中，URL 方案也需要从`http`变为`https`。

```
const { https } = require('catkeys')
const request = async () => {
  const req = await https.request(
    'https://localhost:8080',
    (res) => {
      const data = []
      res.on('data', (chunk) => { data.push(chunk) })
      res.on('end', () => { console.log(data.join('')) })
      res.on('error', console.error)
    }
  )
  req.end()
}
request()
```

让我们重新启动服务器，再次运行客户机请求:

```
$ node request.js
{"isSecure":true}
```

当服务器运行时，我们可以使用`curl`验证服务器是否需要客户端证书:

```
$ curl --insecure [https://localhost:8080/](https://localhost:8080/)
curl: (56) OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert **certificate required**, errno 0
```

目前客户端能够连接到服务器，因为`server.js`和`request.js`共享项目根目录中的同一个`catkeys`目录。如果它们运行在不同的服务器上，您需要在客户端的项目根目录中创建一个`catkeys`目录，并将`catkeys/client.catkey`文件从服务器复制到其中。

## 服务器主机名

TLS 证书通常包含主机名。当客户端连接到服务器时，会将服务器的主机名与其证书中的 common 和 alt 名称进行比较，如果它们不匹配，则会引发错误。

CATKeys 使用稍微不同的方法。CATKeys 希望服务器证书包含一个为服务器密钥保留的特殊通用主机名。这意味着服务器的主机名可以更改，并且不会影响证书的有效性。使用存储在客户端密钥中的 CA 安全地验证服务器的身份。

要看到这种效果，尝试将`request.js`中的主机名从`localhost`更改为`127.0.0.1`，并重复请求。

如果您希望将密钥限制在特定的主机名上使用，您可以通过使用带有`create-keys`命令的 arg `--name`来生成带有主机名的服务器密钥:

```
npx catkeys create-key -k catkeys -s --update --name example.com
```

现在你只能在`example.com`上主持这个键了。以任何其他名称托管服务器将导致客户端在连接时抛出错误。如果您重启服务器并运行客户机请求，您将得到一个类型为`ERR_TLS_CERT_ALTNAME_INVALID`的错误。

```
$ node request.js
events.js:288
      throw er; // Unhandled 'error' event
      ^
Error [ERR_TLS_CERT_ALTNAME_INVALID]: Hostname/IP does not match certificate's altnames: IP: 127.0.0.1 is not in the cert's list:
```

要从服务器密钥的主机名中删除`example.com`，请再次更新密钥并删除`--name`参数:

```
npx catkeys create-key -k catkeys -s --update
```

如果您想要告诉客户端拒绝通常出现在服务器密钥中的特殊的、通用的主机名，并且只连接到具有与其主机名匹配的证书的服务器，您可以提供选项`catRejectMismatchedHostname: true`:

```
const req = await https.request(
  'https://localhost:8080',
  { catRejectMismatchedHostname: true },
  (res) => { ... }
)
```

## 客户名称

默认情况下，客户端密钥被命名为`client`。如果愿意，您可以在多个客户端之间共享一个客户端密钥。或者您可以创建几个密钥来区分不同的客户端。

要创建具有特定名称的客户端密钥，请将`--name`传递给`create-key`命令:

```
npx catkeys create-key --keydir catkeys --name client-key-2
```

如果只有一个客户端密钥，则 CATKeys 将始终使用该密钥进行连接。如果超过 1 个，您将需要定义连接时使用哪个密钥:

```
const req1 = await https.request(
  'https://host1.example.com/',
  { **catKey: 'client'** }
  (res) => { ... }
)
const req2 = await https.request(
  'https://host2.example.com/',
  { **catKey: 'client-key-2'** }
  (res) => { ... }
)
```

## 访问客户端名称

假设您想基于客户机的名称实现某种访问控制，或者您可能只想记录连接到服务器的客户机的名称。您可以通过在服务器上的请求处理器中使用`req.connection.getPeerCertificate()`访问证书来获得客户端的名称:

```
(await https.createServer(
  (req, res) => {
    const clientCert = req.connection.getPeerCertificate()
    console.log('Connection from client: ' + clientCert.subject.CN)
    ...
  }
)).listen(8080)
```

## 撤销客户端的访问权限

有两种方法可以取消对客户端的访问。

如果在您的`catkeys`目录中仍然可以访问您想要撤销的客户端密钥，您可以使用`revoke-key` cli 命令撤销该密钥。

例如，如果您创建了一个名为`client-to-revoke`的键，如下所示:

```
npx catkeys create-key --keydir catkeys --name client-to-revoke
```

那么你可以像这样撤销它:

```
npx catkeys revoke-key --keydir catkeys --name client-to-revoke
```

只有当您有权访问客户端密钥时，才能使用`revoke-key` cli 命令。如果您不再拥有对客户端密钥的访问权，那么您可以通过在调用`createServer()`时传递选项`catCheckKeyExists: true`来限制对存在于`catkeys`目录中的密钥的访问，从而有效地撤销它:

```
(await https.createServer(
  { catCheckKeyExists: true },
  (req, res) => {
    ...
  }
)).listen(8080)
```

CATKeys 将检查 Keys 目录以查看客户端密钥是否存在。删除客户端密钥将撤销访问权限。如果密钥不在磁盘上，那么连接将被关闭，请求处理程序将不会被调用。

## 对节点以外的服务器使用密钥

CATKeys 可以用于节点以外的服务器。密钥只是 TAR 存档，可以扩展。可以提取由 ca、证书和密钥组成的，用于许多支持 TLS/SSL 的服务器。

提取我们之前创建的服务器密钥:

```
npx catkeys extract-key catkeys/server.catkey
```

服务器密钥已经被提取到当前目录中名为`server`的目录中。

```
$ ls -l server
total 32
-rw-r--r-- 1 pommy staff 2053 24 Feb 19:44 ca-crt.pem
-rw-r--r-- 1 pommy staff 2009 24 Feb 19:44 crt.pem
-rw-r--r-- 1 pommy staff 3243 24 Feb 19:44 key.pem
```

这些文件可以在任何支持 pem 格式的 web 服务器上使用。

例如，Nginx 可以配置为在端口 8080 上请求客户端证书，并代理到端口 8081 上的 HTTP 上游服务器，如下所示:

```
server {
  listen 8080 ssl;
  ssl_certificate /path/to/server/crt.pem;
  ssl_certificate_key /path/to/server/key.pem;
  ssl_client_certificate /path/to/server/ca-crt.pem;
  ssl_verify_client on;
  location / {
    proxy_pass [http://localhost:8081;](http://localhost:8081;)
  }
}
```

> *⚠️这个* `*ssl_verify_client on;*` *很重要。它使用* `*ssl_client_certificate*` *中指定的证书验证客户端，并拒绝验证失败的客户端。*

因为 TLS 连接是在服务器端终止的，所以您不能使用选项`catCheckKeyExists: true`来拒绝磁盘上没有密钥的客户端。

## 坦克激光瞄准镜（Tank Laser-Sight 的缩写）

我们只介绍了 HTTPS，但是 CATKeys 也可以支持通过 TLS 升级的套接字进行通信。这对于实时通信很有用。

您可以使用`const { tls } = require('catkeys')`导入`tls`模块。我不打算深入研究如何使用`tls.createServer()`和`tls.connect()`，因为它们与 Node 的`tls`模块中的`tls.createServer()`和`tls.connect()`具有相同的签名(区别也在于它们是`async`方法),并且可以使用我们在`https.createServer()`和`https.request()`中看到的相同的`cat*`选项。

CATKeys 文档包括创建[普通 TLS 服务器和客户端](https://github.com/93million/catkeys/tree/master/examples/tls)的示例，以及与 JsonSocket 一起使用的[——允许使用 JSON 通过 TLS 连接发送结构化数据。](https://github.com/93million/catkeys/tree/master/examples/json-socket)

## 为什么是另一个安全库？

对于那些想知道新东西有什么意义的人来说，CATKeys 并没有真正发明任何新东西。Node 长期以来一直支持客户端认证的 TLS，并且许多可以用作反向代理的 web 服务器(Nginx、HAProxy、Apache)都支持它。安全性由 TLS 协议处理——cat keys 只是作为一个实用程序来创建和打包密钥以便于分发，此外它还包括一组用于加载这些密钥的包装方法。

# 有什么问题吗？

希望这能让您了解如何使用 CATKeys 实现客户端认证的 TLS。我错过了什么或者让你困惑了吗？不要犹豫，在评论中提出问题和建议。