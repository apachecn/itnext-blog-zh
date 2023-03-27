# 使用 Terraform 向 Ingress-Nginx 添加 basic-auth

> 原文：<https://itnext.io/adding-basic-auth-to-ingress-nginx-using-terraform-c9c09f857378?source=collection_archive---------0----------------------->

我一直在做一个内部任务，是关于在我们的 [linkerd](https://linkerd.io/) 仪表板上添加一个认证层。对于那些不知道的人来说，`linkerd`在 [kubernetes](https://kubernetes.io/) 中提供了轻量级服务网格功能，以及其他有用的开箱即用功能，如用于点对点加密的 [mTLS](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/) 。

![](img/191ca4a5e5a789257da44795d45a10bd.png)

对于我们的云基础设施，我们有一个完全声明性的策略，使用 [terraform](https://www.terraform.io/) 和 [terragrunt](https://terragrunt.gruntwork.io/) 描述 [IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code) 中的所有资源。所以作为任务的一部分，我必须使用 terraform 创建所有需要的资源，并使用 terragrunt 配置它们。我按照 [ingress-nginx](https://kubernetes.github.io/ingress-nginx/) 的常规教程为[创建一个 basic-auth 模块](https://kubernetes.github.io/ingress-nginx/examples/auth/basic/)，因为我们已经使用 ingress-nginx 控制器来公开我们的服务，并使用它在我们的 linkerd 仪表板上设置 [basic-auth。这相当简单，我在 terraform 中创建了一个](https://linkerd.io/2.11/tasks/exposing-dashboard/#nginx-with-basic-auth) [random_password](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) 资源。我继续创建 akubernetes_secret 资源，使用用户名(比如 admin)和生成的密码。

从表面上看，这可以很好地在 kubernetes 和`ingress-nginx`模块中创建资源。但是当我试图加载仪表板时，我无法通过！显然，密码不起作用，我不明白为什么！

[](https://terragrunt.gruntwork.io/) [## Terragrunt | Terraform 包装

### 去掉重复的后端代码。定义如何在根目录中管理您的 Terraform 状态，并在…

terragrunt.gruntwork.io](https://terragrunt.gruntwork.io/) 

深入研究之后，我意识到在 nginx-ingress basic-auth 教程中，密码是使用`htpasswd`实用程序生成的，然后在存储到 kubernetes 时被编码为`base64`。但是为什么我生成的密码不起作用呢？看着`htpasswd`的[官方文档](https://httpd.apache.org/docs/2.4/programs/htpasswd.html)，我开始知道这个工具使用`bcrypt`加密密码，一个`MD5.`的版本，突然我意识到我的错误和[感谢 terraform](https://www.terraform.io/language/functions/bcrypt) 的 bcrypt 函数，我不得不引入一个非常小的变化。我很惊讶这样一件小事有时会导致道路堵塞。我的`kubernetes_secret`资源的修改版本如下所示:

这就是所有的魔法！这个密码非常管用！

[](https://www.terraform.io/language/functions/bcrypt) [## 哈希公司的 bcrypt - Functions -配置语言| Terraform

### 搜索 Terraform 文档 bcrypt 使用 Blowfish 密码计算给定字符串的散列，返回一个字符串…

www.terraform.io](https://www.terraform.io/language/functions/bcrypt)