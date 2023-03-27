# Julia 和 Python 中的生成器和迭代器

> 原文：<https://itnext.io/generators-and-iterators-in-julia-and-python-6c9ace18fa93?source=collection_archive---------3----------------------->

## 比较了迭代器在 Julia 和 Python 中是如何实现的，以及生成器在迭代器中扮演的不同角色

Python 中经常使用生成器来实现迭代器。Julia 和 Python 都使用生成器实现列表理解。与其写 say `[x*x for x in 1:4]`，我们可以把表达式列表里面的理解力放在一个单独的对象里面:

```
julia> g = (x*x for x in 1:4)
Base.Generator{UnitRange{Int64},getfield(Main, Symbol("##27#28"))}(getfield(Main, Symbol("##27#28"))(), 1:4)
```

你可以在 Julia 中看到一个`Generator`对象，在 python 中看到一个`generator`对象:

```
python> g = (x*x for x in range(1,5))
python> g
<generator object <genexpr> at 0x10bdeef48>
```

虽然看起来很相似，但它们有很大的不同。Python 生成器是基于协程的，而 Julia 生成器是建立在迭代器之上的。

这就把我们带到了迭代器。我们需要理解它们在两种语言中的作用和实现。

## Python 迭代器

我将做一个有点傻的例子，向你展示如何迭代一个点类的坐标。

```
**class** Point:
    **def** __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z
```

这段代码将允许我们编写:

```
python> p = Point(3, 8, 2)
```

然而，如果我们想为它创建一个迭代器，我们也需要添加这个方法:

```
**def** __iter__(self):
        return PointIterator(self)
```

允许我们在 for 循环中迭代我们的点。为了方便起见，我们也将添加这个方法。它给出了 REPL 中点的字符串表示。

```
**def** __repr__(self):
        return f"Point({self.x}, {self.y}, {self.z})"
```

我们正在构建的功能的实质是在我们返回的迭代器类中。

```
**class** PointIterator:
    **def** __init__(self, point):
        self.point = point
        self.coords = ['z', 'y', 'x']

    **def** __next__(self):
        **if** self.coords:
            return getattr(self.point, self.coords.pop())
        **else**:
            **raise** StopIteration
```

其工作方式是，我们保存一个坐标 x，y 和 z 的列表，当我们迭代该点时，我们要依次访问这些坐标。

`__next__`在每次迭代时被调用，我们从坐标列表中弹出一个坐标来获取。使用`getattr`,我们可以通过名字获得 Python 对象的属性。

通过这个实现，我们可以使用迭代器:

```
python> p = Point(2, 3, 8)
python> for a in p: 
   print(a)

2
3
8

python> it = iter(p)
python> next(it)
2
python> next(it)
3
python> next(it)
8
```

让我们比较一下在进入生成器之前，迭代器是如何在 Julia 中产生的。

## 朱莉娅迭代器

Julia 有一种完全不同的方法来实现迭代器。抽象地说，人们可以说，两种语言都需要实现某种协议来使某些东西可迭代。然而，Julia 并不是以面向对象的方式来做的。没有可以子类化的类，取而代之的是必须实现新版本的`iterate`函数。

首先让我们制作`Point`型。不需要实现`repr`函数来显示类型。它有一个自动表示。

```
**struct** Point
    x::Int
    y::Int
    z::Int
**end**
```

它还有一个默认的构造函数，所以我们也不需要实现它。Julia 中的 for 循环如下所示:

```
**for** i **in** iter   # or  "for i = iter"
    # body
**end**
```

Julia 将其翻译成以下代码:

```
next = iterate(iter)
**while** next !== **nothing**
    (i, state) = next
    # body
    next = iterate(iter, state)
**end**
```

所以要迭代一个点的坐标，我们需要实现两个版本的`iterate`。在 Julia 的术语中，这些是方法，但是对于具有面向对象背景的人来说，这可能会令人困惑。方法是函数在 Julia 中的所有具体实现。所以在 Julia 术语中，我们实现了`iterate`函数的两个方法。

```
iterate(p::Point) = p.x, [:y, :z]

**function** iterate(p::Point, coords)
   **if** isempty(coords)
       nothing
   **else**
       getfield(p, coords[1]), coords[2:end]
   **end**
**end**
```

所以和 Python 中的`__next__`有一些相似之处。Python 谈论对象的属性，而 Julia 谈论对象的字段。因此`getattr`和`getfield`基本上做同样的事情。`getattr`用字符串名指代属性，Julia 用符号。符号与字符串非常相似。如果你想更好地理解为什么 Julia 经常使用符号而不是字符串，这里有一个很好的答案。`"foobar"`是字符串，而`:foobar`是符号。

另一个重要的区别是，Julia 在迭代时不会对任何数据进行突变。看看这个在 Python 中使用`enumerate`的例子:

```
python> it = enumerate(["one", "two"])
python> list(it)
[(0, 'one'), (1, 'two')]
python> list(it)
[]
```

你可以看到迭代器被耗尽了。你不能一直从迭代器中创建列表。但是在 Julia 中可以，因为迭代的状态与迭代器本身是分开的:

```
julia> it = enumerate(["one", "two"])
Base.Iterators.Enumerate{Array{String,1}}(["one", "two"])

julia> collect(it)
2-element Array{Tuple{Int64,String},1}:
 (1, "one")
 (2, "two")

julia> collect(it)
2-element Array{Tuple{Int64,String},1}:
 (1, "one")
 (2, "two")
```

## 使用 Python 生成器实现迭代器

如你所见，在 Python 中实现迭代器相当冗长，这就是为什么在 Python 中实现迭代器的首选方法是使用生成器。

```
**class** Point:
    **def** __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

    **def** __repr__(self):
        **return** f"({self.x}, {self.y}, {self.z})"

    **def** __iter__(self):
        **def** gen():
            **yield** self.x
            **yield** self.y
            **yield** self.z
        **return** gen()
```

你看，使用这种方法，我们根本不需要实现一个`PointIterator`类。`yield`与 return 的不同之处在于，当在协程的上下文中运行时，它会“记住”退出时它在函数中的位置。协程将从它停止的地方重新进入该函数。这在处理迭代时非常实用。让我再举一个例子来说明这一点。

```
**def** generator():
    **for** i **in** range(0, 6):
        **yield** chr(ord('A') + i)
```

让我告诉你如何在 REPL 中使用它来产生从 A 到 f 的字母

```
python> g = generator()
python> list(g)
['A', 'B', 'C', 'D', 'E', 'F']
python> g = generator()
python> next(g)
'A'
python> next(g)
'B'
python> next(g)
'C'
```

这里的巧妙之处在于，你不需要编写代码来维护关于*在 for 循环中的位置的信息。生成器会帮你记录下来。*

## Julia 中的 Python 风格生成器

如果您想创建一个功能类似 Python 生成器的生成器，您必须利用 Julia 中的通道。这可能看起来有点神秘，但是不要担心，我会在最后详细说明这是如何工作的。

```
**function** generator()
    Channel() **do** channel
        **for** i in 0:5
            put!(channel, Char('A' + i))
        **end**
    **end**
**end**
```

这不会像理解一样返回一个`Generator`对象，而是一个`Channel`对象:

```
julia> g = generator()
Channel{Any}(sz_max:0,sz_curr:1)
```

然而，我们可以像使用常规迭代器一样使用这个通道。例如，我们可以收集值并组成一个数组，就像迭代器一样。

```
julia> collect(g)
6-element Array{Any,1}:
 'A'
 'B'
 'C'
 'D'
 'E'
 'F'
```

或者我们可以使用 unpack 语法来创建一个数组。

```
julia> g = generator()
julia> [g...]
6-element Array{Char,1}:
 'A'
 'B'
 'C'
 'D'
 'E'
 'F'
```

将作为参数返回的字符解包为 string，用于将所有汽车连接成一个字符串。

```
julia> g = generator()
julia> string(g...)
"ABCDEF"
```

## 了解任务和渠道的详细信息

我们在代码示例中使用的`Channel`构造函数实际上做了很多事情，所以你看不到它如何工作的所有细节。

这里更详细地说明了正在发生的事情:

```
**function** generator()
   channel = Channel(2)
   **function** make_letters()
       **for** i **in** 0:5
           put!(channel, Char('A' + i))
       **end**
   **end**
   task = Task(make_letters)
   schedule(task)
   channel
**end**
```

你可以把协程想成有点像线程。像线程一样，您可以拥有许多线程，并且可以使用不同的数据结构在它们之间进行通信。通道用于在协同程序之间发送数据。然而，与线程不同的是，只有一个协程运行。不能在不同的 CPU 内核上运行多个协同程序。他们像合作的多任务处理一样工作。调度程序切换到另一个协程，因为当前的协程明确放弃控制权。

例如，当我写道:

```
put!(channel, Char('A' + i))
```

一个字母将被放入通道，如果通道有缓冲区，函数将立即返回。我们指定通道的缓冲区为 2。

一旦缓冲区耗尽，我们当前的协同程序将阻塞，调度程序将选择一个协同程序(任务)等待来自这个特定通道的数据。

```
task = Task(make_letters)
```

这将创建一个任务，这就是我们如何跟踪不同的协程。每个协程都需要一个任务。该任务跟踪执行状态。

然而此时，我们的协程实际上并没有运行。for 循环没有将字母放入通道。为了让它运行，我们需要`schedule`这个任务。

```
schedule(task)
```

这会将任务放入调度程序队列中。每当系统空闲时，它将在其调度器队列中查找要运行的任务。这样做的直接结果是我们的任务开始运行，这导致‘A’和‘B’被放入通道。但是当循环试图放入“C”时，它已经用完了缓冲区并阻塞了。

然后，调度程序将寻找另一个任务来运行。我相信我们可以把 REPL 本身看作是调度程序可以切换到的一个主协程。所以说我救了从`generator`回来的`channel`，我们可以这样做:

```
julia> channel = generator()
julia> take!(ch)
'A'
```

之后，REPL 进入空闲状态，等待您的输入。然后调度程序有机会捡起我们的旧任务并继续运行它。一个字母已从缓冲区中移除，因此它现在有机会将“C”放入通道中。

为了避免设置和运行协程任务的样板文件，有许多快捷方式和宏来帮助我们。这里有一个选择:

```
**function** generator()
   channel = Channel(2)
   @async begin
       **for** i **in** 0:5
           put!(channel, Char('A' + i))
       **end**
   **end**
   channel
**end**
```

`@async`宏负责创建一个函数，将其封装在`Task`中，并调度该任务。它将返回任务对象，但我们不需要为任何事情存储它。

`Channel`构造函数既可以接受一个给出其容量的整数，也可以接受一个函数对象。在后一种情况下，它将把提供的函数对象包装在一个`Task`中并调度它。

```
**function** generator()
   **function** make_letters(channel::Channel)
       **for** i **in** 0:5
           put!(channel, Char('A' + i))
       **end**
   **end
**   Channel(make_letters)
**end**
```

这可能会让我们更清楚最初的例子是如何工作的，在那里我们使用了`do`语法。记住`do`的语法类似于:

```
foobar(x -> stuff(x), y, z)
foobar(y, z) **do** x
 stuff(x)
**end**
```

这是 Python 生成器中隐藏的东西。实际上，在 Python 中可以做类似的事情来处理并发性。我将在后面的故事中探讨这个问题。