# 用 bs-json 解码 ReasonML 中的嵌套 JSON 对象

> 原文：<https://itnext.io/decoding-nested-json-objects-in-reasonml-with-bs-json-4cab75fbe308?source=collection_archive---------5----------------------->

![](img/f1bfa9f766e97d2103c4c56db5ce58d4.png)

在 JavaScript 的狂野西部，解析 JSON 数据是微不足道的。如果我们有下面的 JSON:

```
const myJson = {
  "page": 1,
  "pageCount": 12,
  "pageSize": 25,
  "items": [
    {
      "id": 12,
      "active": true,
      "title": "The Trials and Tribulations of Tristram Shandy.",
      "date": {
         "start": "2004-09-16T23:59:58.75",
         "end": "2004-09-16T23:59:58.79"
      },
      "code": "12345"
    },
    {
      "id": 10,
      "active": true,
      "title": "The Life and Letters of Leonard Lego.",
      "date": {
         "start": "2004-09-16T23:59:58.75",
         "end": "2004-09-16T23:59:58.79"
      }
    }
  ]
};
```

下面的代码将解析它:

```
JSON.parse(myJson);
```

在强类型的推理世界中完成同样的任务可能需要更多的代码。一般来说，原因是显而易见的。在这种情况下，这意味着您需要声明您的数据类型和您的解码器。

在本文中，我们将回顾如何解码嵌套的 JSON，就像上面的例子一样，使用最常见的原因之一 JSON 编码/解码库`bs-json`。

`bs-json`是一个用于 BuckleScript 的复合 JSON 编码/解码库。它提供了一组解码器功能，您可以将它们组合成更复杂的解码器。BuckleScript 将 OCaml/ReasonML 文件转换为 JavaScript。关于`bs-json`的深入细节，请访问他们的文档:([https://github.com/glennsl/bs-json](https://github.com/glennsl/bs-json))。

文档很好地描述了如何设置`bs-json`，并做了基本的 JSON 解析。他们没有详细说明的是如何解码复杂的 JSON 对象。

如果你曾经用 C#或 Java 之类的语言做过 JSON 解码，那么用 ReasonML 解码和编码 JSON 将会是一个非常熟悉的体验。然而，如果你只做过 JavaScript，那么 ReasonML 中的 JSON 解码可能会有点令人生畏。如果那是你，希望这篇文章能帮到你。

**示例应用程序**

假设我们正在处理一个列出待售书籍的应用程序。主页上显示了出售书籍的清单。我们查询一个 API，它返回一个包含所有书籍的 JSON 对象。

**解码过程**

我将假设你已经按照 https://github.com/glennsl/bs-json[网站上的步骤安装了你的应用程序(如果没有，这里还有链接:](https://github.com/glennsl/bs-json))。

解码 JSON 的第一步是编写一个 JSON 解码器。编写解码器包括三个步骤:

1.  编写类型来定义您的数据(如果您还没有)
2.  为每种类型编写一个解码器
3.  打电话给解码器

**写入类型**

让我们使用上面定义的 JSON 数据。为了方便起见，这里又是:

```
const myJson = {
  "page": 1,
  "pageCount": 12,
  "pageSize": 25,
  "items": [
    {
      "id": 12,
      "active": true,
      "title": "The Trials and Tribulations of Tristram Shandy.",
      "date": {
         "start": "2004-09-16T23:59:58.75",
         "end": "2004-09-16T23:59:58.79"
      },
      "code": "12345"
    },
    {
      "id": 10,
      "active": true,
      "title": "The Life and Letters of Leonard Lego.",
      "date": {
         "start": "2004-09-16T23:59:58.75",
         "end": "2004-09-16T23:59:58.79"
      }
    }
  ]
};
```

看数据，我们至少有三种类型。一种类型应该定义项目集。另一种类型应该定义单个项目。最后一个类型应该定义日期。在这个例子中，我们只是为每个对象定义一个类型。在您的代码中，您也可以为单个属性定义类型。

下面是上述 JSON 的定义类型。这些类型并不完美。然而，它们很简单，它们将用来说明我们需要做什么。

```
type bookDate = {
  start: string,
  end: string
}type book = {
  id: int,
  active: bool,
  title: string,
  date: bookDate,
  code: option(string),
};type bookOffers = {
  page: int,
  pageCount: int,
  pageSize: int,
  itemCount: int,
  items: array(book),
};
```

对于每种类型，我们创建一个解码器:

```
let decodeBookDate= json =>
  Json.Decode.{
    start: json |> field("start", string),
    end: json |> field("end", string)
  };let decodeBook= json =>
  Json.Decode.{
    id: json |> field("id", int),
    active: json |> field("active", bool),
    title: json |> field("title", string),
    date: json |> field("date", decodeBookDate),
    code: json |> optional(field("code", string))
  };let decodeBookOffers = json =>
  Json.Decode.{
    page: json |> field("page", int),
    pageCount: json |> field("pageCount", int),
    pageSize: json |> field("pageSize", int),
    itemCount: json |> field("itemCount", int),
    items: json |> field("items", array(decodeBook)),
  };
```

注意解码器是如何相互作用的。例如，`decodeBookOffers`解码器的`items`属性的解码器指向`decodeBook`解码器。同样，`decodeBook`解码器的`date`属性指向`decodeBookDate`解码器。

接下来，也是最后一件事，就是使用你的解码器。这部分很简单:

```
Js.Promise.(
    Fetch.fetch("http://my.api.com/bookOffers?page=1&pageSize=10")
    |> then_(Fetch.Response.json)
    |> then_(json =>
         json |> BookDecoder.decodeBookOffers
              |> (offers => send(OffersFetched(offers))) 
              |> resolve
       )
    |> catch(_err => {
         Js.log(_err);
         Js.Promise.resolve(send(OffersFailedToFetch));
       })
    |> ignore
  );
```

这就是使用`bs-json`解码嵌套对象的方法。尽情享受吧！