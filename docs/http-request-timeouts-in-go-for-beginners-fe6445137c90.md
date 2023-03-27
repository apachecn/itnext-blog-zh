# 初学者在 Go 中的 HTTP 请求超时

> 原文：<https://itnext.io/http-request-timeouts-in-go-for-beginners-fe6445137c90?source=collection_archive---------0----------------------->

![](img/b6df340cd29a60a999f5502a12e0e229.png)

[张浩](https://unsplash.com/@hharvey?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

超时是分布式系统中基本的可靠性概念之一，可以减轻分布式系统不可避免的故障的影响，正如这篇推文中所提到的

# 问题是

> 如何模拟 504 http？StatusGatewayTimeout 有条件响应？

当试图在 [zalando/skipper](https://github.com/zalando/skipper/issues/633) 中实现 OAuth 令牌验证时，我必须理解并实现一个测试，以便在服务器超时时使用`httptest`来[模拟一个](https://stackoverflow.com/questions/51319726/how-to-mimic-504-timeout-error-for-http-request-inside-a-test-in-go) `[504 http.StatusGatewayTimeout](https://stackoverflow.com/questions/51319726/how-to-mimic-504-timeout-error-for-http-request-inside-a-test-in-go)`，但只是在客户端因服务器延迟而超时时。作为一个语言初学者，我做了我们大多数人都会做的事情；创建标准 HTTP 客户端并添加一个超时，如下所示:

```
client := http.Client{Timeout: 5 * time.Second}
```

当想要创建一个客户机来发出 http 请求时，上面的内容看起来非常简单和直观。但是隐藏在下面的是许多底层细节，包括客户端超时、服务器超时和负载平衡器超时。

## 客户端超时

可以在客户端使用多种方法定义 http 请求超时，具体取决于请求周期中的超时帧。请求-响应周期由`Dialer`、`TLS Handshake`、`Request Header`、`Request Body`、`Response Header`和`Response Body`超时组成。根据请求-响应的上述部分，Go 提供了以下创建超时请求的方法

*   `http.client`
*   `context`
*   `http.Transport`

## [http.client](https://godoc.org/net/http#Client) :

`http.client`超时是超时的高级实现，包括从`Dial`到`Response Body`的整个请求周期。实现方式`http.client`是一个结构类型，它接受类型`time.Duration`的可选`Timeout`属性，该属性定义了从请求开始到响应体被刷新的时间限制

```
client := http.Client{Timeout: 5 * time.Second}
```

## [语境](https://godoc.org/context)

go `context`包通过`[WithTimeout](https://godoc.org/context#WithTimeout)`、`WithDeadline`和`WithCancel`方法提供了处理超时、截止日期和可取消请求的有用工具。使用`WithTimeout`，您可以使用`req.WithContext`方法将超时添加到`http.Request`

```
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()
req, err := http.NewRequest("GET", url, nil)
if err != nil {
    t.Error("Request error", err)
}

resp, err := http.DefaultClient.Do(req.WithContext(ctx))
```

## [http。运输](https://godoc.org/net/http#Transport):

您还可以使用低层实现来指定超时，即使用`DialContext`创建自定义`http.Transport`，并使用它来创建`http.client`

```
transport := &http.Transport{
    DialContext: (&net.Dialer{   
        Timeout: timeout,
    }).DialContext,
}client := http.Client{Transport: transport}
```

# 解决方案

所以带着上面的问题和手头的选项，我用`context.WithTimeout()`创建了一个`http.request`。但这仍然失败，并出现以下错误

```
client_test.go:40: Response error Get [http://127.0.0.1:49597](http://127.0.0.1:49597): context deadline exceeded
```

## **服务器端超时**

`context.WithTimeout()`方法的问题是它仍然只模拟请求的客户端。如果请求头或请求体花费的时间超过预算的超时时间，则请求在客户端本身失败，而不是在服务器端失败，状态代码为`504 http.StatusGatewayTimeout`。

创建每次都超时的 httptest 服务器的一种方法如下。

```
httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, _ *http.Request){
    w.WriteHeader(http.StatusGatewayTimeout)
}))
```

但是我希望它只根据客户端超时值超时。为了让服务器基于客户机超时返回一个 504，您可以用一个处理函数`http.TimeoutHandler()`包装处理程序，使服务器上的请求超时。下面是应用这个场景的工作测试

```
func TestClientTimeout(t *testing.T) {
    handlerFunc := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        d := map[string]interface{}{
            "id":    "12",
            "scope": "test-scope",
        }

        time.Sleep(100 * time.Millisecond) //<- Any value > 20ms
        b, err:= json.Marshal(d)
        if err != nil {
            t.Error(err)
        } io.WriteString(w, string(b))
        w.WriteHeader(http.StatusOK)
    })

    backend := httptest.NewServer(http.TimeoutHandler(handlerFunc, 20*time.Millisecond, "server timeout"))

    url := backend.URL
    req, err := http.NewRequest("GET", url, nil)
    if err != nil {
        t.Error("Request error", err)
        return
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        t.Error("Response error", err)
        return
    }

    defer resp.Body.Close()
}
```

上述问题的细节在 [zalando/skipper](https://github.com/zalando/skipper/) 中[tokeninfo _ test . go/testoauth 2 tokentimeout](https://github.com/zalando/skipper/blob/8b23d2c2dd0ec54585b276a9ceb2a5137dc92db0/filters/auth/tokeninfo_test.go#L271)的实现中使用

一个初学 gopher 的人可能会发现理解 http 超时的高级工作方式很有用！如果你想查看 go 中`http timeouts` 的更多细节，这篇来自 [Cloudflare](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/) 的文章是必读书。