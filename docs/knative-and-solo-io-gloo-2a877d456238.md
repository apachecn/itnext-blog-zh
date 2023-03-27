# 使用 Knative 和 Solo 实现服务自动化

> 原文：<https://itnext.io/knative-and-solo-io-gloo-2a877d456238?source=collection_archive---------4----------------------->

![](img/f7483a638e159124d59d8a2cd96baa44.png)

由[路易·里德](https://unsplash.com/@_louisreed?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

Knative 被谈论了很多，尤其是它的功能如何在 Kubernetes 的基础上帮助提供更多标准构建块来构建微服务和无服务器之类的服务，例如，扩展到零和按需扩展。Knative 高层有三个能力领域:构建、服务和事件。这篇文章将提供一些关于 Knative Build 和 Knative Serving 的例子。

Knative Serving 最初包含了所有的 Istio，但只使用了 Kubernetes 集群入口的一小部分功能。最近 Knative 团队增加了 Gloo 作为 Istio 的替代产品。更多细节可在 Solo.io 的 [Gloo、Knative 和无服务器的未来](https://medium.com/solo-io/gloo-knative-and-the-future-of-serverless-902b89f15c2c)和 [Gloo 中获得，是 Istio on Knative](https://medium.com/solo-io/gloo-by-solo-io-is-the-first-alternative-to-istio-on-knative-324753586f3a) 的第一个替代方案。

这篇文章展示了一个快速构建、服务和 Gloo 整合的例子。

所有的 Kubernetes 清单都位于下面的 GitHub repo[https://github.com/scranton/helloworld-knative](https://github.com/scranton/helloworld-knative)中。我鼓励您使用 GitHub 库来帮助自己尝试这些例子。

# 设置

这些说明假设您正在本地运行一个全新的最近的`[minikube](https://kubernetes.io/docs/setup/minikube/)`安装，并且您在本地也有可用的`kubectl`。

## 安装 Gloo

在 Mac 或 Linux 上，最快的选择是使用[自制软件](https://bash.sh/)。Gloo 文档中的完整安装说明。

```
brew install glooctl
```

然后假设你已经有了一个运行的`minikube`，并且`kubectl`是针对那个`minikube`实例设置的，也就是说`kubectl config current-context`返回`minikube`，运行下面的代码来安装 Gloo with Knative Serving。

```
glooctl install knative
```

# 部署现有的示例映像

我已经构建了这个例子，并在我的 [Docker Hub repo](https://hub.docker.com/r/scottcranton/helloworld-go) 中公开了这个图像。要使用 Knative 来提供这个现有的图像，只需要执行下面的命令。

```
kubectl apply --filename service.yaml
```

验证服务的域 URL。应该是`helloworld-go.default.example.com.`

```
kubectl get kservice helloworld-go \
  --namespace default \
  --output**=**custom-columns**=**NAME:.metadata.name,DOMAIN:.status.domain
```

并调用服务。注意:只有在对 minikube 进行本地调用时才需要`curl --connect-to`选项，因为该选项会将正确的主机和 sni 头添加到请求中，并将请求发送到从`glooctl proxy address`返回的主机和端口对。

```
curl --connect-to helloworld-go.default.example.com:80:**$(**glooctl proxy address --name clusteringress-proxy**)** [http://helloworld-go.default.example.com](http://helloworld-go.default.example.com)
```

要进行清理，请删除资源。

```
kubectl delete --filename service.yaml
```

# 本地构建，并使用 Knative Serving 进行部署

使用您的 Docker Hub 用户名运行`docker build`。

```
docker build -t **${**DOCKER_USERNAME**}**/helloworld-go .
docker push **${**DOCKER_USERNAME**}**/helloworld-go
```

部署服务。同样，确保您更新了`service.yaml` 文件中的用户名，即将图像参考`docker.io/scottcranton/helloworld-go`替换为您的 Docker Hub 用户名。

```
kubectl apply --filename service.yaml
```

验证服务的域 URL。应该是`helloworld-go.default.example.com`。

```
kubectl get kservice helloworld-go \
  --namespace default \
  --output**=**custom-columns**=**NAME:.metadata.name,DOMAIN:.status.domain
```

并测试您的服务。

```
curl --connect-to helloworld-go.default.example.com:80:**$(**glooctl proxy address --name clusteringress-proxy**)** [http://helloworld-go.default.example.com](http://helloworld-go.default.example.com)
```

要进行清理，请删除资源。

```
kubectl delete --filename service.yaml
```

# 使用主动构建进行构建，使用主动服务进行部署

要安装 Knative Build，请执行以下操作。我使用的是`kaniko`构建模板，所以你也需要安装它。

```
kubectl apply \
  --filename https://github.com/knative/build/releases/download/v0.4.0/build.yamlkubectl apply \
  --filename [https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml](https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml)
```

要验证 Knative Build 安装，请执行以下操作。

```
kubectl get pods --namespace knative-build
```

我鼓励分叉我的示例 GitHub 库[https://github.com/scranton/helloworld-knative](https://github.com/scranton/helloworld-knative)，这样你就可以推送代码变更并在你的环境中看到它们。

为你的 Docker Hub 帐户创建一个 Kubernetes 秘密，让 Knative build 推广你的形象。您还需要注释这个秘密，以表明它是为 Docker 准备的。更多详情请参见[指导凭证选择](https://www.knative.dev/docs/build/auth/#guiding-credential-selection)。

```
kubectl create secret generic basic-user-pass \
  --type**=**"kubernetes.io/basic-auth" \
  --from-literal**=**username**=${**DOCKER_USERNAME**}** \
  --from-literal**=**password**=${**DOCKER_PASSWORD**}**kubectl annotate secret basic-user-pass \
  build.knative.dev/docker-0**=**https://index.docker.io/v1/
```

它应该会产生如下所示的秘密。

```
kubectl describe secret basic-user-passName:         basic-user-pass
Namespace:    default
Labels:       <none>
Annotations:  build.knative.dev/docker-0: https://index.docker.io/v1/Type:  kubernetes.io/basic-authData
**====**
username:  12 bytes
password:  24 bytes
```

用您的 GitHub 和 Docker 用户名更新`service-build.yaml`。这个清单将使用 Knative Build 创建一个使用`kaniko-build`构建模板的映像，并使用 Knative Serving 和 Gloo 部署服务。

```
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: helloworld-go
  namespace: default
spec:
  runLatest:
    configuration:
      build:
        apiVersion: build.knative.dev/v1alpha1
        kind: Build
        metadata:
          name: kaniko-build
        spec:
          serviceAccountName: build-bot
          source:
            git:
              url: https://github.com/{ GitHub username }/helloworld-knative
              revision: master
          template:
            name: kaniko
            arguments:
              - name: IMAGE
                value: docker.io/{ Docker Hub username }/helloworld-go
          timeout: 10m
      revisionTemplate:
        spec:
          container:
            image: docker.io/{ Docker Hub username }/helloworld-go
            imagePullPolicy: Always
            env:
              - name: TARGET
                value: "Go Sample v1"
```

要进行部署，请应用清单。

```
kubectl apply \
  --filename serviceaccount.yaml \
  --filename service-build.yaml
```

然后，您可以观察构建和部署的进行。

```
kubectl get pods --watch
```

一旦你看到所有的`helloworld-go-0000x-deployment-....`吊舱都准备好了，那么你可以 Ctrl+C 来逃离手表，然后测试你的部署。

验证服务的域 URL。应该是`helloworld-go.default.example.com.`

```
kubectl get kservice helloworld-go \
  --namespace default \
  --output**=**custom-columns**=**NAME:.metadata.name,DOMAIN:.status.domain
```

并测试您的服务。

```
curl --connect-to helloworld-go.default.example.com:80:**$(**glooctl proxy address --name clusteringress-proxy**)** [http://helloworld-go.default.example.com](http://helloworld-go.default.example.com)
```

## 清除

```
kubectl delete \
  --filename serviceaccount.yaml \
  --filename service-build.yamlkubectl delete secret basic-user-pass
```

# 摘要

希望这篇文章能让你体会到 Gloo 和 Knative 如何合作，为你提供一种在 Kubernetes 中按需构建和部署服务的方法。

# 请参见

*   [https://github . com/knative/docs/blob/master/install/getting-started-knative-app . MD](https://github.com/knative/docs/blob/master/install/getting-started-knative-app.md)
*   [https://github.com/knative/docs](https://github.com/knative/docs)
*   [https://gloo . solo . io/getting _ started/kubernetes/gloo _ with _ knative/](https://gloo.solo.io/getting_started/kubernetes/gloo_with_knative/)