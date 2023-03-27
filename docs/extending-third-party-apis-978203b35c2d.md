# 用不同的语言扩展第三方 API

> 原文：<https://itnext.io/extending-third-party-apis-978203b35c2d?source=collection_archive---------4----------------------->

![](img/1339164c6e41d4c84d584f0eae8d444e.png)

越来越短的上市时间要求集成越来越多的第三方库。如果曾经有过的话，现在已经没有时间治疗 T4 国立卫生研究院综合症了。虽然大多数时候，该库的 API 都是现成可用的，但有时需要“调整”它以适应代码库。改编的难易程度很大程度上取决于语言。

例如，在 JVM 中，有几个反应式编程库:RxJava、Project Reactor、哗变和协程。您可能需要一个使用一个库的类型的库，但是您的项目基于另一个库。

在这篇文章中，我想描述如何给现有的对象/类型添加新的行为。我不会使用任何反应型来使它更通用，而是将`toTitleCase()`添加到`String`。当它存在时，继承是**而不是**一个解决方案，因为它创建了一个新的类型。

我提前道歉，下面的实现非常简单:它们旨在强调我的观点，而不是处理角落情况，*例如*，空字符串，非 UTF 8，等等。

# Java Script 语言

JavaScript 是一种动态解释的弱类型语言，在 WASM 接管之前，它运行着万维网？据我所知，它的设计很独特，因为它是基于原型的。原型是该类型的新“实例”的模型。

您可以很容易地向原型添加属性，无论是状态还是行为。

```
Object.defineProperty(String.prototype, "toTitleCase", {
    value: function toTitleCase() {
        return this.replace(/\w\S*/g, function(word) {
            return word.charAt(0).toUpperCase() + word.substr(1).toLowerCase();
        });
    }
});console.debug("OncE upOn a tImE in thE WEst".toTitleCase());
```

注意，在对`defineProperty`的调用之后，从这个原型*创建的对象将提供新的属性；在*之前*创建的对象不会。*

# 红宝石

Ruby 是一种解释型动态强类型语言。虽然没有 Ruby On Rails 框架那么受欢迎，但我仍然在 Jekyll 系统中使用它，这是这篇博客的动力。

在 Ruby 生态系统中，向现有类添加方法或属性是相当标准的。我发现了两种在 Ruby 中向现有类型添加方法的机制:

1.  Use [class_eval](https://apidock.com/ruby/Module/class_eval) :
    "在 mod 的上下文中计算字符串或块，除了给定块时，常量/类变量查找不受影响。这可用于向类添加方法”
2.  只需在现有类上实现该方法。

下面是第二种方法的代码:

```
class String
  def to_camel_case()
    return self.gsub(/\w\S*/) {|word| word.capitalize()}
  end
endputs "OncE upOn a tImE in thE WEst".to_camel_case()
```

# 计算机编程语言

Python 是一种解释型动态强类型语言。我想现在每个开发者都听说过 Python。

Python 允许向现有类型添加函数，但有限制。[让我们试试`str`内置型的](https://www.online-python.com/yv52IK4Mux):

```
import redef to_title_case(string):
    return re.sub(
        r'\w\S*',
        lambda word: word.group(0).capitalize(),
        string)setattr(str, 'to_title_case', to_title_case)print("OncE upOn a tImE in thE WEst".to_title_case())
```

不幸的是，上面的代码在执行过程中失败了:

```
Traceback (most recent call last):
  File "<string>", line 9, in <module>
TypeError: can't set attributes of built-in/extension type 'str'
```

因为`str`是*内置的*类型，所以我们不能动态添加行为。我们可以更新[代码](https://www.online-python.com/w4G0We7EYh)来应对这个限制:

```
import redef to_title_case(string):
    return re.sub(
        r'\w\S*',
        lambda word: word.group(0).capitalize(),
        string)class String(str):
    passsetattr(String, 'to_title_case', to_title_case)print(String("OncE upOn a tImE in thE WEst").to_title_case())
```

现在扩展`String`成为可能，因为它是我们创建的一个类。当然，这违背了最初的目的:我们首先必须扩展`str`。因此，它可以与第三方库一起工作。

使用解释型语言，向类型添加行为相当容易。然而，Python 已经触及了极限，因为内置类型是用 c 实现的。

# Java 语言(一种计算机语言，尤用于创建网站)

Java 是运行在 JVM 上的静态和强类型编译语言。它的静态性质使得不可能向类型添加行为。

解决方法是使用`static`方法。如果你已经做了很长时间的 Java 开发人员，我相信你可能在职业生涯的早期见过定制的`StringUtils`和`DateUtils`类。这些类看起来像这样:

```
public class StringUtils { public static String toCamelCase(String string) {
        // The implementation is not relevant
    } // Other string transformations here
}
```

我希望到现在为止，使用 Apache Commons 和 Guava 已经取代了所有这些类:

```
System.out.println(
    WordUtils.capitalize("OncE upOn a tImE in thE WEst")
);
```

在这两种情况下，*静态方法的使用阻止了流畅的 API 使用*，从而损害了开发人员的体验。但是其他 JVM 语言确实提供了令人兴奋的选择。

# 斯卡拉

像 Java 一样，Scala 是一种编译过的、静态的、强类型的语言，运行在 JVM 上。它最初被设计为面向对象编程和函数式编程之间的桥梁。Scala 提供了许多强大的特性。其中，*隐式*类允许向现有类添加行为和状态。[这里的](https://scastie.scala-lang.org/razUhHKRRcqamn9qlA0mhw)是如何给`String`添加`toCamelCase()`功能:

```
import Utils.StringExtensionsobject Utils {
  implicit class StringExtensions(thiz: String) {
    def toCamelCase() = "\\w\\S*".r.replaceAllIn(
      thiz,
      { it => it.group(0).toLowerCase().capitalize }
    )
  }
}println("OncE upOn a tImE in thE WEst".toCamelCase())
```

虽然我对 Scala 略有涉猎，但我从来都不是一个粉丝。作为一名开发人员，我总是说我工作的很大一部分是使*隐式*需求*显式*。因此，我不赞成故意使用`implicit`关键字。有趣的是，似乎我并不孤单。Scala 3 使用更合适的语法保持了[相同的功能](https://scastie.scala-lang.org/18abIFMKSvWiz8gpbVx2gg):

```
extension(thiz: String)
  def toCamelCase() = "\\w\\S*".r.replaceAllIn(
    thiz,
    { it => it.group(0).toLowerCase().capitalize }
  )
```

注意，在这两种情况下，*字节码*有点类似于 Java 的*静态*方法。然而，API 的使用是流畅的，因为您可以一个接一个地链接方法调用。

# 科特林

像 Java 和 Scala 一样，Kotlin 是一种编译过的、静态的、强类型的语言，运行在 JVM 上。包括 Scala 在内的其他几种语言启发了它的设计。

我的看法是 Scala 比 Kotlin 更强大，但代价是额外的认知负荷。相反，科特林有一个轻量级的方法，更加务实。下面是[科特林版本](https://pl.kotl.in/b67HIw06t):

```
fun String.toCamelCase() = "\\w\\S*"
    .toRegex()
    .replace(this) {
        it.groups[0]
            ?.value
            ?.lowercase()
            ?.replaceFirstChar { char -> char.titlecase(Locale.getDefault()) }
            ?: this
    }println("OncE upOn a tImE in thE WEst".toCamelCase())
```

如果你想知道为什么 Kotlin 代码比 Scala 代码更冗长，尽管我之前说过，这里有两个原因:

1.  我对 Scala 不够了解，所以没有管理好角落案例(空捕获等。)，但科特林让你别无选择
2.  Kotlin 团队从 Kotlin 1.5 的`stdlib`中移除了`capitalize()`函数

# 锈

最后但同样重要的是，Rust 是一种编译语言，静态和强类型。它最初被设计用来产生本地二进制文件。然而，通过相关配置，它还允许生成 Wasm。如果你感兴趣的话，我在学习语言的时候做了 link:/focus/start-rust/[几个笔记]。

有趣的是，尽管是静态类型的，Rust 也允许扩展第三方 API，如[下面的代码显示了](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=8d1daecd7bd46d6352c131cbf8186839):

```
trait StringExt {                                               // 1
    fn to_camel_case(&self) -> String;
}impl StringExt for str {                                        // 2
    fn to_camel_case(&self) -> String {
        let re = Regex::new("\\w\\S*").unwrap();
        re.captures_iter(self)
            .map(|capture| {
                let word = capture.get(0).unwrap().as_str();
                let first = &word[0..1].to_uppercase();
                let rest = &word[1..].to_lowercase();
                first.to_owned() + rest
            })
            .collect::<Vec<String>>()
            .join(" ")
    }
}println!("{}", "OncE upOn a tImE in thE WEst".to_camel_case());
```

1.  创建抽象来保存函数引用。这被称为铁锈的一个特征。
2.  为现有结构实现 trait。

Trait 实现有一个限制:我们的代码必须声明 trait 或 structure 中的至少一个。您不能为现有结构实施现有特征。

# 结论

在写这篇文章之前，我认为解释语言允许扩展外部 API，而编译语言不允许——kot Lin 是个例外。收集材料后，我的理解发生了巨大的变化。

我意识到所有主流语言都提供了这样一个特性。虽然我没有包含 C#部分，但它也包含了。我的结论是可悲的，因为 Java 是唯一一种在这方面没有提供任何东西的语言。

我经常说 Kotlin 相对于 Java 最大的好处是扩展属性/方法。虽然 Java 团队继续为这种语言添加特性，但它仍然没有提供接近上述任何一种语言的开发人员体验。由于我已经使用 Java 二十年了，我觉得这个结论有点悲哀，但不幸的是，事实就是如此。

**更进一步:**

*   [Scala 3 语言参考:扩展方法](https://docs.scala-lang.org/scala3/reference/contextual/extension-methods.html)
*   [科特林扩展](https://kotlinlang.org/docs/extensions.html)

*原载于* [*一个 Java 怪胎*](https://blog.frankel.ch/extending-third-party-apis/)*2021 年 11 月 7 日*