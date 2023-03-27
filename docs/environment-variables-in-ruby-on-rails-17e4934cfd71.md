# Ruby On Rails 中的环境变量

> 原文：<https://itnext.io/environment-variables-in-ruby-on-rails-17e4934cfd71?source=collection_archive---------0----------------------->

(保护您的信息安全)

![](img/ef4f9c372ed64712e6f2759effba3653.png)

[*点击这里在 LinkedIn* 上分享这篇文章](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fenvironment-variables-in-ruby-on-rails-17e4934cfd71)

每个应用程序都需要配置设置，如电子邮件帐户凭证或外部服务的 API 密钥(出于安全目的)。您可以使用环境变量将本地配置设置传递给应用程序。
在 Ruby On Rails 中有几种使用环境变量的方法，我们也有一个像 Figaro gem 这样的 Gem。

将您的环境变量文件保存为私有文件

如果你使用 GitHub 来存储和共享代码，并且你的项目是开源的，这意味着任何开发者都可以访问你的代码。你不想与公众分享你的私人信息或 API 密匙。如果您在一个拥有私有 git 存储库的团队中协作，您的本地设置可能不适合团队的所有成员。

U唱一个 local_env.yml 文件:

我们将创建一个简单的文件，其中包含使用标准 YAML 文件格式的每个环境变量的键/值对。

C **创建一个文件 config/local_env.yml:**

#将“您的 Gmail 用户名”作为 ENV[“Gmail 用户名”]提供

```
MAIL_USERNAME: 'Your_Username'
MAIL_PASSWORD: 'Your_Username'
```

开始了。gitignore

如果您已经为您的应用程序创建了 git 存储库，那么您的应用程序根目录应该包含一个名为. gitignore 的文件。

在你的下面添加一行。gitignore 文件。

这可以防止 config/local_env.yml 文件被签入 git 存储库并供其他人查看。

Rails 应用程序配置文件

设置完 env 变量后，我们必须将 local_env.yml 文件设置到 config/application.rb 文件中。在 config/application.rb 文件中:

添加以下代码:

```
config.before_configuration do
  env_file = File.join(Rails.root, 'config', 'local_env.yml')
  YAML.load(File.open(env_file)).each do |key, value|
    ENV[key.to_s] = value
  end if File.exists?(env_file)
end
```

上面的代码从 local_env.yml 文件中设置环境变量。

Use 代码中的环境变量

在 Rails 应用程序中的任何地方使用 ENV["MAIL_USERNAME"]。

举个例子，

```
ActionMailer::Base.smtp_settings = {
    address: "smtp.gmail.com",
    enable_starttls_auto: true,
    port: 587,
    authentication: :plain,
    user_name: ENV["MAIL_USERNAME"],
    password: ENV["MAIL_PASSWORD"],
    openssl_verify_mode: 'none'
    }
```

享受编码。

**感谢&问候，
Alok Rawat**

*原载于*[*qiita.com*](https://qiita.com/alokrawat050/items/0d7791b3915579f95791)*。*