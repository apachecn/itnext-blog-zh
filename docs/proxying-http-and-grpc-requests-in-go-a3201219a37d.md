# 在 Go 中代理 HTTP 和 gRPC 请求

> 原文：<https://itnext.io/proxying-http-and-grpc-requests-in-go-a3201219a37d?source=collection_archive---------3----------------------->

![](img/afda3899b06e4412f2db5b72e2df263b.png)

golang gopher 吃爆米花

在[主机因子](https://hostfactor.io/)处，所有传入流量都通过我们的代理层，所有传出 TCP 流量都被重定向到我们的应用代理。捕获传出的 TCP 流量非常重要，因为我们所有的内部服务都使用 HTTP 或 gRPC (HTTP/2 ),我们希望防止任何用户服务器与我们的内部服务进行通信。在做了一些调查之后，我们在 Go 中找不到任何可以在一个服务器中代理 HTTP/1 和 HTTP/2 请求的简单库，所以我们决定创建一个。

对于这个例子，让我们假设我们想要阻塞对`blocked.hostfactor.io`的所有请求

# TCP 服务器

因为 HTTP/1 和 HTTP/2 都在幕后使用 TCP，所以我们可以从那里开始。

```
addr, _ := net.ResolveTCPAddr("tcp", ":8080")

list, err := net.ListenTCP("tcp", addr)
if err != nil {
 return err
}

go func() {
 for {
  conn, err := list.Accept()
  if err != nil {
   if errors.Is(err, net.ErrClosed) || errors.Is(err, io.EOF) {
    return
   }
   continue
  }

  go handleConn(conn.(*net.TCPConn))
 }
}()
```

这里，我们在端口 8080 上启动一个简单的 TCP 服务器，并将连接传递给`handleConn` func。将 HTTP/1/2 作为 TCP 处理的好处在于，我们可以让一台服务器监听一个端口来处理这两种情况。让我们深入了解一下`handleConn`

```
func handleConn(conn *net.TCPConn) {
 defer func() {
  _ = conn.Close()
 }()

 buf := make([]byte, 4096)
 n, err := conn.Read(buf)
 if err != nil {
  return
 }

 payload := buf[:n]
 req, err := http.ReadRequest(bufio.NewReader(bytes.NewBuffer(payload)))
 if err != nil {
  return
 }
...}
```

`http.ReadRequest`允许您将原始的`[]byte`转换成`*http.Request` ，这允许我们解析有效载荷。

```
if req.ProtoMajor >= 2 {
 err = l.handleHttp2(bytes.NewBuffer(payload), conn)
} else {
 err = l.handleHttpReq(req, conn)
}
```

`req.ProtoMajor`允许我们区分请求是 HTTP/2 (gRPC)还是 HTTP/1

# 处理 HTTP/1

HTTP/1 是最熟悉的情况，在 Go 中得到最多的支持，所以我们从这里开始。首先，我们必须检查主机是否等于我们禁用的`blocked.hostfactor.io`主机

```
// Check the host of the request
if req.Host == "banned.hostfactor.io" {
 resp.StatusCode = http.StatusNotFound
}
```

还可以检查路径、方法、头等。检查通过后，我们代表原始请求者发送请求

```
// Required to forward the request
req.RequestURI = ""
var err error
resp, err = http.DefaultClient.Do(req)
if err != nil {
 return err
}
```

最后，我们编写对 conn 的响应。

```
return resp.Write(w)
```

总的来说，它看起来像这样

```
func handleHttpReq(req *http.Request, w net.Conn) error {
 resp := &http.Response{}
 // Check the host of the request
 if req.Host == "banned.hostfactor.io" {
  resp.StatusCode = http.StatusNotFound
 } else {
  // Required to forward the request
  req.RequestURI = ""
  var err error
  resp, err = http.DefaultClient.Do(req)
  if err != nil {
   return err
  }
 }
 return resp.Write(w)
}
```

# 处理 HTTP/2 (gRPC)

HTTP/2 和 HTTP/1 非常不同。在 HTTP/1 中，最小的数据单位是一个请求，而在 HTTP/2 中，请求和响应的消息被分解成帧。HTTP/1 也使用纯文本字符串，而 HTTP/2 使用二进制编码，这在处理原始 TCP 连接时会引起一些麻烦。

Go `http`包没有对应于 HTTP/2 的`http.ReadRequest`函数。相反，我们可以利用`golang.org/x/net/http2`包解析出这些通过 TCP 连接传入的帧。

我们首先需要的是一个`Framer`

```
f := http2.NewFramer(conn, conn)
```

第一个参数是一个`io.Writer`，第二个是一个`io.Reader`。一个`net.Conn`实现了这两者，所以我们可以使用相同的 TCP 连接来读和写。

HTTP/2 要求在接收到 RFC 7540 中的初始帧时发送设置确认，所以让我们在连接断开之前完成这项工作

```
err := f.WriteSettingsAck()
if err != nil {
 return err
}
```

现在我们已经建立了连接，我们可以利用`Framer`从连接中读取数据帧。完美！唯一的问题是,`Framer`消耗了来自 TCP 连接的`[]byte`,如果请求有效，我们需要转发它🤔。因此，如果`Framer`正在消耗字节，我们如何在不重新实现 HTTP/2 的情况下将数据转发到原始主机？

这就是`io.TeeReader`前来救援的地方。我们只是将从连接中读取的所有字节复制到一个缓冲区中，并将它们分派给原始主机(如果请求有效)

```
dataBuffer := bytes.NewBuffer(make([]byte, 0))
reader := io.TeeReader(conn, dataBuffer)
f = http2.NewFramer(io.Discard, reader)
decoder := hpack.NewDecoder(1024, nil)
```

现在，每当`Framer`从连接中读取时，字节也将被写入`dataBuffer`🎉

接下来将使用`decoder`通过`golang.org/x/net/http2/hpack`包实际解码报头

接下来，我们需要从`Framer`读取帧，并检查`HeadersFrame`以查看原始主机(以及请求中我们关心的任何其他元数据)

```
auth := ""
for auth == "" {
 frame, err := f.ReadFrame()
 if err != nil {
  return err
 }

 switch t := frame.(type) {
 case *http2.HeadersFrame:
  out, err := decoder.DecodeFull(t.HeaderBlockFragment())
  if err != nil {
   return err
  }

  for _, v := range out {
   if v.Name == ":authority" {
    auth = v.Value
   }
  }
 }
}
```

以上循环遍历这些帧，直到我们找到保存了`:authority`的`HeadersFrame`([相当于主机](https://www.rfc-editor.org/rfc/rfc7540#section-8.1.2.3)的 HTTP/1)。一旦找到，我们就停止读取帧。

酷！现在，我们已经通读了 HTTP/2 帧，并且知道要检查它是否有效的原始主机

```
if auth == "blocked.hostfactor.io" {
  return nil
}
```

但是如果`:authority`有效呢？我们如何将从`Framer`读取的所有数据发送到原始主机？首先，我们需要一个到原始主机的新 TCP 连接

```
dialer, err := net.Dial("tcp", auth)
if err != nil {
 return err
}
```

现在，由于我们为客户机使用了一个原始的 TCP 连接，我们将不得不处理来自该连接的读和写。让我们从阅读方面开始

```
// The WaitGroup ensures that every byte is read before exiting
wg := sync.WaitGroup{}
wg.Add(1)
dataSent := int64(0)
go func() {
 // Copy any data we receive from the host into the original connection
 dataSent, err = io.Copy(conn, dialer)
 wg.Done()
}()
```

现在我们可以处理对原始主机的写入

```
_, err = io.Copy(dialer, io.MultiReader(initial, dataBuffer, conn))
wg.Wait()
```

`initial`是从请求方读取的初始字节，`dataBuffer`是我们从连接中读取以检查帧的字节，`conn`是原始字节。一个`MultiReader`简单地将所有这些链接在一起，这样它们就可以按顺序读取并发送给原始主机。这保证了我们到目前为止读取的所有数据都将被发送到原始主机。将所有这些放在一起看起来像

```
func handleHttp2(initial io.Reader, conn net.Conn) error {
 defer func() {
   _ = conn.Close()
 }()
 dataBuffer := bytes.NewBuffer(make([]byte, 0))
 reader := io.TeeReader(conn, dataBuffer)
 f := http2.NewFramer(conn, conn)
 err := f.WriteSettingsAck()
 if err != nil {
  return err
 }

 f = http2.NewFramer(io.Discard, reader)
 decoder := hpack.NewDecoder(1024, nil)

 auth := ""
 for auth == "" {
  frame, err := f.ReadFrame()
  if err != nil {
   return err
  }

  switch t := frame.(type) {
  case *http2.HeadersFrame:
   out, err := decoder.DecodeFull(t.HeaderBlockFragment())
   if err != nil {
    return err
   }

   for _, v := range out {
    if v.Name == ":authority" {
     auth = v.Value
    }
   }
  }
 }

 if auth == "blocked.hostfactor.io" {
  return nil
 }

 dialer, err := net.Dial("tcp", auth)
 if err != nil {
  return err
 }

 _ = dialer.SetReadDeadline(time.Now().Add(5 * time.Second))

 wg := sync.WaitGroup{}
 wg.Add(1)
 dataSent := int64(0)
 go func() {
  // Copy any data we receive from the host into the original connection
  dataSent, err = io.Copy(conn, dialer)
  wg.Done()
 }()

 _, err = io.Copy(dialer, io.MultiReader(initial, dataBuffer, conn))
 wg.Wait()

 if errors.Is(err, os.ErrDeadlineExceeded) && dataSent > 0 {
  return nil
 }
 return err
}
```

现在你知道了！能够阻止任何通过 HTTP/1 或 HTTP/2 接收的流量的代理服务器。

我们在 [Host Factor](https://hostfactor.io/) 利用大量 Go 和其他几项现代技术来确保我们的客户拥有托管其应用程序的最佳体验。请检查一下！