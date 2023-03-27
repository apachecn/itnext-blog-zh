# 通过逆向工程 jQuery 学习可迭代性

> 原文：<https://itnext.io/learning-iterability-by-reverse-engineering-jquery-b6c05b2f0f9?source=collection_archive---------10----------------------->

![](img/fa53d56eecc8f9b825f187acc23e4e34.png)

我记得我第一次使用 jQuery 时，我对仅仅使用 CSS 选择器和一些链接方法就能操作大量 DOM 元素感到敬畏。当时我不知道 jQuery 如何解析类似于`div .button > a.external`的东西来找到适合选择器的元素，或者允许我一个接一个地链接方法来与这些元素交互。我喜欢能够写作

```
$(...).addClass(...)
    .append(...)
    .attr(...)
    .on(...);
```

在 JavaScript 世界呆了很长时间后，我仍然发现 jQuery 是一个非常有趣的库，但是现在的原因与我第一次使用它时完全不同。

我后来发现 jQuery 有趣的一点是，`$(...)`生成的东西实际上并不是一个数组。在控制台中键入`$("html")`将产生类似于

```
► n.fn.init [html, prevObject: n.fn.init]
```

虽然由于控制台中使用的括号符号，它看起来像一个数组，但在这个输出上调用`Array.isArray()`确认它确实不是。事实上，它只是一个对象(因为怀疑它以`n.fn.init`开头，这表明它是某个构造函数的结果)，并且`Object.prototype.toString.call()`告诉我们它是`[object Object]`。

这个小小的奇怪之处引出了一些有趣的问题，尤其是当 DOM 元素不是数组时，jQuery 如何遍历其属性来操作 DOM 元素。研究这是如何可能的把我带到了迭代器的话题上。因此，在这篇文章中，我们将对 jQuery 的一个简单版本进行逆向工程，来讨论可迭代性以及一些更小的问题，比如对象如何像数组一样在控制台中显示。

# 迭代性

在开始之前，我们应该问:什么是可迭代性？

可迭代性是对象指定如何以编程方式访问其数据的一种方式。具体来说，我们指的是 JavaScript 引擎如何使用`for...of`循环、`...` (spread)操作符或`Array.from()`来访问值。

一些 JavaScript 对象内置了可迭代性。使用上述技术，字符串、数组、映射和集合都可以轻松地访问数据。因此，我们可以编写以下代码并获得有效的输出:

```
const str = "abc";
for (const letter of str) {
    console.log(letter);     // prints "a", "b", "c"
}const arr = [ 1, 2, 3 ];
for (const elem of arr) {
    console.log(elem);      // prints 1, 2, 3
}const map = new Map();
map.set("first", 1);
map.set("second", 2);
for (const mapping of map) {
    console.log(mapping);   // prints ["first", 1], ["second", 2]
}const set = new Set();
set.add("first");
set.add("second");
for (const elem of set) {
    console.log(elem);      // prints "first", "second"
}
```

然而，我们不能用常规对象如`person = { name: "Albert Einstein", age: 139 }`或`arrayLike = { 0: "We", 1: "can't", 2: "iterate", 3: "this" }`开箱即用。那么，我们如何使普通对象可迭代呢？

在过去几年对 JavaScript 的一些 Python 风格的增强中，实现的特性之一是一个`Symbol.iterator`全局常量。通过在我们的对象中使用这个作为方法的名称，我们可以创建一个函数来指定使用`for...of`等遍历对象数据的行为。我们将利用这一点来实现我们的 jQuery-lite 功能。

# 作用域 jQuery-lite

那么，我们想要实现什么呢？为了简单起见，我们将只构建一种方法来查询给定 CSS 选择器的 DOM，并且我们将我们可以执行的方法限制为上面列出的`addClass`、`append`、`attr`和`on`。

我们还将创建一个类似于`$()`的函数，它接受 CSS 选择器字符串或类似数组的对象，并返回我们的 jQuery 类的一个实例。

因为我们这样做是为了说明 JavaScript 的可迭代特性，所以我们不会使用数组(或数组的子类)来充当 jQuery 对象，因为这些对象本来就是可迭代的，不会教给我们任何东西。此外，它们开放了 jQuery 没有的数组方法，这些方法对于 DOM 操作没有任何意义(例如`.reduce()`)。

因此，让我们首先设置我们将要充实的类框架:

```
class JQ {
    constructor(list) {
        let i = 0; while (list[i] !== undefined) {
            this[i] = list[i];
            i++;
        }
    } addClass(className) { ... }
    append(content) { ... }
    attr(name, val) { ... }
    on(event, handler) { ... }
}const $$ = input => {
    if (typeof input === "string") {
        const nodes = document.querySelectorAll(input);
        return new JQ(nodes);
    } return new JQ(input);
}
```

目前这还不是一个可迭代的对象，但是我们已经开始打基础了。在构造函数中，我们接受一个列表参数(由我们的基本`$$`函数计算得到，或者是对应于 CSS 选择器的 HTML 元素列表，或者是先前的 JQ 实例),并使用列表中的索引作为属性键，将每一项指定为 JQ 类的属性。

但是我们如何使它可迭代呢？只需将我们的`Symbol.iterator`方法添加到我们的 JQ 类中:

```
 [Symbol.iterator]() {
        let i = 0;
        const _this = this; return {
            next() {
                return _this[i]
                    ? { done: false, value: _this[i++] }
                    : { done: true };
            }
        }
    }
```

我们首先创建一个闭包，它将存储一个变量`i`和一个对`this`的引用，我们将使用它来遍历所有 JQ 实例的属性，我们希望在其他方法中循环遍历这些属性。

然而，真正的魔力发生在`return`条款中。我们返回的对象有一个`next()`方法，只要`next()`返回一个带有`done: false`属性的对象，JavaScript 引擎就会不断调用这个方法。同一个对象将包含一个来自我们的 JQ 实例的值，当我们最终用完要挑选的值时，我们标记`done: true`。

因此，让我们使用新的可迭代性功能来为剩下的方法添加逻辑

# 利用可迭代性

我们将用来充实 JQ 方法的代码非常简单。我写它的方式将展示我们的可迭代性将被使用的两种方式，虽然它可能不是最好的生产目的，但它对教学是好的。我们充实后的类如下所示:

```
class JQ {
    constructor(list) {
        let i = 0; while (list[i] !== undefined) {
            this[i] = list[i];
            i++;
        }
    } addClass(className) {
        for (const elem of this) {
            elem.classList.add(className);
        } return $$([ ...this ]);
    } append(content) {
        for (const elem of this) {
            elem.append(content.cloneNode());
        } return $$([ ...this ]);
    } attr(name, val) {
        for (const elem of this) {
            elem.setAttribute(name, val);
        } return $$([ ...this ]);
    } on(event, handler) {
        for (const elem of this) {
            elem.addEventListener(event, handler);
        } return $$([ ...this ]);
    } [Symbol.iterator]() {
        let i = 0;
        const _this = this; return {
            next() {
                return _this[i]
                    ? { done: false, value: _this[i++] }
                    : { done: true };
            }
        }
    }
}
```

每种方法都使用了`for...of`循环和`...`操作符。因为我们有迭代器方法，这两个都可以。这就是让你的对象可迭代的全部内容，但是我们还可以用我们的类做一些更有趣的事情。

# 奖励:数组欺骗

现在，如果我们将一个 JQ 实例记录到控制台，它将像一个对象通常会做的那样打印出来。如前所述，jQuery 对象不会。那么我们如何强迫一个对象以数组的形式输出呢？

要做到这一点，只需要满足两个要求:一个`length`属性和一个`splice()`方法。所以赶紧补充那些吧。为了保持简洁，我们将只修改构造函数(添加长度)并添加拼接方法。

```
class JQ {
    constructor(list) {
        let i = 0; while (list[i] !== undefined) {
            this[i] = list[i];
            i++;
        }

        this.length = i;
    }

    splice(...args) {
        return [ ...this ].splice(...args);
    }
}
```

这就是我们要做的！现在它将打印为

```
► JQ [ ... ]
```

# 奖励:使用发电机

我们可以通过为生成器函数切换出我们的`Symbol.iterator`方法来简化我们的类所使用的代码量。简单地说，一个生成器函数返回一个生成器对象，它实现了我们上面手工实现的迭代器规则。

发电机有三个特别之处:

1.  它们使用`yield`关键字将值传递给调用函数。
2.  他们会在通话间隙记住自己的状态。
3.  他们在声明中使用了一个`*`。

所以我们来交换一下。

```
class JQ {
    constructor(list) {
        let i = 0; while (list[i] !== undefined) {
            this[i] = list[i];
            i++;
        }
    }

    *generator() {
        for (let i = 0; i < this.length; i++) {
            yield this[i];
        }
    }
}
```

然而，现在我们以前的所有方法都被破坏了，因为它们需要迭代器来执行它们的值，而生成器即使实现了它们也不是迭代器。如果我们现在尝试在 JQ 对象上调用`.addClass()`，我们会得到下面的错误:

```
Uncaught TypeError: this is not iterable
```

那么我们需要做什么呢？我们需要修改我们的其他方法来使用我们的`*generator()`。一般来说，如果我们做`const genObj = generator()`，我们会得到一个*是*可迭代的对象。因此，我们将在每个方法中添加一行额外的代码，并更改两行代码。

```
class JQ {
    constructor(list) {
        let i = 0; while (list[i] !== undefined) {
            this[i] = list[i];
            i++;
        }
    }

    *generator() {
        for (let i = 0; i < this.length; i++) {
            yield this[i];
        }
    } addClass(className) {
        const iterable = this.generator();

        for (const elem of iterable) {
            elem.classList.add(className);
        } return $$([ ...this.generator() ]);
    }
}
```

我们调用`this.generator()`来获得一个可迭代的生成器对象，这允许我们使用`for...of`循环。我们还必须在我们的`return`语句中再次调用它，因为生成器对象是一次性使用的对象，如果我们重用它，我们将没有数据提供给`$$`函数。

这就是我们如何使用生成器函数来简化我们自己的可迭代性逻辑(代价是我们的方法中多了一点代码)。

我留下以下两个要点来展示我们的 JQ 类使用`Symbol.iterator`和生成器方法的完整实现，以及一个到更技术性的可迭代性解释的链接。

*   [要诀运用](https://gist.github.com/ryandabler/e0916a9454aa42af28c05a981df6d28e) `[Symbol.iterator](https://gist.github.com/ryandabler/e0916a9454aa42af28c05a981df6d28e)`
*   [使用发电机法的要诀](https://gist.github.com/ryandabler/083c34eab2009d0a01ff6b3fd8b44fa1)
*   关于迭代器和生成器的好文章