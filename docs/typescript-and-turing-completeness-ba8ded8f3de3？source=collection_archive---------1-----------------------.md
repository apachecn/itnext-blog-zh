# 类型脚本和图灵完备性

> 原文：<https://itnext.io/typescript-and-turing-completeness-ba8ded8f3de3?source=collection_archive---------1----------------------->

![](img/48c85ebd0dd41124262b72afdfe3ef01.png)

照片由 [Siora 摄影](https://unsplash.com/@siora18?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

*在整篇文章中，我们会看到* `*SomeGeneric<...> extends number ? ... : never*` *的一个常见构造。这是因为我们的一些泛型类型的操作方式使得 TypeScript 无法确定结果是数值型的，尽管我们知道结果是数值型的。如果没有这个检查，我们就不能将* `*SomeGeneric<...>*` *传递给另一个需要数字的泛型类型。这个类型保护允许我们向类型检查器提供这些信息。下面代码中的任何使用仅仅是对 TypeScript 的安抚，而不是我们试图构建的逻辑需求。*

在之前的一篇文章中，我展示了如何在 TypeScript 中使用类型系统来产生一个基本的算术系统。一个自然的问题是，从这个系统或基于它的更复杂的系统中，什么是可以计算的，什么是不可以计算的。换句话说:我们的类型系统可以计算或不计算什么？它能计算任何东西吗？还是在某个计算方面有先天限制？

一旦我们开始问这类问题，我们就进入了可计算性理论的领域，更具体地说，就是图灵完备性。我不会对“图灵完备性”这个短语的含义进行详尽的讨论，我只会给出一个与文章其余部分相关的高层次概述，并让读者参考另一篇文章[更深入地讨论这个问题。](https://evinsellin.medium.com/what-exactly-is-turing-completeness-a08cc36b26e2)

概括地说，当我们说一个系统是图灵完备的时，我们的意思是它可以处理任何可以用有限数量的输入在有限数量的步骤中描述的计算。本质上，它足够强大，可以对算法进行一般性编码。这样的系统可以是通用编程语言、图灵机、lambda 演算和细胞自动机。指定和执行这种算法的方式可能看起来非常不同，因为我们可以从以下事实中看出，我们所有的示例本身看起来彼此非常不同。

大多数编程语言都被认为是图灵完全的，甚至一些类型系统[(比如 Rust 的)](https://sdleffler.github.io/RustTypeSystemTuringComplete/)也被证明是图灵完全的。那么，TypeScript 呢？显然，TypeScript 的 JavaScript 部分是图灵完全的，所以我们将只关注 TypeScript 的类型系统(因为它不能从 JavaScript 继承可计算性),并表明它确实也是图灵完全的。

为了证明这一点，我们可以采取三种通用方法:

1.  证明任何图灵机都可以翻译成 TypeScript。
2.  说明 TypeScript 可以实现一个[通用图灵机](https://en.wikipedia.org/wiki/Universal_Turing_machine)。
3.  表明 TypeScript 可以实现另一个已知的图灵完备系统。

我们将采取的方法是第三种。我们将通过为称为 [Smallfuck](https://esolangs.org/wiki/Smallfuck) 的图灵完全深奥编程语言实现一个解释器来做到这一点。

# 小混蛋

在架构上，Smallfuck 在有限长度的位磁带上运行(初始化为所有的`0`),能够一次扫描该磁带上的一位。由于这种架构，它的操作非常少；事实上，它的语言中正好有五个符号，其中三个对应于涉及该磁带的动作，而其余两个涉及基于当前位的值有条件地绕过程序的指令集。

这些符号是:

*   `*`:翻转当前位
*   `<`:向左移动一点
*   `>`:右移一位
*   `[`:如果当前位为`0`，则向前跳一位超过匹配的`]`
*   `]`:向后跳转到匹配`[`

此外，当 Smallfuck 解释器执行`<`或`>`操作时，它还必须考虑磁带的边界。当它在磁带的开头时不能向左，当它在结尾时不能向右。任何此类尝试将导致该计划立即终止。

我们将创建一小组有代表性的程序来说明这种语言的各种特性，并且我们需要我们的解释器来正确地执行这些程序。这些方案是:

*   `*>*>*`:连续翻转三位，终止于最后翻转的位
*   `>*>*>*[*<]`:将连续三位翻转到`1`，然后在这些位上向后循环，并翻转回`0`
*   `*[]`:将该位翻转到`1`，然后在两个括号字符之间无限循环——该程序永远不会终止
*   `*[>*]`:在磁带上循环，当它超出磁带边界时，在终止之前将每个位翻转到`1`
*   `*[>>*[<*]]`:翻转一个位，然后进入一个循环，在进入一个子循环以向后翻转位之前绕着磁带移动

为了阐明我们自己的解释器正确地评估一个任意的程序所必须采取的步骤，给出一个完整的一步一步的程序评估是有帮助的。为此，我们将选择上面列出的第二个选项(`>*>*>*[*<]).`

```
1\. Initialize a tape with the first bit and first instruction "in scope" v                                         v
Tape: 00000000                    Instructions: >*>*>*[*<]2\. Evaluate `>` and move to next instruction v                                         v
Tape: 00000000                    Instructions: >*>*>*[*<]3\. Evaluate `*` and move to next instruction v                                          v
Tape: 01000000                    Instructions: >*>*>*[*<]4\. Evaluate `>` and move to next instruction v                                          v
Tape: 01000000                    Instructions: >*>*>*[*<]5\. Evaluate `*` and move to next instruction v                                           v
Tape: 01100000                    Instructions: >*>*>*[*<]6\. Evaluate `>` and move to next instruction v                                           v
Tape: 01100000                    Instructions: >*>*>*[*<]7\. Evaluate `*` and move to next instruction v                                            v
Tape: 01110000                    Instructions: >*>*>*[*<]8\. Evaluate `[`. Since the current bit is not `0`, move to next instruction v                                             v
Tape: 01110000                    Instructions: >*>*>*[*<]9\. Evaluate `*` and move to next instruction v                                              v
Tape: 01100000                    Instructions: >*>*>*[*<]10\. Evaluate `<` and move to next instruction v                                                v
Tape: 01100000                    Instructions: >*>*>*[*<]11\. Evaluate `]` and move to matching `[` v                                             v
Tape: 01100000                    Instructions: >*>*>*[*<]12\. Evaluate `[`. Since the current bit is not `0`, move to next instruction v                                              v
Tape: 01100000                    Instructions: >*>*>*[*<]13\. Evaluate `*` and move to next instruction v                                               v
Tape: 01000000                    Instructions: >*>*>*[*<]14\. Evaluate `<` and move to next instruction v                                                 v
Tape: 01000000                    Instructions: >*>*>*[*<]15\. Evaluate `]` and move to matching `[` v                                              v
Tape: 01000000                    Instructions: >*>*>*[*<]16\. Evaluate `[`. Since the current bit is not `0`, move to next instruction v                                               v
Tape: 01000000                    Instructions: >*>*>*[*<]17\. Evaluate `*` and move to next instruction v                                                v
Tape: 00000000                    Instructions: >*>*>*[*<]18\. Evaluate `<` and move to next instruction v                                                  v
Tape: 00000000                    Instructions: >*>*>*[*<]19\. Evaluate `]` and move to matching `[` v                                               v
Tape: 00000000                    Instructions: >*>*>*[*<]20\. Evaluate `[`. Since the current bit is `0`, move to next instruction after matching `]` v                                                   v
Tape: 00000000                    Instructions: >*>*>*[*<]
```

20 步后，我们的程序消耗完所有指令(由试图查看一条不存在的指令的指令指针指示)并暂停。然而，停止并不是图灵完备性的必要条件，我们可以使用同样的逐步技术来证明我们的第三个程序(`*[]`)永远不会停止。

# Smallfuck 解释器

## 预赛

在我们开始实现解释器的内部之前，反思一下上面的评估，了解一下我们想要构建的东西，会对我们有所帮助。

我们可以看到，为了正确评估 Smallfuck 程序，我们需要跟踪四条信息:

*   磁带的状态
*   磁带上的哪个位当前在范围内
*   指令集
*   当前正在执行哪条指令

我们可以很自然地使用元组来表示磁带和指令。这允许我们将指向当前位和指令的指针表示为数字。

这四条信息组成了我们的执行状态，我们的主要任务是实现上面列出的五个动作中的每一个。但是，如果没有一些实用程序操作作为基础，这些操作都无法实现。因此，我们将首先设置这些先决条件，并在它们的基础上构建核心解释器。这些是我们需要支持的初步行动:

*   递增一个数字(一次或直到满足终止条件)
*   递减一个数字(一次或直到满足终止条件)
*   分割数组
*   翻转一点(既包括一般翻转，也包括磁带上特定位置的翻转)

递增/递减数字与我们在上一篇关于实现算术的文章中所涉及的非常相似，因此我们将把它们放在这里，不做过多解释。

```
type ArrayFromNumber<
    N extends number, 
    A extends any[] = []
> = A['length'] extends N ? A : ArrayFromNumber<N, [...A, any]>;type Increment<T extends number> = 
    [...ArrayFromNumber<T>, any]['length'];type Decrement<T extends number> = 
    ArrayFromNumber<T> extends [...(infer U), any] 
        ? U['length'] 
        : never;
```

这处理了执行已知数量的递增/递减操作的简单情况，但没有处理任意数量的情况。毕竟，如果不简单地扫描中间的每个字符，就无法判断一个`]`比它匹配的`[`多多少个字符。因此，我们将编写一个通用的倒带和快进操作。

```
type Rewind<
    Arr extends any[],
    I extends number
> = Arr[I] extends '[' ? I : Rewind<Arr, Decrement<I>>;type FastForward<
    Arr extends any[],
    I extends number
> = Arr[I] extends ']'
    ? Increment<I>
    : Increment<I> extends number
        ? FastForward<Arr, Increment<I>>
        : never;
```

这两种类型接受一个数组(这将是我们的指令集)和一个指向数组中当前指令的索引。接下来会发生的是，我们将询问在`Arr[I]`的任何指令是否等于终止字符，如果是，我们返回索引号。如果没有，我们将索引移动一个位置，再试一次。

请注意，`FastForward`给出了一个比多一个*的数字，它终止于该字符的位置。因此，如果它将程序`[***]>*`从位置 1 (= `[`)推进到位置 5 (= `]`)，它将返回位置 6 (= `>`)。这是因为如果我们被给定位置 5，我们的解释器的下一次执行将把我们倒回到开始的括号，我们将在两者之间无限循环，而不是像预期的那样跳出循环。*

不过，这两种类型还不足以满足我们的需求。Smallfuck 程序可能有嵌套循环(如`*[>*[>*]>*]*`)，如果我们使用刚刚构建的内容从第一个`[`前进到其匹配的`]`，我们将前进到`]`的第一个出现的*，而不是匹配*出现的*。因此，我们将从位置 2 的起点前进到位置 8，而不是位置 11。这意味着我们需要增加我们的`Rewind`和`FastForward`类型来处理深度。*

```
type Rewind<
    Arr extends any[],
    I extends number,
    Depth extends number = 0
> = Arr[I] extends '['
    ? Depth extends 0
        ? I
        : Rewind<Arr, Decrement<I>, Decrement<Depth>>
    : Arr[I] extends ']'
        ? Increment<Depth> extends number
            ? Rewind<Arr, Decrement<I>, Increment<Depth>>
            : never
        : Rewind<Arr, Decrement<I>, Depth>;type FastForward<
    Arr extends any[],
    I extends number,
    Depth extends number = 0
> = Arr[I] extends ']'
    ? Depth extends 0
        ? Increment<I>
        : Increment<I> extends number
            ? FastForward<Arr, Increment<I>, Decrement<Depth>>
            : never
    : Arr[I] extends '['
        ? Increment<Depth> extends number
            ? Increment<I> extends number
                ? FastForward<Arr, Increment<I>, Increment<Depth>>
                : never
            : never
        : Increment<I> extends number
            ? FastForward<Arr, Increment<I>, Depth>
            : never;
```

我们可以看到，计算深度增加了我们操作的复杂性。然而，总的要点是这样的:当遍历命令数组时，我们寻找一个特定的符号来终止。如果我们看到那个符号，我们必须确保我们是深度为 0(这表明这个符号是适当的匹配符号)。如果我们在深度 0 不是 T21，我们继续移动，但是我们从深度减去 1。如果所有的括号都适当地平衡，当我们遇到一个终止符号时，我们将最终到达深度 0。然而，如果我们遇到了终止符号的反义词，我们就进入了一个嵌套循环，因此我们需要将深度增加 1，以免在遇到结束这个嵌套循环的终止符号时过早停止。

因为 TypeScript 的类型系统中没有赋值操作，所以我们不能做类似于`Arr[I] = SomeType`的事情。如果我们想在数组的某个索引`I`处改变类型，我们需要创建一个全新的数组，其中`I`左边的所有内容保持不变，第`I`个位置更新，然后`I`右边的所有内容保持不变。这种限制使得修改数组类型的方式变得冗长，但是使用`Slice`实用程序，我们可以使它变得优雅。

```
type Slice<
    T extends any[],
    S extends number = 0,
    E extends number = T['length'],
    A extends any[] = []
> = S extends E
    ? A
    : Increment<S> extends number
        ? Slice<T, Increment<S>, E, [...A, T[S]]>
        : never;
```

`Slice`将原始数组`T`、起始索引`S`、结束索引`E`和累积数组`A`作为参数。`Slice`然后将检查我们的开始索引是否等于我们的结束索引，如果是，则返回累积数组。否则，它增加我们的起始位置，将`T[S]`处的项目添加到我们的累加器数组，并使用那些更新的值再次递归检查。

我们要做的最简单的动作之一就是翻转一下。

```
type FlipBit<B extends Bit> = B extends 0 ? 1 : 0;
```

因此，要在磁带的任何位置翻转一点，我们只需利用 get 上面的`Slice`操作

```
type FlipBitAtPos<T extends Tape, P extends number> =
    Increment<P> extends number
        ? [
            ...Slice<T, 0, P>, 
            FlipBit<T[P]>,
            ...Slice<T, Increment<P>>
          ]
        : never;
```

## 编写解释器

现在，是时候开始建立起那些起重要作用的部分了。

有两种类型将作为解释器的必要构件:

```
type Command = '>' | '<' | '*' | '[' | ']'
type Bit = 0 | 1;
type Tape = Bit[];
```

我们设计解释器的方式是，我们希望它接收一个由`Command`组成的数组，我们希望在给定的`0`磁带上执行该数组。评估的每一步都将操作磁带(或我们在磁带上的位置),并将指针移动到下一个要执行的命令。每个评估的结果将是我们的命令和磁带的组合状态，我们将递归地调用我们的评估器，直到我们到达评估必须停止的点。

所以我们的最后一块积木将会是

```
type ExecutionState = [
    commands: Command[],
    commandPosition: number,
    tape: Tape,
    tapePosition: number
];type GetCommands<T extends [any, ...any[]]> = T[0];
type GetCommandPtr<T extends [any, any, ...any[]]> = T[1];
type GetTape<T extends [any, any, any, ...any[]]> = T[2];
type GetTapePtr<T extends [any, any, any, any, ...any[]]> = T[3];
```

因为`ExecutionState`是一个 4 元组，所以我们也有一些方便的 getters 来提取该元组的特定部分。

使用这些，让我们构建解释器，从面向用户的入口点开始，向后工作到执行评估的机制。

```
type Smallfuck<
    CS extends Command[],
    T extends Tape
> = Evaluate<[CS, 0, T, 0]>;
```

`Smallfuck`将获取一个要评估的命令列表和一个固定长度的磁带，然后构建一个初始`ExecutionState`以传入`Evaluate`。

本身将充当我们的程序运行程序。它将获取程序的当前状态，执行该程序，获取该执行的结果，并无限地再次执行*直到最后一条指令被执行。`Evaluate`会是什么样子*

```
type Evaluate<S extends ExecutionState> = 
    GetCommands<S>[GetCommandPtr<S>] extends undefined
        ? S
        : Execute<
              GetCommands<S>,
              GetCommandPtr<S>,
              GetTape<S>,
              GetTapePtr<S>
          > extends ExecutionState
              ? Evaluate<
                    Execute<
                        GetCommands<S>,
                        GetCommandPtr<S>,
                        GetTape<S>,
                        GetTapePtr<S>
                    >
                >
              : never;
```

递归类型的基本情况是，要执行的命令的索引超出了命令列表的范围。这代表了我们的解释器在上面第 20 步中所处的情况，它试图读取一个根本不存在的命令。

因为我们的`Execute`类型相当复杂，TypeScript 不能正确地推断出它的返回类型是一个`ExecutionState`类型，所以我们必须添加`extends ExecutionState`检查，原因与我们有时必须进行本文开头讨论的`extends number`检查相同。

我们的`Execute`类型将根据每个命令对我们的`ExecutionState`进行适当的修改。本质上，它所要做的是获取当前命令，并通过一系列 case 语句对我们的执行状态进行适当的更改，然后返回更新后的版本。

命令`*`、`<`、`>`和`]`所需的操作都相当简单，因此我们将列出部分`Execute`类型，然后讨论对`[`命令的特殊处理。

```
type Execute<
    CS extends Command[],
    CPos extends number,
    T extends Tape,
    TPos extends number
> =
    CS[CPos] extends '*'
        ? [CS, Increment<CPos>, FlipBitAtPos<T, TPos>, TPos]
        :
    CS[CPos] extends '>'
        ? [CS, Increment<CPos>, T, Increment<TPos>]
        :
    CS[CPos] extends '<'
        ? [CS, Increment<CPos>, T, Decrement<TPos>]
        :
    CS[CPos] extends ']'
        ? [CS, Rewind<CS, Decrement<CPos>>, T, TPos]
        :
    never;
```

每当我们看到一个命令，我们总是需要增加指向下一个命令的命令指针。其他操作只是简单地执行简单的动作，在磁带上的某一点翻转一点，增加/减少磁带的指针，或者返回命令列表以找到匹配的`[`命令。

然而，我们对`>`和`<`的实现并不完整。请记住，因为我们的磁带是有限的，所以有可能试图超越它的界限。在这种情况下，我们的计划应该停止。我们需要用一个检查来增加这两个命令，看看我们是否将要越界，如果是，立即将我们的命令指针移到越界，而不改变磁带的任何内容。这将触发`Evaluate`类型中止任何更多的执行。

```
type Execute<
    CS extends Command[],
    CPos extends number,
    T extends Tape,
    TPos extends number
> =
    CS[CPos] extends '*'
        ? [CS, Increment<CPos>, FlipBitAtPos<T, TPos>, TPos]
        :
    CS[CPos] extends '>'
        ? TPos extends Decrement<T['length']> 
            ? [CS, CS['length'], T, TPos] 
            : [CS, Increment<CPos>, T, Increment<TPos>]
        :
    CS[CPos] extends '<'
        ? TPos extends 0
            ? [CS, CS['length'], T, TPos]
            : [CS, Increment<CPos>, T, Decrement<TPos>]
        :
    CS[CPos] extends ']'
        ? [CS, Rewind<CS, Decrement<CPos>>, T, TPos]
        :
    never;
```

最后一个命令(我们的`[`指令)是一个条件快进。所以我们必须实现逻辑来确定解释器是一直跳到匹配的`]`命令，还是直接跳到下一个命令。我们称之为我们的`MaybeFastForward`类型。

```
type MaybeFastForward<
    CS extends Command[],
    CPos extends number,
    T extends Tape,
    P extends number
> = T[P] extends 0 ? FastForward<CS, CPos> : CPos;
```

如果我们当前的位是`0`，我们不进入循环，所以需要前进通过我们匹配的`]`。否则我们 *do* 进入循环，并转到该循环中的第一个命令。注意，如果我们进入循环，我们只是返回`CPos`，而不是增加它。我们将在下面看到原因。

我们最终的`Execute`型号将是

```
type Execute<
    CS extends Command[],
    CPos extends number,
    T extends Tape,
    TPos extends number
> =
    CS[CPos] extends '*'
        ? [CS, Increment<CPos>, FlipBitAtPos<T, TPos>, TPos]
        :
    CS[CPos] extends '>'
        ? TPos extends Decrement<T['length']>
            ? [CS, CS['length'], T, TPos]
            : [CS, Increment<CPos>, T, Increment<TPos>]
        :
    CS[CPos] extends '<'
        ? TPos extends 0
            ? [CS, CS['length'], T, TPos]
            : [CS, Increment<CPos>, T, Decrement<TPos>]
        :
    CS[CPos] extends ']'
        ? [CS, Rewind<CS, Decrement<CPos>>, T, TPos]
        :
    CS[CPos] extends '['
        ? Increment<CPos> extends number
            ? [
                  CS,
                  MaybeFastForward<CS, Increment<CPos>, T, TPos>,
                  T,
                  TPos
              ]
            : never
        : never;
```

我们可以在`[`的实现中看到，当我们将`CPos`传递给`MaybeFastForward`时，我们已经增加了它的值(因此我们从那个动作中返回了它，没有改变)。这是因为我们如何追踪深度；如果我们从刚刚执行的同一个命令(*即*)开始快进。我们会立即给我们的日常工作增加一个深度。这将导致快进操作跳过它应该寻找的实际匹配括号`]`。

## 测试解释器

不幸的是，当评估一个程序时，不可能看到 TypeScript 采取的一步一步的动作，但是我们至少可以将我们解释器的磁带输出与另一个解释器的进行比较。我们将使用上面列出的程序，或者比较磁带的最终状态(对于那些终止的程序)，或者确定两者是否都产生某种错误/无限循环(对于那些不会终止的程序)。

我们用作比较的外部解释器将是[这个](https://gist.github.com/cnexans/45276bf6f8e1954ed14de6b5e8ad71c2)(有趣的是用 TypeScript 编写——只是不完全在它自己的类型系统中)。它公开了一个名为`interpreter`的函数，该函数为初始磁带获取一串命令和一串零。当我们的解释器返回整个执行上下文时，我们可以通过用一个`GetTape<...>`调用包装每个`Smallfuck`调用来轻松地获取磁带进行评估。

```
/**
 * Evaluate '*>*>*' on tape 000000
 */
type Test1 = GetTape<
    Smallfuck<
        ['*', '>', '*', '>', '*'],
        [0, 0, 0, 0, 0, 0]
    >
>; // [1, 1, 1, 0, 0, 0]const test1 = interpreter('*>*>*', '000000'); // '111000'/**
 * Evaluate '*[]' on tape 000000
 */
type Test2 = GetTape<
    Smallfuck<
        ['*', '[', ']'],
        [0, 0, 0, 0, 0, 0]
    >
>; // Type instantiation is excessively deep and possibly infinite.const test2 = interpreter('*[]', '000000'); // Environment hangs/**
 * Evaluate '*[>*]' on tape 000000
 */
type Test3 = GetTape<
    Smallfuck<
        ['*', '[', '>', '*', ']'],
        [0, 0, 0, 0, 0, 0]
    >
>; // [1, 1, 1, 1, 1, 1]const test3 = interpreter('*[>*]', '000000'); // '111111'/**
 * Evaluate '>*>*>*[*<]' on tape 000000
 */
type Test4 = GetTape<
    Smallfuck<
        ['>', '*', '>', '*', '>', '*', '[', '*', '<', ']'],
        [0, 0, 0, 0, 0, 0]
    >
>; // [0, 0, 0, 0, 0, 0]const test4 = interpreter('>*>*>*[*<]', '000000'); // '000000'/**
 * Evaluate '*[>>*[<*]]' on tape 000000
 */
type Test5 = GetTape<
    Smallfuck<
        ['*', '[', '>', '>', '*', '[', '<', '*', ']', ']'],
        [0, 0, 0, 0, 0, 0]
    >
>; // [0, 1, 1, 0, 0, 0]const test5 = interpreter('*[>>*[<*]]', '000000'); // '011000'
```

# 评论

我们可以看到，对于上面的示例程序，我们的解释器的输出与另一个解释器的输出相匹配。虽然这与证明这两者总是给出相同的输出相差甚远，但它给了我们一些验证，证明我们已经正确地编写了我们的解释器。幸运的是，Smallfuck 是一种简单的语言，所以如果我们能让它为一些说明该语言主要特征的程序工作，我们就不太可能找到一个更复杂的情况来破坏解释器。

我们对 Smallfuck 的实际实现完全是在 TypeScript 的类型系统中，这意味着我们永远无法确定任何`Type instantiation is excessively deep and possibly infinite.`错误是否表示一个无限循环的程序。由于递归类型对于类型检查器来说有性能问题，因此 TypeScript 有严格的递归限制，防止计算非常递归的类型。在我们的情况下，这意味着一旦达到递归极限，实际上无限的程序*和足够复杂的*程序都将终止。

本文从 TypeScript 是否有图灵完整类型系统的问题开始。我们已经通过 Smallfuck 解释器的实现表明答案是肯定的，尽管有一些限制。

对于初学者来说，TypeScript 有一个递归限制，因此我们永远无法评估复杂的程序。此外，即使没有这种限制，TypeScript 也有与图灵机中不存在的任何其他编程语言相同的限制。有限的内存和身体寿命使得它不可能执行一个需要一百万年才能完成的程序。然而，这些限制是*实现*的限制，并不是我们的解释器的逻辑所固有的，所以如果我们可以取消它们，我们就会有一个真正的*图灵完整类型系统。*

我们证明 TypeScript 可以计算任何东西的一个有趣的特征是，它的计算框架看起来一点也不像我们在以前的文章中实现的算术系统(它执行我们习惯看到的更多“函数”计算)。这是因为“计算”实际上表示一个抽象的术语，并且有许多不同的具体方法来计算同一事物。我们的解释器采用这些具体方法中的一种(在我们的例子中，类似于图灵机)并赋予它实体；如果我们决定实现递归函数或 lambda 演算，它看起来会有所不同。

这也不是在 TypeScript 中证明图灵完备性的唯一尝试。几年前有一个更早的尝试来证明这一点。我们的证明看起来一点也不像那个，因为我们在我们的证明中利用了五年前不可用的 TypeScript 的最新功能。

这里有一个链接，链接到完全实现了解释器的 [TypeScript playground](https://www.typescriptlang.org/play?ssl=25&ssc=87&pln=9&pc=1#code/PTBQIAgWh2og4gUwHZIE4EMA2ECuALgJbZHFIDO0M4YBAngA5IQCC6W9AYugPYC2AOTz8ARhgA8giEgAeBVABMqKEePQAaNjPlKqmFPQDaAXQgBeCKYB8FtkYDk2VAHMCACwdm5ClMojSAPzaAFxsHJjcfEJqkoJaRgB0yaxaBvQm1gDcoKAMzBAAkigAxuhI-KgEEgAqOr7+qmIYtpZJKRFRAsLN6LXWaYYmjs4obp4mOfksACJIZRVVtfV6EE3qreGcPN2xfTW2PqvtiQAURCgAZhgQAKoAlIMZEMG3I64eXhBhaABuGDk8kwWAAlJAAdwuigk7HQKz8+kMpi0hXhjT2WjmjA8aJUezsAAZNrCjIVvLoERAHI5QBA6S8IFicUdKQSGaiwmDIX4YRxMfNypUUNVCgNGQLFsKJEz3NZrLT6WESWTcVSTA4FfT6cFigshdUZYcKejegyuVDeZpxXqlqKUaVBUtDbYfkh-uhNVrvhBzTzYfybVK7eLsbLAdMIFxMBQCFxeOhwZh0NDYar0siiqr1hh+aGs-jLES7MryQ0qF4NVrgjLVQTPVWig7JSL5V7FY3Ay386bglGY3GE0mU3yO46g2K5p3pUhQ3LvX8MJ6lRxSaXVtTK232U39dPZ931PWtzqd7ajWW1nsj1vtZHo7H44nk5b7VPg7qxwaZx45deb-O3UXf92wXD0bzCD9mwkUUDxuXt7wHJ9hytSDd2DQ0APdcNgQgABlUgSiQZYWX8dMTC0XDYLhQstAAUSouwaneMZPnI7QSMRYwzDaTI7EojiIFoz1glYJdRyg3Dz1WbM4WCfCiEI2pX0-CRJLohJkkSVIICY3CTF410sNyCMuFIRgACEyAkczVUsghNhsgS2WCABGb0CWwgpTKICyyFYAgAAVeAoYjjSoGpMGYLQAqozZUKWAKpMpGThKsTT5MUmotAJaKxW83zqiYgLMi0dKCKIrLxN3RLrBMMTQMBEBaGoOB4Fw-gcGwS48BKABrfBiFIcgqFgWggQKABhAQOr8OwHGsBwIAAHypCRFpWhwACp1qpRxlrVDUIzswl9pcqYcIigpLDs0xzoKWjZHmQgiF4FBcIITAFDsIxPRKaaDGUMIpv4GbFGRX7-r8IKKDIF6UB+DFPQ+5gwkupANCRyKkGh2HXoR3pQEmYycOQAhgdBkK6gEox0lK5IyN4ywmIJImI1J8mAYCgh9lVGnDCeOnEgZzYmJc1mSaQAg0dCi8+foAWIFpiBNOFxijAAJnFgpSbRrmeeppXDf5xXjZVpF9JFowAGYifGlgAFlInEPsH0HZ9PQm-iwogDm-HBrUJuhqiMa1KnvbRkP6RigSUst4rawZF3EKHCRPa0QPgpdH3ocasBIFamA8I67Aup6-qLgUdBGHKSuWrGiMHqehRU69i9fbBtiM6oGOMR01UI50oOe96VoPdwowu7XSktsW4IJ9w5SoK7vKzLs-zoaUwfM60GpobMEIx4nvfVXmzc22CXfgtVScVKYpwPgmWw57Tn3x-vliJh3nfj7CefF93ZeX8qpLEvhQWq3xD6TxPmtP8F8h7e2clYF+ntmLjC8EA0B+8kEL2AVKQBOkAy32huAg+Adx5QIEhWBkf8fQQgtC-G+S9iFikqpgiBZCj7BSnv4DcsDcHVGXgxZ+ODHb0GdghR8KcX7xTwcQjBciCFbyoFg0C7D6QNWJvdX4OA8CfSIq3VYjcSjPVeu9PRmxPTs0hsoVS1gjBWJBpzbmtjuFUDwH4JAlwLhIEUHw3Cf4whGMIERBxFNbFaFCU4voalECS2ljEnWWM9a2KSv4IJxBTEfQUH+c+gltHYF0c3HJwFBKPWMUUkplTYlk2sSFNSxSqnVPbsk+pjTgKJOYOEhpVSOnY2cZJbpN5WxtOGZUwyAJNEsHap1bqfUW6qnbhmMOF40abFovkwpREaE5UUSzbIuQmr5wLjpSgBARo0CarSSAW1rCbVuYdC6py3KWGmSXWZvUJCOG2loU+PzvlUgWn89BVgdmguyuCiAezc5Uk2qYB5BQainPVnxYupc5lfIcD8xwPz1RsSMGCyFELQWZCyBASANQcIXBjAYYgn04YQCIN3WQhEKAw3+NgegEBFBIBnCbRQEBGDBRhqIDlDKrgXDIEgRIBzgB0i2kYW56o7YnJjFbFFMyy6fJnlizFALdXaoOniglxrsokplQCu5lqjCbQkEqiMiKYwABZ1VvM1Y4QFML9UeoNb8z1OqgU-LWji4F+KiVhshWa5q8q5SwokJtfS8KWAOoIAAVhdWij5GL-V6p+d6-5G5A36vzeqYNRrw3GrNUAA) ，以及相同代码的[要点](https://gist.github.com/ryandabler/fd7884cb9072e66717d9f5d4b23bd5e8)。