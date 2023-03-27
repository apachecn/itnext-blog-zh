# 仙丹+ Kubernetes =💜(第一部分)

> 原文：<https://itnext.io/elixir-plus-kubernetes-part-1-80129eab14f0?source=collection_archive---------0----------------------->

如何使用 Elixir 1.9 设置和自动扩展 Elixir 集群

![](img/60603126738aac241cac2cf8628de07f.png)

> 这是关于如何创建和部署能够在 Kubernetes 上自动伸缩的 Elixir 应用程序的系列文章的第 1 部分。[第二部分](https://medium.com/@groksrc/https-medium-com-groksrc-elixir-plus-kubernetes-part-2-a0579473d7f9)和[第三部分](https://medium.com/@groksrc/elixir-kubernetes-part-3-9bbd71c9c370)也可用。如果你只是想看源代码，可以在这里找到:【https://github.com/groksrc/el_kube如果你只是想看命令的摘要，可以在这里[找到原话的幻灯片。](https://docs.google.com/presentation/d/1xN2Mi_Q-TfwGHNnJ3OczKTvhq_bFyOcMNBa1H4NPMak/edit?usp=sharing)

本系列旨在向您展示如何使用 Elixir 1.9 和 Kubernetes 创建自动伸缩的 Elixir 集群。那是什么意思？这意味着应用程序将启动，在 erlang 集群配置中自动连接，然后随着 Kubernetes (k8s)配置的变化自动添加和删除 erlang 节点。

在第 1 部分中，我们将着眼于创建应用程序本身。第 2 部分将向您展示如何对应用程序进行 Dockerize，并确认容器可以工作。最后，在第 3 部分中，我们将使用 minikube 在 Kubernetes 上启动应用程序，并演示它如何随着部署的变化而自动伸缩。我们开始吧！

# 第 1 部分—创建应用程序

在你开始之前，你要确保你已经安装了[药剂 1.9](https://elixir-lang.org/install.html) 和 OTP 22。您还需要确保您使用的是当前版本的 [Phoenix](https://hexdocs.pm/phoenix/installation.html#content) ，在撰写本文时是 1.4.9 版。此外，确保安装了 [minikube](https://github.com/kubernetes/minikube) 并且 [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 配置为使用 minikube 上下文。

我们的应用程序将被命名为`el_kube`，顺便说一句，所有这些代码都可以在我在[https://github.com/groksrc/el_kube](https://github.com/groksrc/el_kube)的回购中获得。我在 master 中从一个空项目开始，然后在每个分支中，我只更改进行下一步所必需的文件。我最初在 Simpli.fi 技术讲座上展示了这项工作，因此您应该能够直接从[幻灯片组](https://docs.google.com/presentation/d/1xN2Mi_Q-TfwGHNnJ3OczKTvhq_bFyOcMNBa1H4NPMak/edit?usp=sharing)中发出命令并更改文件。如果你遇到任何麻烦，给我一个回复，我会尽力帮忙的。

## 00:新项目

让我们从开发机器上的工作目录开始一个新的 Phoenix 项目。我把我的叫做 el_kube，因为长生不老药和 Kubernetes 搅在一起。

```
$ mix phx.new el_kube
```

当被要求“获取并安装依赖项”时，一定要说“是”。[Yn]"

## 01:打开项目

现在让我们切换到项目文件夹并打开我们的编辑器。以下所有命令都是从项目文件夹的根目录发出的。此外，我将 [vscode](https://code.visualstudio.com/) 与 ElixirLS 插件一起使用，因此您正在使用 git，您可能希望添加。elixir_ls/文件夹添加到. gitignore。

```
$ cd el_kube && code .
```

## 02:编辑 mix.exs

项目在编辑器中打开后，弹出到项目根目录下的 mix.exs 文件。我们将在这里进行三项更改。

首先把`project`键从 1.5 更新到 1.9。这个项目需要使用 Elixir 1.9。

```
elixir: "~> 1.9",
```

接下来，我们将添加对贵族的依赖，并设置[贵族](https://github.com/mrluc/peerage)在启动时作为`extra_application`启动。正如自述文件中所说，Peerage 可以帮助 erlang 节点找到彼此。我们会用它来提供秘方让它发挥作用。

```
def application do
  [
    mod: {ElKube.Application, []},
    extra_applications: [:logger, :runtime_tools, :peerage]
  ]
end
```

和...

```
defp deps do
    [
      {:phoenix, "~> 1.4.9"},
      {:phoenix_pubsub, "~> 1.1"},
      {:phoenix_ecto, "~> 4.0"},
      {:ecto_sql, "~> 3.1"},
      {:postgrex, ">= 0.0.0"},
      {:phoenix_html, "~> 2.11"},
      {:phoenix_live_reload, "~> 1.2", only: :dev},
      {:gettext, "~> 0.11"},
      {:jason, "~> 1.0"},
      {:plug_cowboy, "~> 2.0"},
      {:peerage, "~> 1.0"}
    ]
  end
```

一旦这三个更改成为 mix.exs，您就可以保存并关闭它。如果您的编辑器没有为您运行`$ mix deps.get`，请不要忘记运行它。

## 02:初始化发布

接下来，我们将使用新的 mix 命令为我们生成一些模板文件。这些模板文件用于帮助生成应用程序启动时执行的脚本。

```
$ mix release.init
* creating rel/vm.args.eex
* creating rel/env.sh.eex
* creating rel/env.bat.eex
```

这个命令在`rel`目录中为您生成三个不同的 Elixir 模板。我们将忽略 env.bat.eex。我不在 windows 上，但如果我在，我需要在那里应用相关的更改。相反，我将更新 env.sh.eex，以便在启动时为我设置一些环境变量。

## 03:更新 rel/env.sh.eex

在`rel/env.sh.eex`中，我们将取消导出 RELEASE_DISTRIBUTION 和 RELEASE_NODE 的行的注释。然后将 localhost IP 更改为将动态传递的环境变量。对文件进行以下更改:

```
export RELEASE_DISTRIBUTION=name
export RELEASE_NODE=<%= @release.name %>@${HOSTNAME}
```

将`RELEASE_DISTRIBUTION`环境变量配置为`name`将会配置 erlang beam，使其能够使用长名称格式连接到集群中的其他节点。`RELEASE_NODE`是使用长名称格式的 erlang 节点的名称。有关配置这些环境变量的更多详细信息，请参见 mix [任务文档](https://hexdocs.pm/mix/Mix.Tasks.Release.html#module-environment-variables)。

出于我们的目的，我们将动态传递 k8s 分配给容器的 IP 地址作为`HOSTNAME`，但是您也可以在这里使用有效的集群 DNS 名称。把那个文件存起来，我们接着做下一个。

## 04:删除配置/产品.机密. exs

该文件不会被使用，因此我们将放弃它。

```
$ rm config/prod.secret.exs
```

## 05:创建配置/版本. exs

相反，我们将使用新的`releases.exs`文件来帮助我们在启动时动态配置应用程序。让我们创建文件并在编辑器中打开它。

```
$ touch config/releases.exs
```

## 06:编辑配置/版本. exs

现在我们准备设置我们的动态配置。将此粘贴到文件中:

```
import Config

service_name = System.fetch_env!("SERVICE_NAME")
db_url = System.fetch_env!("DB_URL")
secret_key_base = System.fetch_env!("SECRET_KEY_BASE")
port = System.fetch_env!("PORT")

config :el_kube, ElKube.Repo, url: db_url

config :el_kube, ElKubeWeb.Endpoint,
  http: [port: port],
  secret_key_base: secret_key_base,
  url: [host: {:system, "APP_HOST"}, port: {:system, "PORT"}]

config :peerage, via: Peerage.Via.Dns,
  dns_name: service_name,
  app_name: "el_kube"
```

`service_name`将是应用程序的内部 DNS 名称。换句话说，集群上的一个节点将能够向 DNS 查询`service_name`，并获得一个包含 IP 地址列表的响应。

`db_url`是 Postgres 连接字符串。`secret_key_base`用于 Phoenix 签署 cookies，而`port`是 web 端点将监听的端口。

您还会注意到在`ElKubeWeb.Endpoint`中也加入了一个`APP_HOST`环境变量。Phoenix 有点古怪，因为在 Elixir 1.9 之前它确实允许一些动态配置。这里我们只是使用这个内置的构造来告诉 Phoenix 从环境中拉出`APP_HOST`和`PORT`。这两个值用于它的内部 URL 助手。把这份文件存档，然后转到下一份。

## 07:编辑配置/产品. exs

现在让我们看看生产配置文件。与以前的 Elixir 版本一样，特定于环境的配置位于基本 config/config.exs 之上，因此我们可以在这里设置一些不能更改的特定于生产的内容。首先，我们需要删除已经移到`config/releases.exs`中的`url`键，接下来我们需要添加端点配置键/值对`server: true`

这告诉 Phoenix 在服务器启动时启动 webserver 端点。为什么这不是这个文件中的默认设置，我不确定，但是让我们做一些修改，然后继续。您的 config/prod.exs 文件现在应该是这样的。

```
use Mix.Config# Lots of comments ...
config :el_kube, ElKubeWeb.Endpoint,
  cache_static_manifest: "priv/static/cache_manifest.json",
  server: true# Do not print debug messages in production
config :logger, level: :info# Lots more comments ...
```

哦！不要忘记向下滚动到文件的最底部，删除`import_config "config/prod.secrets.exs"`，我在几次运行中错过了它，所以这就是我们之前删除它的原因。如果我们忘记了这一步，它现在会向我们抱怨。

## 08:编辑 config/config.exs

这一步不是完全必要的。当然，你可以在现实生活中任意配置你的应用程序。但出于演示的目的，我们将加入一些基本的 Ecto 配置，这样我们就可以证明它确实被选中了。打开 config/config.exs 文件并添加以下内容:

```
config :el_kube, ElKube.Repo,
  adapter: Ecto.Adapters.Postgres,
  pool_size: 10
```

再说一次，所有这些都可能不在这里。在现实生活中，您可能希望从 config/releases.exs 配置您的`pool_size`,所以这纯粹是说教。

## 09:编辑配置/开发文件

最后一个文件修改，我们就可以开始编译了。让我们在开发环境中让 peerage 开心，也因为我们通常不在开发模式下运行集群(peerage 对此大声抱怨),让我们告诉它安静。将以下内容添加到 config/dev.exs 文件中:

```
config :peerage,
  via: Peerage.Via.List,
  node_list: [:"el_kube@127.0.0.1"],
  log_results: false
```

## 10:编译

现在让我们切换到我们的终端，执行编译和冒烟测试。首先，我们需要为 Phoenix 的 javascript 端生成摘要文件。

```
$ mix phx.digest
...
==> el_kube
Check your digested files at "priv/static"
```

接下来，让我们生成发布:

```
$ MIX_ENV=prod mix release
...
...
To list all commands: _build/prod/rel/el_kube/bin/el_kube
```

如果你看到了以上内容，你就取得了很大的进步！最后一步是为我们的应用程序创建一个数据库，以便在启动时进行连接。

```
$ mix ecto.create
The database for ElKube.Repo has been created
```

现在我们准备运行发布，并确保它启动。这个命令必须传递我们之前考虑过的所有环境变量，除了一个我没有提到的变量`RELEASE_COOKIE`。这一点在 mix 发布任务文档中也有涉及，但是我想在这里谈一下。`RELEASE_COOKIE`是一种预共享密钥，erlang beam 在节点连接时使用它进行身份验证。这里并没有严格的必要，但是我想在这里介绍它，因为您将在第 3 部分中再次看到它。

## 11:烟雾测试

下面是启动应用程序的命令:

```
DB_URL=ecto://postgres:postgres@localhost/el_kube_dev \
RELEASE_COOKIE=foo \
SECRET_KEY_BASE=foo \
HOSTNAME=127.0.0.1 \
SERVICE_NAME=localhost.svc \
APP_HOST=localhost \
PORT=4000 \
_build/prod/rel/el_kube/bin/el_kube start
```

我们已经在上面介绍了所有这些环境变量的作用，所以我不再赘述。这里的新内容是，我们实际上正在执行我们的应用程序并传递 start 命令。执行此操作时，您应该会看到以下内容:

```
15:20:47.765 [info] Running ElKubeWeb.Endpoint with cowboy 2.6.3 at 0.0.0.0:4000 (http)
15:20:47.765 [info] Access ElKubeWeb.Endpoint at [http://localhost:4000](http://localhost:4000)
```

现在你可以打开浏览器到那个地址或`$ curl [http://localhost:4000](http://localhost:4000)`，你应该会看到一个网页回来。如果是的话，你就成功了！

接下来，让我们通过启动另一个终端并连接到这个正在运行的实例来确认我们的数据库连接正在工作。一切都和前面的命令一样，除了这次不是在最后传递 start，而是传递`remote`参数:

```
DB_URL=ecto://postgres:postgres@localhost/el_kube_dev \
RELEASE_COOKIE=foo \
SECRET_KEY_BASE=foo \
HOSTNAME=127.0.0.1 \
SERVICE_NAME=localhost.svc \
APP_HOST=localhost \
PORT=4000 \
_build/prod/rel/el_kube/bin/el_kube remote
```

在这里，你应该进入一个药剂提示:

```
Erlang/OTP 22 [erts-10.4.4] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [hipe] [dtrace]Interactive Elixir (1.9.1) - press Ctrl+C to exit (type h() ENTER for help)
iex(el_kube@127.0.0.1)1>
```

此时，您可以发出一个 Ecto 命令，您应该会得到一个`:ok`元组。

```
iex(el_kube@127.0.0.1)1> ElKube.Repo.query("select 1 as test")
{:ok,
 %Postgrex.Result{
   columns: ["test"],
   command: :select,
   connection_id: 41993,
   messages: [],
   num_rows: 1,
   rows: [[1]]
 }}
```

如果你能走到这一步，恭喜你！现在你已经准备好进入第二部分并归档你的申请。

-g