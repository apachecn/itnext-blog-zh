# 将 Ruby on Rails 部署到 Kubernetes

> 原文：<https://itnext.io/deploy-ruby-on-rails-to-kubernetes-b845cd3d5dae?source=collection_archive---------2----------------------->

Kubernetes 上 Ruby on Rails 的优化开发和部署工作流

![](img/8c1ea4b6be200c870428b69c93df43c8.png)

Rails 是一个强大的框架，自带电池。有一样东西不是现成的，那就是将 Rails 项目部署到 Kubernetes 的简单方法，以及在 Kubernetes 上迭代开发 Rails 项目的省时方法。

在本教程中，我们通过展示开发人员如何使用 Kubernetes 构建 Rails 应用程序，将 DevOps 推向了一个新的高度。本教程对于 Kubernetes 初学者来说很棒，但对于那些希望在 Kubernetes 上简化 Rails 开发和部署工作流的高级用户来说也很棒。

> *本教程适用于任何 Kubernetes 集群(minikube、Docker Kubernetes、GKE、EKS、AKS 等。)但是如果您手头没有集群，您仍然可以使用 DevSpace 赞助的一个免费的、完全托管的 Kubernetes 名称空间来学习本教程。*

# 1.创建 Rails 项目

> 显然，如果您已经有了一个 Rails 项目，并希望在本教程中使用，那么您可以跳过这一步。

在创建 Rails 项目之前，让我们用希望 Rails 项目所在的目录来定义一个变量:

```
PROJECT_DIR=/c/Users/[username]/rails-project # for win (git-bash)
PROJECT_DIR=/home/[username]/rails-project # for mac, linux (bash)
```

> *如果您使用 Docker Toolbox，您的* `*$PROJECT_DIR*` *需要在您用户的主目录内，例如* `*/c/Users/[username]*` *。*
> 
> *如果您在 Windows 上使用较新的 Docker 桌面，请确保您的项目所在的驱动器在 Docker 设置中标记为* [*共享驱动器*](https://docs.docker.com/docker-for-windows/#shared-drives) *。*

现在，让我们为 Rails 项目的名称创建一个变量:

```
PROJECT_NAME=app
```

如果您手头没有 Rails 应用程序，这段代码是利用 Docker 快速创建 Rails 项目的好方法:

```
mkdir $PROJECT_DIRdocker run --rm -v /$PROJECT_DIR:/app -w //app ruby:2.6-alpine sh -c "apk add build-base nodejs yarn tzdata sqlite-dev postgresql-client postgresql-dev && gem install pg rails && rails new $PROJECT_NAME --database=postgresql"cd $PROJECT_DIR/$PROJECT_NAME
```

这个命令可能会花费一些时间，但是如果您熟悉 Ruby on Rails，您可能已经知道了。

> *正如您在代码片段中看到的，我们使用了* `*--database=postgresql*` *，因为我们稍后想要添加一个 PostgreSQL 数据库(当然，如果您不需要，可以跳过这一步)。*

# 2.集装箱化项目

因此，让我们将 Rails 项目容器化。为了快速简单地完成这项工作，我们使用了 Kubernetes 的开源开发工具 [DevSpace](https://github.com/devspace-cloud/devspace) 。DevSpace 是一个用于容器化项目的客户端专用 CLI 工具，也是将应用程序交互式部署到 Kubernetes 的最快工具。

没有针对 DevSpace 的 Ruby gem，但是您可以通过 npm 轻松安装 DevSpace:

```
npm install -g devspace
```

更多安装选项，请看一下 [GitHub repo](https://github.com/devspace-cloud/devspace#quickstart) 。

安装 DevSpace 之后，将项目容器化是非常简单的事情。只需运行以下命令:

```
devspace init
```

**当 DevSpace 询问您应用程序正在监听哪个端口时，键入** `**3000**` **并点击** `**Enter**` **。剩下的问题应该很容易回答。如果你不确定，你可以点击`Enter`来选择默认选项。**

初始化过程完成后，您就可以将项目部署到 Kubernetes 了。但是，如果您需要一个数据库，在部署项目之前添加它是明智的。

## 额外收获:添加 Postgres 数据库

打开文件`devspace.yaml`(由`devspace init`创建)并将`deployments`部分从:

```
deployments:
- name: app
  helm:
    componentChart: true
    values:
      containers:
      - image: dscr.io/${DEVSPACE_USERNAME}/app
      service:
        ports:
        - port: 3000
```

收件人:

```
deployments:
- name: database
  helm:
    chart:
      name: stable/postgresql
    values:
      postgresqlDatabase: "db_app"
      postgresqlUsername: "db_user"
      postgresqlPassword: ${DB_PASSWORD}
- name: app
  helm:
    componentChart: true
    values:
      containers:
      - image: dscr.io/${DEVSPACE_USERNAME}/app
        env:
        - name: DATABASE_URL
          value: "postgres://db_user:${DB_PASSWORD}[@database](http://twitter.com/database)-postgresql/db_app"
      service:
        ports:
        - port: 3000
```

搞定了。就这样，现在您可以部署这个 Ruby 项目，并立即拥有一个 PostgreSQL 数据库供应用程序使用。

# 3.选择 Kubernetes 集群

如果您没有 Kubernetes 集群或者想要对托管的 Kubernetes 名称空间使用类似 Heroku 的服务，请运行以下命令在 DevSpace Cloud 中获取一个免费的名称空间:

```
devspace create space ruby-app
```

您可以将 DevSpace Cloud 视为 Kubernetes 的一种 Heroku，但在技术堆栈、可移植性和可伸缩性方面没有限制。因此，您可以部署任何容器，并且只需一个命令就可以轻松地将您的应用程序移动到另一个 Kubernetes 集群。

> *DevSpace 会自动为你在 DevSpace Cloud 中创建的空间添加 kube-context，这样你也可以使用 kubectl、helm 等工具。*

如果您已经有一个 Kubernetes 集群，那就更简单了:只需告诉 DevSpace 使用哪个名称空间(如果不存在，将在部署期间创建):

```
devspace use namespace [my-namespace]
```

# 4.开发项目

在部署这个 Rails 项目之前，我们需要建立一个尽可能高效地开发项目的工作流。DevSpace 为此工作流提供了一个命令:

```
devspace dev
```

> *第一次运行该命令时，DevSpace 会要求您定义一个用于变量* `DB_PASSWORD`的数据库密码。

第一次运行可能需要一段时间，但第二次会快得多。

这个命令实际上是从 Docker 文件构建 Docker 映像，标记它并将其推送到注册表。然后，DevSpace 根据`devspace.yaml`的`deployments`部分部署项目，并启动开发服务(本地主机访问的端口转发、日志流和热重装的文件同步)。

如果一切正常，您应该能够在终端中看到 Rails 日志，浏览器将会打开，在`localhost:3000`上显示著名的 Rails 欢迎页面。

![](img/fac2b79f1dd39ae4a0abb0dee12228c3.png)

Ruby on Rails 欢迎页面——每个 Rails 开发人员最喜欢看的页面。

## 生成索引路由

既然项目已经在 Kubernetes 集群中运行，我们就可以开始开发实际的业务逻辑了。当然，您也可以在本地运行常见的`rails`命令，但是在本教程中，我将向您展示如何在运行容器中直接运行这些命令。

> ***确保*** `***devspace dev***` ***仍在运行*** *后再继续本部分教程。只需让它运行或再次启动它，并打开第二个终端来运行以下命令。*

要为“Hello World”索引页生成控制器，请运行以下命令:

```
devspace enter -- rails generate controller Welcome index
```

> *你可以看到我们正在执行一个常规的* `*rails*` *命令。通过添加前缀* `*devspace enter --*` *，我们告诉 DevSpace 在 Kubernetes 的容器内运行这个* `*rails*` *命令，而不是在本地机器上执行它。您也可以运行容器内的任何其他* `*rails*` *、* `*rake*` *、* `*npm*` *或* `*bundle*` *命令，只需在命令前添加* `*devspace enter --*` *即可。*

然后，像往常一样，将本地文件系统上的`config/routes.rb`从:

```
Rails.application.routes.draw do
  # For details on the DSL available within this file, see [https://guides.rubyonrails.org/routing.html](https://guides.rubyonrails.org/routing.html)
end
```

收件人:

```
Rails.application.routes.draw do
  get 'welcome/index'
  root 'welcome#index'
  # For details on the DSL available within this file, see [https://guides.rubyonrails.org/routing.html](https://guides.rubyonrails.org/routing.html)
end
```

> *DevSpace 确保远程系统上的任何文件更改都会反映在本地文件系统上，反之亦然。这是迭代构建基于 Kubernetes 的应用程序的最快方法。*

如果您只编辑一个文件，您可以看到这有多快。去试试吧:

1.  将“欢迎#索引”替换为“Hello World！”文件内部`app/views/welcome/index.html.erb`
2.  重新加载浏览器以查看更改，并查看从 Kubernetes 集群中的容器实时流出的`devspace dev`命令的日志。

## 额外:添加帖子

如果没有任何数据需要持久化，我们就不需要数据库。因此，出于本教程的目的，让我们为博客文章生成一个 Rails 框架:

```
devspace enter -- rails generate scaffold Post title:string body:text
```

创建支架后，我们需要调用`rake`进行数据库迁移:

```
devspace enter -- rake db:migrate
```

就是这样！一切都已经活了:

1.  转到此 URL:[http://localhost:3000/posts](http://localhost:3000/posts)
2.  添加文章，看看它们是如何保存到数据库中的。
3.  添加新帖子时，观察通过`devspace dev`传输的日志。

# 5.部署项目

如果您已经完成了教程的这一部分，那么您已经掌握了开发流程的诀窍。现在，是时候部署应用程序了。在大多数情况下，您可能希望使用`devspace create space myapp-prod`或运行`devspace use namespace [prod-namespace]`来创建另一个空间，以分离开发和生产。您可以随时使用`devspace use space/namespace`命令切换回开发工作区。

一旦进入正确的 kube 上下文和名称空间，就可以使用以下命令部署项目:

```
devspace deploy
```

在访问已部署的 Rails 项目之前，您需要等到数据库迁移完成并且应用程序开始监听。您可以使用以下命令通过流式传输应用程序容器的日志来检查此过程的状态:

```
devspace logs -f
```

在您看到以下日志行之后:

```
* Listening on tcp://0.0.0.0:3000
```

您可以使用以下命令访问您部署的 Rails 应用程序:

```
devspace open
```

DevSpace 可以为您启动一个私有的端口转发会话(用于类似于`devspace dev`的本地主机访问),也可以为您创建一个 Kubernetes 入口，使您的项目在一个域上公开可用。

# 下一步是什么？

就 Kubernetes 和 Rails 的更高生产率而言，还有很多值得展示的地方。这就是为什么我把这篇文章看作是博客系列的开始。如果你对下一篇文章感兴趣(很可能是关于让 Rails 应用程序在 Kubernetes 上生产就绪和(自动)可伸缩)，请订阅，我的下一篇文章一发表(可能在 3-4 周内)，你就会收到一封简短的电子邮件。

如果这对你有帮助，或者你有任何问题或想法，下一步可能会有什么有趣的博客，请通过 twitter [@LukasGentele](https://twitter.com/LukasGentele) 或电子邮件:lg AT devspace.cloud 联系

期待收到你的来信！

*最初发布于*[*https://devspace . cloud*](https://devspace.cloud/blog/2019/10/21/deploy-ruby-on-rails-to-kubernetes)*。*