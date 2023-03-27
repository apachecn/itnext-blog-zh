# 如何创建用于测试一次性密码服务的存根？

> 原文：<https://itnext.io/how-to-create-a-stub-for-testing-a-one-time-password-service-68e0de6f6641?source=collection_archive---------2----------------------->

![](img/a058df3c8b0c0a900e4a9d9964bce420.png)

由[约书亚·阿拉贡](https://unsplash.com/@goshua13?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/@max_duz?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

# 存根与模拟—有什么区别？

**存根**

我相信最大的区别是你已经用预定的行为写了一个存根。因此，您将有一个实现依赖关系的类(最有可能是抽象类或接口),这是您为了测试目的而伪造的，而这些方法只是用 set responses 来清除。他们不会做任何花哨的事情，并且您已经在测试之外为它编写了存根代码。

**嘲弄**

模拟是你测试的一部分，你必须根据你的期望来设置。模拟不是以预先确定的方式建立的，所以您可以在测试中用代码来完成。模拟在某种程度上是在运行时确定的，因为设置期望的代码必须在它们做任何事情之前运行。

**模拟和存根之间的差异**

用模拟编写的测试通常遵循:

`initialize -> set expectations -> exercise -> verify`图案要测试。而预写的存根将跟随一个`initialize -> exercise -> verify`。

**仿制品和存根之间的相似性。**

两者的目的都是为了消除对一个类或函数的所有依赖的测试，这样你的测试就能更集中和更简单地证明它们所要证明的。

唱存根而不是 web 服务。

![](img/5354cea4a01a8879cdb06b1d52dd7bee.png)

既然我们在谈论测试，很可能我们有所有的可能性，不仅仅是以嗅探器的形式放置网格，还可以完全取代其中一个服务。使用这种方法，人工服务将获得“频繁的”请求，并且我们将能够控制消息目的地的行为。为此，有一个很棒的 WireMock 库。你可以在项目的 [GitHub 页面](https://github.com/tomakehurst/wiremock)查看它的代码。底线是，这是一个具有良好 REST API 的 web 服务，几乎可以以任何方式进行配置。这是一个 JAVA 库，但它能够作为一个独立的应用程序运行，可以在 JRE 中使用。然后是带有详细文档的简单设置。

T21:我们怎么做呢？

```
testing_service -> service_stub : (its log): {message}.
```

**所以我们需要做几件事情:**

启动存根服务，并通过响应 OK 来强制它接受某些消息。这将由 WireMock 完成。

确保消息被传递到存根服务。

失败已经发生的事情。有两种选择——使用 WireMock 工具进行验证，或者从中获取请求列表，对它们应用匹配器。

# 入门指南

如何手动提升服务在 Wiremock 网站的[独立运行](http://wiremock.org/docs/getting-started/)部分有详细描述。

创建 JUnit 规则，该规则将在测试开始时引发所需端口上的服务，并在测试结束后结束:

```
@Rule
**public** WireMockRule **wiremock** = **new** WireMockRule(8091);
```

测试看起来会像这样:

```
@Test
**public void** shouldSend3Callbacks() **throws** Exception {
    stubFor(any(urlMatching(**".*"**)).willReturn(aResponse()
            .withStatus(HttpStatus.OK_200).withBody(**"OK"**)));
```

让我们的存根接受任何消息。

# 作为独立进程运行

WireMock 服务器可以在自己的进程中运行，并通过 Java API、JSON over HTTP 或 JSON 文件进行配置。

一旦[下载了独立的 JAR](https://repo1.maven.org/maven2/com/github/tomakehurst/wiremock-standalone/2.26.3/wiremock-standalone-2.26.3.jar) ,您就可以简单地这样运行它:

```
$ java -jar wiremock-standalone-"number of last version".jar --port 8091
```

# 命令行选项

可以选择在命令行上指定以下内容:

`--port`:设置 HTTP 端口号，如`--port 9999`。使用`--port 0`来动态确定端口。

# JSON 文件配置

您还可以通过文件使用 JSON API。当 WireMock 服务器启动时，它在当前目录下创建两个目录:`mappings`和`__files`。

要通过这种方法创建如上所示的存根，将一个扩展名为`.json`的文件放在`mappings`下，内容如下:

```
{
  "request" : {
    "urlPath" : "/get-code",
    "method" : "GET",
    "queryParameters" : {
      "token" : {
        "matches" : "access_token=([^&]*)"
      },
      "phone" : {
        "matches" : "^((\+7|7|8)+([0-9]){10})$"
      }
    }
  },
    "response": {
        "status": 200,
        "headers": {
            "Content-Type": "application/json; charset=utf-8"
        },
        "jsonBody": {**"code"**:**"1234"}**
    }
}
```

重新启动服务器后，您应该能够执行以下操作:

```
$ curl [http://localhost:8091/get-code](http://localhost:8080/api/mytest)/
```

# 停工

要关闭服务器，向`http://<host>:<port>/__admin/shutdown`发送一个带有空主体的请求。

# JUnit 4.x 规则

JUnit 规则提供了一种在测试用例中包含 WireMock 的便捷方式。它为您处理生命周期，在每个测试方法之前启动服务器，之后停止。

```
@Rule
**public** WireMockRule **wireMockRule** = **new** WireMockRule();
```

此外，规则的构造函数可以接受一个 Options 实例来覆盖各种设置。

```
@Rule
**public** WireMockRule **wireMockRule** = **new** WireMockRule(*options*().port(8091));
```

实现代码可能如下所示:

```
*/**
 * The type Base api.
 */* **public class** BaseApi {

    */**
     * Default constructor.
     */* **public** BaseApi() {
        **super**();
        *//empty* **return**;
    }

    @Rule
    **public** WireMockRule **wireMockRule** = **new** WireMockRule(*options*().port(8091));

    */**
     * Before test.
     */* @BeforeTest(alwaysRun = **true**)
    **public void** beforeTest() {
        **wireMockRule**.start();

        **final** MockApiService mockApiService = **new** MockApiService();
        mockApiService.startMockService();

    }

    */**
     * After test.
     */* @AfterTest
    **public void** afterTest() {
        **wireMockRule**.stop();
    }
}
```

# Java(非 JUnit)用法

如果您想在 JUnit 之外使用来自 Java(或任何其他 JVM 语言)的 WireMock，您可以通过编程创建、启动和停止服务器:

```
*/**
 * The type Base api.
 */* **public class** BaseApi {

    */**
     * Value mWireMockServer.
     */* **private** WireMockServer **mWireMockServer**;

    */**
     * Value HTTP_PORT.
     */* **public static final int *HTTP_PORT*** = 8091;

    */**
     * Default constructor.
     */* **public** BaseApi() {
        **super**();
        *//empty* **return**;
    }

    */**
     * Before test.
     */* @BeforeTest(alwaysRun = **true**)
    **public void** beforeTest() {
        **mWireMockServer** = **new** WireMockServer(*options*().port(***HTTP_PORT***));
        **mWireMockServer**.start();

        **final** MockApiService mockApiService = **new** MockApiService();
        mockApiService.startMockService();

    }

    */**
     * After test.
     */* @AfterTest
    **public void** afterTest() {
        **mWireMockServer**.stop();
    }
}
```

**基本存根** 下面的代码将配置一个状态为 200 的响应，在相对 URL 与/get-code 完全匹配时返回(包括查询参数 token 和 phone)。响应的主体将是“code ”,并且将发送一个带有值 text-plain 的内容类型头。

```
*/**
 * The type Mock service.
 */* **public class** MockApiService {

    */**
     * The constant SUCCESS.
     */* **private static final int *SUCCESS*** = 200;

    */**
     * Default constructor.
     */* **public** MockApiService() {
        **super**();
        *//empty* **return**;
    }

    */**
     * Start mock service.
     */* **public void** startMockService() {
        *configureFor*(**"localhost"**, ***HTTP_PORT***);
        *stubFor*(*get*(*urlPathMatching*(**"/get-code"**))
                .withQueryParam(**"token"**, *matching*(**"access_token=([^&]*)"**))
                .withQueryParam(**"phone"**, *matching*(**"^((\\+7|7|8)+([0-9]){10})$"**))
                .willReturn(*aResponse*()
                        .withStatus(***SUCCESS***)
                        .withHeader(**"Content-Type"**, **"application/json"**)
                        .withBody(**"{\"code\":\"1234\"}"**)));

    }
}
```

本教程介绍了 WireMock 以及如何设置和配置这个库来测试 REST APIs。

这篇文章教会了我们两件事:

*   Stubbing 是一种允许我们配置 HTTP 响应的技术，当我们的 WireMock 服务器收到一个特定的 HTTP 请求时，就会返回这个响应。
*   创建获取一次性密码的服务。

在下一篇文章中，我们将学习如何使用 web UI 测试工具 Selenium WebDriver 来处理一次性密码服务