# golang 1.18+仿制药:好的，坏的，丑的。

> 原文：<https://itnext.io/golang-1-18-generics-the-good-the-bad-the-ugly-5e9fa2520e76?source=collection_archive---------0----------------------->

![](img/af471866813ac0c831ac90156a33f405.png)

# 介绍

泛型很棒，golang 变得比以前更加方便。但是类似于渠道，这可能非常有用，我们不应该仅仅因为它们的存在就到处使用它们(见[阶段 4](https://opensource.com/article/17/9/seven-stages-becoming-go-programmer) )。

有**好的**用例，即使不是简单地用在数据结构中。有一些**坏的**用例，比如一般的日志记录器。有一些解决方案可以使用，但是很难看。

然后就是真的很丑。

让我们来看一个例子！

# 好人

我真正梦想在 golang 中做的，也是我认为我现在最终可以做的，是一个 CRUD 端点的通用提供者:

```
type Model interface{
    ID() string
}type DataProvider[MODEL Model] interface {
    FindByID(id string) (MODEL, error)
    List() ([]MODEL, error)
    Update(id string, model MODEL) error
    Insert(model MODEL) error
    Delete(id string) error
}
```

这是一个很大的接口，你可以根据你的具体用例来缩短它，但是，为了完整性，它就在这里。

现在，您可以定义一个使用 DataProvider 的 HTTP 处理程序:

```
type HTTPHandler[MODEL Model] struct {
    dataProvider DataProvider[MODEL]
}func (h HTTPHandler[MODEL]) FindByID(rw http.ResponseWriter, req *http.Request) {
    // validate request here       id = // extract id heremodel, err := h.dataProvider.FindByID(id)
    if err != nil { 
        // error handling here
        return
    }
    err = json.NewEncoder(rw).Encode(model)
    if err != nil { 
        // error handling here
        return
    }
}
```

如你所见，我们可以为每个方法实现一次，然后就完成了。我们甚至可以在事物的反面创建一个客户端，对于基本方法我们只需要实现一次。

为什么我们在这里使用泛型，而不仅仅是我们已经定义的模型接口？

这里，泛型比使用模型类型本身有一些优势:

1.  使用通用方法，DataProvider 根本不需要知道模型，也不需要实现它。它可以简单地提供它的具体类型，这是非常强大的(但仍然可以抽象为简单的用例)
2.  我们可以扩展这个解决方案，用具体的类型进行操作。让我们看看插入或更新的验证器是什么样子的。

```
type HTTPHandler[MODEL any] struct {
    dataProvider DataProvider[MODEL]
    InsertValidator func(new MODEL) error
    UpdateValidator func(old MODEL, new MODEL) error}
```

通用方法的真正优势在于这个验证器。我们将解组 HTTP 请求，如果定义了一个自定义的 InsertValidator，那么我们可以使用它来验证模型是否检查通过，我们可以用一种类型安全的方式来完成它，并使用具体的模型:

```
type User struct {
    FirstName string
    LastName string
}func InsertValidator(u User) error {
    if u.FirstName == "" { ... } 
    if u.LastName == "" { ... }
}
```

所以我们有一个通用的处理程序，我们可以用定制的回调函数来调整它，这样可以直接得到有效负载。没有类型转换。没有地图。只是结构本身！

# 坏事

让我们来看看我在[上一篇文章](https://medium.com/@mier85/self-referencing-interfaces-in-golang-1-18-bcd6b5701992)中写的通用日志:

```
type GenericLogger[T any] interface {
    WithField(string, string) T
    Info(string)
}
```

这本身还不是很有用。向记录器添加键值字符串对有更简单的方法，并且没有记录器(据我所知)真正实现这个接口。我们也不需要新的日志标准。如果我们想使用例如- [logrus](https://github.com/sirupsen/logrus) 我们必须这样做:

```
type GenericLogger[T any, FIELD map[string]interface{}] interface{
    WithFields(M) T
    Info(string)
}
```

如果我们添加自引用部分，这实际上可以由 logrus logger 实现。但是，让我们考虑在实际的结构中使用它，比如某种处理程序:

```
type MessageHandler[T GenericLogger[T], FIELD map[string]interface{}] struct {
    logger GenericLogger[T, FIELD]
} 
```

正如你所看到的，为了在一个结构中使用这个记录器，我们需要使我们的结构通用，这只是一个记录器。如果 MessageHandler 本身处理一般消息，那么它将变成第三个类型参数！

到目前为止，甚至没有办法用泛型将它赋给变量。因此，尽管我们可以用一个接口来表示这个日志记录器是很棒的，但我还是建议不要这样做。我最喜欢的 log lib (zap) ，由于其字段的性质，甚至不能用它来表示。

# 丑陋的

当我摆弄泛型时，我无意中发现缺少对在方法中引入新的泛型参数的支持。虽然这可能有很好的理由，但它确实需要一些变通办法。让我们想象一下，我们想把一个 map 简化为一个 int。我们理想的做法是使用一个返回新的泛型参数的方法，然后我们可以简单地提供 map reduce 函数。

那么，当我们仍然希望以一种通用的方式缩小地图时，我们该怎么做呢？因为没有方法，所以让我们横向创建一个方法:

```
type GenericMap[KEY comparable, VALUE any] map[KEY]VALUEfunc (g GenericMap[KEY, VALUE]) Values() []VALUE {
 values := make([]VALUE, len(g))
 for _, v := range g {
  values = append(values, v)
 }
 return values
}func Reduce[KEY comparable, VALUE any, RETURN any](g GenericMap[KEY, VALUE], callback func(RETURN, KEY, VALUE) RETURN) RETURN {
 var r RETURN
 for k, v := range g {
     r = callback(r, k, v)
 }
 return r
}
```

如您所见，GenericMap 成为我们的 Reduce 函数的第一个参数。在这种情况下，您可以使用任何类型的 map 作为第一个参数，而不是 GenericMap。然而，我想说的是，如果这个方法本身是泛型映射的一部分，那就太好了。即使它不是，我们仍然可以模仿它的行为。总的来说，我可能仍然会在一些用例中使用这种模式，尽管它实际上很难看。

# 真正丑陋的

有时您可能想要利用工厂模式，它为您提供类似于 DataProviders 的东西。您可能希望在动态注册的端点上获得一个提供者。所以你可以这样做:

```
type DataProviderFactory struct {
    dataProviders map[providerKey]any
}func ProviderByName[MODEL Model](factory *DataProviderFactory, name string) (DataProvider[MODEL], bool) {
        var m MODEL
    prov, has := factory.dataProviders[providerKey{name: name, typ: reflect.TypeOf(m)}]
    if !has {
       return nil, false
    }
    return prov.(DataProvider[MODEL]), true 
}func RegisterProvider[MODEL Model](factory *DataProviderFactory, name string, p DataProvider[MODEL]) {
    var m MODEL
    factory.dataProviders[providerKey{name: name, typ: reflect.TypeOf(m)}] = p 
}
```

虽然这种方法可行，而且可能有用，但它很丑陋。首先，它把丑和更丑的东西结合在一起，这就是反射。

虽然从技术上来说这应该是类型安全的，但是由于我们的名称和反射类型的组合键，它仍然是丑陋的。我会很纠结是否要把它放到生产代码附近。

# 结论

虽然我喜欢泛型，但我认为很难取得平衡，尤其是在开始的时候。所以我们需要确保记住它们为什么存在，在什么情况下我们应该使用它们，以及什么时候我们应该避免它们！