# 如何在项目中使用 mock 和 stubs 进行测试。

> 原文：<https://itnext.io/how-to-use-mocks-and-stubs-for-testing-in-the-project-51d8c5c55e69?source=collection_archive---------0----------------------->

![](img/7592ea95678a75bc23bf6ac88ed7858e.png)

Zan 在 [Unsplash](https://unsplash.com/photos/yBEUD8SWABc) 上的照片

为了方便起见，现有测试用例的自动化测试覆盖和结构测试脚本应该类似于测试用例的结构——前提条件、步骤和后置条件。

得到每个自动化脚本应该有 3 个步骤的规则:

1.  前提条件(应用程序的初始化，准备数据，初始化测试数据)。
2.  步骤(测试)(立即测试)。
3.  后置条件(检查你想要的，删除的，在脚本执行期间创建的测试数据)。

本质上，这无非是**给了**，**当了**，**然后**的 BDD

目前，大多数工程师在自动化项目步骤(测试)上没有问题。所有人都学会了编写简单的自动化脚本。使用 Selenium WebDriver 来使用 click()和 sendKeys()方法。

类似于 step 步骤，大多数自动化工程师学会了执行检查(步骤后置条件)、assertEquals、assertTrue、assertThat 等。学习了如何检查哪些元素是可见的、可访问的、可点击的，以及包含正确的文本。如果有必要，我甚至学会了期待一段时间。

但不幸的是，大多数工程师在自动化的大问题上是以步骤为前提条件的。许多自动化工程师根本不知道如何在运行自动化脚本之前准备数据。

如果你在解决这些问题上没有答案，可以有把握地说这篇文章就是为你写的:

*   如何使用自动化脚本阅读电子邮件？
*   如何使用自动化脚本获取或发送短信。
*   如何传递无效的 SSL 证书？
*   如何关闭弹出窗口？

在我们深入了解仿真器之前，我可以说一件事。上面列表中描述的使用自动化脚本完成的所有操作都是不好的。因为自动化脚本和外部服务非常慢而且不稳定。

对于我自己，我制定了一些规则:

*   总是模拟外部服务。
*   仿真那么慢。
*   仿真一切不受控制的事物。

在自动化中，脚本不应该使用外部依赖或服务。因为它们不在你的控制之下，所以速度慢，不稳定。

在自动化脚本中正确地使用模拟器。因为它们速度快，稳定，可控。

平时和同事交流仿真器的话题，听到的都是不同的意见和分歧。

最受欢迎的模拟器是一个模拟器，它是不一样的。因为模拟器只是模拟。然而，在我们的生活中，有许多不断使用仿真器的例子。别介意它们不是真的。而在某些领域没有模拟器就不行。

例如:

*   仿真芯片。
*   拿着出气筒的拳击手。
*   代表测试汽车引擎。
*   飞机风洞

为什么这些模拟器的例子看起来并不奇怪？

答案只有一个。许多人习惯于做错事。

因为所有人都是如此。似乎唯一正确的方法。

如果我们这样做，没有什么是正确的:

*   夺了生产基地。
*   寻找我们需要的数据。如果数据完全存在于数据库中，他们就运行我们的自动化脚本。看起来没问题。
*   如果我们需要的数据不在数据库里怎么办？或者数据是。但它们与我们需要的略有不同。我们将无法执行我们的测试用例？

或者:

*   我们用的是真正的服务。
*   我们试图理解弹出窗口出现的逻辑。
*   我们试着猜测。在我们的自动化脚本中，我们插入循环，if 或 wait。相应的，这一切都是恐怖的，不稳定的。

熟悉的情况？

试着猜测和匹配数据是不正确的。以及使用模拟器的权利。

我会给你示例代码:

```
**public class** UseMock {**public static void** doSomething() {
        **try** {someList = restClient.loadData();} **catch** (IOException ex) {logger.error(**"Failed to load"**, ex);
            someList = *emptyList*();
        }}
}
```

试着回答这个问题:如何在一个真实的后端系统上测试这段代码？

我想答案是显而易见的。这段代码不可能在实时后端上测试。

这段代码中有一个错误:

```
} **catch** (IOException ex) {logger.error(**"Failed to load"**, ex);
            someList = *emptyList*();
        }
```

如果应用程序无法加载列表，例如，发生了 TimeOutException，或者服务器此时无法访问，列表返回为空。没有给出错误，只是返回一个空列表。

测试此类代码的问题是，要在我们需要的任何时候恢复活跃的后端超时异常，这是不可能的。

只有模拟器可以在任何给定的时间向我们返回我们需要的事件。

**一种你不能完全控制你的系统的情况，后端被称为非托管或托管测试环境。**

现在，我将举例说明如何使用仿真器处理这些代码:

```
RestClient **restClient** = mock(RestClient.**class**);when(**restClient**.get(anyString())).thenReturn(**new** IOException(**"Could not connect"**));
```

模仿者的本质是他们必须模仿所有我们需要的东西，并且在你需要的时候。

我们提前为考试做好准备。

W eb 服务。

每个应用程序都需要一个测试环境。自动化工程师必须能够在测试和生产环境中运行应用程序。

例如，带有 key prod 的 launch 应用程序将在实时生产系统上运行自动化脚本。并在测试环境中运行带有关键测试的应用程序。更多关于如何在本文[中分析的框架中实现这一功能的信息。](/how-to-run-automation-scripts-in-multiple-environments-abc39d11aa20)

为了实现 web 服务的模拟，有几个库用于:

1.  码头
2.  游戏框架
3.  弹簧靴。

```
**public class** UseMock {@BeforeClass
    **public static void** startServer() {Server server = **new** Server(8080);WebAppContext shop = **new** WebAppContext(**"webapp"**, **"/shop"**);
        server.setHandlers(shop);server.start();
        server.join();
    }@Test
    **public void** loginTest() {
        open(**"http://localhost:8080/shop"**);
    }}
```

TTP 服务。

一些库允许你为 HTTP 服务创建一个模拟器。

其中最流行的是 WireMock[http://wiremock.org/docs/getting-started/](http://wiremock.org/docs/getting-started/)

实现代码如下所示:

```
*/**
 * Method Start mock service.
 */* **public void** startMockService() {
    configureFor(**"localhost"**, HTTP_PORT);
    stubFor(get(urlPathMatching(**"/login"**))
            .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader(**"Content-Type"**, **"application/json"**)
                    .withBody(**"Success login."**)));}**private static final** String ***LOGIN*** = **"/login"**;@Test
**public void** testRequestInMinions() {
    **final** String resultApiJson = getResponseFromMock(***LOGIN***);
    *assertEquals*(resultApiJson, **"Success login."**);
}*/**
 * Get login data from mock.
 *
 ** ***@param url*** *the url.
 ** ***@return*** *the login from mock.
 */* **public static** String getResponseFromMock(**final** String url) {
    **final** String json =
            given()
                    .then()
                    .statusCode(200)
                    .log()
                    .all()
                    .when()
                    .get(url)
                    .getBody()
                    .asString();
    **return** json;
}
```

电子邮件。

绿色邮件【http://www.icegreen.com/greenmail/】T2
苏贝塔 SMTP[https://github.com/voodoodyne/subethasmtp](https://github.com/voodoodyne/subethasmtp)

这些库允许您在本地环境 SMTP 服务器上运行并管理它。

```
#Mail configuration
 mail.smtp = mock
 mail.smtp.host=localhost
 mail.smtp.port = 9010
 mail.smtp.user = admin
 mail.smtp.password = admin
```

实现代码如下所示:

```
**private** MailServerEmulator **mailServer** = **new** MailServerEmulator();@Test
**public void** loginWithEmail() {open();*// Find filed OTP code*String email = **mailServer**.getMessage().get(0);
    String otpCode = email.replaceFirst(**"Code: (.*)"**, **"$1"**);
}
```

从上面的示例代码可以看出，这是 SMTP 服务器的完整实现:

```
**public class** MailServerEmulator {
    **private** SMTPServer **smtpServer**;
    **private final** List<String> **messages** = **new** ArrayList<>();**public void** startMailServer() {**int** port = 9010;**smtpServer** = **new** SMTPServer(**new** SimpleMessageListenerAdapter(**new** SimpleMessageListener() {@Override
            **public boolean** accept(String from, String recipient) {
                **return true**;
            }@Override
            **public void** deliver(String from, String recipient, InputStream data) **throws** IOException {**try** {
                    MimeMessage mimeMessage = **new** MimeMessage(Session.getDefaultInstance(**new** Properties()), data);
                    **messages**.add((String) mimeMessage.getContent());
                } **catch** (MessagingException ex) {
                    **throw new** RuntimeException(**"Filed to read message from"**, +from + **" to "** + recipient, ex);
                }}}));**smtpServer**.setPort(port);
        **smtpServer**.start();
    }**public void** shutDownMailServer() {
        **if** (**smtpServer** != **null**) {
            **smtpServer**.stop();
        }
    }
}
```

数据库。

所有自动化脚本必须在由您控制的数据库中运行。理想情况下，这个数据库应该在内存中。当您在测试或生产数据库中运行自动化脚本时，您可能会得到不正确的结果。数据数据库中充满了各种你所知甚少的数据。该数据库必须填充某一数据集，这是必要的，只有你。你对数据有完全的控制权。

为了在内存中处理数据库，https://www.h2database.com/html/main.html 有一个很棒的库 H2

实现代码如下所示:

```
*/**
 * The constant BASE_DRIVER.
 */* **private static final** String ***BASE_DRIVER*** = **"org.h2.Driver"**;*/**
 * The constant BASE_CONN.
 */* **private static final** String ***BASE_CONN*** = **"jdbc:h2:tcp://localhost/~/test"**;*/**
 * The constant BASE_USER.
 */* **private static final** String ***BASE_USER*** = **"sa"**;*/**
 * The constant BASE_PASS.
 */* **private static final** String ***BASE_PASS*** = **"your password"**;*/**
 * Get db connection.
 *
 ** ***@return*** *the db connection
 */* **private static** Connection getDBConnection() {Connection h2DBConnection = **null**;**try** {
        Class.*forName*(***BASE_DRIVER***);
    } **catch** (ClassNotFoundException ex) {
        ***LOG***.info(ex.toString());
    }
    **try** {
        h2DBConnection = DriverManager.*getConnection*(***BASE_CONN***, ***BASE_USER***, ***BASE_PASS***);**return** h2DBConnection;
    } **catch** (SQLException ex) {
        ***LOG***.info(ex.toString());
    }**return** h2DBConnection;
}
```

创建该表的代码如下:

```
*/**
 * Create table.
 *
 ** ***@throws*** *SQLException for method.
 */* **public static void** createTable() **throws** SQLException {**final** String absPath = **new** File(**"src/main/resources/createData.sql"**).getAbsolutePath();**final** Connection connection = *getDBConnection*();**try** {
        connection.setAutoCommit(**false**);RunScript.*execute*(connection, **new** FileReader(absPath));connection.commit();} **catch** (SQLException | IOException ex) {
        ***LOG***.info(ex.toString());
    } **finally** {
        connection.close();
    }***LOG***.info(**"Successfully Created table."**);
}
```

和 createTable.sql

```
**drop table** IF **EXISTS** `WORKERS`;
**create TABLE** WORKERS(employeeid ***int*** auto_increment **primary key**,
              firstname ***varchar***(100), 
              lastname ***varchar***(100), 
              department ***varchar***(100),
              location ***varchar***(10));
```

成功完成测试需要一个非常重要的因素，即在每次测试前重置状态。

实现可能如下所示:

```
@Before
**public void** resetDatabase(){
    **if** (firstRun)
        api.executeSql(**"script drop to 'src/main/resources/createData.sql'"**);**else** api.executeSql(**"runscript from 'src/main/resources/createData.sql'"**);
}
```

启动测试容器中的数据库有一个库 test containers[https://github.com/testcontainers/testcontainers-java](https://github.com/testcontainers/testcontainers-java)

Test containers 是一个支持 JUnit 测试的 Java 库，提供了通用数据库、Selenium web 浏览器或其他任何可以在 Docker 容器中运行的轻量级、一次性实例。[https://testcontainers.org](https://testcontainers.org)

这个容器你可以添加任何类型的基地，你需要的。

例如 Oracle:

```
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>oracle-xe</artifactId>
    <version>1.12.1</version>
</dependency>
```

实现代码如下所示:

```
@Testcontainers
**class** MyTestcontainersTests {@Container
    **public** OracleContainer **oracleContainer** = **new** OracleContainer()
            .withDatabaseName(**"foo"**)
            .withUsername(**"foo"**)
            .withPassword(**"secret"**);@Test
    **public void** someTestMethod() {
        String url = **oracleContainer**.getJdbcUrl();
    }
}
```

我希望读完这篇文章后，我的关于模拟器的规则将成为你的规则。我想再次提醒你:

*   总是模拟外部服务。
*   仿真那么慢。
*   仿真一切不受控制的事物。