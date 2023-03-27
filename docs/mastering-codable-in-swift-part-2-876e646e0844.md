# 用 Swift 用 4 个步骤解码 JSON

> 原文：<https://itnext.io/mastering-codable-in-swift-part-2-876e646e0844?source=collection_archive---------2----------------------->

![](img/a3015b1062239ed0a8b2c9a0c8d2803f.png)

[https://pix abay . com/photos/letters-numbers-blocks-alphabet-691842/](https://pixabay.com/photos/letters-numbers-blocks-alphabet-691842/)

# 介绍

在本文中，我将给出一个在 Swift 中解码 JSON 的步骤列表。

*   **第一步**:剖析你的 JSON
*   **第二步**:使用 String、Double、Bool 等 Swift 可编码类型定义一个匹配 JSON 的 struct
*   步骤 3 :定义你的类型来管理一些 JSON 片段
*   **步骤 4** :为 *JSONDecoder* 设置一个*按键解码策略*

> 这个规则有一些例外，我将在以后的文章中讨论这些特殊的场景。

![](img/4529de39eec643b95548b95ae3e08eee.png)

[https://unsplash.com/photos/6gQjPGx1uQw](https://unsplash.com/photos/6gQjPGx1uQw)

# ✅第一步:剖析你的 JSON

JSON 数据被表示为键/值对或/和数组。

该键总是由一个*字符串表示，*并且一个值可能包含以下内容之一:

1.  *字符串*
2.  *编号*
3.  *布尔型*
4.  *阵列*
5.  *空值*
6.  *另一个 JSON 对象*

一旦你学会了管理这些👆六个可能的值，可以用 Codable 解码任何可能的 JSON。

# ✅步骤 2:使用 Swift 可编码类型定义一个匹配 JSON 的结构

请记住，在 Swift 中处理 JSON 解码的大部分工作是定义一个符合 *Codable* 并且匹配 JSON 结构的模型类型。

可能是

*   头等
*   一个结构
*   枚举

通常，我更喜欢使用一个*结构*(所有属性都定义为常量)，因为:

*   当我从一个端点接收到一个 JSON 时，我认为该数据是不可变的。
*   结构值(位于堆栈中)比类实例(位于堆中)快。

## 线

假设您有一个这样的 JSON，其中只有一个类型为 *String* 的元素。

```
{
    "name": "BoJack Horseman"
}
```

在这种情况下，您需要像这样定义一个结构

```
struct Show: Decodable {
    let name: String
}
```

## 数字

让我们向 JSON 添加另一个元素

```
{
    "name": "BoJack Horseman",
    **"seasons": 3**
}
```

*seasons* 键将始终包含一个数字，特别是一个*整数*。所以我们可以这样更新我们的模型结构

```
struct Show: Decodable {
    let name: String
    **let seasons: Int**
}
```

让我们向 JSON 添加另一个值

```
{
    "name": "BoJack Horseman",
    "seasons": 3,
    **"rate": 8.6**
}
```

*rate* 键包含一个 float 值，这意味着我们应该在我们的结构定义中使用一个 *Float* 或一个 *Double*

```
struct Show: Decodable {
    let name: String
    let seasons: Int
    **let rate: Float**
}
```

## 布尔代数学体系的

根据 JSON 标准，有效的布尔值应该包含*真*或*假*。

> 以下值是**而不是**认为有效的布尔值:是，否，“是”，“否”，“真”，“假”，1，0，“1”，“0”。

现在让我们给 JSON 添加一个布尔值。

```
{
    "name": "BoJack Horseman",
    "seasons": 3,
    "rate": 8.6,
    **"favorite": true**
}
```

同样，我们需要做的就是向我们的*结构*添加一个属性(具有匹配的*名称*和*类型*

```
struct Show: Decodable {
    let name: String
    let seasons: Int
    let rate: Float
    **let favorite: Bool**
}
```

## 排列

在 JSON 中，数组值是用以下语法定义的

```
[value(0), value(1), ..., value(n-1)]
```

所以当你看到一个具有这种结构的 JSON 片段时，你知道你必须使用 Swift 数组。

> 请记住，数组的泛型类型必须符合 codable。所以使用一个字符串数组，一个整数数组是可以的…

让我们用一个数组更新我们的 JSON

```
{
    "name": "BoJack Horseman",
    "seasons": 3,
    "rate": 8.6,
    "favorite": true,
    **"genres":["Animation", "Comedy", "Drama"]**
}
```

让我们相应地更新我们的结构

```
struct Show: Decodable {
    let name: String
    let seasons: Int
    let rate: Float
    let favorite: Bool
    **let genres: [String]**
}
```

## 5.空

有时 JSON 的值可能包含空值。如果根据端点的文档，这不是一个错误，您应该将*结构*中的相关属性设为可选。否则，JSON 解码将不起作用。

例如，如果我们更新先前的 JSON，将 *Null* 关联到*收藏夹*键…

```
{
    "name": "BoJack Horseman",
    "seasons": 3,
    "rate": 8.6,
    **"favorite": null,**
    "genres":["Animation", "Comedy", "Drama"]
}
```

…解码将不起作用。

```
Swift.DecodingError.valueNotFound(Swift.Bool, Swift.DecodingError.Context(codingPath: [CodingKeys(stringValue: "favorite", intValue: nil)], debugDescription: "Expected Bool value but found null instead.", underlyingError: nil))
```

因为我们知道 *Null* 可能是 *favorite* 属性的有效“值”,所以我们可以将其设为可选

```
struct Show: Decodable {
    let name: String
    let seasons: Int
    let rate: Float
    **let favorite: Bool?**
    let genres: [String]
}
```

# ✅步骤 3:定义你的类型来管理一些 JSON 片段

最后，我们的 JSON 可以包含另一个 JSON 对象。它可能看起来像这样

```
{
    "name": "BoJack Horseman",
    "seasons": 3,
    "rate": 8.6,
    "favorite": null,
    "genres":["Animation", "Comedy", "Drama"],
    **"platform": {
        "name": "Netflix",
        "ceo": "Reed Hastings"
    }**
}
```

![](img/dd0cfb7604647e57bce5256b3afba924.png)

[https://unsplash.com/photos/VbGYLwHnw88](https://unsplash.com/photos/VbGYLwHnw88)

## 另一个 JSON 对象

我们如何管理*平台*密钥？不是一个*字符串*，一个*数字*，一个*布尔*或者一个*数组*。我们没有一个 Swift 类型与这个值完全匹配，对吗？

```
"platform" {
    "name": "Netflix",
    "ceo": "Reed Hastings"
}
```

在这种情况下，我们可以**递归地**应用本文中描述的步骤，并专门为这个 JSON 片段定义一个新的结构。

```
struct Show: Decodable {
    let name: String
    let seasons: Int
    let rate: Float
    let favorite: Bool?
    let genres: [String]
    **let platform: Platform** **struct Platform: Decodable {
        let name: String
        let ceo: String
    }**
}
```

> 记住: *Show* 的每个属性必须有一个符合 *Decodable* 的类型。因此*平台*类型也必须符合可解码。

## 关于字符串的更多信息

有时候你的 JSON 可能包含一个 URL。JSON 语法没有 URL 的特定格式。它们只是被编码成字符串。

```
{
    "name": "BoJack Horseman",
    "seasons": 3,
    "rate": 8.6,
    "favorite": null,
    "genres":["Animation", "Comedy", "Drama"],
    "platform": {
        "name": "Netflix",
        "ceo": "Reed Hastings"
    },
    **"url":"**[**https://en.wikipedia.org/wiki/BoJack_Horseman**](https://en.wikipedia.org/wiki/BoJack_Horseman)**"**
}
```

所以你可以添加一个*字符串*属性到你的结构中，对吗？

```
struct Show: Decodable {
    let name: String
    let seasons: Int
    let rate: Float
    let favorite: Bool?
    let genres: [String]
    let platform: Platform
    **let url: String**

    struct Platform: Codable {
        let name: String
        let ceo: String
    }
}
```

这个代码是有效的。然而，有一个问题。即使 *url* 包含一些不是这样的 url 的字符串值，解码也会工作👇

```
{
    "name": "BoJack Horseman",
    "seasons": 3,
    "rate": 8.6,
    "favorite": null,
    "genres":["Animation", "Comedy", "Drama"],
    "platform": {
        "name": "Netflix",
        "ceo": "Reed Hastings"
    },
    **"url":"NOT A URL!!"**
}
```

我们可以强制 Codable 只接受有效 URL 的字符串来代替它

```
struct Show: Decodable {
    ...
    let url: String
    ...
}
```

与这个

```
struct Show: Decodable {
    ...
    let url: URL
    ...
}
```

# ✅步骤 4:为 JSONDecoder 设置一个密钥解码器和日期格式

## 蛇箱呼叫骆驼箱

JSON 键通常使用 *snake_case* 符号。让我们相应地更新我们的 JSON

```
{
    "name": "BoJack Horseman",
    "**num_seasons**": 3, 
    "rate": 8.6,
    "**is_favorite**": null,
    "genres":["Animation", "Comedy", "Drama"],
    "platform": {
        "name": "Netflix",
        "ceo": "Reed Hastings"
    },
    "url": "[https://en.wikipedia.org/wiki/BoJack_Horseman](https://en.wikipedia.org/wiki/BoJack_Horseman)"
}
```

然而，Swift 指南建议使用*驼峰式*符号，因此应该如下所示更新该结构

```
struct Show: Decodable {
    let name: String
    let **numSeasons**: Int
    let rate: Float
    let **isFavorite**: Bool?
    let genres: [String]
    let platform: Platform
    let url: String

    struct Platform: Codable {
        let name: String
        let ceo: String
    }
}
```

幸运的是，我们可以告诉 Codable 自动管理用于键的不同符号(例如，将 *num_seasons* 转换为 *numSeasons* )来设置 key decoding 策略

```
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

这是你可以在操场上运行的完整代码

```
let data = """
{
    "name": "BoJack Horseman",
    "num_seasons": 3,
    "rate": 8.6,
    "is_favorite": null,
    "genres":["Animation", "Comedy", "Drama"],
    "platform": {
        "name": "Netflix",
        "ceo": "Reed Hastings"
    },
    "url": "[https://en.wikipedia.org/wiki/BoJack_Horseman](https://en.wikipedia.org/wiki/BoJack_Horseman)"
}
""".data(using: String.Encoding.utf8)!struct Show: Decodable {
    let name: String
    let numSeasons: Int
    let rate: Float
    let isFavorite: Bool?
    let genres: [String]
    let platform: Platform
    let url: String

    struct Platform: Codable {
        let name: String
        let ceo: String
    }
}let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
do {
    let show = try decoder.decode(Show.self, from: data)
    print(show)
} catch {
    debugPrint(error)
}
```

## 处理不一致的键命名

有时 JSON 没有遵循正确的*键*命名规则

```
{
    "**Name**": "BoJack Horseman",
    "**num_seasons**": 3,
    "rate": 8.6,
    "**isFavorite**": null,
    "genres":["Animation", "Comedy", "Drama"],
    "platform": {
        "name": "Netflix",
        "ceo": "Reed Hastings"
    },
    "url": "[https://en.wikipedia.org/wiki/BoJack_Horseman](https://en.wikipedia.org/wiki/BoJack_Horseman)"
}
```

如你所见，我们有几处不一致

*   **名字**以大写字母开头
*   **num_seasons** 在 snake_case 符号中是小写
*   **isFavorite** 遵循驼峰式符号

在这种情况下，最好的解决方案是在 JSON 中的**键名**和我们的 struct 中的**属性名**之间提供映射。为此，我们向结构中添加一个枚举，如下所示。

```
enum CodingKeys: String, CodingKey {
    case name = "Name"
    case numSeasons = "num_seasons"
    case rate = "rate"
    case isFavorite = "isFavorite"
    case genres = "genres"
    case platform = "platform"
    case url = "url"
}
```

请记住，枚举:

*   必须命名为 *CodingKeys*
*   必须将*字符串*作为 *rawType*
*   必须符合*编码密钥*协议
*   必须在结构内部定义
*   结构的每个属性都必须有一个案例
*   每个 case 都必须有一个与 JSON 中的键名匹配的字符串值

> 请注意，对于那些名称与 JSON ( *rate，platform，url* )中的键匹配的属性，可以省略字符串值

```
enum CodingKeys: String, CodingKey {
    case name = "Name"
    case numSeasons = "num_seasons"
    case rate
    case isFavorite = "isFavorite"
    case genres
    case platform
    case url
}
```

最后，让我们将 enum 添加到我们的结构中，并删除我们设置*密钥解码策略*的那一行。

```
let data = """
{
    "Name": "BoJack Horseman",
    "num_seasons": 3,
    "rate": 8.6,
    "isFavorite": null,
    "genres":["Animation", "Comedy", "Drama"],
    "platform": {
        "name": "Netflix",
        "ceo": "Reed Hastings"
    },
    "url": "[https://en.wikipedia.org/wiki/BoJack_Horseman](https://en.wikipedia.org/wiki/BoJack_Horseman)"
}
""".data(using: String.Encoding.utf8)!struct Show: Decodable {
    let name: String
    let numSeasons: Int
    let rate: Float
    let isFavorite: Bool?
    let genres: [String]
    let platform: Platform
    let url: String

    struct Platform: Codable {
        let name: String
        let ceo: String
    }

 **enum CodingKeys: String, CodingKey {
        case name = "Name"
        case numSeasons = "num_seasons"
        case rate
        case isFavorite = "isFavorite"
        case genres
        case platform
        case url
    }**
}let decoder = JSONDecoder()
do {
    let show = try decoder.decode(Show.self, from: data)
    **// decoder.keyDecodingStrategy = .convertFromSnakeCase**
    print(show)
} catch {
    debugPrint(error)
}
```

# 结论

当我需要构建一个模型类型来匹配给定的 JSON 时，这些是我遵循的基本步骤。按照这些说明，您应该能够解码大部分的 JSONs。

有一些例外(比如在顶层有一个数组的 JSON 或者在编译时有未知键的 JSON)，我将在以后的文章中介绍。