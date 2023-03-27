# 调试 Kubernetes PVCs

> 原文：<https://itnext.io/debugging-kubernetes-pvcs-a150f5efbe95?source=collection_archive---------1----------------------->

有时我发现容器中出现问题，存储在持久卷中的一些数据被破坏。这可能导致我不得不亲自动手，在文件系统中摸索一番。

最近我试着用一个[普罗米修斯](https://github.com/prometheus/prometheus)容器做这件事，发现容器里面没有外壳和环境(最佳实践！).这意味着我需要将 [PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) 连接到一个不同的 pod 上，我可以用它来调试环境。

# 分离该卷

由于数据损坏，容器陷入重新启动循环。所以第一步是将部署规模缩小到零。

```
$ kubectl scale deployment my-deployment --replicas=0
deployment.extensions "my-deployment" scaled
```

# 创建调试窗格

现在，我需要检查部署，找出我想要探索的 PVC。

```
$ kubectl describe deployment my-deployment | grep ClaimName
ClaimName:  my-claim
```

然后，我想创建一个新的 pod 规格，它安装相同的 PVC，但使用不同的 docker 映像。在这个例子中，我将使用 busybox，因为我只需要一个基本的 shell，但是您可以使用任何调试工具映像。

```
# my-pvc-debugger.yamlkind: Pod
apiVersion: v1
metadata:
  name: volume-debugger
spec:
  volumes:
    - name: volume-to-debug
      persistentVolumeClaim:
       claimName: <CLAIM NAME GOES HERE>
  containers:
    - name: debugger
      image: busybox
      command: ['sleep', '3600']
      volumeMounts:
        - mountPath: "/data"
          name: volume-to-debug
```

然后，我创建这个 pod，并在其中运行一个 shell。

```
$ kubectl create -f /path/to/my-pvc-debugger.yaml
pod "volume-debugger" created
$ kubectl exec -it volume-debugger sh
/ #
```

现在我在容器内部，我可以探索在`/data`装载的卷并修复问题。

# 缩小比例

一旦我对这个卷满意了，我就可以在容器中退出我的 shell 并删除 debugger pod。

```
/ # logout
$ kubectl delete -f /path/to/my-pvc-debugger.yaml
```

最后，我可以扩展我的部署。

```
$ kubectl scale deployment my-deployment --replicas=1
deployment.extensions "my-deployment" scaled
```

# 结论

在一个完美的世界里，我们永远都不应该接触到我们的卷，但是偶尔的错误会导致我们不得不去清理东西。此示例显示了一种快速跳转到没有任何用户环境的容器的卷中的方法。