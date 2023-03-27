# aws-vault 验证您的 aws 帐户

> 原文：<https://itnext.io/aws-vault-to-authenticate-your-aws-accounts-696d08446106?source=collection_archive---------0----------------------->

![](img/80772c647a5cfa412685ac3045154acc.png)

图片来源:spot.io

在我当前的项目中，我有机会和团队一起使用`aws-vault`。我以前只是依靠我配置的`aws` cli 工具登录一个帐户，但我意识到，如果你想从 cli 访问或执行多个不同 aws 子帐户的命令，这可能是一个相当棘手的问题。aws-vault 解决了这个问题；它允许您配置通过 cli 访问的多个 aws 子帐户。最好的方法是只需配置一次，就可以轻松地与任何其他可能需要访问 aws 服务的 cli 工具集成。

[](https://github.com/99designs/aws-vault) [## GitHub - 99designs/aws-vault:一个用于安全地存储和访问 aws 凭证的保险库…

### AWS Vault 是一个在开发环境中安全存储和访问 AWS 凭证的工具。AWS 保管库存储 IAM…

github.com](https://github.com/99designs/aws-vault) 

我们有三个不同的 aws 子账户，`dev`、`staging`和`prod`，在我们的 aws 账户上展示我们的变化。我访问这些账户时，最常用的是使用 terraform 进行基础设施开发。每次我想在使用`aws-vault`之前切换到另一个子账户时，我都必须配置`aws/config`文件。设置它也只需要 5 分钟！

配置`aws-vault`的先决条件是已经安装了`aws` cli，然后通过`brew install --cask aws-vault`为 mac 用户安装`aws-vault`。在理想的 aws 设置中，用户将被创建在 root 帐户中，并有一个与之相关联的`mfa`。在每个 aws 子帐户中，一个`admin`或高权限角色将允许用户访问不同的服务。您需要为用户创建访问密钥和密码，以便能够通过 cli 访问该帐户。创建之后，您需要更新您的`aws/config`文件来配置所有的账户。

 [## 安装或更新最新版本的 AWS CLI

### 本主题介绍如何在…上安装或更新最新版本的 AWS 命令行界面(AWS CLI)

docs.aws.amazon.com](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) 

具有多个子账户的典型设置如下所示:

```
[default]
region=eu-central-1
output=yaml# the role_arn is the admin or similar role
# that you need to create in every sub-account[profile dev]
mfa_serial=arn:aws:iam::<root-account-id>:mfa/ali
role_arn=arn:aws:iam::<dev-account-id>:role/admin[profile staging]
mfa_serial=arn:aws:iam::<root-account-id>:mfa/ali
role_arn=arn:aws:iam::<staging-account-id>:role/admin[profile prod]
mfa_serial=arn:aws:iam::<root-account-id>:mfa/ali
role_arn=arn:aws:iam::<prod-account-id>:role/admin
```

一旦你建立了配置文件，你就可以开始为`dev`、`staging`和`prod`添加账号了。那就像执行`aws-vault add dev`一样简单。cli 会提示您输入已经为您的用户创建的`ACCESS_KEY_ID`和`SECRET_ACCESS_KEY`。您可以通过执行`aws-vault exec dev -- aws sts get-caller-identity`并验证您是否能看到您的用户凭证来验证配置是否成功。您可以继续为其余帐户创建`aws-vault`配置，并且您已经安全地设置了您的`aws` cli 访问。使用`aws-vault`的一个优点是，您不需要在`aws/credentials`文件中以纯文本格式存储您的访问凭证。

设置`aws-vault`在很大程度上简化了我的部署过程。我不再需要修改我的`aws/config`文件来进行跨帐户部署。我只需要选择相应的帐户并执行我需要的命令。

```
aws-vault exec dev -- terraform plan
aws-vault exec staging -- terraform plan
aws-vault exec prod -- terraform plan
```

您可以通过`aws-vault exec <sub-account> -- env | grep AWS`查看您的`aws-vault`账户中存储的特定子账户的所有信息。让我知道你如何找到这个工具的用法。