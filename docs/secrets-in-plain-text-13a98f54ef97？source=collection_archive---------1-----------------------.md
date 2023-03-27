# 纯文本的 Kubernetes 秘密

> 原文：<https://itnext.io/secrets-in-plain-text-13a98f54ef97?source=collection_archive---------1----------------------->

![](img/741073f72c13fe8f3a51684ec460be98.png)

我写了一个简单的方法来解码 Kubernetes 的秘密，叫做“Kubernetes Secret Decode ”,它位于 github 上的[。当尝试调试应用程序时，准确地找出哪里出了问题，以及为什么事情没有按预期运行是非常耗时的。我想快速排除的一个难题是，找出我在 pod 中引用的秘密值是否是我期望的值。使用`kubectl`实现这一点的一种方法是编写一个 go 模板，并通过像这样选择每个值来对其进行 base64 解码`kubectl get secrets my-secret -o 'go-template={{index .data "username"}} | base64 -D-`。另一种选择是用`-o yaml`输出整个秘密，然后获取每个值，再一次单独地进行 base64 解码。但是对我来说，这两种选择都比我所希望的稍微耗费时间和负担。](https://github.com/ashleyschuett/kubernetes-secret-decode)

为了解决这个问题，我写了一个小工具，你可以通过管道传输一个秘密，它会以纯文本的形式显示整个秘密和数据。你不必学习任何新的命令语法或者离开`kubectl`接口。你可以简单地运行`kubectl get secret my-secret -o yaml | ksd`。

现在使用上面的命令，你会看到这个

```
$ kubectl get secret my-secret -o yaml | ksd
apiVersion: v1
data:
  password: password
  username: username
kind: Secret
metadata:
  creationTimestamp: “2018–05–09T21:01:37Z”
  name: my-secret
  namespace: default
  resourceVersion: “20229”
  selfLink: /api/v1/namespaces/default/secrets/my-secret
  uid: 29ef8024–53cc-11e8–967d-080027cd91ae
type: Opaque
```

在查看完我所期望的数据后，我可以继续检查其他可能损坏的数据。希望这也能让别人的一天过得更好。在此随意制作 PRs 或问题[。](https://github.com/ashleyschuett/kubernetes-secret-decode)