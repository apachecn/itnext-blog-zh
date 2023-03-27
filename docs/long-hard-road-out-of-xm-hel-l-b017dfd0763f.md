# 走出 XM(hel)L 的漫漫长路

> 原文：<https://itnext.io/long-hard-road-out-of-xm-hel-l-b017dfd0763f?source=collection_archive---------4----------------------->

从大学时代起，我就不是 XML 序列化的狂热爱好者；它引入了开销，但收益不大。然而，现在有很多 API，作为 iOS 开发者，我们需要直面它们。

![](img/4f9df805c9565a368dae5d52b3c0608b.png)

即使在瞬息万变的世界里，我们有时也会哭泣！！！

我非常喜欢 Swift 4 中引入的`Decodable`协议。我在`JSONDecoder`和`PlistDecoder`中多次使用过它，但不幸的是，XML 的标准库中没有任何东西。我在 Shawn Moore 的 [Stack Overflow](https://stackoverflow.com/questions/45787760/implementing-a-custom-decoder-in-swift-4) 上找到了一份出色的解决方案草案，但是有很多未解决的问题，不幸的是，没有经过测试。所有这些基础给了我通过`XMLDecoder`开始这段旅程的动力(目前我既不需要名称空间也不需要编码器)。

# 案例研究:Decodable 和 JSON

首先，我想澄清一下`Decodable`允许一个人做什么。假设我们有这个`JSON`文件:

这些是相关的实体:

现在有了`JSONDecoder`，把它们从`JSON`翻译成`struct`就相当简单了:

convertFromSnakeCase 有助于将 first_name 翻译成 firtsName

`decode`方法分两步进行:

*   用`JSONSerialization.jsonObject(with: data)`从`Data`到`Any`；
*   从`Any`到`Decodable`用解码器对每个实体进行拆箱和造型。

# XML 解析和规范化

第一个目标是将类似于`<root><key>value</key></root>`的东西映射到类似于`["key": "value"]`的东西。我编写了一个`XMLTree`来将`Data`映射到`[String: Any]`(XML 中的*映射*比 JSON 简单，因为`Any`可以只是`String`或`[String: Any]`)。我已经使用事件驱动的`XMLParser`构建了一个可导航的`XMLNode`树。

在这一步中，我主要面临两个问题，总结为两个例子:

```
<root>
  <foo id="1">bar</foo>
</root>
```

我已经选择将*匿名* `value`作为一个带有键值**_ 的属性。**

```
["key": [ "id": "1", "_value": "bar"]]
```

我知道这不是最干净的解决方案，但它足够*快速不脏*通过第一版。

但是继续第二个问题:

```
<root>
    <foo>1</foo>
    <foo>2</foo>
    <foo>3</foo>
    <foo>4</foo>
</root>
```

在第一个`foo`的访问中，我不能说它是否是一个链表元素的标量，所以我必须假设它是一个标量:`dictionary["foo"] = 1`。在第二轮中，我需要知道是否存储了以前的值，以便将新值添加到`dictionary["foo"]`，就像`[1, 2]`。从第三个开始就更简单了，因为可以将新值附加到现有值上。

# 拆箱功能

在第一阶段之后，我们可以专注于第二阶段:拆箱值。

为了实现`Decoder`协议，我应该实现`KeyedDecodingContaier`、`UnkeyedDecodingContainer`和`SingleValueDecodingContainer`，但是我只是意识到我可以安全地使用`JSONDecoder`(和`PlistDecoder`)中的那些。不幸的是，那些实现都是`fileprivate`。我真的很乐意与 swift 社区一起找到解决方案，但在此之前，这只是简单的复制粘贴。

考虑到这一点，我的第二个目标是尽可能少地更改这些实现，以便有机会与未来的版本重新保持一致。

这个问题已经被简化了:我只需要实现函数来将**的值**解装箱为期望的类型。【是`extension _JSONDecoder { … }`内标有*具体值表示*注释的块。]

`JSONDecoder`支持的类型有:

*   `Int`、`Int8`、`Int16`、`Int32`、`Int64`；
*   `UInt`、`UInt8`、`UInt16`、`UInt32`、`UInt64`；
*   `Float`、`Double`、`Decimal`；
*   `Data`、`Date`、`URL`、`String`、`Bool`。

对于前三个家庭，我只修改了`guard`的这一部分:

```
guard let number = value as? NSNumber
```

使用:

```
guard let string = value as? String, let number = string.numberValue
```

声明此`extension`:

而对于第四个家庭，我什么也没做。

# 一的数组

最后一个棘手的部分是`Array`解码。XML 没有办法表示一个项目的列表。在以下情况下:

```
<root>
  <key1>
    <key2>value1</key2>
  </key1>
  <key1>
    <key2>value2</key2>
    <key2>value3</key2>
  </key1>
</root>
```

`XMLTree`输出这个`Dictionary`:

```
["key1": [
  ["key2": "value1"],
  ["key2": ["value2", "value3"]
]]
```

逻辑上`key2`是一个`Array<String>`，但是在一个元素的情况下，解码器没有关于它的证据。所以我改变了最后一个`unbox`回退:

```
{
  self.storage.push(container: value)
  defer { self.storage.popContainer() }
  return try type.init(from: self)
}
```

带有一个`do` / `catch`和一个`extension`:

```
{
  self.storage.push(container: value)
  defer { self.storage.popContainer() }
  do {
    return try type.init(from: self)
  } catch {
    if type is _ArrayProtocol.Type {
      self.storage.push(container: [value])
      return try type.init(from: self)
    } else {
      throw error
    }
  }
}protocol _ArrayProtocol {}
extension Array : _ArrayProtocol {}
```

# 结论

我对我在 SDK 中发现的每个场景进行了单元测试，当然我 100%确定还有我没有涉及的边缘情况，但是我真的很乐意提供帮助和整合建议。专门用于`_value`和`_ArrayProtocol`解决方案。