# JavaScript 国际方法

> 原文：<https://itnext.io/javascript-international-methods-b70a2de09d92?source=collection_archive---------1----------------------->

# 因为日期时间格式很麻烦&国际格式几乎不可能

![](img/95cbb406a6950ef338e41d44bf17e1d7.png)

为什么这么难？？蒂姆·高在 [Unsplash](https://unsplash.com/search/photos/developer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

# 介绍

我很幸运，因为我工作的公司主要位于美国和英国，在墨西哥和加拿大也有一些分部。这使我的工作变得容易多了，因为我不必太担心国际化:不用担心转换成不同的日期格式，不用担心语言翻译产生的特性，当然也不用担心其他国家的列表和货币的奇怪之处(真的吗——当涉及到金钱时，谁会在句号后面加逗号呢？？).

虽然这些不是我的问题，但并不意味着其他工程师不会感到难以置信的痛苦。例如，缺乏国际标准化(以及无法转换数字)导致公制单位与英制单位不匹配，导致 NASA 的火星轨道飞行器在 1999 年完全错过进入火星轨道，损失高达 1 . 25 亿美元。

Oof…来源:[https://www.reactiongifs.com/picard-facepalm/](https://www.reactiongifs.com/picard-facepalm/)

虽然我的软件开发团队所用的资金并不是很大，而且我们的错误会导致代码中的错误，我们通常可以在几个小时或几天内修复这些错误，但我肯定能与 NASA 的工程师感同身受，我不敢想象一个简单的错误，如没有仔细检查每个人在 Orbiter 代码中使用的相同测量值，是如何造成这样的损失的。

这就是为什么，当我听说最近发布了 ECMAScript 国际化 API 时，我很兴奋地看到哪些改进将使全世界的 JavaScript 变得更好，对使用它的每个人来说更标准化。

> 今天，让我们来看看**Intl:**ECMAScript 国际化 API 的一些最佳部分，它提供了语言敏感的字符串比较、数字格式化以及日期和时间格式化。

# 国际日期时间突出显示

像往常一样，Mozilla 的 MDN Web 文档对新的国际 API 及其特性进行了最好的总结:

> INTL 对象提供了对几个构造函数以及国际化构造函数和其他语言敏感函数的访问。— MDN 网络文档

新的`**Intl**`对象有六个属性，但在我看来，其中只有五个值得在这篇博文中占有一席之地。

*   `**Intl.DateTimeFormat**`
*   `**Intl.RelativeTimeFormat**`
*   `**Intl.ListFormat**`
*   `**Intl.NumberFormat**`
*   `**Intl.PluralRules**`

因此，不再赘述，我将深入研究第一个属性:国际日期时间格式。

## [国际日期时间格式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat)

![](img/a9f0f6d66dfcc19edab0b72864684c15.png)

是的，我们都去过那里。是的，这真的是一场关于日期和时间的噩梦。

`**Intl.DateTimeFormat**`对象是支持语言敏感的日期和时间格式的对象的构造函数。

最基本的，日期时间格式接受`locales`和`options`对象，并根据这些选项输出格式化的时间戳。

这里有一个使用来自全球的`locales`来格式化这个`date`变量的例子。

国际机场。日期时间格式 `**Locales**` **示例**

```
var date = new Date(Date.UTC(2012, 11, 20, 3, 0, 0));// using US date time formattingconsole.log(new Intl.DateTimeFormat('en-US').format(date));// expected output: "12/19/2012" // using Great Britain date time formattingconsole.log(new Intl.DateTimeFormat('en-GB').format(date));
// expected output: "19/12/2012"// Include a fallback language, in this case US English with the Buddhist calendarconsole.log(new Intl.DateTimeFormat(['en-US-u-ca-buddhist']).format(date));
// expected output: "12/19/2555"
```

除了能够传入位置之外，用户还可以提供与位置连在一起的附加参数，例如:

*   `nu` —不同国家使用的数字系统，
*   `ca` —要使用的日历，
*   `hc` —小时周期(`h11`、`h24`等)。).

在添加到`locales`之后，是可选的`options`对象，它可以包括如下属性:

*   `timezone` —这可以是`"UTC”`或者类似`"America/New_York"`的东西，
*   `hourCycle` —同样，像`"h12"`或`"h23"`这样的规格，
*   我最喜欢的一个:`formatMatcher` —这个算法可以接受用于描述日期时间的属性，并格式化输出以进行匹配。

`formatMatcher`的子集包括以下内容:

*   `weekday`、`year`、`month`、`day`、`hour`、`minute`、`second`
*   `hour`、`minute`、`second`

即使这样，子集属性也有不同的选项。以下是`weekday`的选项。

*   `"long"`(例如`Thursday`)，
*   `"short"`(例如`Thu`)，
*   `"narrow"`(如`T`)。

看看这个使用`DateTimeFormat`的`options`参数格式化输出的例子。

国际机场。DateTimeFormat `**Options**` **参数示例**

```
var date = new Date(Date.UTC(2012, 11, 20, 3, 0, 0));// request a weekday along with a long date in Japanesevar options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };console.log(new Intl.DateTimeFormat('ja-JP', options).format(date));
// expected output: "2012 年 12 月 19 日水曜日"// an application may want to use UTC and make that visible in Germanoptions.timeZone = 'UTC';
options.timeZoneName = 'short';console.log(new Intl.DateTimeFormat('de-DE', options).format(date));
// expected output: "Donnerstag, 20\. Dezember 2012, UTC"// sometimes you want to be more precise in Australian Englishoptions = {
  hour: 'numeric', minute: 'numeric', second: 'numeric', 
  timeZone: 'Australia/Sydney',
  timeZoneName: 'short'
};console.log(new Intl.DateTimeFormat('en-AU', options).format(date));
// expected output: "2:00:00 pm AEDT"// even the US needs 24-hour time for some occasionsoptions = {
  year: 'numeric', month: 'numeric', day: 'numeric',
  hour: 'numeric', minute: 'numeric', second: 'numeric',
  hour12: false,
  timeZone: 'America/Los_Angeles' 
};console.log(new Intl.DateTimeFormat('en-US', options).format(date));
// expected output: "12/19/2012, 19:00:00"// to specify options but use the browser's default locale, use 'default'console.log(new Intl.DateTimeFormat('default', options).format(date));
// expected output: "12/19/2012, 19:00:00"
```

这是一个很酷的功能。它将使正确翻译日期和时间变得更加容易，并以如此多的特定方式格式化它们，以至于像专门为解析、验证和操作 JavaScript 日期和时间而制作的 [moment.js](https://momentjs.com/) 这样的包将变得没有必要。

由于我已经谈到了日期，所以我想继续讨论的下一个属性是国际相对时间格式。

## [国际相对时间格式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RelativeTimeFormat)

`**Intl.RelativeTimeFormat**`对象是一个对象的构造函数，支持语言敏感的相对时间格式。

与`Intl.DateTimeFormat`类似，该对象接受一个`locales`参数和一个`options`对象。

`locales`参数在这个相对时间格式对象中的功能与在日期时间格式中的功能完全相同，所以我将在这里跳过任何进一步的解释。

至于`options`,`Intl.RelativeTimeFormat`具有它接受的各种附加属性:

*   `localeMatcher` —区域匹配算法，选项有`"best fit"`或`"lookup"`，
*   `numeric` —输出消息的格式，
*   `style` —国际化消息的长度。

`style`遵循与`weekday`相同的线路，因为它有三个不同的值:

*   `"long"`(默认，例如`in 2 months`)，
*   `"short"`(如`in 2 mo.`)，
*   或者`"narrow"`(例如`in 2 mo.`)。

`narrow`和`short`可能相同，具体视国家而定。

以下是一些在各种`locales`中使用`Intl.RelativeTimeFormat`并传递给它们各种`options`的示例。

国际机场。RelativeTimeFormat 示例

```
var rtf1 = new Intl.RelativeTimeFormat('en', { style: 'narrow' });// relative time formats in Englishconsole.log(rtf1.format(3, 'quarter'));
//expected output: "in 3 qtrs."console.log(rtf1.format(-4, 'hour'));
//expected output: "4 hr. ago"var rtf2 = new Intl.RelativeTimeFormat('de', { numeric: 'auto' });// relative time formats in Germanconsole.log(rtf2.format(2, 'day'));
//expected output: "übermorgen"// Create a relative time formatter in your locale
// with default values explicitly passed in.const rtf = new Intl.RelativeTimeFormat("en", {
    localeMatcher: "best fit", // other values: "lookup"
    numeric: "always", // other values: "auto"
    style: "long", // other values: "short" or "narrow"
});// Format relative time in English using negative value (-1).console.log(rtf.format(-1, "day"));
// > "1 day ago"// Format relative time in English using positive value (1).console.log(rtf.format(2, "day"));
// > "in 2 days"
```

再一次，超级方便的是，作为开发人员，我们不必处理格式化天数或小时数甚至一些单词的逻辑，根据传入的数字，在多种语言中 T30 不比 T31 少。没有复杂的`if/else if/else`逻辑块，甚至没有三元表达式。

在`Intl.RelativeTimeFormat`的帮助下，只需指定要使用的语言，`numeric`输出和`style`就可以了。不错！😄

关于国际化 API，接下来让我们看看另一件因语言而异的事情:列表是如何格式化的。

## [国际列表格式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ListFormat)

将一种语言中的数组转换成类似于添加了`ands`或`ors`的可读列表已经够难的了，试着用多种语言来做吧。

对象是对象的一个构造函数，用于支持语言敏感的列表格式。

正如本文反复出现的主题一样，`ListFormat`接受的第一个可选参数是一个`locales`规范。

第二个对象是`options`，其参数如下:

*   `localeMatcher` —区域匹配算法，选项同`Intl.RelativeTimeFormat`，
*   `type` —输出消息的格式。可能的值有`"conjunction"`，代表基于“与”的列表(默认，例如`1, 2, and 3`)，或者`"disjunction"`，代表基于“或”的列表(例如`1, 2, or 3`)。`"unit"`代表带有单位的值列表(如`7 pounds, 11 ounces`)。
*   `style` —格式化消息的长度。值包括`"long"`、`"short"`或`"narrow"`。

下面是一些如何在实践中使用`Intl.ListFormat`对象的例子。

国际机场。列表格式示例

```
const vehicles = ['Cat', 'Dog', 'Fish'];// list formats in Englishconst formatter = new Intl.ListFormat('en', { style: 'long', type: 'conjunction' });console.log(formatter.format(vehicles));
// expected output: "Cat, Dog, and Fish"// list formats in Frenchconst formatter2 = new Intl.ListFormat('fr', { style: 'short', type: 'disjunction' });console.log(formatter2.format(vehicles));
// expected output: "Cat, Dog ou Fish"// list formats in Spanishconst formatter3 = new Intl.ListFormat('es', { style: 'narrow', type: 'unit' });console.log(formatter3.format(vehicles));
// expected output: "Cat Dog Fish"const list = ['BMW', 'Volvo', 'Audi'];// list formats in British Englishconsole.log(new Intl.ListFormat('en-GB', { style: 'long', type: 'conjunction' }).format(list));
// expected output: "BMW, Volvo and Audi"console.log(new Intl.ListFormat('en-GB', { style: 'short', type: 'disjunction' }).format(list));
// expected output: "BMW, Volvo or Audi"console.log(new Intl.ListFormat('en-GB', { style: 'narrow', type: 'unit' }).format(list));
// expected output: "BMW Volvo Audi"
```

终于，*终于*有一个内置的 API 可以让列表变得好看，这有多棒？见鬼，即使它只适用于说英语的国家或只有英语语言的应用程序，我也会接受，这太有用了。

但是等等！我还要介绍另一个非常有用的构造函数:数字格式化程序！🤑

## [国际号码格式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat)💰

各位，下雨吧。

虽然`**Intl.NumberFormat**` 的定义一开始听起来并不令人兴奋，但其含义是很大的。此对象是启用区分语言的数字格式的对象的构造函数。你知道这意味着什么吗？国际货币的格式化是轻而易举的事。

除了要使用的编号系统`locales`和`nu`之外，`Intl.NumberFormat`对象还可以接受以下`options`参数:

*   `localeMatcher`，
*   `style` —要使用的格式样式。可能的值有:`"decimal"`用于普通数字格式，`"currency"`用于货币格式，`"percent"`用于百分比格式；`"decimal”`是默认样式。
*   `currency` —货币格式中使用的货币，
*   `currencyDisplay` —如何显示格式化后的货币。选项有`"symbol"`使用本地化的货币符号，如€，`"code"`使用 ISO 货币代码，或者`"name"`使用本地化的货币名称，如`"dollar"`。
*   `useGrouping` —是否使用分组分隔符，如千位分隔符或千位分隔符。可能的值是`true`和`false`。

围绕数字还有几个选项:`minimumIntegerDigits`，`maximumSignificantDigits`，但我认为它们的用处有限，所以我在这里就不一一定义了。

**国际。数字格式示例**

```
var number = 123456.789;// currency formatting with euros in Germanconsole.log(new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }).format(number));
// expected output: "123.456,79 €"// currency formatting in Japanese where the Japanese yen doesn't use a minor unitconsole.log(new Intl.NumberFormat('ja-JP', { style: 'currency', currency: 'JPY' }).format(number));
// expected output: "￥123,457"// limiting an Indian currency format to three significant digitsconsole.log(new Intl.NumberFormat('en-IN', { maximumSignificantDigits: 3 }).format(number));
// expected output: "1,23,000"// Arabic in most Arabic speaking countries uses real Arabic digitsconsole.log(new Intl.NumberFormat('ar-EG').format(number));
// expected output: ١٢٣٤٥٦٫٧٨٩// India uses thousands/lakh/crore separatorsconsole.log(new Intl.NumberFormat('en-IN').format(number));
// expected output: 1,23,456.789// the nu extension key requests a numbering system, e.g. Chinese decimalconsole.log(new Intl.NumberFormat('zh-Hans-CN-u-nu-hanidec').format(number));
// expected output: 一二三,四五六.七八九// when requesting a language that may not be supported, such as
// Balinese, include a fallback language, in this case Indonesianconsole.log(new Intl.NumberFormat(['ban', 'id']).format(number));
// expected output: 123.456,789
```

在我看来，对于一个国际公司或网站来说，这一定是国际化 API 的一个巨大的*巨大的*卖点。我们不需要为存在的各种货币差异手工编码，我们可以依靠一个 API 来为我们处理它——多么合理和省时啊！

好了，今天我要介绍的最后一个属性是:`Intl.PluralRules`。

## [国际复数规则](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/PluralRules)

当你想到英语中复数的规则时，规则是很难的:“一只狗”变成了“两只狗”，但是“一只鹅”变成了“两只鹅”，“一只”变成了“两只”，但是当描述一些模糊的、大量的项目时，除了“两只”之外，就变成了“几只”或“许多”或“其他”，等等——你开始明白我在说什么了吗？

现在想象一下需要知道像这样的不止一种语言的不成文的规则…唉。😑

这就是国际多元规则成为救命稻草的地方。`**Intl.PluralRules**`对象是支持多种敏感格式和多种语言规则的对象的构造器。

这个构造函数接受`locales`(当然)，它的`options`对象接受以下参数:

*   `localeMatcher` —要使用的区域设置匹配算法。可能的值是`"lookup"`和`"best fit"`；默认为`"best fit"`。
*   `type` —要使用的类型。可能的值有:`"cardinal"`为基数(指事物的数量)。这是默认值。或`"ordinal"`作序数(指事物的排序或等级，如英语中的“第一”、“第二”、“第三”)。

**国际贸易的用途。复数规则**

在代码中使用`Intl.PluralRules`有几种不同的方式。

**基本用法** —在没有指定语言环境的基本用法中，返回默认语言环境中带有默认选项的格式化字符串。这有助于区分单数和复数形式，如“cat”和“cats”。

**地区用法** —使用`locales`时，下面的示例代码显示了本地化复数规则的一些变化。为了获得应用程序用户界面中使用的语言格式，请确保使用`locales`参数指定该语言(可能还有一些备用语言)。

**选项用法** —可以使用`options`参数定制多个结果，该参数有一个名为`type`的属性，您可以将其设置为`ordinal`。这有助于找出顺序指示器，例如“第一”、“第二”、“第三”、“第四”、“第四十二”等等。

下面的代码片段概述了我刚才描述的所有用法。

国际机场。复数规则示例

```
// basic usage examplesvar pr = new Intl.PluralRules();// examples of plural rules if in US English localeconsole.log(pr.select(0));
// expected output: 'other'console.log(pr.select(1)); 
// expected output: 'one'console.log(pr.select(2));
// expected output: 'other'// examples using `locale`
// for example: Arabic has different plural rulesconsole.log(new Intl.PluralRules('ar-EG').select(0));
// expected output: 'zero'console.log(new Intl.PluralRules('ar-EG').select(1)); 
// expected output: 'one'console.log(new Intl.PluralRules('ar-EG').select(2));
// expected output: 'two'console.log(new Intl.PluralRules('ar-EG').select(6));
// expected output: 'few'console.log(new Intl.PluralRules('ar-EG').select(18));
// expected output: 'many'// examples using `options`var pr = new Intl.PluralRules('en-US', { type: 'ordinal' });
// examples using US English and `options type` specificationconsole.log(pr.select(0));
// expected output: 'other'console.log(pr.select(1));
// expected output: 'one'console.log(pr.select(2));
// expected output: 'two'console.log(pr.select(3));
// expected output: 'few'console.log(pr.select(4));
// expected output: 'other'
```

很不错，是吧？我承认，在所有的`Intl`方法中，这种方法没有其他方法应用广泛，但是我确信它仍然会有它的成功时刻。

同样值得注意的是，这种方法仍然处于“草案”状态，在 MDN 的文档中有注释“初始定义”,所以在试图在生产中完全依赖它之前，我会推迟一段时间。

# 结论

翻译适用于不同国家和语言的网站和应用程序已经够困难的了，但是能够正确处理日期时间格式、货币和项目列表等具有许多语言特定规则的事情是一个真正的挑战。

ECMAScript 国际化 API 的最新版本旨在通过让其方法根据`locales`和您可能想要指定的任何其他`options`来为您进行格式化，从而使这些艰巨的任务变得更加容易。

对于处理国际需求的 JavaScript 开发人员来说，这将是一个巨大的福音，我本人也期待着将这些新特性融入到我自己的应用程序中，仅仅是为了这些特性提供的便利。

过几周再来看看，我会写关于 JavaScript、ES6 或其他与 web 开发相关的东西，所以请关注我，这样你就不会错过了。

感谢您的阅读，我希望您能看到 ECMAScript 国际化 API 能够为您的应用程序提供的价值，并能够在未来将一些属性整合到您的 web 应用程序中。如果你觉得有帮助，请与你的朋友分享！

如果你喜欢读这篇文章，你可能也会喜欢我的其他一些博客:

*   [同时运行多个 Node.js 或 NPM 命令的 4 个解决方案](/4-solutions-to-run-multiple-node-js-or-npm-commands-simultaneously-9edaa6215a93)
*   [视口单位，你不知道但应该知道的 CSS](/viewport-units-the-css-you-didnt-know-about-but-should-24b104483429)
*   [JavaScript 的异步/等待与承诺:大辩论](/javascripts-async-await-versus-promise-the-great-debate-6308cb2e10b3)

**参考资料和更多资源:**

*   ECMAScript 国际化 API，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Global _ Objects/Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)
*   国际日期时间格式，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Global _ Objects/datetime Format](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat)
*   国际相对时间格式，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Global _ Objects/RelativeTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RelativeTimeFormat)
*   国际列表格式，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Global _ Objects/List Format](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ListFormat)
*   国际号码格式，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Global _ Objects/Number Format](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NumberFormat)
*   国际复数规则，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Global _ Objects/Plural Rules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/PluralRules)