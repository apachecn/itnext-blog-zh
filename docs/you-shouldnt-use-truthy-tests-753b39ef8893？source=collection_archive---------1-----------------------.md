# 你不应该使用真实性测试

> 原文：<https://itnext.io/you-shouldnt-use-truthy-tests-753b39ef8893?source=collection_archive---------1----------------------->

我讨厌 truthy 测试，我认为你不应该使用它们，每当它们出现在 Pull 请求中时，我都反对。

![](img/defbb5ebec94f678f9990ee6777f1fe8.png)

为什么？因为以我的经验来看，更多的 bug 来自于错误的真值测试，而不是其他类型的错误。而且来自不可靠测试的错误也是最难追踪的。

如果你不知道我说的真值测试是什么意思，那就是这个(用 Python，但是我们一会儿会看一些其他语言):

```
if some_variable:
    do_something()
```

这将询问`some_variable` *是否在执行某些条件动作之前对*到`true`求值。

正是这个词*评估了*，真实性测试的所有问题都出现了。看，有些情况真的是`true`或`false`，就像这样:

```
my_var = True
if my_var: do_something()
```

`my_var`实际上是`true`，因为我们只是将其设置为`True`。

这些呢:

```
my_var = 0
my_var = 74
my_var = 0.0
my_var = ""
my_var = "false"
my_var = []
my_var = {}
```

那些*中的一部分评估*到`true`，另一部分*评估*到`false`。[这里有一页](https://docs.python.org/2.4/lib/truth.html)告诉你哪些 Python 值等同于`false`。

错误出现是因为这些东西*实际上并不相同*，试图将它们视为相同必然会导致意想不到的行为。

这里有一个简单的 JavaScript 例子。向函数传递参数时，通常会将它们默认为如下形式:

```
function whatever(params) {
    var myparam = params.myparam || 1;
}
```

这会将`myparam`设置为你传入的值，如果你没有传入什么，那么它会被设置为`1`。

**挑战**:将`myparam`设置为`0`。

**解决方法**:不能。

JavaScript 中的 truthy 测试，当然是把`0`当成`false`，所以初始测试失败，把`myparam`设为`1`。该模式实际上是为了检查`params.myparam`是否为`undefined`(即未指定)，其也评估为`false`，但它也捕捉评估为`false`的所有其他*实际值*。

正确的做法应该是:

```
function whatever(params) {
    var myparam = params.hasOwnProperty("myparam") ? 
                      params.myparam : 1;
}
```

也就是:*如果我们真的意味着参数应该是未定义的，那么我们应该* ***这么说*** 。

尽管如此，在 JavaScript 代码中到处都可以看到懒惰、容易出错的方法。

我听到开发人员为使用真值测试辩护的一些理由:

*   它们写起来更快
*   因为 [PEP8](https://www.python.org/dev/peps/pep-0008) 这么说
*   当你不确定你在处理什么样的变量时，它们是很好的选择(例如，在鸭式语言中)

让我尽快解决这些问题:

他们写起来更快。那又怎样？现在允许了吗？我为你写了这个软件——它坏了，但这比写一些有用的东西要快。

**因为 PEP8 这么说**。并没有。在[编程建议](https://www.python.org/dev/peps/pep-0008/#programming-recommendations)中说，你不应该使用`==`操作符来比较带有`True`或`False`的布尔值。也就是说，您可以使用 truthy 风格测试，其中操作数是*实际上是一个布尔值*。

当你不确定你在处理什么样的变量时，它们很有用。的确，在鸭式语言中，有时你不确定变量实际上是什么，而真值测试可以是避免找出答案的一种便捷方式。我的回答是，如果你不知道你在处理什么样的变量，对它的价值进行价差押注会有什么帮助？如果你的传播范围不够广怎么办？如果给你变量的函数用`-1`来表示“我不知道答案”，而不是用`0`或`None`或其他什么，那会怎样？在这种情况下，你的真实性测试救不了你。找出变量是什么，适当对待它。

对于我们这些跨多种语言工作的人来说，一个复杂的因素是真实性的规则不是共享的。因此，很容易犯错误并测试在一种语言中可以工作但在另一种语言中不能工作的东西。我们已经链接到 falsy 值的 [Python](https://docs.python.org/2.4/lib/truth.html) 列表，这里是 [Ruby](http://ruby-doc.org/core-2.1.1/FalseClass.html) ，这里是 [JavaScript](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) ，这里是 [PHP](http://php.net/manual/en/language.types.boolean.php) ，这里是 [Perl](https://perldoc.perl.org/perlsyn.html#Truth-and-Falsehood) 。

你会看到一些共性，但也有一些关键的区别。例如，Ruby 相对来说比较理智，它只在实际的`false`和`nil`上计算`false`，而大多数其他语言都包含了空字符串和`0`这样的东西。这里有一些非常讨厌的错误，它们肯定会给你的代码逻辑带来问题:

*   JavaScript 说`document.all`是`false`——这是有原因的，它们与语言的历史使用有关，而不是任何原因。
*   Perl 和 PHP 说字符串`"0"`是`false`——如果这不是等待发生的灾难，我不知道什么是。
*   在 Python 中，用户定义的对象可以自己决定它是真还是假。
*   关于空列表/字典是真的还是假的，有相当大的差异。

如果你使用的语言注重真实性，那么你可以做一些事情来让人们的生活变得更容易，如果你给他们提供代码来集成的话。例如，您的函数不应该返回空字符串，当您指的是`false`时，它应该返回`false`。或者，当函数无法获得合适的返回类型时，可以使用异常。

反过来，代码的消费者也应该尊重你的函数。如果他们问你要一份清单，他们应该认为空清单是一个有效的回答，并在继续之前检查清单的长度。

所以让我回来总结一下。当你不能确定你正在处理的是一个纯布尔值时，你不应该使用真值测试，理由如下:

*   没有借口不知道你在处理什么样的变量。如果那是因为给你变量的函数是模糊的，那么那个函数很烂，应该被修正。如果你不能修复函数，艰难，你仍然需要弄清楚它。
*   空字符串、`"0"`、`0`、`false`、空列表、`document.all`等等都是不一样的，把它们一视同仁不是好主意。
*   这是一个如此重要的错误来源，从长远来看，不这样做可以节省你的时间。

或许最重要的是:

> **明确你要测试的是优秀的、自我记录的、干净的代码，未来的开发者会感谢你的。**

所以说真的。不要使用它们。

*Richard 是软件开发咨询公司*[*Cottage Labs*](https://cottagelabs.com)*的创始人和高级合伙人，专门研究数据生命周期的各个方面。他偶尔会在推特上发*[*@ Richard _ d _ Jones*](https://twitter.com/richard_d_jones)