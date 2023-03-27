# JavaScript 中样板文件 if/else/switch 的替代方案。拥抱开放/封闭原则。

> 原文：<https://itnext.io/an-alternative-to-boilerplate-if-else-switch-in-javascript-embrace-the-open-closed-principle-408446cf562d?source=collection_archive---------3----------------------->

![](img/c5f83c2348176c010ce2f94ce9d759c8.png)

软件产品成功或失败的一个关键方面是它在新需求到来时改变的能力和灵活性。一个系统的组件扩展、增长或适应起来并不复杂，并且在这样做的时候，新的变化不会带来混乱，可能会存在更长的时间，并且有更好的成功机会。我们都见过每个人都害怕接触的代码库，因为没有人知道什么会被破坏，或者因为完成一件看似简单的事情需要太长时间。

有所有众所周知的模式、原则和实践，将帮助任何开发人员成为更好的专业人员，并构建更好的软件。几乎我们这个领域的每个人都听说过**SOLID**(【https://en.wikipedia.org/wiki/SOLID】T2)，这是一套帮助你做到这一点的原则，*成为一个更好的专业人士，打造更好的软件*。尽管我在实践中所看到的，以及一些有面试经验的软件开发人员可以证明的是，大量的开发人员知道他们是什么，但并不真正理解，或者实践其中的一些或大部分。今天，我想谈谈它的第二个原理，即**中的 **O** 固体**，即**开/关原理**。我想展示一个违反这一原则的常见不良实践，以及我如何避免这个陷阱，并在方便的时候尝试重构。

我必须说，这个想法不是我想出来的，我是从 Jimmy Bogard(auto mapper 和 MediatR 的创始人)那里学到的，如果我没记错的话，他在一次演讲中谈到了加强薄弱的业务领域。从那以后我就一直使用这种技术，他用 C#实现了这种技术(使用**多态**，但是这篇文章是要和你分享我用 JavaScript 实现这种技术的方法，顺便说一下，这种方法没有使用 ECMAScript 6“类”，而是使用了一个简单的*工厂* **函数**。

**初始要求:**

您需要创建一个函数，它接受一个参数，可能的值为“偶数”和“奇数”，并返回一个数组，数组中的偶数或奇数(取决于参数的值)介于 0 和 10 之间(两端都不包括)。为了简单起见，限制 10 将是固定的，因为我们这次练习的目标不是算法创建，而是代码设计。

**常见解决方案:**

```
const getSequence = type => {
    if (type === 'even') {
        return getEvenNumbers();
    } else if (type === 'odd') {
        return getOddNumbers();
    } else {
        throw Error(`Invalid argument value ${type}`);
    }
};
```

我们还没有实现 **getEvenNumbers** 或 **getOddNumbers** ，老实说，我们不需要它们来跟进，尽管为了完整起见，它们将被包括在内。除此之外，它看起来像非常普通的代码，带有覆盖可能有效场景的 **if/else-if/else** 语句(出于本文的目的，我们对边缘情况并不真正感兴趣)。

上述代码的一个变体是:

```
const getSequence = type => {
    switch(type) {
        case 'even':
            return getEvenNumbers();
        case 'odd':
            return getOddNumbers();
        default:
            throw Error(`Invalid argument value ${type}`);
    }
};
```

在这种情况下，使用了一个 **switch** 语句来实现相同的结果。你使用这两种方法中的哪一种并不重要，它们所带来的问题是一样的，请原谅我。

由于 **getEvenNumbers** 的实现与 **getOddNumbers** 的实现有很多共同之处，我将这些共同部分重构为一个名为**includeIfConditionIsMet**的函数，您可以在下面看到它们的实现:

```
const getEvenNumbers = limit => includeIfConditionIsMet(limit,    number => number % 2 === 0);

const getOddNumbers = limit => includeIfConditionIsMet(limit, number => number % 2 !== 0);

const includeIfConditionIsMet = (limit = 10, predicate) => {
    return (function inner(array, number) {
        if (number === limit) {
            return array;
        } return inner(predicate(number) ? [...array, number] : array, number + 1);
    })([], 1);
};
```

最后一个函数使用默认的**限制**10 和一个**谓词**(唯一在 **getEvenNumbers** 和 **getOddNumbers** 之间有所不同的东西)，来构建并返回所需的数组。如您所知，最后一个函数可以用不同的方式实现。由于这里我们并不真正关心它的实现细节，我只是使用递归来实现它，只是因为我不喜欢用**代替**、 **foreach** 、 **while** 或 **for of loops** (我不喜欢命令式编程，你可以在我的另一篇文章[https://it next . io/a-more-declarative-solution-to-the-T9-problem-in-both-JavaScript 中找到“为什么”](/a-more-declarative-solution-to-the-t9-problem-in-both-javascript-and-c-13ab7f7a859b)

**第二个要求:**

你对你的代码和它的工作方式非常满意。现在你的老板要求你在你的程序中添加另一个功能，给定相同的输入“even”或“odd”以及一个整数数组，你需要返回一个新的数组，该数组包含偶数或奇数位置的元素(取决于参数的值)。

让我们坚持第一个和第二个“需求”的 **if/else** 解决方案。当老板看到你完成得如此之快时，你肯定会给他留下深刻印象。现在，您的代码如下所示:

```
const getElementsByPositionType = (type, elements) => {
    if (type === 'even') {
        return getElementsInEvenPosition(elements);
    } else if (type === 'odd') {
        return getElementsInOddPosition(elements);
    } else {
        throw Error(`Invalid argument value ${type}`);
    }
};
```

为了完整起见，包括 getElementsInEvenPosition 的实现，你很容易就能想到 getElementsInOddPosition 的 T2

```
const getElementsInEvenPosition = elements => elements.reduce((a, c, i) => i % 2 === 0 ? a.concat(c) : a, []);
```

但事实是，你的老板在代码审查会议上马上就发现了问题。函数 **getSequence** 和**getElementsByPositionType**之间似乎有很多重复。 **if/else-if/else** 检查子句是相同的，现在他非常确定您复制并粘贴了第一个子句，并对其进行了修改(上帝知道是我做的)。他想知道是否每次有一个像这样的新“需求”时，你都会做同样的事情，他让你知道他的担心。

**问题(第一部分):**

重复不是一件好事，它会带来很多问题，我相信你也意识到了。相信我，我见过 15 加 **if/else** 或 **switch** 语句检查相同范围的可能值的代码库。我遇到过的最典型的情况就是与 C#中的**枚举**有关。例如，一个 **Person 类**具有一个**类型属性**，这是一个**枚举**，具有值 Manager、VP 等。；有一个不同的逻辑来计算其中每一项的内容，如奖金，但它们在其他几个操作中也互不相同，因此对于每个不同的基于类型的计算，都有一个基于枚举类型属性的 switch 语句。当有一天需要向**枚举**中添加一个新值时，必须这样做的可怜人必须检查整个代码库，寻找那些 **switch** 语句，并向其中的每一个添加一个新的 **case** 子句。不要错过一个，因为这将是你的错，你将是被绞死的人，而不是创造了**枚举**的人，或者是增加了**开关/案例/默认**地狱的士兵大军。

但是到目前为止，你还没有这个问题，你只是有一个复制的问题…等着瞧吧。

**第三个要求:**

在你的老板看起来不太满意你解决最后一个“需求”的方式之后，你仍然没有从你的程序员的自负中恢复过来；然而，他有一个新的任务给你。“我们的产品需要支持质数的类似功能”，这句话是你在美好的第一次约会之夜无法忘记的，因为她/他可能不会再见到你，但你的老板会。

在你第一次尝试时，你想到了一些东西，你在代码中寻找所有需要添加*质数*逻辑的地方，并修改代码，看起来像这样:

```
const getSequence = type => {
    if (type === 'even') {
        return getEvenNumbers();
    } else if (type === 'odd') {
        return getOddNumbers();
    } else if (type === 'prime') {
        return getPrimeNumbers();
     } else {
        throw Error(`Invalid argument value ${type}`);
    }
};

const getElementsByPositionType = (type, elements) => {
    if (type === 'even') {
        return getElementsInEvenPosition(elements);
    } else if (type === 'odd') {
        return getElementsInOddPosition(elements);
    } else if (type === 'prime') {
        return getElementsInPrimePosition();
     } else {
        throw Error(`Invalid argument value ${type}`);
    }
};
```

好消息是只有两个地方可以去，两年后当项目变得更大更复杂时，你可能就没那么幸运了。

*可以实现***getElementsInPrimePosition****作为练习*。*

*你内心的某些东西告诉你，你一定不能把这个给你的老板看，如果你看了，你可能没有足够的钱支付下一个约会之夜(我知道你第一个晚上做得很好，他/她会再见到你)。*

***问题(第二部分):***

*你首先违反了**干原则**https://en.wikipedia.org/wiki/Don%27t_repeat_yourself，而这一次，你违反了**开/关原则**https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle。这最后一个基本上陈述了，"*软件实体(类，模块，函数，等等。)应该对扩展开放，但对修改关闭*”。你确实在那两个地方做了手术并修改了代码。任何时候你像那样修改代码，特别是在许多不同的地方，你都冒着破坏东西的巨大风险，并且你也创建了更难遵循、维护和扩展的代码。不要自私，想想那个可怜的灵魂会在你之后来到那个代码，那也可能是你自己的，明年，下个月或者两个小时之后。*

*如果你想了解更多关于**开放/封闭原则**和一般的**坚实原则**的方法，我推荐你阅读罗伯特·C·马丁的《C# 中的*敏捷原则、模式和实践》。**

***解决方案:***

*你不知道任何更好的，但你有一个你信任的 JavaScript 开发者的电话号码，这将帮助你。她告诉你，我会这样开始。*

```
*const asEnumeration = dictionary => {
    return Object.freeze({
        fromValue: value => {
            if (dictionary[value]) {
                return dictionary[value];
            }
            throw Error(`Invalid enumeration value ${value}`); 
        }
    });
};*
```

*我使用枚举作为这个函数名的一部分，以纪念 Jimmy Bogard 给他的抽象类起的名字，这是他的 C#设计的基石，旨在解决上述所有问题。*

*反正这个**函数**是要带一个**类字典对象**作为参数的，它要用一个单独的方法 **fromValue** 返回一个**不可变对象**(也是为了纪念 Jimmy 的**枚举**类而命名的)。这样，您可以基于传入的**字典**创建类似**枚举的对象**。*

```
*const numbersEnumeration = asEnumeration({
    'even': {
        getSequence: getEvenNumbers
    },
    'odd': {
        getSequence: getOddNumbers
    }
});*
```

*现在我可以有一个数字枚举，哪些键是我们的业务域的可能值，“偶数”，“奇数”和某一天“质数”或任何可能的情况。然后我可以像这样实现 **getSequence** :*

```
*const getSequence = type => numbersEnumeration.fromValue(type).getSequence(10);*
```

*然后，当“按位置类型排列的元素”的新“需求”出现时，不是在两个或者也许 15 个不同的地方重复你自己，而是在一个地方，为**字典**中的每个对象添加一个新的**属性**，就像这样:*

```
*const numbersEnumeration = asEnumeration({
    'even': {
        getSequence: getEvenNumbers,
        getElementsByPositionType: getElementsInEvenPosition,
    },
    'odd': {
        getSequence: getOddNumbers,
        getElementsByPositionType: getElementsInOddPosition,
    }
});*
```

*那么**getElementsByPositionType***就会是:**

```
**const getElementsByPositionType = (type, elements) => numbersEnumeration.fromValue(type).getElementsByPositionType(elements);**
```

**您可以重构代码，并像这样调用 **getSequence** 和**getElementsInPrimePosition**:**

```
**const fromValue = type => numbersEnumeration.fromValue(type);

const getSequence = type => fromValue(type).getSequence(10);

const getElementsByPositionType = (type, elements) => fromValue(type).getElementsByPositionType(elements);

console.log(getSequence('even'));
console.log(getElementsByPositionType('odd', [6, 7, 8, 9, 10]));**
```

**现在，当*质数*逻辑或者任何其他逻辑出现时，您只需向映射**字典**中添加一个新的**对象**(再次仅在一个地方发生变化)，如下所示:**

```
**const numbersEnumeration = asEnumeration({
    'even': {
        getSequence: getEvenNumbers,
        getElementsByPositionType: getElementsInEvenPosition,
    },
    'odd': {
        getSequence: getOddNumbers,
        getElementsByPositionType: getElementsInOddPosition,
    },

    'prime': {
        getSequence: getPrimeNumbers,
        getElementsByPositionType: getElementsInPrimePosition,
    }
});**
```

**在整个代码库中，不再有 **if/else/switch** 语句需要追踪和更改。我不得不承认，没有完全不允许修改的代码，所以不要从字面上理解，或者如果你发现自己在修改现有代码的同时试图不违反这个原则，那么你会感觉很糟糕，如果不修改代码，就没有办法在系统中添加或更改功能。但是，通过在一个地方而不是多个地方添加或更改代码来实现这一点肯定是更好的做法，并且最好是添加/扩展而不是修改。**

**我还必须说，这种技术并不是要取代所有的 **if/else/switch** 语句，而是要取代那些处理固定值集的语句，比如那些**enum**类型所包含的语句，这些语句往往会产生大量的重复。我使用这种技术已经很多年了(我开始用 C#，然后用 JavaScript)，它从未让我失望过。**

**无论你是用 C#这样的语言，通过类、继承和多态(或者更好，组合[https://en.wikipedia.org/wiki/Composition_over_inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance))还是用 JavaScript 这样的语言(使用工厂函数、对象甚至对象组合)编码，如果你想创建一个*强域*(在我看来这与贫血域[https://www.martinfowler.com/bliki/AnemicDomainModel.html](https://www.martinfowler.com/bliki/AnemicDomainModel.html))**类似枚举的**类型应该有行为，而不仅仅是“原始类型”。**

**编码快乐！**