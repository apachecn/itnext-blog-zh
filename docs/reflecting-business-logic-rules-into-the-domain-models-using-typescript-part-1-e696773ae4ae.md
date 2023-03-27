# 使用 typescript 将业务逻辑规则反映到域模型中—第 1 部分

> 原文：<https://itnext.io/reflecting-business-logic-rules-into-the-domain-models-using-typescript-part-1-e696773ae4ae?source=collection_archive---------2----------------------->

![](img/a86e0f73d52fa2b9de709cf5d33b222f.png)

Typescript 是一个强大的类型检查工具，但是如果您没有根据业务逻辑规则定义您的类型，它可能会让您感到困扰。

如果您的模型没有反映业务逻辑规则，过一段时间后，它将在业务逻辑所说的内容和您的代码行为之间产生差距。

我们用一个例子来解释这个。

我们需要定义一个存储`Person`数据的模型。规则和约束是:

*   一个人的名字必须限制在 50 个字符以内
*   一个人的姓氏长度必须限制在 50 以内
*   一个人必须有联系信息。联系信息可以是电子邮件或邮政编码，或者两者都有。

把`Person`模型做成这样似乎很合理。

```
type Person = {
     firstName: string
     lastName: string
     email?: string
     postalCode?: string
}
```

但是这种建模存在一些问题。

*   `firstName`和`lastName`不是任何字符串。他们不能有任何长度，他们应该有一个有限的长度为 50。
*   `email`不是任何字符串。它应该有一些特定的形状(有@字符和域名)
*   `postalCode`不是任何字符串。它应该包含 10 个数字。

我们如何将这些规则放到我们的模型中呢？

# 受歧视的工会

根据文件显示

> 您可以组合单例类型、联合类型、类型保护和类型别名来构建一个高级模式，称为*区分联合*，也称为*标记联合*或*代数数据类型*。

[https://www . typescriptlang . org/docs/handbook/advanced-types . html # discribed-unions](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions)

让我们先解决`firstName`和`lastName`的问题。
我们需要一个`string`应该有一个 50 有限的长度。所以我们创造了一种新的类型，比如:

```
type String50 = {
    kind: "String50"
    value: string
}
```

然后我们可以这样重新定义我们的`Person`模型:

```
type Person = {
     firstName: String50
     lastName: String50
}
```

现在我们可以理解，一个人必须有名和姓，这两个名字的长度都应该限制在 50 以内。

现在让我们回到联系信息规则。
我们可以将`Email`和`PostalCode`定义如下:

```
type Email = {
    kind: "Email"
    value: string
}type PostalCode = {
    kind: "PostalCode"
    value: string
}
```

然后我们更新`Person`模型:

```
type Person = {
     firstName: String50
     lastName: String50
     email?: Email
     postalCode?: PostalCode
}
```

但是有一个问题。你看到了吗？根据业务逻辑，一个人必须有联系信息，联系信息可以是电子邮件或邮政编码，也可以是两者都有。但是在我们的模型中，一个人可能没有任何联系信息，因为他们都是可选的。

所以我们像这样改变我们的模型:

```
type Person = {
     firstName: String50
     lastName: String50
     email: Email
     postalCode: PostalCode
}
```

它也有一个问题。现在一个人必须同时拥有电子邮件和邮政编码！

我们应该如何解决这个问题？

# 工会类型

联合类型将多个类型组合成一个类型。这允许您将现有的类型添加到一起，以获得可能是其中之一的单个类型。

你可以在这里阅读更多关于 union type 的内容:
[https://www . typescriptlang . org/docs/handbook/advanced-types . html # union-types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#union-types)

对于我们的例子，我们可以这样定义`ContactInfo`类型:

```
type ContactInfo =
   | Email
   | PostalCode
```

它说`ContactInfo`可能是`Email`也可能是`PostalCode`

现在让我们再次重新定义我们的`Person`模型:

```
type Person = {
     firstName: String50
     lastName: String50
     contactInfo: ContactInfo
}type ContactInfo =
   | Email
   | PostalCode
```

让我们回顾一下我们的业务规则:

*   一个人的名字必须限制在 50 个字符以内
*   一个人的姓氏长度必须限制在 50 以内
*   一个人必须有联系信息。联系信息可以是电子邮件或邮政编码，或者两者都有。

最后一条规则有一些其他约束。你看到了吗？

我们说`ContactInfo`可能是`Email`或者`PostalCode`。如果一个人既有电子邮件又有邮政编码怎么办？

我们应该为这种情况创建一个新类型:

```
type EmailAndPostalCode = {
    email: Email
    postalCode: PostalCode
}
```

然后更新`ContactInfo`

```
type ContactInfo =
   | Email
   | PostalCode
   | EmailAndPostalCode
```

现在联系信息可以是电子邮件或邮政编码，或者两者都有。

让我们再次回顾一下业务逻辑规则，并用我们的类型来检查它们:

```
type Person = {
     firstName: String50
     lastName: String50
     contactInfo: ContactInfo
}type ContactInfo =
   | Email
   | PostalCode
   | EmailAndPostalCodetype EmailAndPostalCode = {
    email: Email
    postalCode: PostalCode
}
```

*   一个人的名字必须限制在 50 个字符以内
*   一个人的姓氏长度必须限制在 50 以内
*   一个人必须有联系信息。联系信息可以是电子邮件或邮政编码，或者两者都有。

正如你现在看到的,`Person`模型反映了所有的业务逻辑规则，它是非常可读和干净的代码。
没有任何额外的文档或注释或任何`if else`语句，它显示了业务逻辑规则，并且是自文档化的。

在这一集里，我们学习了如何使用有区别的联合和联合类型来创建可读和自记录的类型。
但他们只是类型。他们什么也不做。

在下一集，我们尝试创建一些代码来创建和验证`Person`模型。

[继续下一篇。](https://medium.com/@mohsen.sareminia/reflecting-business-logic-rules-into-the-domain-models-using-typescript-part-2-61a19fba069d)