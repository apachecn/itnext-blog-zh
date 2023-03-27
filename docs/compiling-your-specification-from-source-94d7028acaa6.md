# 从源代码编译您的规范

> 原文：<https://itnext.io/compiling-your-specification-from-source-94d7028acaa6?source=collection_archive---------1----------------------->

![](img/22a9ca2a3c8d3d0218619282b744d49c.png)

规格，尤其是*开放* *标准*而不是一次性的，必须非常具体和详细。它们还必须清楚地展示出来，以便实施者有机会做对。

作为细节要求的结果，规范包含了许多细节，这些细节适用于某些上下文，而不适用于其他上下文，并且为了清楚起见，这些信息在上下文中重复出现。

这反过来给规范的所有者带来了维护上的麻烦。随着想法和需求的发展(正如他们应该*，*通过不断的社区反馈)，在保持内部一致性的同时更新文档变得非常具有挑战性。规格越大，问题越大，有些[规格非常非常大](https://www.loc.gov/preservation/digital/formats/fdd/fdd000428.shtml)，需要大量支持信息。

我抽象地谈论这一点，但是我想用一个具体的用例来说明下面的内容:SWORD 3.0 规范。SWORD 3.0 是一种协议，使客户端和服务器能够围绕复杂的数字对象进行通信，特别是在支持将这些对象存放到类似数字存储库的服务方面。我一直是这个规范的 2.0 和 3.0 版本的技术负责人，所以亲眼目睹了汇编文档的挑战。

在这篇文章中，我想分享一些经验和方法，我们采取了最大限度地提高剑 3.0 规格的准确性，同时尽量减少维护工作。

# 标准化

你能做的最重要的一件事就是尽可能地将你的规范中的信息规范化。这可以最大限度地减少原始资料中的重复，从而最大限度地提高规范的准确性。

但是打住，我们刚才不是说了为了帮助实现者，需要上下文中的重复吗？

当然，一旦我们使上下文正常化，我们将引入一些机制来产生最终的文档，它需要所有的重复，并且没有管理开销。

## 将专业知识以专业形式呈现

规范通常是用散文写的(尽管这是一种高度技术性的风格)，而实现是用代码写的。这给实现者造成了巨大的鸿沟。我们可以通过尽可能用代码编写我们的规范来跨越这个鸿沟。

在这种情况下，我的意思是您应该尽可能使用现有的标准来记录规范。在我们针对 SWORD 3.0 的具体案例中，我们使用了以下代码:

1.  [OpenAPI](https://swagger.io/specification/) 为规范定义的 REST API。
2.  [JSON 模式](http://json-schema.org/)为规范定义的所有 JSON 文档的结构

通过这样做，我们可以在可能需要这些文档的地方简化实现过程，并使那些工件 ***成为规范*** *的主要信息来源。*

## 制作可重复使用的示例

你应该写出可重用的例子，并在适当的上下文中展示给读者，而不是编写内联用法的例子。例如，如果您已经定义了 JSON 格式，那么在您的规范源代码树中，将它们的完整的具体例子存储为实际的`.json`文件。

## 将内容列表

规范内容的主要部分是定义、解释或扩展特定的术语或术语组合。所有这些都适合制表，我的意思是**把它放入电子表格**。我建议将您的工作表作为 CSV 文件存储在您的源代码树中，这样它们就可以被机器读取了。

例如，如果您有一组在整个文档中使用的 URL，请在如下表格中定义它们

```
| URL          | Definition                                      |
| ------------ | ----------------------------------------------- |
| Service-URL  | The root URL for the service                    |
| Resource-URL | The URL to be used to access a defined resource |
```

以机器可读的方式这样做的好处是，当我们将我们的规范编译成 HTML(这就是我们的方向)时，我们总是可以链接回这些定义，或者在我们讨论规范的另一个方面时在上下文中呈现它们，所有的*都不必实际重复内容本身。*

你也可以用这种列表方法做非常复杂的事情，你可以阅读我的帖子[关于我们如何为 SWORD 3.0 做这件事](https://medium.com/@richard.jones/normalising-requirements-in-multi-operation-protocol-specs-191136e22729)来获得一些想法。

# 定义您的输出

除了标准化内容之外，我们还需要决定我们想要产生什么样的输出。对于规范，这可能包括以下内容

*   规格说明:这将是你正式的、没有虚饰的、纯粹的规格说明定义。这不会完全正常化，毫无疑问会有一些重复，我们会尽量减少。
*   **附录和支持信息**:这可能包括以前的工作，如需求列表或用例，或背景阅读，或对规范领域结果的深入研究。
*   **实施指南**:大规格可能会很枯燥，让实施者不知道从哪里开始。附加文档，如实现指南、教程或工作示例，可以增加获得高质量实现的机会。
*   **演示和营销材料**:信不信由你，成功的规范(尤其是开放的规范)需要大量的营销。演示文稿、博客帖子、信息表等。

你很快就会明白，相同类型的内容会在不同的上下文中反复出现。

例如，如果您已经定义了一组用于与一组 API 端点交互的规则，并且您已经在主规范中抽象了这些规则，那么实现者可能不会立即清楚哪些规则适用于特定的*端点。在这种情况下，您可以在一个单独的文档中重用主规范中的相同内容，以绝对明确地说明如何实现每个端点。你可以在我的另一篇文章中看到关于这种方法的详细讨论，这里有一个简单的例子:*

假设我们有以下(简单的、人为的)规则:

1.  所有端点都必须进行身份验证
2.  发送正文内容的客户端必须提供`Content-Length`头
3.  发送 JSON 的客户端必须提供头`X-JSON-Type`(我们只是为了讨论才发明的头)
4.  发送二进制内容的客户端必须提供`Content-Type`头

然后，在我的支持信息中，我可以提供一个部分，内容如下:

> **向资源 URL 发送二进制内容**
> 
> 1.必须认证
> 
> 2.必须提供`Content-Length`标题
> 
> 3.必须提供`Content-Type`表头

重要的是，当我改变主规范中的一个规则时，支持文档也得到更新。这是我们规范化我们的规范内容，并将其转换成机器可读格式的强大动力，否则我们必须手动完成。

# 定义编译过程

现在我们知道了我们需要什么样的文档，并且我们在规范化的数据源中有了所有的基础信息，是时候开始将它们编译成最终的文档了。

这里我将简单描述一下我们的 SWORD 3.0 编译器。我们的编译器是为这个目的定制的，尽管它与静态站点生成器有很多相似之处，你也可以使用类似于 Jekyl 或 Hugo 的工具。我将在以后的文章中详细描述我们的编译器。

每个文档，例如主规范或实施者指南，都有自己的文本文件，其中包含所有的程序集信息。我们的是用 [Markdown](https://daringfireball.net/projects/markdown/syntax) 写的，摆脱了手工写 HTML 的需要。它们还包含自定义标签，告诉我们的编译器做什么。这些文档中的每一个都被输入到编译器中，编译器最终输出 HTML。

以下是需要采取的基本步骤:

1.  **读入输出文件**的主文件。在很大程度上，我们的主文件只是一个按适当顺序排列的部分列表
2.  **导入包含文件**。includes 也是包含规范文本的 Markdown 文件，以及告诉编译器从标准化的源数据生成什么文档的函数。
3.  **索引文档**。这意味着我们寻找所有的标题，并在它们前面加上合适的章节号(如`2.4.17`)，然后存储所有的信息，以便以后生成一个目录。
4.  **运行文档中的所有功能。**这将从标准化的源中生成所有文本。这包括如下操作:从 OpenAPI 规范中描述 API，或者包括定义表，或者在部分之间创建链接。
5.  **将文件渲染成 HTML** 。将结果文件从 Markdown 转换成 HTML，启用一些有用的扩展，比如那些覆盖表和代码块的扩展(这些不是基本 Markdown 语言的一部分)。

为了给你一个更具体的描述，下面是主规范文件的一个片段:

```
Last modified: {% date now,%Y-%m-%d %}
{% include sections/INTRODUCTION.md %}
{% include sections/TERMINOLOGY.md %}
{% include sections/STRUCTURE.md %}
{% include sections/HTTP_HEADERS.md %}
{% include sections/PROTOCOL_OPERATIONS.md %}
{% include sections/PROTOCOL_REQUIREMENTS.md %}
```

它基本上说:为构建打印一个时间戳，然后按顺序包含以下部分。

如果您深入研究这些文件，您会发现如下函数:

```
{% json_extract
    source=examples/service-document.json,
    keys=byReferenceDeposit
%}
```

或者:

```
{% http_exchange
    source=schemas/openapi.json,
    method=get,
    url=/Service-URL,
    response=200
%}
```

这些命令告诉编译器从我们的标准化源数据中代入一些信息。

例如，`json_extract`函数加载指定的 JSON 文件，并在引用的块中打印其中的一个或多个键。例如，您可以用它来解释一个特定的特性在文档中是如何表示的。该函数的上述用法用于描述“byReferenceDeposits”是如何在服务描述中公布的，并为我们提供:

```
{
    "byReferenceDeposit": true
}
```

这样做的好处是，如果我们选择更改示例文件中该字段的值，摘录中的值也会得到更新。

或者考虑一下`http_exchange`——它为我们提供了一个关于 OpenAPI 规范的视图，这是规范中常见的一种形式；它详细描述了客户机和服务器之间通过 HTTP 进行的在线交互。上面的函数调用将输出如下内容:

```
GET /Service-URL HTTP/1.1 
Authorization: ... 
On-Behalf-Of: ... HTTP/1.1 200 
Content-Type: application/json [Service Document]
```

同样，这是利用了标准化的数据，如果我们选择将 HTTP 头、主体内容或此交换的任何其他方面添加到 OpenAPI 定义中，这些将会反映在我们的示例中。

# 来自 SWORDv3 的示例

要看一些具体的例子，你可以看一下我们的 SWORD 规范 git 库(如果你这样管理你的规范，版本控制也很容易):

*   使用开放标准的标准化源数据的目录
*   一份[目录列表](https://github.com/swordapp/swordv3/tree/master/src/tables)来源数据
*   来自规范的文件类型的示例的[目录](https://github.com/swordapp/swordv3/tree/master/src/examples)
*   主规范[来源](https://raw.githubusercontent.com/swordapp/swordv3/master/src/MAIN.md)和[结果文档](https://swordapp.github.io/swordv3/swordv3.html)
*   详细说明每个可能的协议操作要求的实施者支持指南:[来源](https://raw.githubusercontent.com/swordapp/swordv3/master/src/BEHAVIOURS.md#)和[结果文档](https://swordapp.github.io/swordv3/swordv3-behaviours.html)
*   根据与主规范相同的标准化源材料构建的概述演示:[源](https://raw.githubusercontent.com/swordapp/swordv3/master/src/presentations/OVERVIEW.md#)和[结果演示](https://swordapp.github.io/swordv3/overview)

多年来，我参与了许多规范工作。不仅是剑 1 到 3，还有 [OAI 矿石](https://www.openarchives.org/ore/)，以及[资源同步](http://www.openarchives.org/rs/toc)(通过正式 [NISO](https://www.niso.org/) 程序)。更不用说许多特定于客户端的 API 定义了。在所有这些版本中，都需要管理大量的文档，而在 SWORD 3.0 中，我们有机会探索一种使这变得更容易的方法。

我希望这种方法可以帮助您完成这类工作，并且您会发现它不仅适用于开放标准，而且适用于技术规范的所有方面。

随着时间的推移，我们也许能够正式发布组装规范的软件。现在这是一个专门为 SWORD 3.0 编写的原型，尽管欢迎你来[看一看](https://github.com/swordapp/swordv3/blob/master/builder/builder.py)。

*理查德是软件开发咨询公司*[*Cottage Labs*](https://cottagelabs.com)*的创始人和高级合伙人，专门研究数据生命周期的各个方面。他偶尔会在推特上发*[*@ Richard _ d _ Jones*](https://twitter.com/richard_d_jones)