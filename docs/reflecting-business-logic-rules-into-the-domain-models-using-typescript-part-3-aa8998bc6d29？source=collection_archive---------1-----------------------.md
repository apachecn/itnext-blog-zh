# 使用 typescript 将业务逻辑规则反映到域模型中—第 3 部分

> 原文：<https://itnext.io/reflecting-business-logic-rules-into-the-domain-models-using-typescript-part-3-aa8998bc6d29?source=collection_archive---------1----------------------->

![](img/a86e0f73d52fa2b9de709cf5d33b222f.png)

在上一篇文章中，我们声明了类型并创建了一个函数来创建和验证类型。

但是如果我们想要为我们想要使用的每个模型创建这些函数和验证器，这将花费太多的时间。

在本文中，我们将使用`io-ts`库来自动化这些枯燥的任务。

[](https://github.com/gcanti/io-ts) [## gcanti/io-ts

### 为了安装稳定版，发布了实验性特性(*)，以便从…获得早期反馈

github.com](https://github.com/gcanti/io-ts) 

期末专题可以在这里找到:

[](https://github.com/mohsensaremi/reflecting-business-logic-rules-into-the-domain-models/tree/using-io-ts) [## mohsensaremi/将业务逻辑规则反映到领域模型中

### 为 mohsensaremi/reflecting-business-logic-rules-into-domain-models 开发作出贡献，方法是创建一个关于…的帐户

github.com](https://github.com/mohsensaremi/reflecting-business-logic-rules-into-the-domain-models/tree/using-io-ts) 

# 装置

```
yarn add io-ts fp-ts
```

那么什么是`fp-ts`？

`fp-ts`是 typescript 的函数式编程工具，是`io-ts`的对等依赖。

在本文中，我们不打算讨论函数式编程。

我们只讨论一点关于单子。

# 多媒体数字信号编解码器

`io-ts`用一种叫做编解码器的东西来代表我们的类型。

编解码器具有`decode`和`encode`功能。

我们使用`decode`函数创建并验证输入数据，并使用`encode`函数将数据恢复正常。

编解码器看起来是这样的:

首先，我们来谈谈泛型类型。

## 通用类型 A

这就是我们正在建模的。例如在`String50`类型中，`A`代表:

```
{
     kind: "String50",
     value: string,
}
```

## 普通 O 型

这是输出类型。这表明当我们想要将数据传递给其他人时，数据应该是什么样子。

例如，我们需要返回`String50`作为 API 响应。我们应该转换这种类型:

```
{
     kind: "String50",
     value: string,
}
```

来点正常的`string`。

所以这个例子中的输出类型是`string`。

## 通用类型 I

这是输入类型。它指明了我们的类型可以验证什么样的数据。

通常是`unknown`型。因为我们想从每一个可能的数据中制作和验证我们的类型。我们不限制输入数据。

现在我们知道了泛型类型。先说构造函数。

要创建一个编解码器，我们需要 4 个参数。

## 名字

编解码器的名称

## 类型-高斯函数

这个函数接收未知的输入，并评估输入是否是类型`A`(记住`A`是我们的建模类型，就像`String50`)

## 验证功能

这个函数就像上一篇文章中的`make...`函数。它接收一个类型为`I`(输入类型通常为`unknown`)的参数，并尝试验证和创建该类型。

如果输入有效，它将返回`t.success`。否则将返回`t.failiur`。

## 编码功能

该函数接收类型为`A`(我们的建模类型)的参数，并返回类型为`O`(编解码器的输出类型)的参数。

例如，它将接收一个`String50`并返回一个正常的`string`。

现在让我们使用 codec 逐个定义我们的基本类型。

# String50Codec 编解码器

通用类型:

*   `A` = > `String50`
*   `O` = > `string`
*   `I` = > `unknown`

type-guard 函数检查输入值的形状，类似于`String50`。

验证函数检查输入是否为字符串，最大长度为 50。这个函数是我们检查业务逻辑规则的地方。
如果输入有效，则返回成功，否则返回失败。

编码函数应该返回一个`string`。所以只需要归还`String50`的`value`财产。

再来说说验证函数。

在上一篇文章中，我们使用了`CustomTypeError`异常来表示故障状态。
但是这里我们返回`t.success`和`t.failure`。这些函数是实际值的包装器。

当我们尝试使用编解码器的`decode`函数来验证输入时，它不会为无效输入抛出错误。而是会返回一个`Either`。

什么是要么？

`Either`是一个函数式编程单子。我们在这里不讨论函数式编程。

把一个`Either`想象成一个围绕值的盒子。它有两种状态。`left`和`right`状态。我们用`Either.left`表示失败状态，用`Eihter.right`表示成功状态。

就当`right`是成功状态，`left`是失败状态吧。

因此，如果输入有效，验证函数返回`right`，如果输入无效，则返回`left`。

我们将定义下一个编解码器。

# EmailCodec

# 后酒精分解

# EmailAndPostalCodeCodec

这里我们使用了来自`io-ts`的`t.type`来创建`EmailAndPostalCodeCodec`。

它组成`EmailCodec`和`PostalCodeCodec`来创建新的编解码器。

现在是定义`PersonCodec`的时候了。

# 个人编解码器

在上一篇文章中，我们将`ContactInfo`定义为:

```
ContactInfo =
    | Email
    | PostalCode
    | EmailAndPostalCode
```

它是一个联合类型，并且`io-ts`有一个将这种类型转换成编解码器的功能。它叫`t.union`。

`t.union`接收编解码器数组并返回联合类型编解码器。

现在我们的`PersonCodec`完成了。让我们用这些编解码器进行测试。

如你所见，我们移除了`try catch`块。

相反，我们有一个从`decode`函数返回的`personEither`。
它是一个`Either`单子，可以是`right`(表示成功)或`left`(表示失败)

我们使用`fp-ts/Either`库中的`isRight`函数检查它是否正确。

记得我说过`Either`就像一个围绕实际值的盒子。因此，为了获得实际的个人价值，我们使用`personEither.right`来获得个人。

我们打印了个人价值和个人编码价值。

`PathReporter`是一个显示错误细节的实用函数。

让我们看看输出:

您会看到编码后的值就像一个可以传递给其他人的普通值(例如，作为 API 响应返回)。

而`PathReporter`显示了更详细的错误。您可以找到导致错误的值和字段。

这是将业务逻辑规则反映到领域模型中的最后一部分。

我希望你能从这些文章中学到一些东西，写出更干净的、自文档化的代码。

如果你对编写干净且人类可读的代码感兴趣，我强烈推荐你学习函数式编程，并在日常项目中使用`fp-ts`库。