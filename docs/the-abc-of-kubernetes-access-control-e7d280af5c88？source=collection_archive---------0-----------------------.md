# Kubernetes 访问控制基础知识

> 原文：<https://itnext.io/the-abc-of-kubernetes-access-control-e7d280af5c88?source=collection_archive---------0----------------------->

![](img/1877e0408e8ae02e9e36d5976b5505f4.png)

ABC 就像是在**A**l ways**B**e**C**控制——是的，我知道，我从 1992 年的电影《格伦加里·罗斯》(Glengarry Glen Ross)中的角色布莱克那里拿走了 [ABC 这个名字——我的意思是你不仅应该让](https://www.youtube.com/watch?v=Yz246_Pjjkc) [RBAC 启用](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)，显然，你还应该为你的应用程序创建并使用专用的[服务账户](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)。至少对于 Kubernetes 的本地应用来说是这样，但是如果你养成了这样的习惯，这并没有什么坏处:

```
$ kubectl create ns myapp
$ kubectl -n myapp create sa thesa
```

所以现在你已经在名称空间`myapp`中准备了一个专用的服务帐户`thesa`,为了你自己，我希望你总是使用专用的名称空间，对吗？

现在，如果您启动您的应用程序，[使用专用服务帐户](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) `thesa`，例如，像这样:

```
$ kubectl -n myapp run theapp \
          --image=quay.io/whatever/theapp:0.42 \
          --serviceaccount=thesa
```

## 为什么？

因为现在你可以在任何时间点让应用程序访问你喜欢的任何资源，或者，也可以撤销访问。想要在命名空间中赋予应用程序超能力吗？只需这样做:

```
$ kubectl -n myapp create rolebinding givemyappssuperpower \
          --clusterrole=cluster-admin \
          --serviceaccount=myapp:thesa
```

想让你的应用在整个集群中拥有超能力吗(说真的，不要)？把上面的`create rolebinding`换成`create clusterrolebinding`就行了。想要[限制您的应用程序对名称空间中的资源进行只读访问](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles)？执行以下操作:

```
$ kubectl -n myapp delete rolebinding givemyappsuperpower$ kubectl -n myapp create rolebinding givemyappreadonlyaccess \
          --clusterrole=view \
          --serviceaccount=myapp:thesa
```

请注意，以上所有操作只能以干净的方式进行，因为您已经提前创建了服务帐户`thesa`，并使用该服务帐户启动了应用程序。

那么，有什么启示呢？不管你*是否需要*立即给予一个应用程序一定的权限，*总是*创建一个专用的服务帐户并使用它启动应用程序，最好是在一个专用的名称空间中。换句话说:在现实世界的设置中不要使用默认的名称空间和/或默认的服务帐户，但是*总是控制*；)