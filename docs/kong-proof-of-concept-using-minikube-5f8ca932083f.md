# Kong 使用 Minikube 进行概念验证

> 原文：<https://itnext.io/kong-proof-of-concept-using-minikube-5f8ca932083f?source=collection_archive---------5----------------------->

# 如何在大约 15 分钟内完成概念验证

本指南是对我之前的一个故事的更新。

为什么是另一个向导？因为它以更简单的方式解决了同样的问题。
一般来说，简化是提取信息的艺术。这都是关于组织想法和概念，只提取有意义的部分。

本指南将带您了解一个从头开始设置 API 网关的工作示例，该网关将使用 JWT 和 ACL 来授权用户访问端点。其他部分可参考[孔优秀文档](https://docs.konghq.com/)。

> "简单是最高级的复杂。"莱昂纳多·达·芬奇(1452-1519)

# 上下文:研究不同的 API 网关

在 Sumup 上，我们希望研究不同的 API 网关，在这种情况下，我们正在构建 API 服务，需要基于角色允许或限制某些调用，使用 JWTs、速率限制等身份验证令牌。简而言之，我们需要一些微服务请求网关，通过插件处理负载平衡、日志记录、认证、速率限制、转换等等。[当然这个应该可以缩放](https://konghq.com/wp-content/uploads/2019/01/GigaOm-API-Platform-Benchmark-Kong-Apigee-1-25-19-1.pdf)。

我和@limoges 决定建立一个快速的概念验证来展示像 Kong 这样的工具的能力。这篇文章是在他的帮助下完成和审阅的。

![](img/280b22f905d8b9e25c27ca3cfeb6bc38.png)

“老办法”对孔

> 关于插件和集成的更多信息，你可以查看 [Kong Hub](https://docs.konghq.com/hub/) 。

# 为孔做一个展示

在进一步讨论 API 网关之前，我们希望创建一个概念验证，以便对问题空间和可用工具进行自我教育。

本指南是一个简短的示例，说明如何在一个示例应用程序中使用 setup Kong。在 SumUp，我们在 Kubernetes 中运行应用程序，因此我们希望在 Kubernetes 集群上进行概念验证。

这个概念验证将帮助我们探索 API 网关的所有*非功能性需求*。

严肃地说，这不是一个生产就绪的环境，只是一种快速而肮脏的方式来创建他们可以用来开发的开发人员沙箱。

# 1.使用 Minikube 创建我们的“集群”

我们将使用 minikube-version 1.15.6，因为 1.16 上的一些 api 被弃用

```
**> minikube start --kubernetes-version v1.15.6**
😄  minikube v1.5.2 on Ubuntu 19.10
✨  Automatically selected the 'virtualbox' driver (alternates: [none])
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.15.6 on Docker '18.09.9' ...
💾  Downloading kubeadm v1.15.6
💾  Downloading kubelet v1.15.6
🚜  Pulling images ...
🚀  Launching Kubernetes ... 
⌛  Waiting for: apiserver
🏄  Done! kubectl is now configured to use "minikube"
```

# 2.在我们的集群上设置 Helm v2

我们希望使用 Kong helm 图表来简化这个过程，所以我们需要在我们的集群上安装 Tiller (Helm 的服务器端组件)。

```
**> curl -L** [**https://git.io/get_helm.sh**](https://git.io/get_helm.sh) **| bash # Install helm****> helm init # Setup Tiller**
Helm v2.16.1 
Run 'helm init' to configure helm.
$HELM_HOME has been configured at /home/pablo/.helm.Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
```

# 3.设置孔

第一步是简单地用[舵图](https://hub.helm.sh/charts/stable/kong)安装[孔](https://konghq.com/)。因为我们是好公民，所以我们将把与 Kong 相关的所有内容放在一个名为 *kong* 的名称空间中。

```
**> kubectl create ns kong****> helm install --version 0.26.1 \
--name kong stable/kong \
--namespace kong \
--set ingressController.enabled=true \
--set image.tag=1.4 \
--set admin.useTLS=false**
```

孔掌舵图安装会给你一个获取*代理*地址和*管理*地址的命令。正如您可能已经知道的，代理地址将被用作您的服务的入口点，而 *Admin* 让您可以访问 Kong HTTP API。

```
**> kubectl get services --namespace kong**
NAME                        TYPE        ...
kong-kong-admin             NodePort    ...
kong-kong-proxy             NodePort    ...
kong-postgresql             ClusterIP   ...
kong-postgresql-headless    ClusterIP   ...
```

因为我们会多次引用*管理*和*代理*地址，所以让我们导出它们。

```
**> export PROXY_ADDR=$(minikube service -n kong kong-kong-proxy --url | head -1)****> export ADMIN_ADDR=$(minikube service -n kong kong-kong-admin --url | head -1)****> echo $PROXY_ADDR # This will differ for you.**
[http://192.168.64.7:30076](http://192.168.64.7:30076)**> curl -i $PROXY_ADDR**
HTTP/1.1 404 Not Found
Date: Mon, 25 Nov 2019 16:50:59 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
X-Kong-Response-Latency: 2
Server: kong/1.4.0
{“message”:”no Route matched with those values”}%
```

因为我们没有在集群上运行 http 服务，所以我们得到一个 404。

# 4.设置 KONGA(可选)

Konga 是一个优秀的图形化管理界面，用于管理 Kong，我们可以用下面的清单简单地设置它。

```
# We can create a file containing this manifest 
# or pipe it to `kubectl`.
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: konga
  namespace: kong
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: konga
        app: konga
    spec:
      containers:
      - name: konga
        image: pantsel/konga
        ports:
        - containerPort: 1337
        env:
          - name: NO_AUTH
            value: "true"
---
apiVersion: v1
kind: Service
metadata:
  name: konga-svc
  namespace: kong
spec:
  type: NodePort
  ports:
  - name: kong-proxy
    port: 1337
    targetPort: 1337
    nodePort: 30338
    protocol: TCP
  selector:
    app: konga
```

因为我们将通过浏览器查看 Konga，所以让我们获取那个地址。

```
export KONGA_ADDR=$(minikube service -n kong konga-svc — url | head -1)
```

在浏览器中打开`$KONG_ADDR`，我们将不得不设置与*孔管理 HTTP API* 的连接。

我们可以给这个连接起一个名字，并将 URL 设置为`$ADMIN_ADDR`。

![](img/a889c57e2e2ad174fa3177c2d0c05261.png)

加载 Konga 仪表板

# 5.安装 HTTP 应用程序

因为我们最感兴趣的是 HTTP 请求，所以我们可以使用简单的 echo-server 应用程序，它只是将 HTTP 请求打印回给我们，并附带一些关于 pods 之类的其他细节。

我们将应用程序放在 Kong 后面，并使用 Kong 的入口类。

```
**> curl -sL bit.ly/echo-server | kubectl apply -f -**
service/echo created
deployment.apps/echo created
```

## 使用 Kong 代理(需要检查下级)

创建一个入口规则来代理之前创建的 echo 服务器:

```
**> echo "
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo
  annotations:
    # This annotation is optional since the ingress class will
    # default to "kong" behind the scene.
    # You can validate the right ingress is being used by the
    # `X-Kong-Upstream-Latency` header being added to requests.** [**kubernetes.io/ingress.class**](http://kubernetes.io/ingress.class)**: "kong"
spec:
  rules:
  - http:
      paths:
      - path: /foo
        backend:
          serviceName: echo
          servicePort: 80
" | kubectl apply -f -**
ingress.extensions/demo created
```

验证入口正在工作:
*(注意孔入口发回的* `*X-Kong-**` *响应头。)*

```
> **curl -i $PROXY_IP/foo** 
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Date: Mon, 25 Nov 2019 17:19:13 GMT
Server: echoserver
*X-Kong-Upstream-Latency: 4
X-Kong-Proxy-Latency: 4*
Via: kong/1.4.0Hostname: echo-599d77c5c7-m9zbhPod Information:
 node name: minikube
 pod name: echo-599d77c5c7-m9zbh
 pod namespace: default
 pod IP: 172.17.0.15Server values:
 server_version=nginx: 1.12.2 - lua: 10010Request Information:
 client_address=172.17.0.14
 method=GET
 real path=/
 query=
 request_version=1.1
 request_scheme=http
 request_uri=[http://192.168.99.110:8080/](http://192.168.99.110:8080/)Request Headers:
 accept=*/*  
 connection=keep-alive  
 host=192.168.99.110:30400  
 user-agent=curl/7.65.3  
 x-forwarded-for=172.17.0.1  
 x-forwarded-host=192.168.99.110  
 x-forwarded-port=8000  
 x-forwarded-proto=http  
 x-real-ip=172.17.0.1Request Body:
 -no body in request-
```

# 5.设置授权

> [*ACL*](https://docs.konghq.com/hub/kong-inc/acl/) *(访问控制列表)
> 通过将使用任意 ACL 组名的消费者列入白名单或黑名单来限制对服务或路由的访问。该插件要求已经在服务或路由上启用了* [*认证插件*](https://docs.konghq.com/about/faq/#how-can-i-add-authentication-to-a-microservice-api) *。*
> 
> *J*[*WT*](https://docs.konghq.com/hub/kong-inc/jwt/)*(JSON Web 令牌)
> 验证包含 HS256 或 RS256 签名的 JSON Web 令牌的请求(如*[*RFC 7519*](https://tools.ietf.org/html/rfc7519)*中所指定)。您的每个消费者都将拥有 JWT 凭证(公钥和私钥)，必须使用这些凭证来签署他们的 jwt。*

为了做到这一点，我们必须采取 5 个步骤:

1.  创建一个名为 **hello 的消费者。**
2.  将**你好**添加到一个群中，我们将其命名为**允许**。
3.  给消费者增加一个 JWT。
4.  将 JWT 插件添加到路线中。
5.  将 ACL 插件添加到路由中。

因此，让我们从创建一个消费者开始:

![](img/dbbb2c47104ef868456b5530e61fd0d8.png)

使用 Konga 创建新消费者

然后，我们将我们的使用与我们创建的组相关联:

![](img/58022b5d7d20507fbf8b34750342cbcc.png)

在消费者面板下可以找到组

然后，我们为消费者创建一个 JWT；缺省值是可取的，否则您将玩弄密钥和声明:

![](img/9c3bdddee17d08abbe8ada7e0c70d88c.png)

什么都不做，只需按提交

太好了。消费者设置完成。现在让我们来看看路线。

我们添加 ACL 插件，并将之前创建的组列入白名单:

![](img/fef83ebdb5b2e7e0479035fc4bdcb51d.png)

然后我们添加 JWT 插件:
(这是唯一的关键步骤。您绝对需要**授权**头名称，否则在 HTTP 头中设置令牌将不起作用。)

![](img/3e57c2ed8d2d32515de3e06c3e4545be.png)

确保在标题名称中添加**授权**

就是这样。我们已经设置了使用 ACL+JWT 的路由。现在，我们要做的就是使用它。让我们测试一下端点:

```
**> curl -i $PROXY_ADDR/foo**
HTTP/1.1 401 Unauthorized
Date: Tue, 26 Nov 2019 14:11:27 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 26
X-Kong-Response-Latency: 8
Server: kong/1.4.0{"message":"Unauthorized"}
```

太好了。所以现在我们需要得到令牌。
前往 [jwt.io](https://jwt.io/) 并使用神奇工具:

1.在有效负载中:将`iss`设置为您的消费者的 JWT 凭证中的`key`的值。

2.在验证签名中:将`secret`设置为`secret`的值，这也可以在消费者的 JWT 凭证中找到。

![](img/7f302ea619de8e23f4657f62fc904326.png)![](img/6271dfbfe69c9befaf63f5eb2a3b8928.png)

现在，我们来试试这个:

```
› curl -i -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJWVGlIeWNGUGszbzdJQkhTZVhlQ2xwYXpsWGxnTWNqOCJ9.XODg0ay6fioNZZqkPiVW_u0N_4UpZgHqX3cVevAzSXo' $PROXY_ADDR/foo
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Date: Tue, 26 Nov 2019 10:22:19 GMT
Server: echoserver
X-Kong-Upstream-Latency: 4
X-Kong-Proxy-Latency: 12
Via: kong/1.4.0Hostname: echo-599d77c5c7-ktlr7Pod Information:
        node name:      minikube
        pod name:       echo-599d77c5c7-ktlr7
        pod namespace:  default
        pod IP: 172.17.0.15Server values:
        server_version=nginx: 1.12.2 - lua: 10010Request Information:
        client_address=172.17.0.12
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=[http://192.168.64.7:8080/](http://192.168.64.7:8080/)Request Headers:
        accept=*/*
        authorization=Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJWVGlIeWNGUGszbzdJQkhTZVhlQ2xwYXpsWGxnTWNqOCJ9.XODg0ay6fioNZZqkPiVW_u0N_4UpZgHqX3cVevAzSXo
        connection=keep-alive
        host=192.168.64.7:30585
        user-agent=curl/7.54.0
        x-consumer-groups=baddies-allowed
        x-consumer-id=5ed91a70-12fe-450f-b0a1-3dbbbe9a7997
        x-consumer-username=baddies
        x-forwarded-for=172.17.0.1
        x-forwarded-host=192.168.64.7
        x-forwarded-port=8000
        x-forwarded-proto=http
        x-real-ip=172.17.0.1Request Body:
        -no body in request-
```

大获成功！

![](img/d60ee56a63ef9c7687af3d3f4f5b9935.png)

所以，就这样了。我希望你觉得它有用，并且步骤容易遵循。如果我遗漏了什么，请评论和/或提问——很高兴得到反馈。