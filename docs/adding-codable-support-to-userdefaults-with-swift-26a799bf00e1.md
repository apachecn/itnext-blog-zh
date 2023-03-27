# 使用 Swift 为用户默认值添加可编码支持

> 原文：<https://itnext.io/adding-codable-support-to-userdefaults-with-swift-26a799bf00e1?source=collection_archive---------6----------------------->

![](img/c1fd6b890598506895a7613c755879dd.png)

[https://unsplash.com/photos/E0AHdsENmDg](https://unsplash.com/photos/E0AHdsENmDg)

如果你是一名 iOS 开发者，你很有可能在过去使用过**用户默认值**来保存和加载本地数据。

目前，UserDefaults 支持以下数据类型:

```
- URL
- Any (NSData, NSString, NSNumber, NSDate, NSArray, or NSDictionary)
- Bool
- Double
- Float
- Int
- String
```

# 好吧，但是 Codable 呢？🤔

假设我们有一艘符合**代码**的**星际飞船**结构。

```
struct Starship: Codable {
    let name: String
    let captain: String
}let enterprise = Starship(name: "Enterprise D", captain: "Jean Luc Picard")
```

我们如何**保存**企业*常量到用户默认值？*

目前，唯一的方法是在保存之前将企业数据转换为数据。稍后，当获取值时，我们需要将**数据**转换成 Starship。

很无聊吧？😴

有几个方法可以让**保存**和**加载**一个可编码的值，难道没有用吗？

# 改善用户默认值

好，让我们将下面的 UserDefault 扩展添加到我们的项目中。

```
extension UserDefaults {func set<Element: Codable>(value: Element, forKey key: String) {
        let data = try? JSONEncoder().encode(value)
        UserDefaults.standard.setValue(data, forKey: key)
    }func codable<Element: Codable>(forKey key: String) -> Element? {
        guard let data = UserDefaults.standard.data(forKey: key) else { return nil }
        let element = try? JSONDecoder().decode(Element.self, from: data)
        return element
    }}
```

*   第一个方法接收一个**可编码值**，并将其保存到 UserDefaults。
*   第二种方法返回与给定键相关联的可编码值。

多亏了这些方法，我们现在可以对任何可编码类型使用 UserDefaults。

# 试验🔨

最后，我们来做一个测试。

```
struct Starship: Codable {
    let name: String
    let captain: String
}let enterprise = Starship(name: "Enterprise D", captain: "Jean Luc Picard")UserDefaults.standard.set(value: enterprise, forKey: "enterprise")let value: Starship? = UserDefaults.standard.codable(forKey: "enterprise")
```

我们现在可以打印出**值** 常数*。*

```
Optional(Starship(name: "Enterprise D", captain: "Jean Luc Picard"))
```

# 为什么我们需要声明要获取的值的类型？🤔

好问题，为什么我们需要写这个

```
let value**: Starship?** = UserDefaults.standard.codable(forKey: "enterprise")
```

而不是这个？

```
let value = UserDefaults.standard.codable(forKey: "enterprise")
```

与其他 UserDefaults 方法(如**userderfaults . standard . integer(forKey:)**)不同，我们的 **codable(forKey:)** 方法并不总是返回相同的类型。

喂👇

```
struct Starship: Codable {
    let name: String
    let captain: String
}struct SpaceStation: Codable {
    let name: String
}let enterprise = Starship(name: "Enterprise D", captain: "Jean Luc Picard")
let deepSpace9 = SpaceStation(name: "Deep Space Nine")UserDefaults.standard.set(value: enterprise, forKey: "enterprise")
UserDefaults.standard.set(value: deepSpace9, forKey: "deepSpace9")let value0: Starship? = UserDefaults.standard.codable(forKey: "enterprise")
let value1: SpaceStation? = UserDefaults.standard.codable(forKey: "deepSpace9")
```

观察最后两行，我们的方法是返回一艘**星际飞船**，然后返回一艘**空间站**。

这就是为什么我们需要告诉我们的方法我们期望的类型。