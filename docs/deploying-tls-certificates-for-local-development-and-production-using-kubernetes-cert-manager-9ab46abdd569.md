# 使用 Kubernetes、cert-manager、mkcert 和 Let's Encrypt 为本地开发和生产部署 TLS 证书

> 原文：<https://itnext.io/deploying-tls-certificates-for-local-development-and-production-using-kubernetes-cert-manager-9ab46abdd569?source=collection_archive---------0----------------------->

最近在我的项目中，我深入研究了如何在生产中以及开发环境中在我们面向客户端的基础设施上启用 TLS。

我是这样处理这件事的！

![](img/41f4f40b77506267f53084297f08890d.png)

# 宗旨

作为我的项目中安全通信的第一次迭代，我的主要目标只是在我们的 Kubernetes 集群的边缘(即入口级别)放置 TLS 终端，并在生产中向客户端提供 Let's Encrypt 证书。

当我有更多的时间时，我肯定会回来实现完整的端到端加密，但是，正如俗话所说:“你必须在跑之前学会走路”:)

我为第一次迭代设定了两个子目标:

*   为地方发展启用 TLS
*   为所有生产和类似生产的环境(例如，试运行和生产)启用 TLS。

为了让开发环境尽可能真实，我希望两种情况都有一个类似的解决方案，但是我真的不能花一整个月的时间来实现某些东西。

另一个要求是完全自动化/可复制性(即，能够轻松删除/重新创建所有内容)。

# 证书管理器

经过一些初步的研究，并且因为我们已经“深入”到 Kubernetes，我的选择已经决定了 [cert-manager](https://cert-manager.io/) 一个由 [Jetstack](https://twitter.com/JetstackHQ) 的优秀人员创建的开源解决方案(向他们致敬！).

cert-manager 自动管理和发放来自不同来源的 TLS 证书*。*

*一旦[安装了](https://cert-manager.io/docs/installation/)(使用 helm 或项目提供的 [yaml 文件非常容易)，cert-manager 的控制器就会监视它所支持的自定义资源定义(CRD)，例如"](https://github.com/jetstack/cert-manager/releases/download/v0.12.0/cert-manager.yaml)[颁发者](https://cert-manager.io/docs/concepts/issuer/)或"[证书](https://cert-manager.io/docs/concepts/certificate/)"资源类型。*

# *证书管理器部署步骤*

*实际上，cert-manager 的部署过程是这样的:*

1.  *为证书管理器创建一个名称空间(例如，kubectl 创建名称空间证书管理器)*
2.  *安装证书管理器*
3.  *创建一个或多个发行者(例如，每个名称空间一个，一个全局的，等等)*
4.  *创建证书资源*
5.  *让魔法流动吧*

*顺便说一下，看看 cert-manager 的[官方文档，他们在那里做了一件非常棒的工作！](https://cert-manager.io/docs/)*

*要安装 cert-manager，最简单的方法是使用 kubectl:*

```
*kubectl apply --validate=false -f [https://github.com/jetstack/cert-manager/releases/download/v0.12.0/cert-manager.yaml](https://github.com/jetstack/cert-manager/releases/download/v0.12.0/cert-manager.yaml)*
```

# *证书管理器颁发者*

*[发布者](https://cert-manager.io/docs/concepts/issuer/)是 cert-manager 中可以发布证书(因此得名)的元素，开箱即可支持多种类型，例如:*

*   *[自签名](https://cert-manager.io/docs/configuration/selfsigned/):简单的自签名发行人*
*   *[CA](https://cert-manager.io/docs/configuration/ca/) :代表证书颁发机构(CA)的颁发者，有权访问相应的私有/公共密钥来颁发新证书*
*   *ACME :最酷的类型，支持 ACME 协议，因此兼容[让我们加密](https://letsencrypt.org/)！*

# *作战计划*

*计划如下:*

*   *通过使用 [mkcert](https://github.com/FiloSottile/mkcert) 生成的自签名证书，使用 CA 发行者进行本地开发，允许我们使用可信证书(使用 mkcert 轻松添加到本地信任存储中)进行 TLS 支持的本地开发*
*   *在生产和类似生产的环境中使用 ACME 发行者，使用“让我们加密”来获取证书*
*   *将证书存储在 K8S 秘密中*
*   *使用我的 [NGINX 入口](https://github.com/kubernetes/ingress-nginx)到[TLS 配置选项](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)中的那些秘密*

# *瞬间达到顶点*

*ACME 代表“**A**automatic**C**certificate**M**M**E**n environment”是一个很酷的协议(我想？)标准化得益于 Let's Encrypt，它允许 ca 和“申请人”(即证书申请人)自动执行所有权/控制权验证和证书颁发流程。ACME 现在在 [RFC8555](https://tools.ietf.org/html/rfc8555) 中定义。*

*我不知道 ACME 协议的所有细节，但一个重要的部分是所有权的验证，这可以通过两种主要方式完成:使用 http01 或 dns01 挑战。这些挑战旨在验证证书申请中 DNS 域的所有权/控制权。*

*http01 挑战要求申请人通过在一个特定的 URL 上公开一个具有特定内容的文件来证明所有权(详情请查看 RFC ),该 URL 可通过端口 80 由 Let's Encrypt 访问。*

*另一方面，dns01 挑战要求通过添加特定的 dns 记录来证明所有权。*

*我选择 http01 是因为它在 Kubernetes 集群中实现起来更加简单，并且是独立的，而 dns01 挑战则要求我与 dns 区域进行交互，并处理 DNS 复制延迟和其他微妙之处:)*

*在实践中，当然有一些试验和错误涉及到让整个事情工作，但一切都很容易按照计划进行。*

# *HTTPS 地方发展组织*

*对于本地开发来说，这真是轻而易举，因为 mkcert 使得动态创建自签名根 CA 证书并将其添加到主机的不同信任存储区变得非常简单:*

```
***echo** "Creating self-signed CA certificates for TLS and installing them in the local trust stores"CA_CERTS_FOLDER=$(pwd)/.certs# This requires mkcert to be installed/available
**echo** ${CA_CERTS_FOLDER}
rm -rf ${CA_CERTS_FOLDER}
mkdir -p ${CA_CERTS_FOLDER}
mkdir -p ${CA_CERTS_FOLDER}/${ENVIRONMENT_DEV}# The CAROOT env variable is used by mkcert to determine where to read/write files# Reference: [https://github.com/FiloSottile/mkcert](https://github.com/FiloSottile/mkcert)
CAROOT=${CA_CERTS_FOLDER}/${ENVIRONMENT_DEV} mkcert -install*
```

*基本上，在这个步骤执行之后，一个单独的伪根 CA 证书就可用了，然后通过 mkcert -install 将它添加到本地信任存储中。就这么简单！*

*下一步是在目标名称空间中创建一个秘密，包含证书和私钥:*

```
***echo** "Creating K8S secrets with the CA private keys (will be used by the cert-manager CA Issuer)"kubectl -n some-namespace create secret tls my-ca-tls-secret --key=${CA_CERTS_FOLDER}/${ENVIRONMENT_DEV}/rootCA-key.pem --cert=${CA_CERTS_FOLDER}/${ENVIRONMENT_DEV}/rootCA.pem*
```

*如果您希望每个环境有不同的根 CA 证书，那么复制它是非常简单的*

*接下来是 CA 颁发者定义:*

```
*# Certificate Issuer (CA)apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
name: tls-ca-issuer
namespace: some-namespace
labels: ...
annotations: ...spec:
  ca:
    secretName: my-ca-tls-secret*
```

*在本例中，我选择为每个名称空间定义一个颁发者，因为在生产环境中，我希望限制每个颁发者服务的 DNS 域，以避免开发环境为生产域请求证书。*

*作为简单部署的替代方案，cert-manager 还支持 ClusterIssuer 资源类型，这几乎是相同的。*

*这个 issuer 定义可以像任何其他 Kubernetes 资源一样进行部署，我们亲爱的朋友 kubectl apply:)*

*如您所见，CA 发行者需要知道在哪里可以找到 CA 密钥，这是通过 secretName 指定的。*

*准备就绪后(假设 cert-manager 已经安装到集群中)，就可以交付证书了！*

*这里有一个例子:*

```
*# Reference: https://cert-manager.io/docs/usage/certificate/apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: my-tls-certificate
  namespace: some-namespace
labels: ...spec:
  secretName: my-app-tls-secret
  dnsNames:
    - my-app.dev.local
  issuerRef:
    name: tls-ca-issuer
    # Alternative: ClusterIssuer if there is a cluster-wide issuer available
    kind: Issuer*
```

*一旦应用到集群上，cert-manager 将检测到它，并将证书(基本上是各种 CSR)对象传递给发行者，发行者将依次处理其余的事情。“其余的”后面有相当多的细节需要了解，但我不会在这里深究那些细节。最后，如果一切顺利，那么发行者会将新生成的证书保存在 my-app-tls-secret secret 中。*

*一旦有了这些，接下来就是使用它的问题了，使用 ingress-nginx 也很容易:*

```
*apiVersion: extensions/v1beta1
kind: Ingress
namespace: some-namespace
metadata: ...
labels: ...
annotations: ...
spec:
  tls:
    # Hosts list must match those in the certificate
    - hosts:
      - my-app.dev.local
    secretName: my-app-tls-secret
  ...*
```

*如果证书可用，那么 NGINX 将使用它并执行 [TLS 终止](https://kubernetes.github.io/ingress-nginx/examples/tls-termination/)。*

*作为拼图的最后一块，我简单地将条目添加到我的 hosts 文件中，这样 my-app.local DNS 名称就可以工作了，而不必摆弄 DNSmasq 之类的东西。*

*这就是 HTTPS 当地的发展，干净而简单。*

# *生产设置*

*对于生产，设置非常相似。变化的主要是发行者，它是一个 ACME 发行者:*

```
*# Certificate Issuer (ACME)
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: tls-acme-issuer
namespace: some-namespace
labels: ...
annotations: ...
spec:
  acme:
    # Production URL: https://acme-v02.api.letsencrypt.org/directory
    server: [https://acme-staging-v02.api.letsencrypt.org/directory](https://acme-staging-v02.api.letsencrypt.org/directory)
    email: <valid mail for certificate expiry notifications, API changes, etc>
    # Where to store the Let's Encrypt account key
    privateKeySecretRef:
      name: acme-tls-issuer-account-secret-key
    # Use the HTTP01 challenge
    # Reference: [https://cert-manager.io/docs/configuration/acme/http01/](https://cert-manager.io/docs/configuration/acme/http01/)
    solvers:
    - http01:
      ingress:
        class: nginx
    # This issuer is configured to only provide certificates
    # for these specific DNS names
    # Reference: [https://cert-manager.io/docs/configuration/acme/#adding-multiple-solver-types](https://cert-manager.io/docs/configuration/acme/#adding-multiple-solver-types)
    selector:
      dnsNames:
      - my-app.dev.local*
```

*这里没有更多的东西需要了解:*

*   *服务器是 ACME 端点，在这种情况下，让我们加密的登台环境*
*   *privateKeySecretRef 是秘密的名称，cert-manager 将在其中存储它将创建的帐户的私钥，以便与 Let's Encrypt ACME endpoint 交互(值得备份！)*
*   *解决者定义如何处理 ACME 挑战；在这种情况下使用 http01“方法”*
*   *选择器允许更多的选择；在这种情况下，我将限制该发行者可以处理的 DNS 名称。我认为这是一种非常简洁方法，可以限制每个发行者的权限*

*更棘手的部分是实际生产基础设施的 ACME 挑战，目前托管在[数字海洋](https://www.digitalocean.com/)上。*

*最初，我在负载平衡器级别以及 NGINX 入口上启用了代理协议，以便在集群中获得真实的客户端 IP，但我最终禁用了它，因为它给 ACME on Digital Ocean 带来了麻烦。*

*我已经尝试了各种解决方法(包括一些负载平衡器注释)，但似乎没有一个适合我的情况。*

*因为我想前进，所以我现在接受了这个缺点，但是肯定有解决的办法。也许改天我会再写一篇关于数字海洋负载平衡器的文章..；-)*

*此外，我最终还是选择了我最初选择的 http01 ACME 解算器，因为如果没有很好地实现，替代方案(dns01)似乎更加危险。*

*在我排除故障的过程中，我很高兴地发现在 Kubernetes Slack 的#cert-manager 频道中有超级友好/乐于助人的人:【http://slack.kubernetes.io/。再次感谢他们的宝贵帮助！*

*在官方文档中还有一个关于故障排除的有用页面:[https://cert-manager.io/docs/faq/acme/](https://cert-manager.io/docs/faq/acme/)*

# *关于让我们加密速率限制*

*上一节我没有坚持那一点，但是值得知道的是，Let's Encrypt 在制作上有相当严格的速率限制。*

*这很有道理，因为他们的目标是向尽可能多的人/组织交付尽可能多的证书，但你必须知道这些速率限制，否则当被禁一周时，你可能会遇到麻烦的生产问题:)*

*详情请看以下页面:[https://letsencrypt.org/docs/rate-limits/](https://letsencrypt.org/docs/rate-limits/)*

*一般的建议是在更加友好的试运行环境中执行所有的测试。*

# *那都是乡亲们！*

*还有很多细节需要讨论，但这应该能让你对整个过程有一个很好的了解。*

*到目前为止，我对这个解决方案以及整个解决方案的简单明了非常满意。*

*Let's Encrypt 非常棒，支持全球数百万个网站，并提供 A+级证书。*

*我还非常喜欢这样一个事实，即我可以拥有一个相对真实的开发环境，这使得使用服务工作者之类的东西没有太多的麻烦。*

*我当然可以改进很多部分，比如像[这样的事情，但是我现在没有足够的时间去做。](https://github.com/smallstep/step-issuer)*

*今天到此为止！*