# 使用网真和 konfig 为 Kubernetes 开发 Go 服务

> 原文：<https://itnext.io/developing-go-services-for-kubernetes-with-telepresence-and-konfig-13018203e819?source=collection_archive---------2----------------------->

最初发表[此处](https://milad.dev/posts/telepresence-with-konfig)。

# 问题是

作为一名开发人员，当您在本地机器上开发 Kubernetes 应用程序时，如果您想测试或调试某些东西，您有以下选择:

*   使用 [docker-compose](https://docs.docker.com/compose) 运行的完整环境。
*   在本地 Kubernetes 集群中运行的完整环境( [Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube) 或 [Docker-for-Desktop](https://www.docker.com/products/docker-desktop) )
*   通过 CI/CD 管道推送插装代码、构建、测试和部署到一个 *dev* Kubernetes 集群。

前两个选项的问题是，您获得的环境无论如何都不接近您的实际最终环境(*准备*和*生产*)。而且，最后一个选项对于开发人员来说非常耗时，每次都要做一个小的更改并遍历完整的 CI/CD 管道。

![](img/5da68be6579138bde875fd0b40672f22.png)

# 什么是网真？

[网真](https://www.telepresence.io/)是由[数据线](https://www.datawire.io/)创建的 [CNCF](https://www.cncf.io/) 项目。对于开发人员来说，这是一个非常好的工具，他们可以在本地调试和测试他们的代码，而无需经历整个部署到 Kubernetes 集群的过程。

网真在一个 *Pod* 和一个在你的机器上本地运行的进程之间创建了一个双向网络代理。 *TCP 连接*、*环境变量*、*卷*等。从您的 pod 代理到本地进程。本地进程的网络也被透明地改变了，所以来自本地进程的 *DNS* 调用和 *TCP 连接*将被代理到远程 Kubernetes 集群。

在[安装](https://www.telepresence.io/reference/install)远程呈现后，您可以尝试以下操作:

```
telepresence --swap-deployment <name> --run-shell
```

这将在本地运行一个*外壳*，同时来自 pod `<name>`的所有 *TCP 连接*、*环境变量*、*卷*都可用。

# konfig 是什么？

[konfig](https://github.com/moorara/konfig) 是一个最小的非个人化库，用于读取 Go 应用中的配置值。你可以在这里阅读更多关于它和如何使用它的信息[。使用`konfig`，读取和解析配置值就像定义一个 struct 一样简单！这里有一个例子:](https://milad.dev/projects/konfig)

```
package mainimport (
  "fmt"
  "net/url"
  "time" "github.com/moorara/konfig"
)var config = struct {
  Port        int
  LogLevel    string
  Timeout     time.Duration
  DatabaseURL []url.URL
} {
  Port:     3000,   // default port
  LogLevel: "info", // default logging level
}func main() {
  konfig.Pick(&config)
  // ...
}
```

# konfig 有什么帮助？

当使用 Kubernetes [Secrets](https://kubernetes.io/docs/concepts/configuration/secret) 时，您希望将它们作为挂载的卷和文件来访问。这样，您可以为已安装的机密设置文件权限，当您更改机密时，这些权限会自动更新。

在远程呈现会话中，包括所有卷的 pod 文件系统从`TELEPRESENCE_ROOT`环境变量中指定的路径挂载。如果你想在远程呈现会话中运行你的应用程序，就像在你的 pod 中一样，你需要在你的应用程序中建立一些逻辑来考虑`TELEPRESENCE_ROOT`。

`konfig`是一种开箱即用的解决方案，可在真实 Pod 环境或远程呈现会话中透明地读取您的配置值。

# 一个例子

你可以在这里找到这个例子[的源代码。](https://github.com/moorara/konfig/tree/master/examples/4-telepresence)

让我们构建并推送 Docker 映像，然后将资源部署到 Kubernetes。

```
# current directory: examples/4-telepresence
make docker k8s-deploy
```

如果我们获得了 pod ( `kubectl logs <pod_name>`)的日志，我们应该会看到类似下面这样的内容:

```
2019/07/14 23:45:54 making service-to-service calls using this token: super-strong-secret
```

现在，我们将看看如何使用*网真*和`konfig`。首先，让我们看看网真是如何工作的。运行以下命令:

```
go build -o app
telepresence --swap-deployment example --run ./app
```

如果你运行`ls`命令，你会看到你的本地文件。如果您运行`env`命令，您会看到`AUTH_TOKEN_FILE`环境变量存在。接下来，尝试运行`echo $TELEPRESENCE_ROOT`，然后运行`cd $TELEPRESENCE_ROOT`。这是安装 pod 文件系统的地方。在远程呈现会话中，您需要在已挂载卷的路径前添加`$TELEPRESENCE_ROOT`。我们将看到`konfig`如何自动检测远程呈现会话，并像在 pod 中一样读取您的秘密。

现在，让我们对代码进行如下更改:

```
konfig.Pick(&Config, konfig.Telepresence(), konfig.Debug(5))
log.Printf("auth token: %s", Config.AuthToken)
```

无需构建新的 Docker 映像并推送它，我们只需编译我们的应用程序并使用`telepresence`命令运行它。

```
go build -o app
telepresence --swap-deployment example --run ./app
```

您将看到我们的应用程序正在本地运行，并且从远程呈现环境中成功读取了`AuthToken`。

```
2019/07/14 19:58:24 ----------------------------------------------------------------------------------------------------
2019/07/14 19:58:24 Options: Debug<5> + Telepresence
2019/07/14 19:58:24 ----------------------------------------------------------------------------------------------------
2019/07/14 19:58:24 [AuthToken] expecting flag name: auth.token
2019/07/14 19:58:24 [AuthToken] expecting environment variable name: AUTH_TOKEN
2019/07/14 19:58:24 [AuthToken] expecting file environment variable name: AUTH_TOKEN_FILE
2019/07/14 19:58:24 [AuthToken] expecting list separator: ,
2019/07/14 19:58:24 [AuthToken] value read from flag auth.token:
2019/07/14 19:58:24 [AuthToken] value read from environment variable AUTH_TOKEN:
2019/07/14 19:58:24 [AuthToken] value read from file environment variable AUTH_TOKEN_FILE: /secrets/myappsecret/auth-token
2019/07/14 19:58:24 [AuthToken] telepresence root path: /tmp/tel-deewv8wa/fs
2019/07/14 19:58:24 [AuthToken] value read from file /tmp/tel-deewv8wa/fs/secrets/myappsecret/auth-token: super-strong-secret
2019/07/14 19:58:24 [AuthToken] setting string value: super-strong-secret
2019/07/14 19:58:24 ----------------------------------------------------------------------------------------------------
2019/07/14 19:58:24 auth token: super-strong-secret
```

# 结论

[网真](https://www.telepresence.io/)是为 Kubernetes 开发应用程序的一个伟大工具。 [konfig](https://github.com/moorara/konfig) 使得读取和解析 Go 应用程序的配置值变得极其容易。它还可以透明地读取远程呈现会话中的配置值。