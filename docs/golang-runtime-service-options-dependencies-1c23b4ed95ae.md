# golang——如何向结构中动态注入依赖关系

> 原文：<https://itnext.io/golang-runtime-service-options-dependencies-1c23b4ed95ae?source=collection_archive---------4----------------------->

![](img/457046f44c35b6255981dd9858679f63.png)

如何在 Go 中动态地给“服务”注入依赖关系？

该解决方案(方法)基于:

1.  `variadic functions`([)https://yourbasic.org/golang/variadic-function/](https://yourbasic.org/golang/variadic-function/))和
2.  `self-referential functions`([https://www.geeksforgeeks.org/self-referential-structures/](https://www.geeksforgeeks.org/self-referential-structures/))。

乍一看，这似乎有点混乱，但它实际上是一个简单的解决方案。
这种方法可以用来初始化整个应用程序，并加载配置和其他服务。

使用`variadic function`:

```
type Option func(*accs)func (f *accs) AddOptions(opts ...Option) {
    for _, opt := range opts {
        opt(f)
    }
}
```

使用`self-referential functions`:

```
func BindSubscriptions(v subscriptions.Client) Option {
    return func(f *accs) {
        f.srvSubscriptions = v
    }
}
```

首先，您应该创建主服务。在我们的片段中是`accounts`:

```
accs := accounts.New()
```

初始化第二个服务(它将是主服务的属性):

```
srvSubscriptions := subscriptions.New(111)
```

之后，您应该绑定第二个服务并为第一个服务设置属性:

```
sbs := accounts.BindSubscriptions(srvSubscriptions)
accs.AddOptions(sbs)
```

最后，我们处理业务逻辑(流程):

```
accs.Process(222)
```

同样，我们可以更改第二个服务中的任何属性(在我们的代码片段中是 accounts 或 billing):set value = 222

此外，通过使用`variadic functions`，我们可以将一个或多个属性添加到第一个服务中:

```
accs.AddOptions(sbs, bl)
```

或者通过这种方式:

```
accs.AddOptions(sbs)
accs.AddOptions(bl)
```

完整代码:

```
.
├── cmd
│   └── main.go
├── pkg
│   ├── accounts
│   │   └── service.go
│   ├── billing
│   │   └── service.go
│   └── subscriptions
│       └── service.go
└── README.md
```

所有代码:

```
package mainimport (
    "github.com/yourUserName/projectName/pkg/billing"
    "github.com/yourUserName/projectName/pkg/subscriptions"
    "github.com/yourUserName/projectName/pkg/accounts"
    "fmt"
)func main() {
    fmt.Println("Variant_1:")
    variant1() fmt.Println("Variant_2:")
    variant2() fmt.Println("Variant_3:")
    variant3()
} func variant1() {
    accs := accounts.New()
    srvAccounts := subscriptions.New(111)
    sbs := accounts.BindSubscriptions(srvAccounts)    accs.AddOptions(sbs)
    accs.Process(222)
}func variant2() {
    accs := accounts.New() srvAccounts := subscriptions.New(333)
    srvBilling := billing.New() sbs := accounts.BindSubscriptions(srvAccounts)
    bl := accounts.BindBilling(srvBilling) accs.AddOptions(sbs, bl) accs.Process(444)
}func variant3() {
    accs := accounts.New() srvAccounts := subscriptions.New(555)
    srvBilling := billing.New() sbs := accounts.BindSubscriptions(srvAccounts)
    bl := accounts.BindBilling(srvBilling) accs.AddOptions(sbs)
    accs.AddOptions(bl) accs.Process(777)
}
```

输出结果:

```
Variant_1:
        a.Prop (before): 111
        a.Prop (after): 222Variant_2:
        a.Prop (before): 333
        a.Prop (after): 444Variant_3:
        a.Prop (before): 555
        a.Prop (after): 777
```

来源回购:[https://github.com/romanitalian/go-service-options](https://github.com/romanitalian/go-service-options)

公关是受欢迎的。