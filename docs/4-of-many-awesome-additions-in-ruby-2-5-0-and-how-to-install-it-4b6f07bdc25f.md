# Ruby 2.5.1 中 4 个令人惊叹的新增功能(以及如何安装)

> 原文：<https://itnext.io/4-of-many-awesome-additions-in-ruby-2-5-0-and-how-to-install-it-4b6f07bdc25f?source=collection_archive---------2----------------------->

## 还有一个关于 Ruby 结构以及如何利用它的快速旁白

![](img/55cac99237da8777e923a7261bcd8399.png)

ruby 2 . 5 . 1:2018 年 3 月 28 日发布

上周二，我参加了[雾城 Ruby](http://www.fogcityruby.com/)——这是一个在旧金山举行的月度聚会，有三场涉及 Ruby 和 Ruby 相关话题的讲座。当晚的主要话题——[*Ruby 2.5:由*](https://speakerdeck.com/havenwood/a-deep-dive-into-new-ruby-features) *[Shannon Skipper](https://github.com/havenwood) 从 [Square](https://squareup.com/) 带来的对 Ruby 特性* 的深入探究——是对 Ruby 2.5 中实现的一些新特性的精彩概括。(他的原始幻灯片，点击[这里](https://speakerdeck.com/havenwood/a-deep-dive-into-new-ruby-features))

下面，我列出了我从他的演讲中学到的 2.5 版的四个显著的新特性。

# 1.前置和追加数组

如果您曾经认为`unshift`听起来像是数组方法的一个笨拙的名字，那是因为它确实是。

Ruby 2.5 现在允许你使用`prepend`在数组的开头添加一个值。对每个人来说更容易阅读和记忆。

```
a = [2,3,4]# THE OLD WAY
a.unshift(1) #=> [1,2,3,4]# NEW IN RUBY 2.5
a.prepend(0) #=> [0,1,2,3,4]
```

你也可以`append`给一个数组赋值，而不是把它推入或者铲入。

```
b = [1,2,3]# THE OLD WAYS
b << 4 #=> [1,2,3,4]
b.push(5) #=> [1,2,3,4,5]# NEW IN RUBY 2.5
b.append(6) #=> [1,2,3,4,5,6]
```

# 2.SecureRandom .字母数字

如果你想创建一个随机的字母数字字符串，Ruby 2.5 可以满足你。`SecureRandom`现在有了一个`alphanumeric`方法。只要确保`require 'securerandom'`在文件的顶部，然后像下面这样调用`SecureRandom.alphanumeric`:

```
require 'securerandom'alnum = SecureRandom.alphanumeric 
#=> "gKS5JuHBb2Xu3JtR"alnum.size #=> 16SecureRandom.alphanumeric(1) 
#=> "x"SecureRandom.alphanumeric(2) 
#=> "8M"SecureRandom.alphanumeric(20)
#=> "95SqPm8TNixZwonrjZqw"
```

# 3.#屈服 _ 自我

Ruby 2.5 增加了一个名为`yield_self`的新方法，它将给定的块产生给定的接收者，并返回块中最后一条语句的输出。

```
2.yield_self { |num| num + 3 } #=> 5"Harry".yield_self { |str| "Hermione, #{str}, and Ron" }#=> "Hermione, Harry, and Ron"
```

# 4.关键字结构

## 4a。除了什么是结构

在 Shannon 向我们介绍这个新的关键字 Structs 概念之前，他要求小组成员如果以前在他们的 Ruby 代码中使用过`Struct`请举手。

房间里大约一半的人举起了手，我不在其中。然后我看了这篇[博文](https://www.leighhalliday.com/ruby-struct)现在知道是什么了。我在这里总结一下:

直接来自 Ruby 文档， [Struct](http://ruby-doc.org/core-2.1.2/Struct.html) 是一种通过使用访问器方法将许多属性捆绑在一起的便捷方式，而不必编写显式类。

要建立一个 Ruby 类，(这里我将使用带有 Fido 实例的陈词滥调 Dog 类)，可以像教科书一样编写它:

```
class Dog
  attr_accessor :name, :breed def initialize(name, breed)
    [@name](http://twitter.com/name) = name
    [@breed](http://twitter.com/breed) = breed
  end def introduce_self
    "Hi, my name is #{name} and I'm a #{breed}."
  endendfido = Dog.new('Fido', 'Corgi')
puts fido.introduce_self #=> Hi, my name is Fido and I'm a Corgi.
```

有了`Struct`，你可以大大缩短让一只狗自我介绍的过程，改为写下:

```
Dog = Struct.new(:name, :breed) do
  def introduce_self
    "Hi, my name is #{name} and I'm a #{breed}."
  end
endfido = Dog.new('Fido', 'Corgi')
puts fido.introduce_self #=> Hi, my name is Fido and I'm a Corgi.
```

不需要键入带有实例变量的`attr_accessors`或`initialize`方法，`Struct`会为您处理所有这些。

## 4b。Ruby 2.5 中的关键字结构

回到谈论 Ruby 2.5…

假设我们想用`Struct`创建蜘蛛侠，如下所示:

```
Superhero = Struct.new(:name, :real_name) do
  def introduce_self
    "Hi, I'm #{name}, but my friends call me #{real_name}."
  end
endspiderman = Superhero.new("Spiderman", "Peter")puts spiderman.introduce_self
#=> Hi, I’m Spiderman, but my friends call me Peter.
```

假设三个小时后，你想创造蝙蝠侠，但你总是忘记名字的顺序。是先说他的真名，还是先说他的超级英雄名？

多亏了 Ruby 2.5 的关键字 structs，您现在可以使用关键字参数创建结构的实例，如下所示:

```
Superhero = Struct.new(:name, :real_name, :keyword_init => true) do
  def introduce_self
    "Hi, I'm #{name}, but my friends call me #{real_name}."
  end
endbatman = Superhero.new(real_name: "Bruce", name: "Batman")puts batman.introduce_self
#=> Hi, I'm Batman, but my friends call me Bruce.
```

请注意顶部的`:keyword_init => true`行，以及我们现在如何通过指定其键值对来创建超级英雄的新实例。

# 如何安装

那么，如何使用 Ruby 2.5 呢？

首先，检查一下自己是不是已经有了。在您的终端中键入:

`ruby -v`

它应该会输出您当前拥有的版本。我的是:

`ruby 2.3.1p112`

我落伍了。

通过在终端键入以下命令，将 [Ruby 版本管理器](https://rvm.io/)下载到您的机器上:

```
curl -L [https://get.rvm.io](https://get.rvm.io) | bash -s stable
```

然后键入:

```
rvm install ruby-2.5.1
```

下载它应该需要时间。

下载完成后，通过输入以下命令将其设置为默认的 Ruby 版本:

```
rvm --default use 2.5.1
```

# 结束语

以上方法只是 Ruby 2.5 为了让你的生活变得更简单而引入的几个方法中的一小部分。请务必查看官方文档和前面提到的[幻灯片](https://speakerdeck.com/havenwood/a-deep-dive-into-new-ruby-features)，了解所有性能改进的列表和新的默认 gem 列表。

还有一个关于 Ruby 2.5 的非常优秀的博客系列[。在我写这篇文章的时候，我也经常参考这个系列。](https://blog.bigbinary.com/categories/ruby-2-5/)

所以一定要更新你的 Ruby，保持最新！

*感谢阅读！请随意在下面留下一些掌声！*