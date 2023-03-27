# 通过专用网络启用 AWS 弹性容器注册表自定义域和匿名访问

> 原文：<https://itnext.io/enable-aws-elastic-container-registry-custom-domain-and-anonymous-access-over-private-network-123d07663709?source=collection_archive---------3----------------------->

![](img/d1b9ddebd1eceab1b8278610ab05e9ce.png)

詹姆斯·萨顿在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

> [点击这里在 LinkedIn 上分享这篇文章](http://https%3A%2F%2Fitnext.io%2Fenable-aws-elastic-container-registry-custom-domain-and-anonymous-access-over-private-network-123d07663709%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

几个月前，我们启动了 AWS 弹性容器注册中心(ECR ),作为供应商解决方案的替代方案，这让我们有些难过。

从运营的角度来看，这非常好，因为一旦您通过生命周期策略将 ECR 设置为过期并删除历史映像，它就变得几乎无需维护。然而，作为一名开发人员，我们发现这种体验有点违反直觉:

*   对于不熟悉 AWS、跨账户访问和 Docker 本身的人来说，登录 Docker 是很难理解的。此外， [ECR 访问凭证每天都会过期](https://medium.com/@xynova/keeping-aws-registry-pull-credentials-fresh-in-kubernetes-2d123f581ca6)使得初始设置变得更加复杂。
*   注册表 URL 不是很友好(**[account id]. dkr . ECR . AP-southeast-2 . Amazon AWS . com**)，而且还没有办法定制它。
*   ECR 目前不支持为专用网络 ip 范围配置匿名访问的策略。

很明显，我们必须做些什么来让 ECR 更容易被普通开发者使用。上周，我们终于建立了一个 Kubernetes Nginx 服务，该服务为 ECR 提供了一个自定义域名，并允许未经认证的 docker 从 VPCs 和 OpenVPN 中提取内容。

该服务的组成部分如下:

*   带有标记化 Nginx 配置文件的 ConfigMap，用于按计划使用新 ECR 凭据和域名进行处理。
*   代理 ECR 注册表的 Nginx 服务。
*   一个 CronJob，按计划使用新的 pull secrets 处理 ConfigMap，并退回反向代理服务。

## 配置图

*   Nginx 配置模板( **aws-registry-proxy-tpl** )极其简单。它代理 ECR 注册表，强制主机头并为请求设置 Docker 基本身份验证凭证。(点击此处阅读更多内容[https://docs.docker.com/registry/recipes/nginx/](https://docs.docker.com/registry/recipes/nginx/))

## Nginx 服务

代理服务也没什么特别的。它只是一个普通的 Nginx Pod(我们碰巧使用 Nginx Plus)，通过一个 ConfigMap 卷加载之前处理过的模板(**AWS-registry-proxy-config**)。Kubernetes Pod 的前端是一个服务和一个 TLS 入口路由。

请注意，该规范包含一个 imagePullSecrets 指令，指向一个**AWS-registry**Kubernetes secret，用于针对 ECR 进行认证。这个秘密被一个 CronJob 刷新，我们很快就会看到。

## 克朗乔布

最后，我们到达了 Kubernetes CronJob，它将所有的碎片粘合在一起:

*   它调用 **aws ecr get-login** 命令来获取一个新的 ecr 令牌。
*   它将 ECR 令牌保存为 **aws-registry** pull secret，这样我们就可以提取我们的 Nginx-Plus 容器(不需要它，因为您使用的是公共映像)。
*   它呈现了**AWS-registry-proxy-TPL**config map 模板，并将其保存为 Nginx 容器使用的**AWS-registry-proxy-config**config map。
*   它为 Nginx Pod 部署创建了一个小的更新，以便 Kubernetes 对它执行滚动更新。

我一直使用 Kubernetes 作业来触发模板的第一次执行，因为目前似乎没有办法按需调用 Kubernetes CronJob 运行。

## 临时演员

下面是用于运行 CronJob 的 **xynova/aws-kubectl** 容器 Dockerfile 文件。它基本上将 aws-cli、kubectl 和 [gomplate](https://gomplate.hairyhenderson.ca/) 二进制文件组合在一个容器中。

对于使用 aws-cli 请求 docker 登录命令的容器，它必须在 Kubernetes 节点上运行，并具有以下角色附加策略。

## 包扎

正如您所看到的，只需要一点点路由技巧就可以绕过 AWS ECRs 的一些限制。对我们来说，这是一个巨大的胜利，因为现在开发人员可以连接到 OpenVPN，直接从**registry.shrd.ourdomain.net**获取图像，而无需使用任何黑魔法来验证。

干杯