# GraphQL 最被低估的优势

> 原文：<https://itnext.io/the-most-understated-benefit-of-graphql-95bac3651403?source=collection_archive---------1----------------------->

![](img/15cfc3dcea3cc24fa1abe0444a1d221a.png)

我完全相信 GraphQL 是 API 架构中的下一件大事。看到一项技术解决了 REST 固有的一大堆问题，并且使用起来如此令人愉快，真是令人兴奋。GraphQL 的主要优势是众所周知的，但是有一个被低估的优势人们谈论得不够:*我们终于可以停止争论 URL 了！*

你曾经浪费时间和精力争论如何构造 REST URLs 吗？例如，如果我想从我的 API 中检索属于某个公司的所有导师，那么 GET URL 应该是什么？

`/companies/:id/mentors
/mentors?companyId=:id
/mentors?filters="companyId = :id"`

这个端点应该返回什么？它应该是一个结构与从`/mentors/:id`返回的对象完全相同的对象列表吗？还应该包含公司信息吗？如果我们只返回特定用例所需的信息，成本会是多少？这些都不是无关紧要的问题，我从未在一个不浪费时间讨论这些问题的组织工作过。用 URL 命名和构造复杂的关系是一件困难的事情！

GraphQL 解决了这个问题，它为开发人员提供了一个单一的端点来控制返回的数据，包括相关数据和内置过滤！命名根查询的方式与命名函数的方式非常相似。在 GraphQL 中，您的查询可能如下所示:

```
query {
    company(id: "abc123") {
        id
        name
        mentors {
            id
            name
        }
    }
}
```

这是 GraphQL 最大的好处吗？当然不是。作为一名久经沙场的 REST API 开发人员，这只是我喜欢的那些*真正*好的副作用之一。

感谢您的阅读，祝您编码愉快！🤙

# 跟我来。

如果你喜欢这篇文章，请关注我！或者至少给我一两下掌声。你能省下一点掌声，对吧？

**网址**:[https://sreisner . github . io](https://sreisner.github.io) **中**:[@ Shawn . web dev](https://medium.com/@shawn.webdev)
**Twitter**:[@ reisner Shawn](https://twitter.com/ReisnerShawn)

# 查看我的更多作品

[](/heres-why-mapping-a-constructed-array-doesn-t-work-in-javascript-f1195138615a) [## 这就是为什么映射一个构造的数组在 JavaScript 中不起作用

### 以及如何正确地去做

itnext.io](/heres-why-mapping-a-constructed-array-doesn-t-work-in-javascript-f1195138615a) [](/understanding-the-react-context-api-through-building-a-shared-snackbar-for-in-app-notifications-6c199446b80c) [## 学习 React Context API，并将其应用到您的应用中

### 为应用内通知构建共享素材 UI Snackbar

itnext.io](/understanding-the-react-context-api-through-building-a-shared-snackbar-for-in-app-notifications-6c199446b80c) [](/a-quick-practical-use-case-for-es6-generators-building-an-infinitely-repeating-array-49d74f555666) [## ES6 发电机的快速实用用例

### 构建无限重复的数组

itnext.io](/a-quick-practical-use-case-for-es6-generators-building-an-infinitely-repeating-array-49d74f555666)