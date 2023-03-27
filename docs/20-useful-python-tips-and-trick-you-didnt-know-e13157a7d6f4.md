# 您不知道的 20 个有用的 Python 技巧和诀窍

> 原文：<https://itnext.io/20-useful-python-tips-and-trick-you-didnt-know-e13157a7d6f4?source=collection_archive---------0----------------------->

![](img/ce5d0f911a1688c3aa679428c2a7f687.png)

[Duomly —编程在线课程](https://www.duomly.com)

本文原载:[https://www . blog . duomly . com/20-essential-python-tips-and-tricks-you-should-know/](https://www.blog.duomly.com/20-essential-python-tips-and-tricks-you-should-know/)

Python 是一种流行的、通用的、广泛使用的编程语言。它用于数据科学和机器学习、许多领域的科学计算、后端 Web 开发、移动和桌面应用程序等等。许多知名公司都使用 Python:谷歌、Dropbox、脸书、Mozilla、IBM、Quora、亚马逊、Spotify、NASA、网飞、Reddit 等等。

Python 是免费和开源的，与其相关的大部分产品也是如此。此外，它还有一个由程序员和其他用户组成的大型、专注且友好的社区。

其语法的设计考虑了简单性、可读性和优雅性。

本文介绍了 20 个可能有用的 Python 技巧和诀窍。

# 1.Python 的禅

Python 的禅宗(Zen of Python)也称为 PEP 20，是 Tim Peters 的一篇小文章，代表了设计和使用 Python 的指导原则。它可以在 Python 网站上找到，但是您也可以在您的终端(控制台)或 Jupyter 笔记本上用一条语句获得它:

```
>>> import this
The Zen of Python, by Tim PetersBeautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

# 2.链式分配

如果需要多个变量引用同一个对象，可以使用链式赋值:

```
>>> x = y = z = 2
>>> x, y, z
(2, 2, 2)
```

逻辑优雅对吧？

# 3.链式比较

通过链接比较运算符，可以将多个比较合并到一个 Python 表达式中。如果所有比较都正确，则该表达式返回 True，否则返回 False:

```
>>> x = 5
>>> 2 < x ≤ 8
True
>>> 6 < x ≤ 8
False
```

这类似于(2 < x)和(x ≤ 8)以及(6 < x)和(x ≤ 8 ),但更紧凑，并且只需要对 x 求值一次。

这也是合法的:

```
>>> 2 < x > 4
True
```

您可以链接两个以上的比较:

```
>>> x = 2
>>> y = 8
>>> 0 < x < 4 < y < 16
True
```

# 4.多重赋值

您可以使用元组解包在单个语句中分配多个变量:

```
>>> x, y, z = 2, 4, 8
>>> x
2
>>> y
4
>>> z
8
```

请注意，第一条语句中的 2，4，8 是一个等价于(2，4，8)的元组。

# 5.更高级的多重赋值

普通的多重赋值不是 Python 能做的全部。您不需要左侧和右侧的元素数量相同:

```
>>> x, *y, z = 2, 4, 8, 16
>>> x
2
>>> y
[4, 8]
>>> z
16
```

在这种情况下，x 取第一个值(2 ),因为它最先出现。z 是最后一个，取最后一个值(8)。y 接受打包在列表中的所有其他值，因为它有星号(*y)。

# 6.交换变量

您可以应用多重赋值以简洁优雅的方式交换任意两个变量，而无需引入第三个变量:

```
>>> x, y = 2, 8
>>> x
2
>>> y
8
>>> x, y = y, x
>>> x
8
>>> y
2
```

# 7.合并词典

合并两个或更多字典的一种方法是将它们解压成一个新字典:

```
>>> x = {'u': 1}
>>> y = {'v': 2}
>>> z = {**x, **y, 'w': 4}
>>> z
{'u': 1, 'v': 2, 'w': 4}
```

# 8.连接字符串

如果需要连接多个字符串，最终它们之间有相同的字符或一组字符，可以使用 str.join()方法:

```
>>> x = ['u', 'v', 'w']
>>> y = '-*-'.join(x)
>>> y
'u-*-v-*-w'
```

# 9.高级迭代

如果要遍历一个序列，并且需要序列元素和相应的索引，应该使用 enumerate:

```
>>> for i, item in enumerate(['u', 'v', 'w']):
...     print('index:', i, 'element:', item)
... 
index: 0 element: u
index: 1 element: v
index: 2 element: w
```

在每次迭代中，您将获得一个元组，其中包含索引和序列的相应元素。

# 10.反向迭代

如果要以相反的顺序遍历序列，应该使用 reversed:

```
>>> for item in reversed(['u', 'v', 'w']):
...     print(item)
... 
w
v
u
```

# 11.聚集元素

如果您要聚合来自几个序列的元素，您应该使用 zip:

```
>>> x = [1, 2, 4]
>>> y = ('u', 'v', 'w')
>>> z = zip(x, y)
>>> z
<zip object at 0x7f9e98df3148>
>>> list(z)
[(1, 'u'), (2, 'v'), (4, 'w')]
```

您可以遍历获得的 zip 对象，或者将其转换为列表或元组。

# 12.转置矩阵

虽然人们在处理矩阵时通常使用 NumPy(或类似的库),但是您可以用 zip 获得矩阵的转置:

```
>>> x = [(1, 2, 4), ('u', 'v', 'w')]
>>> y = zip(*x)
>>> z = list(y)
>>> z
[(1, 'u'), (2, 'v'), (4, 'w')]
```

# 13.独特的价值观

如果元素的顺序不重要，可以通过将列表转换为集合来删除列表中的重复项，从而获得唯一值:

```
>>> x = [1, 2, 1, 4, 8]
>>> y = set(x)
>>> y
{8, 1, 2, 4}
>>> z = list(y)
>>> z
[8, 1, 2, 4]
```

# 14.排序序列

默认情况下，序列按其第一个元素排序:

```
>>> x = (1, 'v')
>>> y = (4, 'u')
>>> z = (2, 'w')
>>> sorted([x, y, z])
[(1, 'v'), (2, 'w'), (4, 'u')]
```

但是，如果您想要根据它们的第二个(或其他)元素对它们进行排序，您可以使用参数键和一个适当的 lambda 函数作为相应的参数:

```
>>> sorted([x, y, z], key=lambda item: item[1])
[(4, 'u'), (1, 'v'), (2, 'w')]
```

如果您想获得相反的顺序，情况类似:

```
>>> sorted([x, y, z], key=lambda item: item[1], reverse=True)
[(2, 'w'), (1, 'v'), (4, 'u')]
```

# 15.分类词典

您可以使用类似的方法对通过。items()方法:

```
>>> x = {'u': 4, 'w': 2, 'v': 1}
>>> sorted(x.items())
[('u', 4), ('v', 1), ('w', 2)]
```

它们是根据关键字排序的。如果您希望根据它们的值对它们进行排序，您应该指定对应于 key 并最终反转的参数:

```
>>> sorted(x.items(), key=lambda item: item[1])
[('v', 1), ('w', 2), ('u', 4)]
>>> sorted(x.items(), key=lambda item: item[1], reverse=True)
[('u', 4), ('w', 2), ('v', 1)]
```

# 16.原始格式化字符串

PEP 498 和 Python 3.6 引入了所谓的格式化字符串或 f 字符串。您可以在这样的字符串中嵌入表达式。把一个字符串既当作原始的又当作格式化的是可能的和直接的。您需要包括两个前缀:fr。

```
>>> fr'u \ n v w={2 + 8}'
'u \\ n v w=10'
```

# 17.获取当前日期和时间

Python 有一个内置的模块 datetime，它是处理日期和时间的通用模块。它的方法之一是。now()返回当前日期和时间:

```
>>> import datetime
>>> datetime.datetime.now()
datetime.datetime(2019, 5, 20, 1, 12, 31, 230217)
```

此结果对应于 2019 年 5 月 20 日凌晨 01:12:31。

# 18.获取最大(或最小)元素的索引

Python 不提供直接获取列表或元组中最大或最小元素的索引的例程。幸运的是，(至少)有两种优雅的方法可以做到这一点:

```
>>> x = [2, 1, 4, 16, 8]
>>> max((item, i) for i, item in enumerate(x))[1]
3
```

如果有两个或多个元素具有最大值，此方法将返回最后一个元素的索引:

```
>>> y = [2, 1, 4, 8, 8]
>>> max((item, i) for i, item in enumerate(y))[1]
4
```

要获得第一个匹配项的索引，您需要稍微修改一下前面的语句:

```
>>> -max((item, -i) for i, item in enumerate(y))[1]
3
```

另一种方式可能更优雅:

```
>>> x = [2, 1, 4, 16, 8]
>>> max(range(len(x)), key=lambda i: x[i])
3
>>> y = [2, 1, 4, 8, 8]
>>> max(range(len(y)), key=lambda i: x[i])
3
```

要查找最小元素的索引，请使用函数 min 而不是 max。

# 19.获得笛卡尔积

内置模块 itertools 提供了许多潜在有用的类。其中之一是用于获得笛卡尔积的乘积:

```
>>> import itertools
>>> x, y, z = (2, 8), ['u', 'v', 'w'], {True, False}
>>> list(itertools.product(x, y, z))
[(2, 'u', False), (2, 'u', True), (2, 'v', False), (2, 'v', True),
 (2, 'w', False), (2, 'w', True), (8, 'u', False), (8, 'u', True),
 (8, 'v', False), (8, 'v', True), (8, 'w', False), (8, 'w', True)]
```

# 20.矩阵乘法运算符

PEP 465 和 Python 3.5 引入了矩阵乘法专用的中缀运算符。您可以使用 __matmul__ 、__rmatmul__、和 __imatmul_ 方法为您的类实现它。这就是向量或矩阵相乘的代码看起来有多优雅:

```
>>> import numpy as np
>>> x, y = np.array([1, 3, 5]), np.array([2, 4, 6])
>>> z = x @ y
>>> z
44
```

# 结论

您已经看到了 20 个让它变得有趣而优雅的 Python 技巧和诀窍。还有很多其他的语言特性值得探索。

编码快乐！

![](img/d5ee4c5640193ff931b57af57d9cd1d4.png)

[Duomly —编程在线课程](https://www.duomly.com)

感谢您的阅读！

我们的队友米尔科准备了一篇文章。