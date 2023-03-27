# 揭秘 JSON。解析()

> 原文：<https://itnext.io/demystifying-json-parse-383bb4907d79?source=collection_archive---------3----------------------->

![](img/7ab23eb5a0ed04396a714cdde433ddef.png)

卡里姆·甘图斯在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

软件工程的一个迷人的方面是将文本文件中的一系列字符转换成一组可以在 CPU 上执行的指令，以产生有意义的结果。对于一个阅读代码的人来说，期望的结果是很明显的(嗯，也许不那么明显，[取决于语言](http://www.jsfuck.com/))。

这是因为人们的思维已经发展到能够有效地将一连串的字符组织成有意义的组，这些组以有意义的顺序出现。这是使人们能够交流和回应的东西，也是一种经常被认为是理所当然的活动。

这完全是计算机的另一个故事。他们没有自动构建内容的心理过程，这些内容可以用来指导他们的操作；一串字符就是一串字符。为了让它成为别的东西，必须教会计算机如何理解它。

这并不是一项困难的任务(至少不再是了，这要感谢在这一领域开拓创新的前几代计算机科学家和工程师)，软件工程师每天用来解决问题的多种编程语言、文件格式和反序列化数据的方法证明了这一点。任何工程师都可以参与添加一种编程语言或一种文件格式或一种序列化技术。唯一真正的先决条件是具备一些文本解析的知识和经验。

# 从语法上分析

解析本质上是将一系列字符(或者更一般地说，一系列位)转换成结构化数据。出于本文的目的，它将被用来处理字符，尽管像文件格式这样的东西处理位。

简单地说，将字符串转换成有结构和有意义的东西的过程有两个步骤:

1.  词法分析:单个字符的流被转换成记号流(*即*，分组字符)
2.  语法分析:标记流被转换成一个抽象语法树(一种数据结构，它表达了文本中标记之间的关系，也可以记录关于它们的重要元数据)

通常，解析只是更大努力的第一步。例如，将 Golang 文件编译成可执行文件的第一步需要进行解析，但还需要做更多的工作，进一步将 AST 转换成 CPU 指令。

不管最终结果如何，任何成功的解析阶段都需要指定一个[形式语法](https://en.wikipedia.org/wiki/Formal_grammar)。这是一个人类可读的规范，解释了什么是相关的令牌以及它们应该出现的顺序。

最简单的形式是，语法可以被描述为一套重写字符串的规则。某些字符串可以根据其他字符串重写，而有些字符串不能再重写，它们分别被称为非终结符和终结符。在词汇阶段识别的记号包括该语法的终结符号集，该终结符号集进一步包括其非终结符号集(在非终结符号与终结符号之间存在一对多的关系)。

计算器可以用来解析数学表达式的简单语法示例如下

```
EXPRESSION
    : EXPRESSION '+' EXPRESSION
    | EXPRESSION '-' EXPRESSION
    | EXPRESSION '*' EXPRESSION
    | EXPRESSION '/' EXPRESSION
    | number
    ;
```

(这实际上是一个糟糕的语法，原因超出了本文的范围；它仅用于说明稍后将使用的表单类型。)

这个语法指定了终端符号`+`、`-`、`*`、`/`和`number`，并指出它们可以用五种不同的方式组织——其中四种允许递归组织。举个例子，`EXPRESSION '+' EXPRESSION`可以扩展为`EXPRESSION '+' EXPRESSION '-' EXPRESSION`。这允许用非常少的规则来表达复杂的令牌组。在这种语法下，像`12 + 41 * 3`这样的标准算术表达式可以通过以下重写匹配为一个表达式:

```
 EXPRESSION '+' EXPRESSION (rewrite the rightmost EXPRESSION)
-> EXPRESSION '+' EXPRESSION * EXPRESSION
-> number '+' number '*' number
```

解析是如何发生的，语法是如何被用来识别一个有效的数学表达式，这是本文的主要焦点。有两种主要的解析方法:[使用众多解析器生成器库中的一个](https://en.wikipedia.org/wiki/Comparison_of_parser_generators)(实现了成熟的解析算法)，或者手工编写一个。使用生成器可以从开发人员那里抽象出大量的工作，对于复杂的情况非常有用，但是当试图理解这个过程是如何工作的时候，最好还是手工编写一个解析器。

为此，本文将为 JavaScript 开发人员每天使用的东西创建一个解析器:JSON。到本文结束时，将会有一个工作版本的`JSON.parse`方法，它探索了将 JSON 文本转换成 JavaScript 值时必须采取的路径和必须做出的决策。

# 解析 JSON (lite)

为了在本文的其余部分保持词汇的一致性，“解析”将被理解为将非结构化字符串转换为结构化数据的整个过程。解析将由标记化步骤(词法分析)和紧随其后的(语法)分析步骤组成。

## 语法

为了搭建标记器和分析器，将给出一个精确的语法来表达要解析的 JSON。这种语法不会读取匹配完整 JSON 规范的所有可能的字符串；这样做将需要钻研细节，并偏离广泛理解解析管道的更高层次的目标。

JSON 字符串可以表示的值可能是字符串、数字、布尔值、空值以及对象和数组(两者都可以包含任意数量的 JSON 值)。这给了语法最重要的规则:

```
JSON
    : JSON_STRING
    | number
    | null
    | JSON_BOOL
    | JSON_ARRAY
    | JSON_OBJECT
    ;
```

本语法中用于命名符号的约定如下:

*   非终结符将全部大写
*   代表一类字符的终端符号将全部小写
*   表示文字字符的终止符号将被写在单引号之间

为了完整起见，这个语法的四个非终结符需要指定它们的规则。

JSON 中的字符串可以是两个引号之间的任何内容。数组将是两个方括号之间的任何 JSON 值的逗号分隔列表。对象将是一个逗号分隔的列表，在花括号之间是键/值对。这将语法扩展为以下内容:

```
JSON
    : JSON_STRING
    | number
    | null
    | JSON_BOOL
    | JSON_ARRAY
    | JSON_OBJECT
    ;JSON_STRING
    : '"' string '"'
    ;JSON_BOOL
    : true
    | false
    ;JSON_ARRAY
    : '[' JSON_ARRAY_ELS ']'
    ;JSON_ARRAY_ELS
    : JSON_ARRAY_ELS ',' JSON
    | JSON
    | epsilon
    ;JSON_OBJECT
    : '{' JSON_OBJECT_KV_PAIRS '}'
    ;JSON_OBJECT_KV_PAIRS
    : JSON_OBJECT_KV_PAIRS ',' JSON_OBJECT_KV_PAIR
    | JSON_OBJECT_KV_PAIR
    | epsilon
    ;JSON_OBJECT_KV_PAIR
    : JSON_STRING ':' JSON
    ;
```

这种语法有几个有趣的特点需要指出。

引入了一个叫做`epsilon`的特殊符号。这个符号代表空字符串，在这个语法中用来表示一个数组或者对象可以是空的。这将允许`"[]"`和`"{}"`都是有效的 JSON 字符串。

语法也是递归的。这反映了 JSON 值本身的递归性质，因为数组和对象中可以嵌套的内容没有限制。这使得`"[1,[true,[]]]"`成为一个有效的 JSON 字符串，可以被解析成一系列嵌套数组。此外，这些递归规则中的一些是*左*递归的，这意味着产生式规则将自身作为规则的开始:

```
JSON_ARRAY_ELS
    : JSON_ARRAY_ELS ',' JSON
    ;
```

对于使用某些解析算法的分析器来说，这可能导致无限循环，因为解析器为了解析`JSON_ARRAY_ELS`，首先必须查看它是否匹配`JSON_ARRAY_ELS.`，然后它必须查看*是否匹配*`JSON_ARRAY_ELS`、*以及无限的*。因为本文是手工实现分析器，所以分析器处理左递归的方式有很大的灵活性:可以很容易地重写为迭代。

## 标记器

记号赋予器的目的是将待解析的文本分成字符组(即使其中一些组只包含一个字符)。这些标记表示语法中的终止符号，通常由正则表达式定义并给定名称。某些标记也可以被认为是无意义的，可以完全跳过——这是对空白字符经常采用的方法。

上面定义的语法给出了以下标记:

```
const tokens = [
    [/^\s+/, null], // skip whitespace
    [/^\[/, '['],
    [/^]/, ']'],
    [/^\{/, '{'],
    [/^}/, '}'],
    [/^:/, ':'],
    [/^,/, ','],
    [/^"/, '"'],
    [/^\d+/, 'number'],
    [/^null\b/, 'null'], // Add word boundaries to ensure that
    [/^true\b/, 'true'], // `null`, etc., is matched exactly.
    [/^false\b/, 'false'],
    [/^[^"]*/, 'string'],  // Match anything not a quotation mark.
];
```

每个元组代表一个模式/名称对。每个模式都被锚定在字符串的开头，以确保记号赋予器总是生成下一个*记号，该记号应该被提供给分析器。通过给空白赋予一个`null`名称，这将向标记器表明所有的空白都应该被忽略。*

记号赋予器的逻辑是遍历记号列表，找到第一个匹配字符串开头的记号。这意味着对令牌进行排序的方式很重要，即更具体的令牌在更一般的令牌之前。例如，如果某个编程语言有两个关键字`def`和`define`，那么`define`应该排在前面，这样，对字符串`define add(a, b)`进行标记就不会首先匹配`def`，从而留下一个无效的`ine add(a, b)`供标记器以后处理。

这给出了以下结构作为记号赋予器的框架

```
class Tokenizer {
    constructor(tokens) {} read(string) {} next() {}
}
```

可以充实到

```
class Tokenizer {
    // List of tokens recognized by the tokenizer
    #tokens; // Position in the input string from which it will 
    // read to get the next token
    #cursor; // String to turn into tokens
    #string; constructor(tokens) {
        this.#tokens = tokens;
    } read(string) {
        this.#cursor = 0;
        this.#string = string;
    } next() {
        // If at end of input, no more tokens to generate
        if (this.#cursor === this.#string.length) {
            return undefined;
        } // Find substring beginning at position of cursor
        const str = this.#string.slice(this.#cursor); for (const [pattern, type] of this.#tokens) {
            const [match] = pattern.exec(str) || []; if (!match) {
                continue;
            } this.#cursor += match.length; // Skip tokens with null types
            if (type === null) {
                return this.next();
            } return { token: match, type };
        } // Could not extract any tokens, so throw error
        throw new Error(`Unrecognized input: ${str[0]}`);
    }
}
```

既然字符流可以被转换成记号流，那么是时候将这些记号转换成抽象语法树了。

## 分析者

分析器比记号赋予器要多做一点工作，因为它必须实现语法的产生式规则。由于这个分析器是手工编写的，它的大小和复杂性与它必须实现的规则数量成正比。

由于各种手动创建的分析器(称为"[递归下降](https://en.wikipedia.org/wiki/Recursive_descent_parser)")直接实现语法规则，一个好的框架应该是:

```
class Analyzer {
    // Tokenizer class
    #tokenizer; // Current token being analyzed
    #lookahead; constructor(tokenizer) {} #eat(tokenType) {} read(string) {} #JSON() {} #JSON_STRING() {} #JSON_number() {} #JSON_null() {} #JSON_BOOL() {} #JSON_ARRAY() {} #JSON_OBJECT() {} #JSON_ARRAY_ELS() {} #JSON_OBJECT_KV_PAIRS() {} #JSON_OBJECT_KV_PAIR() {}
}
```

这些方法大多直接匹配语法；两个是新的。`read`是一个公共方法，将在一个字符串上被调用以将其转换成一个抽象语法树。`#eat`是分析仪常用的一种方法，用于确保当前令牌是所需的类型。如果令牌与类型匹配，它将被返回，分析器可以继续派生。否则，分析器应该抛出一个错误。`#eat`的用法将在本文后面变得更加明显。

分析器上还有一个`#lookahead`属性。这是解析最重要的方面之一:在使用当前令牌之前检查它的能力。`#lookahead`存储当前令牌，允许分析器确定它应该采用语法中的哪条路径。它的存在对于实施`#JSON`方法至关重要。一般来说，任何数量的标记都可以用作 lookaheads，尽管一个标记是非常常见的数量，足以实现一个 JSON 解析器。

充实了非语法方法后，分析器将看起来像

```
class Analyzer {
    #tokenizer;
    #lookahead; constructor(tokenizer) {
        this.#tokenizer = tokenizer;
    } read(string) {
        this.#tokenizer.read(string); // Set the first token as the lookahead
        this.#lookahead = this.#tokenizer.next(); return this.#JSON();
    } #eat(tokenType) {
        const token = this.#lookahead; if (!token) {
            throw new Error(
                `Unexpected end of input; expected ${token.type}`
            );
        } if (this.#lookahead.type !== tokenType) {
            throw new Error(
                `Expected ${tokenType} === ${token.type}`
            );
        } // Set the next token as the lookahead
        this.#lookahead = this.#tokenizer.next(); return token;
    }
} 
```

随着`#eat`和`read`的实现，构建 AST 的方法就可以实现了。

```
class Analyzer {
    // ... methods from above #JSON() {
        switch (this.#lookahead.type) {
            case '"':
                return this.#JSON_STRING(); case 'number':
                return this.#JSON_number(); case 'null':
                return this.#JSON_null(); case 'true':
            case 'false':
                return this.#JSON_BOOL(); case '[':
                return this.#JSON_ARRAY(); case '{':
                return this.#JSON_OBJECT(); default:
                throw new Error(
                    `Received token which cannot be valid JSON`
                );
        }
    }
}
```

从该方法中可以看出，前瞻标记在允许分析器确定应该解析哪个 JSON 值的*中起着至关重要的作用。没有这些知识，分析器将无法继续。因为只有语法的标记集的子集可以开始一个 JSON 值，如果 lookahead 不在某个`case`语句中，那么分析器试图读取格式错误的 JSON 并抛出一个错误。*

其余的方法(其中一些举例说明了`#eat`的用法)有:

```
class Analyzer {
    // ... previous methods #JSON_STRING() {
        // The quotation marks are necessary for the JSON grammar,
        // but contribute nothing to the semantic content of the
        // AST, so ensure they exist but do not use
        this.#eat('"');
        const string = this.#eat('string').token;
        this.#eat('"'); return {
            type: 'JSON_STRING',
            value: string,
        }
    } #JSON_number() {
        const number = this.#eat('number').token; return {
            type: 'JSON_number',
            value: +number,
        }
    } #JSON_null() {
        this.#eat('null'); return {
            type: 'JSON_null',
            value: null,
        }
    } #JSON_BOOL() {
        const bool = this.#lookahead.type === 'true'
            ? this.#eat('true')
            : this.#eat('false'); return {
            type: 'JSON_BOOL',
            value: bool.token === 'true',
        }
    }
}
```

这负责语法的非递归规则。每个规则返回一个对象，该对象指定它在 AST 中表示的节点的类型以及该节点的解释值(这意味着如果分析器正在读取一个数字，则返回的节点具有一个值*，该值是一个数字*，而不是该数字的文本标记)。

JSON 值的这些最终情况可以递归地组合到数组和对象中，因此接下来的规则将在某种程度上涉及到通过最顶层的`#JSON`方法进行递归。

```
class Analyzer {
    // ... previous rules #JSON_ARRAY() {
        this.#eat('[');
        const elements = this.#JSON_ARRAY_ELS();
        this.#eat(']'); return {
            type: 'JSON_ARRAY',
            elements,
        }
    } #JSON_ARRAY_ELS() {
        const elements = []; while (this.#lookahead.type !== ']') {
            elements.push(this.#JSON()); // The final element of an array will not have a comma
            // following it, so conditionally consume any comma
            // characters
            this.#lookahead.type === ',' && this.#eat(',');
        } return elements;
    }
}
```

这是第一个为 AST 节点使用不同形状的产生式规则:代表终端节点的规则都有`value`属性，但是这个规则使用了`elements`。AST 节点可以有任何可能的形状，只要形状使用一致就没有问题。在这种情况下，对于数组节点用`elements`代替`value`更有语义意义。

`JSON_ARRAY_ELS`的语法规则是左递归的，如上所述，这可能导致某些算法分析器的无限递归。然而，手动分析器通过使用一个`while`循环来分析数组元素，从而绕过了这个问题。由于递归和循环是密切相关的概念，通常可以进行这样的替换，在这种情况下，它可以防止语法发生变化，同时仍然给出相同的结果。

方法`#JSON_ARRAY_ELS` 本身不返回 AST 节点，因为它是一个实用函数，而不是读取 JSON 的主要方法。

读取对象的方法与读取数组的方法非常相似:

```
class Analyzer {
    // ... previous methods #JSON_OBJECT() {
        this.#eat('{');
        const kvPairs = this.#JSON_OBJECT_KV_PAIRS();
        this.#eat('}'); return {
            type: 'JSON_OBJECT',
            entries: kvPairs,
        }
    } #JSON_OBJECT_KV_PAIRS() {
        const entries = []; while (this.#lookahead.type !== '}') {
            entries.push(this.#JSON_OBJECT_KV_PAIR());
            this.#lookahead.type === ',' && this.#eat(',');
        } return entries;
    } #JSON_OBJECT_KV_PAIR() {
        // Get key
        this.#eat('"');
        const key = this.#eat('string').token;
        this.#eat('"'); this.#eat(':'); // Get value
        const value = this.#JSON(); return [key, value];
    }
}
```

## 抽样输出

既然解析管道已经完成，就可以把记号赋予器和分析器放在一起，展示给定各种 JSON 字符串输入的示例输出。

```
const tokenizer = new Tokenizer(tokens);
const analyzer = new Analyzer(tokenizer);const str1 = '"abc"';
const str2 = '145';
const str3 = '[true, { "k": null }, []]';analyzer.read(str1);
// {
//   type: 'JSON_STRING',
//   value: 'abc',
// }analyzer.read(str2);
// {
//   type: 'JSON_number',
//   value: 145,
// }analyzer.read(str3);
// {
//   type: 'JSON_ARRAY',
//   elements: [
//     { type: 'JSON_BOOL', value: true },
//     { type: 'JSON_OBJECT', entries: [
//       [ 'k', { type: 'JSON_null', value: null } ],
//     },
//     { type: 'JSON_ARRAY', elements: [] },
//  }
```

## 将字符串转换为 JavaScript 值

到目前为止，分析器将 JSON 字符串转换为抽象语法树。然而，目标是将所述字符串转换成 JavaScript 值(从而模仿实际的`JSON.parse`方法)。

要做到这一点，需要利用 AST 的结构已经很好地反映了 JSON 本身的结构这一事实:每个返回 AST 节点的方法都会返回一些信息，这些信息包含了创建实际的底层 JSON 值所必需的所有信息。这意味着，通过非常小的改变，分析器*可以*返回值，而不是 ASTs。

要做到这一点，一点重构是必要的。不是分析器的方法*总是*返回 AST 节点，而是调用工厂函数，返回代表工厂“主题”的东西。在最初的案例中，主题是 AST 节点；在下一种情况下，它将是 JavaScript。

分析器的重构版本看起来像这样

```
const astFactory = {
    JSON_STRING(token) {
        return {
            type: 'JSON_STRING',
            value: token.token,
        };
    }, JSON_number(token) {
        return {
            type: 'JSON_number',
            value: +token.token,
        };
    }, JSON_null() {
        return {
            type: 'JSON_null',
            value: null,
        };
    }, JSON_BOOL(token) {
        return {
            type: 'JSON_BOOL',
            value: token.token === 'true',
        };
    }, JSON_ARRAY(elements) {
        return {
            type: 'JSON_ARRAY',
            elements,
        };
    }, JSON_OBJECT(entries) {
        return {
            type: 'JSON_OBJECT',
            entries,
        }
    }
};class Analyzer {
    // ... non-refactored methods and properties #factory; constructor(tokenizer, factory) {
        this.#tokenizer = tokenizer;
        this.#factory = factory;
    } #JSON_STRING() {
        this.#eat('"');
        const string = this.#eat('string');
        this.#eat('"'); return this.#factory.JSON_STRING(string);
    } #JSON_number() {
        const number = this.#eat('number'); return this.#factory.JSON_number(number);
    } #JSON_null() {
        this.#eat('null'); return this.#factory.JSON_null();
    } #JSON_BOOL() {
        const bool = this.#lookahead.type === 'true'
            ? this.#eat('true')
            : this.#eat('false'); return this.#factory.JSON_BOOL(bool);
    } #JSON_ARRAY() {
        this.#eat('[');
        const elements = this.#JSON_ARRAY_ELS();
        this.#eat(']'); return this.#factory.JSON_ARRAY(elements);
    } #JSON_OBJECT() {
        this.#eat('{');
        const kvPairs = this.#JSON_OBJECT_KV_PAIRS();
        this.#eat('}'); return this.#factory.JSON_OBJECT(kvPairs);
    }
}
```

现在，无论相关的`#factory`函数返回什么，分析器都会返回。此时，需要做的就是编写一个 JavaScript 工厂，而不是 AST 工厂。

```
const jsFactory = {
    JSON_STRING(token) {
        return token.token;
    }, JSON_number(token) {
        return +token.token;
    }, JSON_null() {
        return null;
    }, JSON_BOOL(token) {
        return token.token === 'true'
    }, JSON_ARRAY(elements) {
        return elements;
    }, JSON_OBJECT(entries) {
        return entries.reduce(
            (obj, [k, v]) => {
                obj[k] = v;
                return obj;
            },
            {}
        );
    }
};
```

使用与上面相同的字符串，JSON 工厂输出真正解析的 JSON 值。

```
const tokenizer = new Tokenizer(tokens);
const analyzer = new Analyzer(tokenizer, jsFactory);const str1 = '"abc"';
const str2 = '145';
const str3 = '[true, { "k": null }, []]';analyzer.read(str1); // "abc"
analyzer.read(str2); // 145
analyzer.read(str3); // [true, { k: null }, []]
```

# 结论

手动编写定制的记号赋予器和分析器并不像最初看起来那样具有挑战性，并且为工程开辟了许多其他途径，例如构建正则表达式引擎、CSS 引擎，甚至是自己的编程语言。事实上，最具挑战性的部分通常是恰当地建模语法。有了定义良好的语法，解析器本身的运行速度相当快。

虽然这个解析器没有解析完整的 JSON 规范，但它已经很接近了，对于任何对实现完整语法感兴趣的人来说，道格拉斯·克洛克福特的 RFC [可以在这里找到](https://www.ietf.org/rfc/rfc4627.txt)。事实上，如果有人对扩展它的功能感兴趣，这个解析器可以扩展成 JSON+语法。规范中不支持像`Infinity`、`undefined`和注释这样的值，但是可以很容易地添加到当前的语法中。

[这里是所有解析器代码的一个要点](https://gist.github.com/ryandabler/1dcdcb88890f72677bfdaed661d64c8c)。