# 使用 typescript 将业务逻辑规则反映到域模型中—第 2 部分

> 原文：<https://itnext.io/reflecting-business-logic-rules-into-the-domain-models-using-typescript-part-2-61a19fba069d?source=collection_archive---------3----------------------->

![](img/a86e0f73d52fa2b9de709cf5d33b222f.png)

在之前的文章中，我们学习了如何创建自文档化的类型和模型。但是它们只是类型，没有任何功能。

在本文中，我们将编写一些代码来创建和验证这些类型。

在本文的最后，我们的类型可以用于现实生活中的项目。

项目源代码可从以下网址获得:

[](https://github.com/mohsensaremi/reflecting-business-logic-rules-into-the-domain-models) [## mohsensaremi/将业务逻辑规则反映到领域模型中

### 为 mohsensaremi/reflecting-business-logic-rules-into-domain-models 开发作出贡献，方法是创建一个关于…的帐户

github.com](https://github.com/mohsensaremi/reflecting-business-logic-rules-into-the-domain-models) 

# 引导程序

首先，我们需要一个 typescript 项目。所以让我们来创造它。

```
yarn init -y
yarn add --dev typescript ts-node
./node_modules/.bin/tsc --init
```

然后我们创造我们的类型。

# 字符串 50

在顶部，有类型声明。之后就是`makeString50`功能了。这个函数接受`any`参数并试图创建`String50`。

它检查参数类型是否是一个`string`并且它的长度是否小于 50。

如果参数不满足规则和约束，它将抛出一个`CustomTypeError`异常。

这里是`CustomTypeError`

这只是一个简单的显示错误的类。

我们在`try catch`块中使用它来处理我们的自定义类型错误。

让我们继续定义其他类型。

# 电子邮件

我们使用一个正则表达式来验证该参数确实是一封电子邮件。

# 邮政编码

就像那封邮件一样。使用正则表达式验证。

# 电子邮件和邮政编码

首先，我们检查是否提供了参数。然后我们使用`Email`和`PostalCode`类型来进行验证。

# ContactInfo

`ContactInfo`类型是一种联合类型。可能是`Email`或`PostalCode`或两者都是。

在`makeContactInfo`函数中，我们首先尝试创建一个`Email`，然后尝试创建一个`PostalCode`，最后尝试创建`EmailAndPostalCode`。

如果它们都失败了，那么异常将抛出。

现在我们有了所有想要的类型，可以创建`Person`模型了。

# 人

在`makePerson`函数中，我们需要做的就是调用类型生成器函数。

如果它们都返回一个有效的值，我们就创建了我们的人模型。

让我们使用`makePerson`功能创建人员，并测试我们创建的内容。

`createAndLogPerson`函数只尝试创建一个人并打印其值。如果失败，它将打印错误。

让我们看看输出:

*   人员 1 有效，并有联系信息的电子邮件
*   人员 2 有效，并且有联系信息的邮政编码
*   人员 3 是有效的，并且具有用于联系信息的电子邮件和邮政编码
*   人员 4 无效，因为它没有任何联系信息。所以它会抛出一个错误
*   人员 5 无效，因为它有联系信息的电子邮件，但它不是正确的电子邮件格式。所以它会抛出一个错误

在本文中，我们让我们的类型实际工作并验证输入数据。

但是正如你所看到的，这是一个很无聊的工作。

首先，我们需要声明类型，然后我们应该编写一些函数来创建和验证它。这种类型的模型结构只在我们的领域和项目中有意义。如果我们需要将这些数据发送给其他人。例如，如果我们需要将这些数据作为 rest API 调用的响应返回，该怎么办？

我们需要另一个函数将`Person`作为参数，并返回一个正常的结构，如下所示:

```
{
    "firstName": "pf1",
    "lastName": "pl1",
    "email": "pf1@gmail.com",
    "postalCode": "3483848392",
}
```

我们需要定义类似于`encodePerson`函数的东西。

正如你所做的，我们必须为每种类型创建`make...`和`encode...`函数。

我们需要一些方法来自动化这些功能。

幸运的是，有一个 typescript 库可以做到这一点。你可以在这里阅读:

[](https://github.com/gcanti/io-ts) [## gcanti/io-ts

### 为了安装稳定版，发布了实验性特性(*)，以便从…获得早期反馈

github.com](https://github.com/gcanti/io-ts) 

在下一篇文章中，我们将使用`io-ts`库来创建和验证我们的模型。

[继续下一篇。](https://medium.com/@mohsen.sareminia/reflecting-business-logic-rules-into-the-domain-models-using-typescript-part-3-aa8998bc6d29)