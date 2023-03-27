# Go 中 HTTP 和单元测试的一些模式

> 原文：<https://itnext.io/some-patterns-for-http-and-unit-testing-in-go-221097a0597b?source=collection_archive---------0----------------------->

![](img/bcc04646e6b917115265bcd34cf0cc1d.png)

# 每个人都写测试

我想马上得到一分。几年前，我和一个团队一起工作，他们不断地向构建贡献错误的代码。当我们研究该团队的开发实践时，我们发现了一个非常糟糕的反模式，即他们将团队分为“开发人员”和“测试人员”，而测试人员单独负责编写测试代码(包括单元测试)。没有人想成为一名测试人员，测试人员在确保测试覆盖率方面一直处于水下(更不用说确保测试实际上是好的)。

在我的团队中，*每个人都写测试*。我不知道你在这个问题上的立场，但是虽然我对很少的事情有宗教信仰，但是我在执行测试驱动的开发实践上立场相当坚定。如果你不知道如何测试你的代码，你不应该把它添加到构建中。实际上，如果您不知道如何测试您的代码，并且您不能花时间编写可靠的单元测试来防止回归，那么就不要提交。

你不必同意我的立场，但我想提供一些背景。我认为，不仅每个人都需要知道如何编写测试代码，而且每个人*都应该*编写测试代码。

本文面向 Go 开发人员，他们可能已经使用过 Go 的内置`testing`包，但是(a)不熟悉一些可以帮助他们改进测试的好的测试工具，和/或(b)不确定如何为一些特定的场景编写单元测试——在本例中是 HTTP 客户端和服务器。

我经常看到开发人员仅仅依赖于客户端/服务器案例的集成测试，但是这样做太晚了。让集成测试驱动所有可能的负面(错误)案例通常是不可行的，这意味着在代码已经交付之后，边缘案例可能会泄漏并表现为 bug。因此，我认为，掌握我将要演示的一些技术(或者类似的，甚至可能更好的技术)是 HTTP 开发人员技能/工具集的一个非常重要的部分。

# 文章的代码示例

我在文章中使用的所有代码示例都可以在[https://github.com/ScarletTanager/go-http-testing](https://github.com/ScarletTanager/go-http-testing)找到。您可以自由地复制/使用/修改您认为合适的代码，并且我欢迎对文章和/或代码示例的评论。

# Golang 测试工具箱

我将首先列出一些我在测试中使用(并提倡使用)的工具，我建议每个 Go 开发者都尝试一下并熟悉它们——我认为它们是值得投资的。

## 银杏树

Onsi Fakhouri 的[银杏包](https://github.com/onsi/ginkgo)为 golang 提供了一个健壮的行为驱动开发(BDD)框架。Ginkgo 提供了一个库，开发人员可以用它来以一种更清晰、更有组织的方式构建测试，同时还提供了命令行工具`ginkgo`来运行测试。银杏与`testing`包完全兼容(例如，你可以用`go test`毫无问题地运行银杏测试)，因为它建立在这个包之上，而不是取代它。

我曾听开发人员称银杏为语法糖，但我认为这是对银杏的漠视——它不仅仅是表达测试的一种方式。Ginkgo 提供了状态管理(安装和拆卸都受支持)、嵌套上下文(例如，用于异常测试)和其他值得探索的特性。

银杏通常与匹配器库(提供测试结果的实际条件评估)一起使用，这里我推荐 Onsi Fakhouri 的另一个包…

## 戈梅加

Gomega 是一个非常健壮的 Golang 测试匹配器库。你可以阅读优秀的[文档](https://onsi.github.io/gomega/)来找出它提供的一切，但是特性包括特定类型匹配器(字符串、布尔值、数字、列表类型等。)，同步和异步匹配器，并支持例如针对流数据的匹配。

Gomega 提供的一个特性是它为 http 客户端提供的特定于 http 的匹配器库，但我从未真正尝试过使用它(考虑到本文的主题，这有点奇怪)。也许有一天评估它并将其与我自己的模式进行比较将是另一篇文章的主题…

## 伪造者

我最喜欢的一个工具是 Max Brunsfeld 的[伪造者](https://github.com/maxbrunsfeld/counterfeiter)，这是一个为 golang 接口类型生成 doubles(测试假货)的工具。在 Go 中还有其他可行的生成 doubles 的方法，但是如果可以选择的话，我每次都要使用伪造者。

伪造者的问题是你需要使用接口，如果你的代码不容易使用接口，你可能无法从中获得同样多的价值，但幸运的是伪造者也可以用于为你不拥有的接口生成伪造品(例如，你想要伪造的来自 go 标准库包之一的接口)。

*在文章中，我通常交替使用“假”和“双”。一个术语相对于另一个术语的使用没有特别的含义。*

所有这些工具都被积极地维护着，我几乎没有因为推荐它们而创新。但是如果您还没有尝试过，它们非常值得您去研究。

# 模仿来自 HTTP 服务器的响应

一个最常见的用例是 API(通常是 RESTful 的，但不仅仅是 RESTful 的)客户端，它需要能够处理来自服务器的特定响应。例如，您知道 API 服务器在某些情况下可能会返回 401——在这种情况下，您的客户端需要做什么？

在这种情况下，开发人员需要模仿(模拟)服务器的行为，以便测试代码可以保证客户端处理的是 401，并且只是 401(或者客户端需要处理的任何其他代码/响应)。我们如何做到这一点？

如果我们坚持核心库所提供的，那么典型的方法是旋转一个`httptest.Server`的实例，并将我们的客户端请求发送给它。我们可以定制服务器的行为，这样我们就知道它会以一种特定的方式做出响应，这样我们就可以确信我们的客户端会为我们的测试处理正确的响应。

虽然这是一个完全有效的方法，并且是我们将在下面重新讨论的方法，但它也比我通常喜欢在客户端单元测试中使用的方法要“沉重”一些。一种简单得多(在测试执行方面也更快)的方法是模仿 *http 客户端*，这样它就能产生正确的响应。

让我们假设我们有一个如下所示的 API 客户端:

```
package clientimport (
  "errors"
  "io"
  "net/http"
)type ApplicationClient struct {
  HttpClient *http.Client
}func (c *ApplicationClient) PerformQuery(body io.Reader) error {
  r, _ := http.NewRequest(http.MethodPost, "/resource", body)
  resp, _ := c.HttpClient.Do(r)
  if resp.StatusCode == http.StatusUnauthorized {
    return errors.New("Unauthorized")
  }
  return nil
}
```

因此，在这里，与其产生创建一个`httptest.Server`实例并定制其行为的开销(通过编写一个返回特定期望响应的处理函数)，我更喜欢模拟出`http.Client`。但是怎么做呢？

首先，伪造者只支持模仿`interface`类型，不支持`struct`类型。所以我需要我的`ApplicationClient`来包装一个`interface`。但是我从哪里得到这个`interface`？答案很简单，这要感谢 Go 处理`interface`类型的方式。

我需要做的就是定义我自己的简单的`MyHttpClient`接口，并重新定义我的`ApplicationClient`来包装它:

```
type MyHttpClient interface {
  Do(*http.Request) (*http.Response, error)
}type ApplicationClient struct {
  HttpClient MyHttpClient
}
```

敏锐的读者会注意到这是`*http.Client`已经实现的方法之一——这意味着类型`*http.Client`的对象自动满足接口规范。所以到目前为止，这个简单的重构不应该破坏任何东西。

为了模仿接口，我首先需要生成假的，为此我向类型定义添加了一个注释:

```
//counterfeiter:generate . MyHttpClient
type MyHttpClient interface {
 Do(*http.Request) (*http.Response, error)
}
```

`counterfeiter`允许多种注释模式——查看文档并选择对您有意义的一种，但是一旦您运行(通常从您的项目根中运行)`go generate ./...`，最终结果将是相同的。在这里，我们会看到在我们的`client`包下，有一个名为`clientfakes`的子包。该包将包含对类型为`FakeMyHttpClient`的结构的定义，这是一个全功能的`MyHttpClient`接口的双重实现。

为了在测试中使用生成的假代码，我可能会这样做:

```
package client_testimport (
 "net/http" . "github.com/onsi/ginkgo"
 . "github.com/onsi/gomega" "myapp/client"
 "myapp/client/clientfakes"
)var _ = Describe("Client", func() {
  var (
    appClient  *client.MyApplicationClient
    httpClient *clientfakes.FakeMyHttpClient
  ) BeforeEach(func() {
    httpClient = &clientfakes.FakeMyHttpClient{}
    appClient = &client.MyApplicationClient{
      HttpClient: httpClient,
    } httpClient.DoReturns(&http.Response{}, nil)
  }) Describe("PerformQuery", func() {
    It("Executes and returns nil", func() {
      Expect(appClient.PerformQuery()).NotTo(HaveOccurred())
    })
  })
})
```

这是一个微不足道的快乐路径测试——除了演示如何使用我们生成的 fake 之外，它真的没有做什么。这里需要注意的重要一行是:

```
httpClient.DoReturns(&http.Response{}, nil)
```

在这里，我们模拟了`Do()`方法，使它返回我们希望它返回的内容，而不执行任何实际的底层 http 调用。为了做一些稍微更有意义的事情，我们可以将参数改为`DoReturns()`，以便测试我们的错误情况。如果您还记得`PerformQuery()`是如何工作的，那么只有当我们从服务器获得`http.StatusUnauthorized` (a 401)时，它才会返回一个错误。因此，让我们强制`Do()`返回一个带有该状态代码的响应，并测试我们是否返回了一个错误，正如我们期望我们的代码所做的那样:

```
Describe("PerformQuery", func() {
  It("Executes and returns nil", func() {
    Expect(appClient.PerformQuery()).NotTo(HaveOccurred())
  }) Context("When we get a 401", func() {
    JustBeforeEach(func() {
      httpClient.DoReturns(&http.Response{
        StatusCode: http.StatusUnauthorized,
      }, nil)
    }) It("Returns an error", func() {
      Expect(appClient.PerformQuery()).To(HaveOccurred())
    })
  })
})
```

# 关于句法的一些注释

## 构建银杏测试文件

银杏的一个好处是你可以灵活地组织你的测试文件。银杏提供了多种嵌套结构，惯例是按照从最外层到最内层的顺序使用它们:

1.  “这就是我所描述的(或测试的)”
2.  `Context()`——“我在测试 it 的哪个方面(或者在什么条件下)”
3.  `When()`——“在什么特定条件下”
4.  `It()` —“这是预期的行为(断言)”

如果你看我的代码，你会发现我不使用`When()`——这只是习惯，而不是有意识的选择。我倾向于使用两个级别的`Describe()`——一个用于我正在测试的包或(例如，在包包含多个复杂类型的情况下)类型，然后一个用于我正在测试的每个方法。对我来说，这更有意义，因为我正在描述一个特定方法的行为*，然后`Context()`表示方法行为被测试的*条件。*我知道其他 golang 程序员也使用其他模式。*

我喜欢广泛使用的一个特性是通过`BeforeEach()`和`JustBeforeEach()`模块嵌套前提条件/测试设置。刚接触银杏的开发人员有时会感到困惑，所以至少让我花一分钟来描述一下我是如何使用它们的。

从语义上来说，这两个块做同样的事情——这些函数接受一个匿名函数作为参数，该函数的主体在相同嵌套层(或更低嵌套层)的任何测试执行之前执行。因此它们通常用于测试设置。(如你所料，也有一个`AfterEach()`功能用于拆除——例如关闭测试服务器、销毁频道等。).但是在`BeforeEach()`和`JustBeforeEach()`之间有一个严格的顺序——假设我们有一个这样的文件:

```
Describe("A Thing", func() {
  BeforeEach("**Setup1**", func() { ... } ) JustBeforeEach("**Setup2**", func() { ... } ) Context("Some conditions", func() {
    BeforeEach("**Setup3**", func() { ... } ) JustBeforeEach("**Setup4**", func() { ... } ) When("Something is true", func() {
      BeforeEach("**Setup5**", func() { ... } ) JustBeforeEach("**Setup6**", func() { ... } ) It("Does something", func() { ... } )
    })
  })
})
```

在这里，我们在实际测试周围有三个嵌套层次。每一关我们都有一个`BeforeEach()`和一个`JustBeforeEach()`。为什么都是？正如您所料，当我们执行测试时，这些是按照从最外层到最内层的顺序执行的——`It()`调用。但是……它比那更复杂。这些命令的实际执行顺序如下:`Setup1`、`Setup3`、`Setup5`、`Setup2`、`Setup4`和`Setup6`。*中的所有* `*BeforeEach()*` *调用将执行之前* ***中任何*** *括起来的* `*JustBeforeEach()*` *调用。*这使得`JustBeforeEach()`在您需要保证封装的代码是测试前运行的最后一个代码时特别有用。

因此，在我之前的测试中，我的`JustBeforeEach()`肯定会在之前对`BeforeEach()`的调用中覆盖对`DoReturns()`的调用。因此，我可以确定测试将在什么条件下执行，而不考虑之前的封闭设置。

但是，如果你在一个封闭层中调用`JustBeforeEach()`(就像我上面做的那样),然后在一个较低的层中调用`BeforeEach()`,这可能会导致问题——例如:

```
Describe("A thing", func() {
  var (
    myValue int
  ) JustBeforeEach(func() {
    myValue = 20
  }) Context("When the value is 10", func() {
    BeforeEach(func() {
      myValue = 10
    }) It("Equals 10", func() {
      Expect(myValue).To(Equal(10))
    })
  })
})
```

正如你可能已经猜到的那样，这是行不通的——`JustBeforeEach()`将在和`BeforeEach()`之后执行*,尽管处于封闭级别。所以在使用`BeforeEach()`和`JustBeforeEach()`时要小心，因为如果不小心很容易绊倒自己。*

## 造假者嘲讽

对于被模仿界面中的每个方法，`counterfeiter`提供了许多方法来定制被模仿的行为。在上面的例子中，我使用了`DoReturns()`——对于任何方法`Foo()`，`FooReturns()`将指定对`Foo()`的调用返回特定的传递值——例如，如果`Foo()`返回一个`bool`，那么`FooReturns(true)`将意味着*对`Foo()`的所有*后续调用都将返回`true`。`counterfeiter`耗材，除了`XXXReturns()`方法，还有一个`XXXReturnsOnCall()`方法。如果您知道您在要测试的函数中调用了`Foo()`三次，并且您希望它第一次返回`true`，第二次返回`false`，第三次返回`true`，您可以这样做:

```
FooReturnsOnCall(0, true)
FooReturnsOnCall(1, false)
FooReturnsOnCall(2, true)
```

如果`Foo()`被调用了十次，并且您只希望第七次调用返回`false`，但是其他所有调用都返回 true，您可以这样做:

```
FooReturns(true)
FooReturnsOnCall(6, false)
```

如果您想模仿函数做一些事情，而不仅仅是简单地返回一个硬编码的值，那么您可以将`XXXStub`变量设置为您希望在调用`XXX()`方法时执行的函数体——当然，签名必须与原始的`XXX()`方法的签名相匹配。

# 在服务器中添加

在这一点上，我们已经看到，从客户端模拟 http 行为可以非常非常简单，并且在执行和设置方面比使用测试服务器要轻一些。但有时你真的需要一个测试服务器，所以接下来我们将看看这方面的一些情况。

## 也许我们所需要的只是一个操纵者…

好吧，我撒谎了。我们还不打算建立一个完全成熟的测试 HTTP 服务器。如果您正在编写服务器端 HTTP 代码，您通常希望将处理程序与路由(mux)代码分开(这既是出于模块化的一般原因，也是因为您有多个路径调用同一个处理程序)。所以让我们从测试一些处理程序开始。没有服务器我们怎么做呢？

测试一个处理器不需要使用`counterfeiter`——但是我们将使用标准的`httptest`包。Go 中的 HTTP 处理函数通常有一个签名`(http.ResponseWriter, *http.Request)`——虽然您当然可以使用不同的签名，并且仍然可以在您的处理函数中找到一种方法，但是没有什么好的理由偏离这种模式(这意味着您的处理函数可以与您选择的任何 mux 一起使用，无论是 Gorilla、Echo、Gin 还是 Go 已经有的内置 mux)。

让我们编写一个非常基本的处理程序——我们将检查单个必需的头(`X-Account`)，如果没有找到，则返回 400(错误请求)。我们还将检查请求方法——取决于您如何编写处理程序并将它们连接到 mux，这可能完全是多余的——一个好的模式是拥有特定于方法的处理程序，并通过在 mux 层中指定方法来路由到它们(大多数，但不是所有，mux 实现提供这种功能 OOTB——例如，默认的 Golang mux 不提供)。我们可能会得到这样的结果:

```
package serverimport (
  "net/http"
)const (
  HEADER_KEY_X_ACCOUNT = "X-Account"
)func HandleGET(w http.ResponseWriter, r *http.Request) {
  // Check our required header
  account := r.Header.Get(HEADER_KEY_X_ACCOUNT) if account == "" {
    w.WriteHeader(http.StatusBadRequest)
  } else {
    if r.Method != http.MethodGet {
      w.WriteHeader(http.StatusInternalServerError)
    } else {
      // Process request
      w.Write([]byte("Content!"))
    }
  }
}
```

测试这一点非常简单——我们只需要给它一个请求，然后检查响应……但是我们如何在运行实际服务器之外做到这一点(至少，我们如何在测试用例中轻松做到这一点)？

我们将利用`httptest`包中的两个函数— `httptest.NewRequest()`和`httptest.NewRecorder()`。第一个返回一个被填充的`*http.Request`，就好像它是由服务器路由的一样(与客户端生成的相反)。这意味着请求将把目标设置为`example.com`，除非所提供的 URL 是一个完全合格的 URL。第二个返回类型为`*httptest.ResponseRecorder`的实例——真实的服务器会传入一个用于将输出路由回客户机的实现，但是在我们的例子中，我们只想捕获请求处理的结果。所以这基本上是一个方便的实现。

下面是我们的测试文件此时的样子:

```
package server_testimport (
  "net/http"
  "net/http/httptest" . "github.com/onsi/ginkgo"
  . "github.com/onsi/gomega" "myapp/go-http-testing/examples/server"
)var _ = Describe("Server", func() {
  var (
    req     *http.Request
    handler http.Handler
    writer  *httptest.ResponseRecorder
  ) Describe("HandleGET", func() {
    BeforeEach(func() {
      handler = http.HandlerFunc(server.HandleGET)
      req = httptest.NewRequest(http.MethodGet, "/", nil)
      req.Header.Add(server.HEADER_KEY_X_ACCOUNT, "myaccount")
      writer = httptest.NewRecorder()
    }) It("Processes a GET request successfully", func() {
      handler.ServeHTTP(writer, req)
      Expect(writer.Code).To(Equal(http.StatusOK))
    }) Context("When we get the wrong method", func() {
      JustBeforeEach(func() {
        req.Method = http.MethodPost
      }) It("Returns an internal server error", func() {
        handler.ServeHTTP(writer, req)
        Expect(writer.Code).To(
          Equal(http.StatusInternalServerError))
      })
    })
  })
})
```

我们首先将我们的处理函数包装在一个`http.HandlerFunc`中，它添加了`http.Handler`接口所需的`ServeHTTP`方法。当然，我们可以直接调用我们的`server.HandleGET()`函数，但是包装会强制执行标准的调用签名(这在这里本身就是一个很好的实践)。`writer`变量是我们的`*httptest.ResponseRecorder`对象，在调用`ServeHTTP()`之后，`writer`保存处理结果——在本例中是我们在断言中检查的 HTTP 状态代码。

## 好，现在我们启动一个服务器

将我们的函数包装在`http.HandlerFunc`中还有另一个目的——创建一个测试服务器，我们需要传递一个`http.Handler`的实现，包装给了我们一个。

为什么我们要在这个时候启动服务器？我们已经有了我们认为是工作的处理程序，我们已经有了我们认为是工作的客户，我们都很好，对吗？嗯，确保 API 契约的双方实际上一起工作是一个好主意，这允许我们这样做。

在上面的测试文件中，我们在最后一个`})`之前添加了以下内容:

```
Describe("Server-side Request handling", func() {
  var (
    c   *client.MyApplicationClient
    srv *httptest.Server
  ) BeforeEach(func() {
    handler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
      Expect(r.URL.Path).To(Equal("/resource"))
      w.WriteHeader(http.StatusCreated)
    })
  }) JustBeforeEach(func() {
    srv = httptest.NewServer(handler)
    c = client.NewApplicationClient(srv.Client())
  }) AfterEach(func() {
    srv.CloseClientConnections()
    srv.Close()
  }) It("Checks the resource path", func() {
    c.PerformQuery()
  }) Context("Use a different handler", func() {
    BeforeEach(func() {
      handler = http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        Expect(r.Method).To(Equal(http.MethodPost))
        w.WriteHeader(http.StatusCreated)
      })
    }) It("Checks the request method", func() {
      c.PerformQuery()
    })
  })
})
```

注意我们对银杏测试设置/拆卸工具的使用——我们的第一个`BeforeEach()`调用设置了一个包含实际测试断言的默认处理程序。我们使用一个`JustBeforeEach()`来确保在测试之前执行的最后一个动作是启动服务器(作为参数传入我们的处理程序)。在我们的嵌套案例中(在内部`Context()`中)，我们使用另一个`BeforeEach()`来覆盖默认的处理程序——因为服务器本身是在一个`JustBeforeEach()`中启动的，我们保证服务器将在我们设置了我们的`handler`变量的值之后启动*,确保处理程序将测试正确的断言。*

我们还添加了对`AfterEach()`的使用——通常，测试服务器的拆除是在测试用例的清理阶段执行的。

# 整理想法

对于用 golang 测试 HTTP 代码来说，这是一组既不全面也不规范的例子。如果你在网上搜索，你肯定会看到不同的模式，包括一些使用我在这里使用的相同工具的模式。还有许多可能的排列我没有在这里演示(例如，测试你的多路复用/路由层)。我也没有试图深入研究银杏和(尤其是)Gomega 所提供的东西，只是解释我的例子所需要的东西——两者都有很多很多更多的特性，值得作为通用 golang BDD/TDD 框架来研究。

此外，正如我在上面提到的，我没有演示 Gomega 的`ghttp`包——也许这是以后文章的主题。

但是我希望这将被证明是有用的，特别是对于刚接触 golang 的开发人员和/或不知道如何开始测试 HTTP 代码的 go 开发人员。