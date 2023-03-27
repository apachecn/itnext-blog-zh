# 科特林和杰克逊的无痛 JSON

> 原文：<https://itnext.io/painless-json-with-kotlin-and-jackson-fc6d66fbf94a?source=collection_archive---------5----------------------->

似乎许多提供 REST API 的后端最终被美化成代理，将 JSON 从一个地方转移到另一个地方。如果你想让这些后端尽可能简单，这一点尤其正确([微服务](https://microservices.io/patterns/microservices.html)有人知道吗？).因此，拥有好的工具来解析和生成 JSON 可以对保持代码整洁和紧凑产生很大的影响。我想说说我用[科特林](https://kotlinlang.org/)和[杰克逊](https://github.com/FasterXML/jackson)做这个的经历。

我记得在过去，用 Java 处理 JSON 非常痛苦，因为必须编写大量代码来映射对象。事实上，这也是我最初使用 Ruby 的原因。事情发生了很大的变化(向好的方面！)从此。如今使用 *Kotlin* 和 *Jackson* 你可以用最少的努力来处理 JSON。*杰克逊*是一个非常强大的图书馆，但是你很容易迷路。如果用它来解析一堆具有不同需求的类的例子，加上一些代码来把它集成到你的工作流中。在这种情况下，我将使用[跳靴](https://spring.io/)。

# 序列化/反序列化

我们将使用[数据类](https://kotlinlang.org/docs/reference/data-classes.html)来表示将从 JSON 转换成的实体。它们相当于使用 *Lombok* 中的 [@Value](https://projectlombok.org/features/Value) 注释，并具有该语言的一流支持。它们是不可改变的(耶！)，并且有开箱即用的`equals`和`toString`之类的便利方法。

您可以使用一个[对象映射器](https://fasterxml.github.io/jackson-databind/javadoc/2.7/com/fasterxml/jackson/databind/ObjectMapper.html)来进行解析，尽管这可以在 *SpringBoot* 中配置为大部分自动完成，我将在后面展示。我有一个`User`实体，它有两个字段，我想把它们转换成 JSON，然后再转换回来。

```
data class User(val id: String, val age: Int) fun User.toJson(): String = ObjectMapper().writeValueAsString(this) fun String.toUser(): User = ObjectMapper().readValue(this)
```

对于简单的情况，只要定义数据类就足够了，只要你有[右模块](https://github.com/FasterXML/jackson-module-kotlin)。不过，您可以在此基础上做一些额外的配置。其中许多可以用注释来控制，注释可以使代码更加紧凑。它们也可以让你的代码变得不可维护，所以不要滥用它们。

## 可空性

如果有些字段是可选的，您可以提供默认值以防它们不存在

```
data class User(val id: String = "")
```

或者允许它们成为`null`

```
data class User(val id: String?)
```

不做任何事情会导致解析失败，并出现一个异常，实际上我发现这是一件好事。

## 错认假频伪信号

如果您从使用不同属性名称的不同来源解析您的对象，但是仍然希望保持一个*规范的*表示，`@JsonAlias`是您的朋友。

```
data class User(   
  @JsonAlias("userId")   
  val id: String 
)
```

这将正确地解析如下内容

```
{   
  "userId": "123" 
}
```

## 忽略属性

也许您正在解析一个包含大量您不需要的字段的对象。如果你不打算在你的代码中使用它，你真的应该避免添加它们，因为这会使你更难理解什么是需要的，什么是不需要的。`@JsonIgnoreProperties`可用于此。

```
@JsonIgnoreProperties(ignoreUnknown = true) 
data class User(val id: String)
```

## 不同的表现

如果您的后端充当代理，您将从某个地方读取数据并将其传递给客户端。在这种情况下，您可能希望跳过序列化中的一些字段，以便为您的客户端提供他们需要的字段。您可以通过自定义访问来实现这一点。

```
data class User(   
  val id: String = "",   
  @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)   
  val age: Int
)
```

这个对象的序列化将不包含`age`，但是它在我们的代码中是可用的。然而，这种方法不能很好地扩展。如果您发现同一个实体有两种不同的表示，并且添加了大量的注释以便只使用一个类，那么最好将它分成两个不同的类，并为一个类提供一个方法。

这强调了重要的一点。你不需要对所有的事情都使用注释和隐式转换，也许在某些地方使用专用的转换器更具可读性，如果你想给这个过程附加一些逻辑的话就更是如此。

## 如果你想要更多…

这只是可能做到的事情的一小部分。您可以真正控制序列化/反序列化过程的每个方面。如果你想知道其他选择，看看这篇文章。

# 远离无类型字符串

在 JSON 中，您倾向于使用字符串来表示许多实体。任何 id 类型，比如用户 id，或者语言代码。我更喜欢在我的代码中将它们映射到专用类。我见过许多错误，其中使用了错误的实体，而编译器可以直接阻止这种错误。以一个`UserId`为例，我喜欢这样建模:

*   它应该是一个不可变的数据类
*   它不应该强制改变 JSON 的结构(即没有嵌套)
*   序列化/反序列化应该开箱即用

通过使用一个数据类，我们得到了一个代表这个实体的不可变对象。我们能做的相对较少。在这种情况下，我们甚至不想访问内部字段。我们将直接比较实例，如果我们需要获得一个字符串表示，我们将通过`toString`方法来完成。

序列化通过`@JsonValue`注释进行，在这里我们直接使用值。如果我们修改我们之前一直使用的`User`类，它看起来会像这样。

```
data class User(val id: UserId, val age: Int)
```

该类将被序列化为这个 JSON

```
{   
  "id": "123",   
  "age": 20 
}
```

这符合大多数客户端(特别是前端)对这种结构的预期，而不会牺牲后端的任何安全性。

反序列化会自动发生。然而，我喜欢定义一个静态构造函数(使用`@JvmStatic`和`@JsonCreator`注释),这样我就可以在生成实例之前处理输入。这有助于确保我们的模型处于一致的状态。

从 Kotlin 1.3 开始，引入了一个叫做[内联类](https://kotlinlang.org/docs/reference/inline-classes.html)的新概念，这可能更适合这个用例。从 16/06/19 开始，Jackson 在嵌套对象中很难正确地反序列化它，所以到目前为止我还不能用它替换我的数据类。Github 中有一个[未解决的问题需要关注。](https://github.com/FasterXML/jackson-module-kotlin/issues/199)

# SpringBoot 集成

这是拼图的最后一块。我们可以手动使用一个`ObjectMapper`并显式地转换事物。如果它自己发生，事情就简单多了。好消息是，除了添加 [jackson-module-kotlin](https://github.com/FasterXML/jackson-module-kotlin) 作为依赖项之外，这里没有太多事情要做:

```
implementation("com.fasterxml.jackson.module:jackson-module-kotlin:${jacksonVersion}")
```

如果你使用的都是最新版本，那就足够了。如果 spring 魔术不是自己发生的(spring 做了很多我不太懂的魔术)，你可以手动做。您可以使用一个`@Configuration`以便您的控制器可以自动映射到 JSON 和从 JSON 映射。

如果您向另一个服务发出 REST 请求，您可以构建一个自定义的`RestTemplate`来做同样的事情:

同样，所有这些都应该通过将库添加到类路径中来实现。如果由于某种原因不工作，使用这个作为后备。此外，该模板可以扩展为使用基本 url、接收环境变量(例如包括 keystore)或自动向请求添加某些头。

## PathVariables 不是 JSON

现在我们已经深入到自动化 JSON 映射中，我开始变得雄心勃勃。如上所述，我们不再使用普通字符串，而是使用适当的域类。假设你有一条像`GET /users/:userId`这样的路线。控制器应该是这样的:

如果您向这个路由发出请求，`userId`将被自动解析，但是我们的自定义`create`方法不会被调用，因为这是一个 URL，而不是 JSON。我们走到这一步并不是为了再次手动解析字符串。让我们通过使用自定义转换器来解决这个问题。

就是这样。现在我们可以确定，在请求流的任何一点上，这些讨厌的字符串都不会在我们的应用程序中流动。

# 摘要

Jackson 是一个非常强大的库，老实说，你可以用所有的注释来过度使用它。但是，如果您明智地使用它们，使用 JSON 会变得非常容易，同时在这个过程中保持很好的类型安全性。对于测试，这与使用 WireMock 的[记录的 API 配合得非常好。](https://hceris.com/recording-apis-with-wiremock/)

*原载于 2019 年 6 月 16 日*[*https://hceris.com*](https://hceris.com/painless-json-with-kotlin-and-jackson/)*。*