# 使用通道的 Go 中的迭代器模式

> 原文：<https://itnext.io/iterator-pattern-in-go-using-channels-73a36746c7ac?source=collection_archive---------7----------------------->

![](img/5392698df038d28de2730fec9cd0736a.png)

[Edvinas daugordas](https://unsplash.com/@edvinasd?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/staircase?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

通道和 Goroutines 是 Go 中并发模型的两个特性之一。它们在语言中如此普遍，以至于运行库在语法中为它们提供了内在支持。一个这样的语法支持是在`for range`循环中。您可以使用`for range`循环对*打开的*通道中的值进行范围调整(当通道*关闭*时，范围循环终止)

```
ch := make(chan int)
for i := ch {
    fmt.Println(i)
}
```

这种语法支持允许我们在 go 中创建迭代器模式，而不需要明确使用`next()`或`isDone()`方法。

## 问题

我们最近有一个用例，在这个用例中，我们必须定期从服务中获取分页数据，并将聚合数据上传到 AWS S3

## 解决办法

为了解决这个问题，我们使用 go 通道并发地向远程服务发出迭代 RPC 请求，并聚合结果。该解决方案包含以下组件:

1.  定期运行`fetch-and-upload`作业的 [K8S Cronjob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) 。
2.  这个问题的获取部分是对远程 REST 端点的 RPC 调用。
3.  请求分页数据的迭代器(*我们将只关注解决方案的这一部分*
4.  上传部分使用 go 中的`aws-sdk`上传聚合文件

## 使用通道的迭代器模式

创建迭代器的先决条件是能够清楚地区分迭代器何时完成。在大多数迭代器的实现中，这是使用类似`isDone`(双关语)的方法来完成的。在我的例子中，远程服务允许基于光标的响应，当数据结束时，字段`next`为空。这非常适合迭代器模式，在迭代器模式中，我们需要某种东西来识别分页数据的结尾。

因此，获取分页用户信息的`iterator`使用通道编写为:

```
import (
  "log"
  "github.com/dummy-package/service"
)func UserIterator() <-chan User {
  ch := make(ch User)
  offset := 0
  limit := 1000 go func() {
    for {
      userResponse, err := service.GetUsers(offset)
      if err != nil {
          log.Println("Cannot fetch data for offset", offset)
      }

      offset += limit
      ch <- userResponse.Items if userResponse.Next == nil {
        log.Println("Finished fetching all users")
        close(ch)
        return
      }
    }
  }()
  return ch
}
```

在这种情况下,`service.GetUsers(offset)`的签名如下

```
type NextCursor struct {
  Href string
}type UserResponse struct {
   Items []User
   Next NextCursor
}func GetUsers(offset int) UserResponse {
  ...
}
```

有了上面的迭代器，我们可以声明性地遍历通道，并将所有返回的用户响应组合到一个`slice`中

```
package mainfunc main() {
  var allUsers []User
  for users := range UserIterator() {
    allUsers = append(allUsers, users...)
  } UploadToS3(allUsers)
}
```

这种模式允许我们直观地推理远程服务的分页行为并聚合响应，而不必处理 HTTP 响应中有太多信息的问题。