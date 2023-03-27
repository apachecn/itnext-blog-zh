# 规范化还是反规范化，这是一个问题

> 原文：<https://itnext.io/to-normalize-or-to-denormalize-that-is-the-question-4cc3968439a8?source=collection_archive---------1----------------------->

数据是您最宝贵的资产之一。而且，因为数据非常有价值，所以在开始定义任何模式之前，花时间仔细考虑并提出尽可能多的问题非常重要。数据将如何被消费？多久更新一次？我们对最终的一致性满意吗？我们需要什么总量或报告？而且，多得多。如果你对学习如何在 NoSQL 建立关系数据模型感兴趣，你可能会对我以前的文章感兴趣 [NoSQL 不是指！关系型](/nosql-does-not-mean-relational-8aac79ce6b9c)。

![](img/3ba923ea6e7f67e9c45e47888bbdbe25.png)

图片由 unDraw.co 提供

我发现采用 [Cloud Firestore](https://cloud.google.com/firestore) 和其他 NoSQL 数据库的一个好处是，它们非常灵活，但永远不要忘记“强大的力量带来巨大的责任”——本叔叔(“斯坦·李”)。NoSQL 数据库通常被称为无模式数据库，你可以动态地修改数据以适应你的模型，但是 9/10 的这种想法会给你带来比你意识到的更多的痛苦。

到目前为止，我已经说了很多，没有太多的解释，所以让我们开始吧。我们将查看一个类似 CRM 的票证应用程序，试图了解我们何时应该规范化，何时应该反规范化。

首先，对于那些不知道的人来说，数据规范化是将数据组织到多个容器(例如，表)中以减少数据重复和提高数据完整性的技术。

```
**SQL Model****Organizations
**  - id
- name
  - address
  - city
  - state
  - postal_code
  - phone**Users
**  - id
- name
  - email
  - phone
  - organization_id**Tickets
**  - id
- submitter_name
  - note
  - assigned_to // REFERENCES App_User(id)
```

查看上面的 SQL 模型，您可以看到我们已经规范化了数据——我们使用`assigned_to`引用了`Tickets`中的`User`,通过`organization_id`引用了`User`中的`Organization`。

对于那些不知道什么是反规范化数据的人来说，反规范化数据是一种策略，用于以前规范化的数据库，通过复制数据来提高读取性能，代价是写入性能。

但这并不完全准确，反规范化数据也可以用于数据完整性。在我的[上一篇文章](/nosql-does-not-mean-relational-8aac79ce6b9c)中，我讨论了一个点餐应用程序，它有`dishes`、`orders`和`order_items`，一个`order`有多个`order_items`，一个`dish`有一个由`restaurant`设置的`price`。当 a `user`选择一个`dish`进行订购时，我们创建一个`order`和一个`order_item`。现在，如果我们通过一个外键引用`dish`,由于原料短缺，餐馆提高了`dish`的价格，我们的历史报告数据会发生什么情况？报告数据会是错误的！例如，顾客将无法从餐馆调出他们的订单历史，并看到他们之前为这道菜支付了 10 美元，但是如果我们从`dish`记录中反规格化并复制`price`呢？我们保留客户在订单时支付的`price`。

回到我们的 ticket 应用程序，让我们来看一个非规范化的模型，这次我将使用一个文档数据库模型。

```
**Document Model - Firestore
users {** user_id{
    name,
    email,
    phone,
    organization: {
      id: ${organization_id}
      name: ${organization.name}
    }
  }
**}
organizations {
**  organization_id {
    name,
    address,
    city,
    state,
    postalCode,
    phone,
  }

  **tickets (**sub-collection)
    id {
      submitted_name,
      note,
      assigned_to: {
        id: ${user_id},
        name: ${user.name}
      }
    }
**}**// users & organizations are collections
// tickets are a sub-collection under the organization
```

现在查看文档模型并将其与 SQL 模型进行比较，您会看到我已经将 **SQL** `Tickets`上的`assigned_to`数据规范化，但是在**文档** `tickets`中，我已经通过将用户的`name`直接包含在标签上而将`assigned_to`非规范化。公平地说，我们可以用 RDBMS 做完全相同的非规范化模式，这不是文档或 NoSQL 数据库所独有的。许多工程师会看到这一点，并说，等一下只需引用`user`并加入数据，这是一个逻辑解决方案，但它可能不是最佳解决方案。

在票证控制面板中，您通常会看到一些有限的数据—您通常会看到提交者的姓名、创建时间、分配给谁以及可能的最后更新时间戳。现在，假设分配到票证的员工结婚了，并将他们的姓名从`Jane Doe`更改为`Jane Smith`，如果使用规范化模型，登录并查看其票证的客户(“提交者”)可能会感到困惑，为什么当他们看到`Jane Smith`时`Jane Doe`不再是他们的`assigned`代表。在规范化模型中，您不能选择旧名称是否应该保留在票据上，或者新名称是否应该出现在票据上。现在，看看这个非规范化的模型，你会发现我们有一个选择，更好的是我们可以将这个选择提供给`Jane`或`organization`。我们可以让`Jane`或`organization`决定是用她的新名字更新她的所有票，还是只让新票使用她的新名字。

那么，如果 Jane 决定让所有的票都用她的新名字，我们该如何更新名字呢？在 SQL 中这很容易，让我们假设我们已经反规范化了上面的 SQL 模型，现在`Tickets`包括了`assigned_to_id`和`assigned_to_name`。我们的 SQL 查询如下所示

```
**UPDATE** Tickets 
**SET** assigned_to_name = $assigned_to_name 
**WHERE** assigned_to_id = $user_id;
```

大多数文档数据库要求您查询和更新每个文档。使用文档数据库的一个例子是

```
**Firestore Node.js (Admin-SDK)**// Function to update the ticket
// In the event of an error we push the data to pubsub
**async function updateTicket**({ticketId, data}){
  const docRef = firestore
    .doc(`organizations/{organizationId}/tickets/{ticketId}`); try {
    await docRef.update(data);
  } catch error {
    // handle your error
    logging(error);
    pubsub.add(JSON.stringify({ticketId, data});
  }
}**;***// retrieve all of the tickets for the user by the user id*
**const tickets** = await firestore
  .collection(`organizations/${organizationId}/tickets`)
  .where('assigned_to.id', '==', ${userId})
  .get();**const promises** = tickets.docs.map(doc => {
  // use our updateTicket function to write data
  return updateTicket({doc.id, assigned_to_name: $assigned_to_name})
});**await Promise.all(promises);** // This is only an example
```

正如您在上面看到的，文档数据库更新确实需要更多的工作。您很可能希望在后台执行此操作，云函数之类的东西对于此用例来说是一个很好的工具。

对于非规范化数据还有其他注意事项，您必须确保在更改外键/引用时更新非规范化数据。这意味着什么呢？如果我们决定将一张票从`Jane`重新分配给`John`，在一个规范化的数据库中，我们不需要担心将票上的名字更新为`John`我们将在`JOIN`数据时得到他的名字。但是，在一个非规范化的模型上，我们需要更新`assigned_to.id`和`assigned_to.name`。这没什么大不了的，因为您正在为新 id 更新文档/记录，您只需要确保记住也要更新 name 字段— *提示:这是一些测试的好地方。*

那么什么时候决定是否应该反规格化呢？我通常这样想，如果我的演示/视图需要显示数据，我会把它放在我的文档中。如果我的数据需要保留历史记录，我会将数据反规范化。然而，如果数据有经常变化的趋势，并且我们不需要保留历史值，我选择规范化模型。

en.wikipedia.org 反规格化[https://en . Wikipedia . org/wiki/反规格化#:~:text =反规格化%20is%20a%20strategy%20used，data % 20 or % 20 by % 20 group % 20 data](https://en.wikipedia.org/wiki/Denormalization#:~:text=Denormalization%20is%20a%20strategy%20used,data%20or%20by%20grouping%20data)。