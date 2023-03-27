# 使用关键字参数和 merge 创建 Ruby 对象

> 原文：<https://itnext.io/removing-argument-order-dependencies-c5e2482ba208?source=collection_archive---------6----------------------->

![](img/f693839752537d3661551bc9a3f8e8f0.png)

通读 Sandi Metz 的书[*Ruby 中的实用面向对象设计(POODR)*](http://www.poodr.com/) *，*我发现自己又一次发现了如何以 Sandi Metz 的方式做事的最佳实践。

在本帖中，我们将揭示在不使用散列或关键字参数的情况下创建 Ruby 对象的一些问题，并提供一些在初始化 Ruby 对象时处理默认值的解决方案。

首先，让我们从创建一个基本的 Ruby `Movie`类开始。

```
class Movie
  attr_accessor :title, :year, :runtime def initialize(title, year, runtime)
    [@title](http://twitter.com/title) = title
    [@year](http://twitter.com/year) = year
    [@runtime](http://twitter.com/runtime) = runtime
  endendmatrix = Movie.new("The Matrix", 1999, 150)matrix.title #=> "The Matrix"
matrix.year #=> 1999
matrix.runtime #=> 150
```

很基本的东西。

然而，正如梅斯总是提到的…

> 设计与其说是实现完美的行为，不如说是保持可变性的艺术。

因此，让我们看看如果我们决定向我们的`Movie`类添加更多的属性会发生什么。

```
class Movie
  attr_accessor :title, :year, :runtime, :box_office, :oscar_noms

  def initialize(title, year, runtime, box_office, oscar_noms)
    [@title](http://twitter.com/title) = title
    [@year](http://twitter.com/year) = year
    [@runtime](http://twitter.com/runtime) = runtime
    [@box_office](http://twitter.com/box_office) = box_office
    [@oscar_noms](http://twitter.com/oscar_noms) = oscar_noms
  endendmatrix = Movie.new("The Matrix", 1999, 150, 463517383, 4)
```

现在创建电影实例变得相当混乱，容易出错。

这四个数字又是什么意思？

这些论点必须按什么顺序排列？

如果我们不一定想谷歌《黑客帝国》的票房收入是多少呢？我们应该在那里输入什么默认值呢？

# 使用关键字参数进行初始化

为了消除参数顺序依赖性，我们可以像这样使用[关键字参数](https://robots.thoughtbot.com/ruby-2-keyword-arguments)来重构我们的`#initialize`方法。

```
class Movie
  attr_accessor :title, :year, :runtime, :box_office, :oscar_noms

  def initialize(
    title:, 
    year: "19XX", 
    oscar_noms: "0?", 
    runtime: "Longer than an hour", 
    box_office: "More than a million?"
  )

    [@title](http://twitter.com/title) = title
    [@year](http://twitter.com/year) = year
    [@runtime](http://twitter.com/runtime) = runtime
    [@box_office](http://twitter.com/box_office) = box_office
    [@oscar_noms](http://twitter.com/oscar_noms) = oscar_noms end
endmatrix = Movie.new(
  year: 1999, 
  title: "The Matrix", 
  runtime: 150,
  oscar_noms: 4
  box_office: 463517383,
)
```

使用关键字参数，我们可以直接在`#initialize`方法中指定键及其默认值。

更重要的是，初始化*矩阵*现在看起来容易多了，最棒的是，我们不再受限于向`#initialize`方法提交参数的固定顺序。

虽然代码比使用固定顺序的参数更冗长，但是重构的`#initialize`方法在可读性和防止任何意外阅读障碍方面更胜一筹。(即变量的交换顺序)

# 默认值选项#2:使用#merge

指定默认值的另一种方法是创建一个单独的`#defaults`方法，并利用 Ruby 的`#merge`方法，如下所示:

```
class Movie
  attr_accessor :title, :year, :stars_keanu

  def initialize(movie_args)
    movie_args = defaults.merge(movie_args) [@title](http://twitter.com/title) = movie_args[:title]
    [@year](http://twitter.com/year) = movie_args[:year]
    [@stars_keanu](http://twitter.com/stars_keanu) = movie_args[:stars_keanu] end private def defaults
    {year: "No earlier than 20th century, stars_keanu: "???"}
  endendinception = Movie.new(
  year: 2010, 
  title: "Inception", 
  stars_keanu: false
)inception.stars_keanu #=> false
```

方法`#merge`首先接受由`#defaults`方法返回的默认散列，然后*将*合并到用户输入的参数中。

换句话说，在`#defaults`方法下，用户提供的任何键值对都将优先于默认的键值对。

我们的`#initialize`方法看起来更干净，尽管可能比使用关键字参数更难理解。

话虽如此，如果默认值开始变得比简单的字符串和数字更复杂，建议为`#defaults`创建一个单独的包装方法并利用`#merge`。

*希望你喜欢这篇文章。请随意在下面留下一些掌声！*