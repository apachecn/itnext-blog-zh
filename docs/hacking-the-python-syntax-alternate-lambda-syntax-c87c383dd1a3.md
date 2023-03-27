# 破解 Python 语法:备用 lambda 语法

> 原文：<https://itnext.io/hacking-the-python-syntax-alternate-lambda-syntax-c87c383dd1a3?source=collection_archive---------5----------------------->

![](img/3be49c7ad5e04cef75d37fd8f99ba166.png)

由 [Unsplash](https://unsplash.com/s/photos/jenga?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的 [La-Rel Easter](https://unsplash.com/@lastnameeaster?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

## 在 60 行代码中

[简介](https://remykarem.medium.com/hacking-the-python-syntax-part-0-introduction-9a1c054ec1e6) | [三元运算符](https://remykarem.medium.com/hacking-the-python-syntax-part-1-ternary-operator-bbcb04aa6ecb) | **交替 lambda 语法** |函数中无返回关键字(即将推出)|列表理解++(即将推出)

*变更日志:
2022 年 12 月 31 日—使用 Medium 的新代码块来突出显示语法*

# 目录

1.  [匿名功能](#64cd)
2.  [当前语法](#7440)
3.  其他人是怎么做的？
4.  [目标语法](#606a)
5.  [添加代币](#0198)
6.  [改变语法](#f443)
7.  [例题](#f88e)

关于环境设置，请阅读*简介*。完整的代码，看我的分叉[这里](https://github.com/remykarem/cpython/tree/lambda-syntax)。

# 1.匿名函数

这是我的最爱(我私下希望 Python 在未来几年内会考虑改变这一点或类似的东西)！

一个没有名字的函数，或者一个 [**匿名函数**](https://en.wikipedia.org/wiki/Anonymous_function) ，是一个在[函数编程](https://en.wikipedia.org/wiki/Functional_programming)范例中普遍使用的构造，其中函数经常作为对象传递。

# 2.当前语法

在 Python 中，你写一个[**λ表达式**](https://docs.python.org/3/reference/expressions.html#lambda) 来创建一个匿名函数:

```
lambda x: x+1
```

下面是一个例子，我们使用匿名函数根据项目的长度对列表进行排序:

```
>>> items = ["applesss", "apple"]
>>> sorted(items, key=lambda x: len(x))
["apple", "applesss"]
```

好吧，所以我的问题是为什么“lambda”这个词？感觉这个关键词不够直观。

这个想法可能来自[λ演算](https://en.wikipedia.org/wiki/Lambda_calculus)，但这是一个深奥的概念(我最近才知道)。它也可能来自[Lisp](https://www.gnu.org/software/emacs/manual/html_node/eintr/lambda.html)——他们对匿名函数使用`lambda`关键字。

```
(lambda (x) (1 + x))
```

那么我们该怎么办呢？

# 3.其他人是怎么做的？

使用 Python 定义参数加 3 的方式，

```
lambda x: x+3
```

让我们看看其他语言是如何做到这一点的(如果有不止一种选择，我将挑选出我认为更简洁的一种)。

[JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) (对于可以推断类型的[类型脚本](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#anonymous-functions)也是如此)

```
x => x+3
(x) => x+3
```

[Java 8](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)

```
(x) -> x+3
(x) -> { x + 3}
```

[科特林](https://kotlinlang.org/docs/lambdas.html#lambda-expressions-and-anonymous-functions)

```
{ x -> x+3 }
```

[生锈](https://doc.rust-lang.org/rust-by-example/fn/closures.html)🦀

```
|x| x+3
|x| -> { x+3 }
```

红宝石

```
{ |x| puts x+3 }
```

哈斯克尔

```
\x -> x + 3
```

[C++11](https://en.cppreference.com/w/cpp/language/lambda)

```
[](int value) { return x + 3; }
```

雨燕

```
{ $0 + 3}
```

[仙丹](https://elixir-lang.org/crash-course.html#partials-and-function-captures-in-elixir)

```
&(&1 + 3)
```

[Scala](https://docs.scala-lang.org/overviews/scala-book/anonymous-functions.html)

```
i => i + 3
_ + 3  // Are you kidding me? I love this!
```

Clojure(以下是[前缀标注](https://en.wikipedia.org/wiki/Polish_notation)；假设我们想要一个[中缀](https://en.wikipedia.org/wiki/Infix_notation)版本，它看起来像`#(% + 3)`。)

```
#(+ % 3) 
```

基于这些语言，以下是我观察到的一些特征(并非相互排斥):

*   使用`&`、`#`、`\`或`{ ... }`的匿名函数的明确指示。在 Python 中，这是`lambda`。
*   使用类似于`->`或`=>`的分隔符在参数和正文之间使用“分隔符”,或者按照类似于`|x|`或`(x)`的语法将参数和正文分组，并将`{ ... }`作为正文。
*   对参数或位置参数的引用，如`_`、`%1`或`$0`。
*   表示返回值的关键字，如`return`或`puts`。

# 4.目标语法

今天，我们将删除关键字，并用另一种语法替换它😈。我们需要一些能立即显示起点`x`和结果`x+1`的东西，并使用一些符号来建立这种关系。

所以我在想…

```
|x| -> x+1
```

1.  删除匿名函数的指示(`lambda`)
2.  用`|...|`和`->`改变参数和主体之间的“分隔线”,只是因为它更常见。

以下是一些其他注意事项:

*   管道操作符`|`目前只被用作二元操作符(需要 LHS 和 RHS 参数),所以使用`|...|`应该对当前规则没有什么干扰
*   `=>`第一次对我来说失败了，有点发毛，所以`->`就是了，虽然目前是用来注释[函数返回类型](https://docs.python.org/3/library/typing.html)。
*   增强的按位 or `|=`会有问题

# 5.添加令牌

不需要新的令牌。

*   `|`令牌值(令牌名`VBAR`)已经用于[位或](https://docs.python.org/3/library/operator.html#mapping-operators-to-functions)。
*   `->`令牌值(令牌名`RARROW`)也已经被用于[注释函数返回类型](https://docs.python.org/3/library/typing.html#module-typing)。

# 6.改变语法

改变这种语法稍微复杂一些。所以去喝杯咖啡吧！

## 6.1 lambdef

λ定义定义为`lambdef`。让我们在第 5–7 行创建 lambda 的新定义:

```
lambdef[expr_ty]:
    | 'lambda' a=[lambda_params] ':' b=expression {
        _PyAST_Lambda(
            (a) ? a : CHECK(arguments_ty, _PyPegen_empty_arguments(p)), b, EXTRA) }
    | '|' a=[lambda_params_new] '|' '->' b=expression {
        _PyAST_Lambda(
            (a) ? a : CHECK(arguments_ty, _PyPegen_empty_arguments(p)), b, EXTRA) }
```

它遵循以下伪语法:

```
| <lambda_params_new> | -> <expression>
```

现有的`lambda_params`定义及其嵌套定义看起来非常复杂。这就是为什么我们创造了一个新的定义叫做`lambda_params_new`。

## _ params _ new

lambda 参数的现有规则(`lambda_params`)进一步定义了 lambda 函数的结构，如斜线、星号、默认关键字等。

```
lambda_params[arguments_ty]:
    | invalid_lambda_parameters
    | lambda_parameters

# lambda_parameters etc. duplicates parameters but without annotations
# or type comments, and if there's no comma after a parameter, we expect
# a colon, not a close parenthesis.  (For more, see parameters above.)
#
lambda_parameters[arguments_ty]:
    | a=lambda_slash_no_default b[asdl_arg_seq*]=lambda_param_no_default* c=lambda_param_with_default* d=[lambda_star_etc] {
        _PyPegen_make_arguments(p, a, NULL, b, c, d) }
    | a=lambda_slash_with_default b=lambda_param_with_default* c=[lambda_star_etc] {
        _PyPegen_make_arguments(p, NULL, a, NULL, b, c) }
    | a[asdl_arg_seq*]=lambda_param_no_default+ b=lambda_param_with_default* c=[lambda_star_etc] {
        _PyPegen_make_arguments(p, NULL, NULL, a, b, c) }
    | a=lambda_param_with_default+ b=[lambda_star_etc] { _PyPegen_make_arguments(p, NULL, NULL, NULL, a, b)}
    | a=lambda_star_etc { _PyPegen_make_arguments(p, NULL, NULL, NULL, NULL, a) }

lambda_slash_no_default[asdl_arg_seq*]:
    | a[asdl_arg_seq*]=lambda_param_no_default+ '/' ',' { a }
    | a[asdl_arg_seq*]=lambda_param_no_default+ '/' &':' { a }

lambda_slash_with_default[SlashWithDefault*]:
    | a=lambda_param_no_default* b=lambda_param_with_default+ '/' ',' { _PyPegen_slash_with_default(p, (asdl_arg_seq *)a, b) }
    | a=lambda_param_no_default* b=lambda_param_with_default+ '/' &':' { _PyPegen_slash_with_default(p, (asdl_arg_seq *)a, b) }

lambda_star_etc[StarEtc*]:
    | '*' a=lambda_param_no_default b=lambda_param_maybe_default* c=[lambda_kwds] {
        _PyPegen_star_etc(p, a, b, c) }
    | '*' ',' b=lambda_param_maybe_default+ c=[lambda_kwds] {
        _PyPegen_star_etc(p, NULL, b, c) }
    | a=lambda_kwds { _PyPegen_star_etc(p, NULL, NULL, a) }
    | invalid_lambda_star_etc

lambda_kwds[arg_ty]: '**' a=lambda_param_no_default { a }

lambda_param_no_default[arg_ty]:
    | a=lambda_param ',' { a }
    | a=lambda_param &':' { a }
lambda_param_with_default[NameDefaultPair*]:
    | a=lambda_param c=default ',' { _PyPegen_name_default_pair(p, a, c, NULL) }
    | a=lambda_param c=default &':' { _PyPegen_name_default_pair(p, a, c, NULL) }
lambda_param_maybe_default[NameDefaultPair*]:
    | a=lambda_param c=default? ',' { _PyPegen_name_default_pair(p, a, c, NULL) }
    | a=lambda_param c=default? &':' { _PyPegen_name_default_pair(p, a, c, NULL) }
lambda_param[arg_ty]: a=NAME { _PyAST_arg(a->v.Name.id, NULL, NULL, EXTRA) }
```

有很多逻辑。为了不弄乱现有的逻辑，我们将创建上面的副本，将`_new`附加到所有的规则上。然后，我们在这些站点用新语法替换旧语法(注意`'|'`):

*   `lambda_slash_no_default_new`
*   `lambda_slash_with_default_new`
*   `lambda_param_no_default_new`
*   `lambda_param_with_default_new`
*   `lambda_param_maybe_default_new`

```
# Lambda functions new syntax
# ---------------------------

lambda_params_new[arguments_ty]:
    | invalid_lambda_parameters_new
    | lambda_parameters_new

# lambda_parameters etc. duplicates parameters but without annotations
# or type comments, and if there's no comma after a parameter, we expect
# a colon, not a close parenthesis.  (For more, see parameters above.)
#
lambda_parameters_new[arguments_ty]:
    | a=lambda_slash_no_default_new b[asdl_arg_seq*]=lambda_param_no_default_new* c=lambda_param_with_default_new* d=[lambda_star_etc_new] {
        _PyPegen_make_arguments(p, a, NULL, b, c, d) }
    | a=lambda_slash_with_default_new b=lambda_param_with_default_new* c=[lambda_star_etc_new] {
        _PyPegen_make_arguments(p, NULL, a, NULL, b, c) }
    | a[asdl_arg_seq*]=lambda_param_no_default_new+ b=lambda_param_with_default_new* c=[lambda_star_etc_new] {
        _PyPegen_make_arguments(p, NULL, NULL, a, b, c) }
    | a=lambda_param_with_default_new+ b=[lambda_star_etc_new] { _PyPegen_make_arguments(p, NULL, NULL, NULL, a, b)}
    | a=lambda_star_etc_new { _PyPegen_make_arguments(p, NULL, NULL, NULL, NULL, a) }

lambda_slash_no_default_new[asdl_arg_seq*]:
    | a[asdl_arg_seq*]=lambda_param_no_default_new+ '/' ',' { a }
    | a[asdl_arg_seq*]=lambda_param_no_default_new+ '/' &'|' { a }

lambda_slash_with_default_new[SlashWithDefault*]:
    | a=lambda_param_no_default_new* b=lambda_param_with_default_new+ '/' ',' { _PyPegen_slash_with_default(p, (asdl_arg_seq *)a, b) }
    | a=lambda_param_no_default_new* b=lambda_param_with_default_new+ '/' &'|' { _PyPegen_slash_with_default(p, (asdl_arg_seq *)a, b) }

lambda_star_etc_new[StarEtc*]:
    | '*' a=lambda_param_no_default_new b=lambda_param_maybe_default_new* c=[lambda_kwds_new] {
        _PyPegen_star_etc(p, a, b, c) }
    | '*' ',' b=lambda_param_maybe_default_new+ c=[lambda_kwds_new] {
        _PyPegen_star_etc(p, NULL, b, c) }
    | a=lambda_kwds_new { _PyPegen_star_etc(p, NULL, NULL, a) }
    | invalid_lambda_star_etc_new

lambda_kwds_new[arg_ty]: '**' a=lambda_param_no_default_new { a }

lambda_param_no_default_new[arg_ty]:
    | a=lambda_param_new ',' { a }
    | a=lambda_param_new &'|' { a }
lambda_param_with_default_new[NameDefaultPair*]:
    | a=lambda_param_new c=default ',' { _PyPegen_name_default_pair(p, a, c, NULL) }
    | a=lambda_param_new c=default &'|' { _PyPegen_name_default_pair(p, a, c, NULL) }
lambda_param_maybe_default_new[NameDefaultPair*]:
    | a=lambda_param_new c=default? ',' { _PyPegen_name_default_pair(p, a, c, NULL) }
    | a=lambda_param_new c=default? &'|' { _PyPegen_name_default_pair(p, a, c, NULL) }
lambda_param_new[arg_ty]: a=NAME { _PyAST_arg(a->v.Name.id, NULL, NULL, EXTRA) }
```

我们还没有完全完成，因为我们还没有创建一些定义，即`invalid_lambda_`定义。

## 6.3 无效的 _lambda_*

如果遇到不正确的语法，我们希望相应地引发错误。

快速搜索`invalid_lambda_`可以看到三个现有的规则:`invalid_lambda_parameters`、`invalid_lambda_parameters_helper`和`invalid_lambda_star_etc`。

因此，让我们创建一个副本，进行适当的编辑以适应新的语法，并将`_new`添加到规则名称中:

```
invalid_lambda_parameters_new:
    | lambda_param_no_default_new* invalid_lambda_parameters_helper_new a=lambda_param_no_default_new {
        RAISE_SYNTAX_ERROR_KNOWN_LOCATION(a, "non-default argument follows default argument") }

invalid_lambda_parameters_helper_new:
    | a=lambda_slash_with_default_new { _PyPegen_singleton_seq(p, a) }
    | lambda_param_with_default_new+

invalid_lambda_star_etc_new:
    | '*' ('|' | ',' ('|' | '**')) { RAISE_SYNTAX_ERROR("named arguments must follow bare *") }
```

而且…我们应该准备好了！重新生成相关文件并运行解释器:

```
**make** regen-pegen && **make** -j8 -s && **./python.exe**
```

# 7.例子

只有一个参数的匿名函数:

```
>>> identity = |x| -> x
>>> identity("hello")
"hello'
```

具有多个参数的匿名函数:

```
>>> product = |a, b, c| -> a*b*c
>>> product(1, 8, 9)
72
```

没有参数的匿名函数:

```
>>> display_hello = || -> print("hello")
>>> display_hello()
'hello'
```

带有默认参数的匿名函数:

```
>>> exp = |x, power=2| -> x**power
>>> exp(10)
100
>>> exp(3, 3)
27
```

匿名函数，用于列表排序:

```
>>> items = ["applesss", "apple"]

>>> sorted(items, key=lambda x: len(x))
["apple", "applesss"]

>>> sorted(items, key=|x|->len(x))
["apple", "applesss"]
```

一个匿名函数，用在 Spark 这样的转换管道中(见[https://spark.apache.org/examples.html](https://spark.apache.org/examples.html)):

```
>>> text_file = sc.textFile("hdfs://...")
>>> counts = (
...     text_file
...     .flatMap(|line| -> line.split(" "))
...     .map(|word| -> (word, 1))
...     .reduceByKey(|a, b| -> a + b)
... )
```

模糊一行程序中的匿名函数，改编自 Python docs[https://docs . Python . org/3/FAQ/programming . html # is-it-possible-to-write-obfuscated-one-liners-in-Python](https://docs.python.org/3/faq/programming.html#is-it-possible-to-write-obfuscated-one-liners-in-python)。这个程序生成前十个斐波那契数。

```
>>> list(map(|x,f=x,f|->(f(x-1,f)+f(x-2,f)) if x>1 else 1|->f(x,f),range(10)))
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

通过使用第 1 部分中的三元运算符，我们可以使上面的内容更加简洁:

```
>>> list(map(|x,f=|x,f|->x>1?f(x-1,f)+f(x-2,f):1|->f(x,f),range(10)))
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

暂时就这样吧！谢谢你让我和你分享我的黑客之旅。敬请关注函数中的**不返回关键字和**列表理解++** ！**

如果你有点迷茫，你可能想先看看[破解 Python 语法:三元运算符](https://remykarem.medium.com/hacking-the-python-syntax-part-1-ternary-operator-bbcb04aa6ecb)。

我发表关于人工智能、机器学习、编程语言和生产力的文章。

*如果您喜欢阅读更多关于编程语言的内容，您可以通过我的推荐链接* [*订阅*](https://remykarem.medium.com/subscribe) *以接收我发布的更新，或者* [*注册*](https://remykarem.medium.com/membership) *！请注意，您的会员费的一部分将作为介绍费分摊给我。*

# 参考

*   改变 CPython 的语法([devguide.python.org](https://devguide.python.org/grammar/))
*   CPython 解析器指南([devguide.python.org](https://devguide.python.org/parser/))