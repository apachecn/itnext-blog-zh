# 让我们用 Terraform 加密证书

> 原文：<https://itnext.io/lets-encrypt-certs-with-terraform-f870def3ce6d?source=collection_archive---------0----------------------->

SSL 证书的免费、安全和自动化存储

![](img/fc75d1e7967414fdc6f3d5d6002d01e5.png)

克里斯·巴巴利斯在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

我不知道你对使用[让我们加密](https://letsencrypt.org/)的印象如何，但我的印象是，与市场上其他昂贵的替代品相比，它们是为我们的网站获得**生产就绪免费 SSL 证书**的绝佳资源。

虽然⚠️ ( [参考](https://letsencrypt.org/docs/rate-limits/))也有一些收获:

*   证书生成依赖于我们管理下的有效 DNS 区域。这是必要的，因为证书生成过程将需要放入 DNS 记录，以验证我们对要颁发的域的权限。
*   一个证书最多可以包含 100 个(子)域。
*   证书(重新)生成限制为每周 5 次。我艰难地认识到，超过这个限制将不会允许你在 7 天内生成新版本的证书😅

这意味着我们必须非常小心地为我们的应用程序生成证书。我建议采取以下行动来避免我所经历的头痛:

*   出于测试目的(即，每当我们不完全确定将要颁发的证书时)，我们可以使用 let ' s encrypt[staging environment](https://letsencrypt.org/docs/staging-environment/)。尽管这些证书在生产应用程序中不起作用(用户将在其浏览器中看到一个*红色锁*),但它们是验证这些证书生成的一个很好的替代方法，其限制比生产证书高得多。
*   自动生成和存储这些证书，以便在过期前定期重新生成它们，并从负载平衡器或 web 服务器等其他组件使用它们。

实现这些目标的最简单的开源方法之一是使用 **Terraform** 来自动生成证书(重新生成)。

在我们下面的简短示例中，我们将通过 Terraform 使用 **Route53** 来验证 DNS 质询，以自动放置 Let's encrypt 所需的验证记录。

作为第一步，我们将指定要使用的 Terraform 版本& ACME 提供者:

正如你可能看到的，这个地形定义是非常基本的。那是有目的的🙂因为我只关注创建，所以让我们加密资源。您可以随意为您的代码的其余部分添加任何额外的配置(比如远程后端，这总是一个好主意！).

正如 Terraform Registry 中所记录的，使用 [ACME provider](https://registry.terraform.io/providers/vancluever/acme/latest) 发布 let's encrypt 证书非常容易:

正如你从上面的定义中看到的，我们在这里:

*   使用“让我们加密试运行环境”,并保留生产环境以备将来使用。
*   使用一个**现有的** Route53 DNS 区域，以便放置我们新 SSL 证书的 DNS 挑战记录。
*   使用加密 Terraform 资源创建 SSL 证书。

如果你喜欢冒险，可以在你的管理下更新 *base_domain* 数据源的 DNS 区域。如果 Route53 中还没有 DNS 区域，请随意将此数据源转换为 Terraform 资源，创建一个新的 DNS 区域并将其用于加密验证。

在这种情况下，我们不会使用电子邮件作为确认质询。但是，在注册资源中输入真实的电子邮件地址是一个好主意，这样可以将证书与您在 let's encrypt records 上的身份相关联。

与提供者[文档](https://registry.terraform.io/providers/vancluever/acme/latest/docs/resources/certificate#using-dns-challenges)相比，本例使用 Terraform 基于 *AWS_ACCESS_KEY_ID* 、 *AWS_SECRET_ACCESS_KEY* 和 *AWS_DEFAULT_REGION* 变量继承的默认 AWS 凭证。但是，请记住，可以在 *config* 块中显式设置 AWS 凭证，甚至可以使用其他 DNS 提供商，如 Azure、GCP，甚至自定义 HTTP/TLS 挑战。

下一步是将这个证书存储在一个安全的地方，根据我们的需要在其他地方使用。在这种情况下，我们将介绍三种实现方式:使用 **S3 对象**(可以使用外部服务/应用程序下载)、使用 **Terraform 输出**(可以手动复制以备将来使用)，以及使用**亚马逊证书管理器(ACM)** (稍后可以在其他 AWS 资源中使用，如负载平衡器、API 网关等)。

⚠️坚持住！:SSL 证书密钥被视为域有效性的敏感材料。使用以下任一替代方法存储和暴露它们时要小心:

## 第一种选择:将证书存储为 S3 对象

我们可以扩展 Terraform 代码，将我们的证书保存为 S3 对象，指定存储桶名称和保存每个证书密钥的路径:

如上面的代码所示，我们指定了一个现有的 S3 桶的名称**。如果没有资源，考虑使用 *aws_s3_bucket* 资源创建一个新的 bucket。**

## 第二种选择:将证书显示为 Terraform 输出

如果我们想使用一次证书，而他们的密钥可以手动获取，那么使用 Terraform 输出可能是一个不错的选择。对于这种情况，我们必须创建三个输出对象，如下所示:

请注意，私钥对象是一个敏感的元素。这就是为什么我们使用[非敏感](https://www.terraform.io/docs/language/functions/nonsensitive.html)函数在控制台中显示其内容，⚠️小心在公共场所暴露敏感内容，如证书私钥，如 GitHub repos 或公共 CI 管道。

## 第三个选项:将证书存储在 AWS 证书管理器(ACM)中

也许新生成的证书的用例将用于其他 AWS 资源，如负载平衡器、CloudFront 或 API 网关。如果这是您的情况，我们可以使用如下方式将 Let's encrypt 证书作为新的 ACM 证书导入:

这将在 AWS 中被视为一个新的 ACM 证书(不是由 Amazon 发布的):

![](img/eb2130eefd60bf1172266fd341d879f1.png)

就是这样！从现在开始，你可以在你的负载平衡器中使用你的全新证书，如 ELB、Nginx 或 HAProxy🔐

我真的很感激任何类型的反馈，疑问或评论。请随意在此给我[留言，或者在这个帖子里发表任何评论。](https://www.linkedin.com/in/nicosingh)

[](https://github.com/sponsors/nicosingh) [## GitHub 赞助商上的赞助商@nicosingh

### 非常感谢你的支持😊我感谢你对做开源软件和分享知识的认可…

github.com](https://github.com/sponsors/nicosingh)