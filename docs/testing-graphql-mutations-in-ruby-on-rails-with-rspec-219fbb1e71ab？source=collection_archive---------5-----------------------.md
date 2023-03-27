# 用 RSpec 测试 Ruby On Rails 中的 GraphQL 突变

> 原文：<https://itnext.io/testing-graphql-mutations-in-ruby-on-rails-with-rspec-219fbb1e71ab?source=collection_archive---------5----------------------->

![](img/a0570cd3bfb1a654d9e92c47ac7c0432.png)

在上一篇[文章](https://medium.com/@IreneuszSkrobis/graphql-mutations-in-ruby-on-rails-4f93f91b1f2d)中，我们在测试应用程序中添加了一些 GraphQL 变体。是时候为它们中的每一个创建自动化测试了。但是首先，我们需要建立 RSpec 来编写更好的规范。

# RSpec 和 FactoryBot

安装 Rspec 和 Factory Bot 是微不足道的。首先，你必须将这些宝石添加到你的`Gemfile`:

运行`bundle install`

然后跑`rails generate rspec:install`

我建议做的最后一件事是将这一行添加到`spec/rails_helper.rb`

多亏了这一行，我们将能够使用简化的语法通过工厂来构建和创建对象。这意味着我们将能够简单地编写`create(:book)`，而不是`FactoryBot.create(:book)`。

# 工厂

我提到过，由于 FactoryBot，我们将能够轻松地创建对象。但是为了有这种可能性，我们必须先定义那些工厂。我们需要创建其中的两个:

`specs/factories/author_factory.rb`

`specs/factories/book_factory.rb`

现在我们已经万事俱备，可以开始写规格了。但请注意，现在，我们将只涵盖幸福的道路。

# 创建图书规格

在创建对象的过程中，我们可以编写的最重要的断言是检查对象是否被创建以及它在数据库中的字段是否正确。但是当我们测试 GraphQL 端点时，我们也希望确保它返回正确的对象。了解了所有这些之后，让我们来看看测试的最终版本。按照 RSpec 约定，我们将把我们的文件保存在`specs`文件夹中，与原始文件在同一个树目录中。简而言之，我们将我们的规范保存在`specs/graphql/mutations/books/create_book_spec.rb`中

正如你所看到的，我们测试了`.resolve`方法，我们检查了我之前提到的东西，因此编写并返回了对象。有两个独立的规范，我们希望对它们使用相同的查询。因此，当我们传递对我们重要的参数时，我们定义了`query`函数。在这种情况下，我们只需要通过`author_id`。

# 更新手册规格

在`updateBook`规范中，我们将测试一个对象在操作后是否改变，以及是否返回正确的对象。代码将与前一个非常相似，我们将它放在`specs/graphql/mutations/books/update_book_spec.rb`

# 销毁书规格

最后一个规格将为`destroyBook`突变而写。我们将重点检查对象是否被移除，以及在操作完成后是否返回移除的对象。正如您可能猜到的，我们将代码保存在`specs/graphql/mutations/books/destroy_book_spec.rb`中

# 摘要

为 GraphQL 变体编写规格说明与为控制器的动作编写规格说明没有什么不同。唯一的区别可能是定义了`query`方法。我强烈建议您使用`<<~`来编写多行查询并增加可读性。除此之外，你只需要专注于你想要在你的规范中涵盖的事情和边缘情况。仅此而已。编码快乐！

【selleo.com】最初发表于[](https://selleo.com/blog/testing-graphql-mutations-in-ruby-on-rails-with-rspec)**。**