# DTO 的替代品

> 原文：<https://itnext.io/alternative-dto-cb6caf9f4ff8?source=collection_archive---------0----------------------->

![](img/5cde6a6869fd6a2691e35bb6d4bb60b8.png)

十多年前，我[写过](https://blog.frankel.ch/dto-in-anger)关于 DTO:

> *数据传输对象是在进程间传送数据的对象。使用它的动机是进程间的通信通常依靠远程接口来完成，其中每个调用都是一个昂贵的操作。因为每个调用的大部分开销都与客户端和服务器之间的往返时间有关，所以减少调用次数的一种方法是使用一个对象(d to ),该对象聚集本应由多次调用传输的数据，但只由一次调用提供服务。*
> 
> *—* [*百科*](https://en.wikipedia.org/wiki/Data_transfer_object)

我过去认为(现在仍然认为)这应该成为过去。然而，它的用法似乎仍然很普遍。

我不否认有一些有效的理由来转换数据。但是，除了传统的 DTO 流程，还有其他选择:

1.  从服务层返回一个业务对象。请注意，在我之前参与的项目中，我们直接将业务对象映射到从数据库中读取的实体。
2.  将业务对象转换为表示层中的 DTO
3.  从表示层返回 DTO

# 返回实体本身

当实体的属性是需要显示的属性的超集时，不需要聚合其他属性。将实体转化为 DTO 不仅是矫枉过正。它会影响性能。

在这种情况下，最好的方法是返回实体本身。

# JPA 投影

我们在特定的上下文中请求特定的数据。因此，当调用到达数据访问层时，所需数据的范围是完全已知的:执行适合该范围的 SQL 查询是有意义的。

为此，JPA 提供了预测。本质上，查询中的投影允许精确地选择想要的数据。这里有一个例子:给定一个`Person`实体类和一个`PersonDetails`常规类:

```
CriteriaQuery<PersonDetails> q = cb.createQuery(PersonDetails.class);
Root<Person> c = q.from(Person.class);
q.select(cb.construct(PersonDetails.class,
  c.get(Person_.firstName),
  c.get(Person_.lastName),
  c.get(Person_.birthdate)
));
```

# 杰克逊转炉

具体到 JSON，我们可以将提供正确数据的过程委托给序列化程序框架，*，例如* [Jackson](https://github.com/FasterXML/jackson) 。其背后的思想如下:主代码照常处理实体，在边缘，Jackson 转换器将其转换为所需的 JSON 结构。

如果需要更少的数据，那就太容易了。如果更多，那么转换器需要更多的依赖项来获取数据。当然，如果这些数据来自同一个数据存储，这不是很好，上面的替代方法更合适。如果没有，这是一个选择。

# GraphQL

最后但同样重要的是，可以返回完整的实体，让客户端决定哪些数据在其上下文中有意义。

GraphQL 就是围绕这个想法建立的:脸书创建了它，现在它已经完全开源了。它的主要优点是提供了一个规范，并在此基础上提供了许多特定语言的实现[。](http://graphql.github.io/code/#server-libraries)

> *你的 API 的查询语言*
> 
> GraphQL 是一种 API 查询语言，也是一种用现有数据完成这些查询的运行时语言。GraphQL 为 API 中的数据提供了完整且易于理解的描述，使客户能够准确地要求他们需要的东西，使 API 更容易随时间发展，并支持强大的开发工具。
> 
> *—* [*GraphQL 网站*](http://graphql.github.io)

# 结论

当业务模型和表示模型之间存在差距时，很容易回到古老的“模式”，如 DTO。然而，以上任何一种选择都可能更相关。

**更进一步:**

*   [将投影查询映射到 DTO 的最佳方式](https://vladmihalcea.com/the-best-way-to-map-a-projection-query-to-a-dto-with-jpa-and-hibernate/)
*   [实体或 dto——何时应使用哪种投影？](https://www.thoughts-on-java.org/entities-dtos-use-projection/)
*   [GraphQL](http://graphql.github.io/)

*原载于* [*一个 Java 极客*](https://blog.frankel.ch/alternatives-dto/)*2022 年 3 月 6 日*