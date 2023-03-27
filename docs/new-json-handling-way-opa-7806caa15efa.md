# 新的 JSON/YAML 处理方式= OPA

> 原文：<https://itnext.io/new-json-handling-way-opa-7806caa15efa?source=collection_archive---------2----------------------->

## “opa eval”的通用 JSON/YAML 运算

![](img/7882a16c54e36f141b0e111fe139a0bc.png)

照片由[考夫曼商业](https://unsplash.com/@kaufmann_mercantile?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

## [更新于 2021 年 4 月 12 日]

我注意到**这个方法也可以应用到 YAML……**这篇文章关注的是 JSON 处理，但是 YAML 之类的`kubectl`输出可以传递给`jr`命令。我会写另一篇关于这个的文章。

## TL；速度三角形定位法(dead reckoning)

使用 OPA 可以进行以下 JSON 处理。`jr`包括`opa eval`。

```
$ data='{"name":"chris", "friends":["alice", "bob"]}'
$ echo $data | **jr 'i.friends' | jr 'i[0]'**
"alice"
```

# 处理 JSON 的现有方法

当我处理 JSON 数据时，我使用了许多命令工具:`cat`、`grep`和`jq`。`jq`有一个用于处理 JSON 的语法(比如创建一个新的对象或数组，从输入中过滤掉一些值)。下面的`jq`命令提取部分 JSON 对象。

```
$ data='{"name":"chris", "friends":["alice", "bob"]}'
$ echo $data | **jq '.name'**
"chris"
$ echo $data | **jq '.friends'**
[
  "alice",
  "bob"
]
```

# OPA 是什么？

![](img/2dd3434a9e2e6d48730a70c1ece64e25.png)

[**OPA(开放策略代理)**](https://www.openpolicyagent.org/) 是通用策略引擎。“策略引擎”听起来像一个复杂的工具，但它不是。

OPA 很简单。OPA 接收一些 JSON 输入，根据定义的规则修改它，并回复一个 JSON 输出。就是这样。事实上，OPA 拥有强大的规则解析器，可以解决复杂的规则。这就是 OPA 被称为“策略引擎”的原因。但是我们不需要部署复杂的规则。

我们可以按照[正式文件](https://www.openpolicyagent.org/docs/latest/#running-opa)安装`opa`命令行工具。当您使用 MacOS 时，只需运行以下命令。

```
**brew install opa**
```

# 作为 JSON/YAML 处理工具的“opa eval”命令

我们可以使用`opa`命令来处理 JSON 数据。请看这个例子。

```
$ data='{"name":"chris", "friends":["alice", "bob"]}'
$ echo $data | **opa eval -I --format=raw 'input.name'**
"chris"
$ echo $data | **opa eval -I --format=raw 'input.friends'**
[
  "alice",
  "bob"
]
```

令人惊讶的是，这些命令返回的结果与`jq`相同。

让我们简化这些命令。用了一招之后，我们可以这样写`opa eval`。

```
$ data='{"name":"chris", "friends":["alice", "bob"]}'
$ echo $data | **jr 'i.name'**
"chris"
$ echo $data | **jr 'i.friends' | jp i**
[
  "alice",
  "bob"
]
```

可以看到用法更类似于`jq`。`jr`和`**jp**`只是以下命令的别名。

```
opa eval -I --import 'input as i' [--format=pretty / --format=raw]
```

让我们配置您的终端以允许这些别名。

# 配置 jp / jr 命令

如果你使用 zsh，在`~/.zshrc`中写下下面几行

```
alias jo="opa eval -I --import 'input as i'"
# The above line works after v0.27.1\. Before v0.27.0, the next works
# alias jo=”opa eval -I --import ‘input as i’” --package p
alias jp="jo --format=pretty"
alias jr="jo --format=raw"
```

## **含义**

*   `opa eval`
    `opa`的强大子命令。它评估类似`'input.name'`的任意策略。
*   `-I`
    该选项表示 JSON 输入来自标准输入。
*   `--import 'input as i'`
    该选项将`input`缩短为`'i'`。`input`是指 JSON 输入数据的特殊关键字。使用该选项后，我们可以用`i`指定输入数据。
*   `--format=pretty / --format=raw`
    该选项使`opa eval`生成 JSON 输出。

# 基本用法

## 准备:设置一个输入数据

```
$ data='{"people":[{"name":"chris", "friends":["alice", "bob"]}, {"name":"elena", "friends":["dave"]}]}'
$ echo $data | jp i
{
  "people": [
    {
      "friends": [
        "alice",
        "bob"
      ],
      "name": "chris"
    },
    {
      "friends": [
        "dave"
      ],
      "name": "elena"
    }
  ]
}
```

## 用法 1:过滤一些字段

基本用法是`i.<some field>`选择单个字段。
这个语法和`i.<some>.<child>`一样是递归的。
或者我们可以使用管道`|`来连接不同的`jr`命令。当你想以美化的格式打印输出时，`jp`命令很有用。

```
$ echo $data | jr **'i.people[1]' | jp i**
{
  "friends": [
    "dave"
  ],
  "name": "elena"
}$ echo $data | jr **'i.people[1].friends'**
["dave"]$ echo $data | jr **'i.people[1]' | jr 'i.friends'**
["dave"]
```

> **技术#1** 。`i.<some field>` *(* 选择单个字段)

## 用法#2:检查是否包含一些值(条件)

这种用法用于检查 JSON 数据是否满足某些条件。
最常用的运算符是`==`，当**单值等于另一个值**时，返回`true`。
并且当**候选包括单个值**时，它也返回`true`。

```
$ echo $data | jr **'"chris" == i.people[0].name'** 
true$ echo $data | jr **'"chris" == i.people[1].name'** 
false$ echo $data | jr **'"chris" == i.people[_].name'
** # this means any elements have name "chris"
true$ echo $data | jr **'"dave" == i.people[_].friends[_]'**
true$ echo $data | jr **'"dave" == i.people[x].friends[y]'**
true
```

> **技术#2** : `[_], *[x], [<arbitrary>]*`(指所有考生)

## 用法 3:向上列表

第三种常见用法是列出所有元素。
在之前的技术中，显示了`[_]`。这也可以用于列表。

```
$ echo $data | jr **'i.people[_]'**
{"friends":["alice","bob"],"name":"chris"}
{"friends":["dave"],"name":"elena"}
$ echo $data | jr **'i.people[_].friends[_]'**
alice
bob
dave
```

## 用法#4:格式化字符串

当我们想创建一个格式化的字符串时，`sprintf`在许多编程语言中都会用到。OPA 也不例外。我们可以在表达式中使用`sprintf`。出于类似的原因，`concat`可以用来从许多字符串数据中创建一个字符串数据。

```
$ echo $data | jr **'sprintf("name=%s, friends=%s", [i.people[1].name, i.people[1].friends])'** "name=elena, friends=[\"dave\"]"# Change 1 to x
$ echo $data | jr **'sprintf("name=%s, friends=%s", [i.people[x].name, i.people[x].friends])'**
name=chris, friends=["alice", "bob"]
name=elena, friends=["dave"]$ echo $data | jr **'concat(" and ", [i.people[0].friends[0], i.people[0].friends[1]])'**
alice and bob
```

> **技术#3** : `sprintf`(格式化)`concat`(加入数组)
> 其他内置函数见[官方文档](https://www.openpolicyagent.org/docs/latest/policy-reference/#strings) t。

## 用法#5:展平到数组

技术#2 `[_]`允许我们获得所有的元素，但是不足以将所有的元素组合在一起。看下面这个例子。

```
$ echo $data | **jr 'i.people[_].name'**
chris
elena
```

这个结果不是单个数组对象，而是字符串类型的独立结果。
如果你想得到一个数组，你需要稍微改变一下策略。
`[<any name>|<operation>]`返回。另一种语法是`{<any name>|<condition>}`。严格来说，这两种语法有不同的含义(第一种生成数组，第二种生成集合)。

```
$ echo $data | **jr '[r|r := i.people[_].name]'**
["chris","elena"]$ echo $data | **jr '[r|r := i.people[_].friends[_]]'**
["alice","bob","dave"]
```

> **技巧四:** `[<any>| <any> := <operation>]`(生成数组)
> 
> **技巧#5:** `*{<any>| <any> := <operation>}*`(生成一组)

## 用法#6:条件过滤器

如果你有一个数组或者一个集合，通过执行用法#5，你可以更进一步到高级用法。
高级用法之一是条件过滤器。这是技术#2 和技术#4，5 的结合。在生成数组的过程中，您可以收集选择性元素。
以下示例在集合生成`{|}`中使用了条件表达式`==`。

```
$ echo $data | **jr '{rtn|rtn := i.people[_]; rtn.name == "chris"}'**
[{"friends":["alice","bob"],"name":"chris"}]
```

`rtn.name == "chris"`是过滤`people[_]`元素的条件。您可以像这样添加任意数量的条件。

```
echo $data | **jr '{rtn|rtn := i.people[_].friends[_]; rtn != "dave"; rtn != "peter"}'**
["alice","bob"]
```

一个更高级的例子是

```
$ echo $data | jr **'{rtn|
  a := i.people[_]
  a.name == "chris"
  rtn := a.friends[_]
  rtn != "alice"}'**
["bob"]
```

我们使用`;`来分隔多个表达式。但是`breakline`也一样工作。
而且上一个例子和下一个例子没有区别。条件行位于不同的位置。这也是 OPA 强大的原因之一。OPA 说它具有声明性。**“声明性”是指规则只需要声明，不需要考虑处理的顺序。**

```
$ echo $data | jr **'{rtn|
  a := i.people[_]
  rtn := a.friends[_] # switched
  a.name == "chris" # switched
  rtn != "alice"}'** ["bob"]$ echo $data | jr **'{rtn|
  a := i.people[_]
  rtn := a.friends[_]
  rtn != "alice" #switched
  a.name == "chris"}' #switched** ["bob"]
```

# OPA 的高级用途和通往深层世界的大门

## 高级用法#1:多个字段

当技术#2、3、4 结合起来时，我们可以为 JSON 中的多个元素自动化一个模板化工作。

```
echo $data | jr **'{r|
 a := i.people[x].name;
 b := i.people[x].friends;
 c := concat(", ", b)
 r := sprintf("(name=%s, friends=%s)", [a,c])}'**
["(name=chris, friends=alice, bob)","(name=elena, friends=dave)"]
```

## 高级用法#2:创建新函数(=规则)

上面的例子很有用。但是我们可以走得更远。当我深入使用这些技术时，我不想在命令中重复输入相同的行。我想创建一个函数来避免重复。
是我们创建一个`.rego`文件的时间。

创建一个`filter.rego`并写下这几行。最后 6 行与高级用法#1 相同。

```
package f
import input as i
format = {r |
  a := i.people[x].name
  b := i.people[x].friends
  c := concat(", ", b)
  r := sprintf("(name=%s, friends=%s)", [a,c])
}
```

现在，我们可以运行以下命令。我们将得到与上一个没有内联策略编写的例子相同的结果。

```
echo $data | jr **-d filter.rego** **'data.f.format'**
["(name=chris, friends=alice, bob)","(name=elena, friends=dave)"]
```

含义

*   `-d <file>` :
    这是`opa eval`选项。它允许导入 rego 文件或 json 数据文件。这个选项可以重复。这些文件中的每个规则或数据都存储在`data.<label>`中。对于 rego 文件，`<label>`与 rego 文件的第一行相同。上面写的是`package f`，所以我们可以把函数`format`称为`data.f.format`。

我们可以在一个文件中定义多个函数(OPA 世界中的=“rules”)。
创建 rego 文件后，我们可以重用现有的规则。

# 更进一步

如果你有兴趣，这里有一些先进的技术。
[https://gist . github . com/onelittle night music/DAE 68899 fee 9 EC 967 e 56 cdfd 49985 d6c](https://gist.github.com/onelittlenightmusic/dae68899fee9ec967e56cdfd49985d6c)

# 摘要

`jp`和`jr`只是简单的命令别名，因此`opa`可以用于任何 JSON 操作。原来，`opa eval`命令在处理 JSON 对象上是如此强大。尤其是，`--format=raw`选项使这种想法成为可能。[这个增强版](https://github.com/open-policy-agent/opa/pull/3207)是由
[贾斯珀·范德耶乌特](https://github.com/jaspervdj-luminal)在两周前创作的，并由[斯蒂芬·雷纳图斯](https://github.com/srenatus)和
[托林·桑德尔](https://github.com/tsandall)进行了讨论。不仅如此，`opa`也在积极研发中。为了 JSON 的处理，我还写了`opa`的[一点点增强](https://github.com/open-policy-agent/opa/pull/3240)。任何人都可以投稿`opa`。
OPA 旨在成为微服务、Kubernetes、CI/CD 管道、API 网关中的通用策略实施([官方文件](https://www.openpolicyagent.org/docs/latest/))。多亏了它，OPA 在 JSON 处理中有了这么多有用的函数。浏览这些文档是开始 OPA 的好方法。
如果这篇文章对任何人的 JSON 操作的套路有所帮助，我很乐意。

![](img/43d8ac361775cc5c6267b225e7d5fc00.png)

马库斯·斯皮斯克在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片