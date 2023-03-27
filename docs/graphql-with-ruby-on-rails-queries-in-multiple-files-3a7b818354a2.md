# 使用 Ruby On Rails 的 GraphQL:多个文件中的查询

> 原文：<https://itnext.io/graphql-with-ruby-on-rails-queries-in-multiple-files-3a7b818354a2?source=collection_archive---------1----------------------->

![](img/d70b9df02ee0268b6fa5f0cbbe563aae.png)

在上一集，我们添加了针对 GraphQL 突变的 RSpecs 测试。如果你需要更多关于这个话题的见解，你可以在这里查看文章。今天，我们将为 GraphQL 查询添加规范(同样，我们将只关注愉快的路径)。但是对于当前的查询实现，有一点我不喜欢。我们在一个文件中定义了一切:`app/graphql/types/query_type.rb`

您可以想象这个文件会随着时间的推移而增长。我想向您展示如何将逻辑分成许多更小的部分。

# 每个重构的第一步

没有合适的规范，很难进行重构。因此，我们将首先添加规格。但是为了使文章更简短，更容易理解，我们将采用我们公司非常重视的方法。我们将进行测试驱动的开发。这意味着我们将编写最终规格。因此，我们将为未来的实现添加规范，而不是在以前的实现中添加规范，在未来的实现中，每个查询都定义在一个单独的文件中。

我们希望有一个结构，所有的查询都在`app/graphql/queries/`文件夹内。因此，我们的规格将在`specs/graphql/queries/`目录中结束。

`specs/graphql/queries/author_spec.rb`

`specs/graphql/queries/authors_spec.rb`

`specs/graphql/queries/book_spec.rb`和`specs/graphql/queries/books_spec.rb`几乎相同，所以我不打算在这里粘贴代码。相反，如果你愿意，你可以检查 gist 上的文件: [book_spec.rb](https://gist.github.com/kayinrage/f668ba82b492058e47bc26dcf56fb5e2) 和 [books_spec.rb](https://gist.github.com/kayinrage/bc272ac8cffb54ec4c31425736f1d1b1)

正如你所看到的，我们的新规格看起来真的很像我们写的突变。我们有一个`query`方法，用于构建 GraphQL 查询，并检查查询结果是否符合我们的预期。

# 将查询分成单独的文件

现在，当所有需要的规范都准备好了，我们终于可以进行重构了。我不打算在本文中展示一个合适的 TDD，因为它需要一遍又一遍地执行这些操作:运行 spec，查看错误，修复错误，再次运行 spec 查看另一个错误，修复那个错误，等等。这很难理解，所以我将给出最终的解决方案，并解释其背后的想法。

我们希望有一个与突变相似的结构，但它不被支持，我们需要寻找替代方案。我们通常用来拆分查询的是`Resolvers`。

免责声明:并不总是推荐。有关更多信息，请查看文档。

正如我多次提到的，我们希望我们的`query_type.rb`像`mutation_type.rb`一样工作，这意味着`app/graphql/types/query_type.rb`的新版本应该是这样的:

我们去掉了所有的逻辑。相反，我们定义什么查询是可用的，以及该查询的解析器(一个我们可以找到它的地方)。

如果我们要遵循变异的做事方式，我们需要一个额外的文件:`app/graphql/queries/base_query.rb`

它现在是空的，但在未来的剧集中，我们会用它来做一些很酷的事情。

我们终于可以看到新查询的实现了:

`app/graphql/queries/author.rb`

`app/graphql/queries/authors.rb`

`app/graphql/queries/book.rb`

`app/graphql/queries/books.rb`

每个查询的代码都从`query_type.rb`中删除，没有任何改变。使之成为可能的神奇之处在于从`Queries::BaseQuery`继承而来，而后者又继承自`GraphQL::Schema::Resolver`。就是这样。

# 摘要

在这一点上，我们已经测试了所有的端点，我们(几乎)有了一个易于构建的结构。在接下来的几集中，我们将讨论分页、认证、授权等等。

*原载于*[*selleo.com*](https://selleo.com/blog/graphql-with-ruby-on-rails-queries-in-multiple-files)*。*