# 你应该是 2019 年的林挺

> 原文：<https://itnext.io/you-should-be-linting-in-2019-a813021eb45e?source=collection_archive---------3----------------------->

![](img/d5d3dec54b750cd89004c4dd202a2d99.png)

[野蝉公园](https://unsplash.com/@yechan0422?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

现代网络已经走过了漫长的道路。我们现在有 TypeScript 和 ES2019，Sass，GraphQL，Webpack，NodeJS， [@pika](https://www.pika.dev/about/) ，Loopback 4，Aurelia 和 Ember。随着工具、平台和语言的改进，更多的逻辑从后端转移到了前端；从 java/c#到 JavaScript/TypeScript；从 Tomcat 到 NodeJS。这是一场革命，或者说，是一场网络复兴。然而，这些技术也增加了前端解决方案的复杂性。这种复杂性也使得实现代码标准比以往任何时候都更加重要。开始提高你的代码标准的最好方法是通过[林挺](https://en.wikipedia.org/wiki/Lint_%28software%29)。

## 什么是*林挺*？

林挺帮助定义公认的代码语法和模式。这对于开发团队来说是确保代码质量和一致性的关键。简而言之，林挺就是将可接受的代码与好的代码、伟大的代码与好的代码区分开来的东西。

## **为什么** **你** **要当林挺？**

因为你希望你的代码更好。因为您希望团队中的每个人都保持相同的代码风格。不要被另一个简单的错误弄得措手不及。谁不想要更好的代码呢？

## tldr

1.  **开药语法一致。从奇怪的间距和糟糕的缩进到函数调用中参数的重复，大多数团队都有不一致的代码。通用林挺规则意味着你团队的代码可以有一致的语法。**
2.  **预防常见 bug**。从影子变量到未声明的变量，从未使用的变量到无法访问的代码，一个配置良好的 linter 可以帮助识别常见问题，以便您可以在它们进入生产之前解决它们。
3.  **改进代码评审。有了林挺鼓励的已定义的编码标准，代码评审可以关注质量和可维护性。**
4.  **提高代码质量。当代码评审聚焦于质量时，你的团队将能够讨论更大的问题，比如技术债务和小代码。**

# 1.一致的语法

这是大多数人认为林挺支持的唯一一件事……这就是为什么新球队不重视林挺。这是一个错误。

当代码不一致时，很难关注质量。不同的巨大标签？从业务逻辑中转移注意力的奇怪间距？不一致的语法会使代码难以阅读。一旦你标记了这些东西，你就可以用棉绒一步搞定它们。这里有一些规则来说明什么是可能的:

**强制一致缩进(indent)。经过几年的前端开发，如果有一件事让我抓狂，那就是四个或更多的空间标签。Java 开发者更喜欢八个空格制表符吗？这个规则，当被你的团队定义时，可以给疯狂带来秩序。**

**不允许多空行(no-multiple-empty-lines)。**为什么有些开发人员会在代码行之间添加四行或更多行？！他们是按代码行付费的吗？过度滚动使得理解代码更加困难，这个规则可以让你标记它或者自动修复它。

**强制使用一致的反斜线、双引号或单引号(** [**引号**](https://eslint.org/docs/rules/quotes) **)。**多年来，我参与了多个前端项目，逐渐体会到了其中的不同。单引号并不意味着字符，因为没有字符，只有字符串。HTML 可能到处都需要双引号，但是 JavaScript 不需要。反斜杠在 ECMAScript 中有特殊的含义—它们是模板字符串。这些规则将有助于使您的报价风格更加一致。

**强制逗号周围的间距(**[](https://eslint.org/docs/rules/comma-spacing#enforces-spacing-around-commas-comma-spacing)****)。我肯定你见过奇怪的逗号间距——比如逗号前有空格或者逗号后没有空格。****

****要求拉条式(** [**拉条式**](https://eslint.org/docs/rules/brace-style#require-brace-style-brace-style) **)。**【1TBS】为赢——为什么还要浪费尚未*的另一条*线呢？设置此规则来修复它。**

****需要遵循花括号约定(** [**花括号**](https://eslint.org/docs/rules/curly#require-following-curly-brace-conventions-curly) **)。花括号很重要。它们作为一个代码块的视觉提示…这个规则可以让你确保它们总是在那里。****

****要求山茶案变量名(**[](https://eslint.org/docs/rules/camelcase#require-camelcase-camelcase)****)。** MY_SPECIAL_CONST，Firstname，firstname 或 FIRstname。叹气。为什么你想要几十种命名类型？选一个。您的团队可以通过启用此规则来带来秩序。****

****当代码在开发人员中看起来一样时，它就成为了一个预期的标准:*这是我们在这里做事的方式*。当无论你在看什么模块，代码看起来都一样时，就更容易理解和维护。然而，当代码风格从一个模块到下一个模块完全不同时，它促进了区域编码…例如，一个模块只有一个开发人员理解。****

****用右手。你可以修复 90%的代码格式不一致。通过在 VSCode 中选择`Format All`或在 cmd 行运行`eslint --fix`，可以自动修复语法问题。虽然不是所有的代码格式问题都可以自动修复，但至少 eslint 会显示警告或错误。在 VSCode 中，您会看到红色的曲线，并且可以配置保存时应用的格式规则。****

# ****2.防止常见错误****

****林挺还可以执行 [*静态代码分析*](https://en.wikipedia.org/wiki/Static_program_analysis) 来标记编程错误，防止常见错误并识别可疑结构。以下是捕捉这些问题的一些规则:****

******不允许变量声明隐藏在外部作用域中声明的变量(** [**无阴影**](https://eslint.org/docs/rules/no-shadow#disallow-variable-declarations-from-shadowing-variables-declared-in-the-outer-scope-no-shadow) **)。**不对不对。我已经在外部作用域中声明了那个变量。我是想隐藏那个变量吗？我应该使用外部变量吗？如果不是，也许我们应该重命名这个内部变量以避免混淆。****

******不允许因使用** `**await**` **或**`**yield**`**(**[**require-atomic-updates**](https://eslint.org/docs/rules/require-atomic-updates#disallow-assignments-that-can-lead-to-race-conditions-due-to-usage-of-await-or-yield-require-atomic-updates)**)而导致竞态条件的赋值。**在 ACID 中,“A”代表原子更新。通过要求原子更新，你是说我将只给一个可观察对象赋值一次(比如更新 UI 的东西)。通过启用此规则，有助于在您的代码中停止这种争用情况。****

******不允许空块语句(** [**非空**](https://eslint.org/docs/rules/no-empty#disallow-empty-block-statements-no-empty) **)。**一个什么都没有的 if 块？为什么？！也许这个逻辑有更清晰的写法？不要强迫其他开发人员(或你自己)在 6 个月内浪费时间去弄清楚意图。保持你的代码清晰明了。****

******return、throw、continue、break 语句后不允许不可达代码(**[**no-unreachable**](https://eslint.org/docs/rules/no-unreachable#disallow-unreachable-code-after-return-throw-continue-and-break-statements-no-unreachable)**)。如果你在扫描代码时没有立即看到那个返回或抛出…你会想为什么你写的代码没有到达或者假设它运行了。如果你的棉绒标记它，那么你的团队可以避免歧义。******

******不允许对象文字中有重复的关键字(** [**无重复关键字**](https://eslint.org/docs/rules/no-dupe-keys#disallow-duplicate-keys-in-object-literals-no-dupe-keys) **)。最近，我的同事在我们的应用程序中检查一些多年前的棕色地带代码。他轻笑着问道:“这里到底发生了什么？当初的开发者怎么就没看到这一点呢？”简单。他们不是林挺……也没有这条规定。******

******不允许对本机对象或只读全局变量赋值(** [**【无全局赋值】**](https://eslint.org/docs/rules/no-global-assign#disallow-assignment-to-native-objects-or-read-only-global-variables-no-global-assign) **)。**嗯，为什么我要在文档中附加一个变量？！这条规则将有助于防止这种糟糕的编码实践。****

# ****3.改进代码审查****

****你的团队是否在代码评审中苦苦挣扎……有许多来回和史诗般的争论？如果是这样，定义的标准和林挺的好处应该是显而易见的。在林挺规则集中包含已定义的代码标准，可以作为准备评审的代码的基线。通过确保一致的语法和防止常见的错误，您的代码审查将更快，并提供更多的价值。当你的代码评审不是一场语法大战时，你和你的评审可以专注于*业务逻辑。*****

# ******4。提高代码质量******

****有了一致的代码，并关注代码评审中的业务逻辑，开发人员就可以关注更大的画面:质量。基本的软件工程原则，像[固体](https://en.wikipedia.org/wiki/SOLID)现在可以成为你努力的焦点。代码是否不允许修改，但允许扩展？我们是否在多个地方复制/粘贴了这个实用函数？这段代码是把 15 件事做差了还是把一件事做好了？我们能否将这种逻辑转移到服务中以确保一致的行为？****

# ******总结******

****在这篇文章中，我展示了**为什么**你应该成为 2019 年的林挺。许多团队忽略或降低了林挺的优先级，因为它的优势并不明显。希望这篇文章为你的团队今天开始林挺之旅提供了一个令人信服的案例。****

****[**史蒂夫·哈特佐格**](http://about.me/steve.hartzog) 是一名暗物质开发者和敏捷[认证的 ScrumMaster](https://www.scrumalliance.org/community/profile/shartzog) ，他作为软件工程师、系统经理和系统管理员为许多公司和政府机构提供咨询。他最喜欢的技术有 [TypeScript](http://www.typescriptlang.org/) 、 [Aurelia](https://aurelia.io/) 和 [@pika](https://github.com/pikapkg/web) 。他非常喜欢巴黎和附近的凡尔赛，但目前和他的疯猫 Scotty 住在 DC 郊区。他在 Twitter[*@ s*teve . hart zog](https://twitter.com/SteveHartzog)上，并在 Code Camps 和 meetups 上就多个主题发表过演讲。****