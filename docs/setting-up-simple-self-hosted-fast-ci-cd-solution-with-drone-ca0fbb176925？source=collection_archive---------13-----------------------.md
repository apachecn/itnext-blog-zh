# 使用无人机设置简单、自托管且快速的 CI/CD 解决方案

> 原文：<https://itnext.io/setting-up-simple-self-hosted-fast-ci-cd-solution-with-drone-ca0fbb176925?source=collection_archive---------13----------------------->

![](img/1fb565d47879558b3374bef22a03e58c.png)

在过去的几年里，持续集成和持续交付一直是 DevOps 世界中非常流行的话题。随着越来越多的组织努力改进他们的测试和发布过程，我们观察到在这个生态系统中出现了许多新的工具。所有主要的云提供商都在提供他们的自动化系统([有些非常不成功](https://toxicbakery.github.io/vsts-devops/microsoft-devops-ci/))，所以很难选择一个适合你的用例的。

有相当多的服务(Travis，CircleCI，Google Cloud Builder)和自托管解决方案来构建、测试和部署您的代码，但实际上，只有少数是免费、开源和自托管的。

最知名的 CI/CD 工具是 Jenkins 和 GitLab CI(我最近在 [dev.to post](https://dev.to/krusenas/whats-your-favourite-cicd-tool-and-why-52ll) 上问过这个问题)。然而，Jenkins 占用了大量内存，因为它运行在 Tomcat (Java)上，并且工作流是在 Jenkins 内部定义的，而不是在用于运行这些测试/构建作业的存储库中。

至于 GitLab CI，它非常好，但需要你运行自己的 GitLab(这很大)或在 gitlab.com 上(虽然现在你可以将 Github repos 镜像到 GitLab 以自动开始构建，但是，这增加了复杂性)。你可以独立运行你自己的跑步工具，它们看起来很可靠。我帮助进行后端开发的一个组织正在使用 Gitlab CI，并取得了巨大的成功，但正如我之前提到的，业务定价和服务器运行成本可能会很高。

> *免责声明:几年前我曾尝试过使用无人机，但并不满意。我不确定这是实际的产品问题还是文档问题，但是我无法实现我想要的。这一次似乎很不一样。产品是稳定的，文档是好的，一些非常好的例子就在他们自己的* `*.drone.yml*` *文件中:*[*【https://github.com/drone/drone/blob/master/.drone.yml】*](https://github.com/drone/drone/blob/master/.drone.yml)*。构建快速可靠。*

# 输入无人机

我研究了许多备选方案，并决定至少对我来说，目前最好的 CI/CD 解决方案是 [Drone](https://drone.io/) 。这是一个用 Go 编写的开源、非常轻量级的应用程序，可以作为 SaaS 或自托管服务器使用。无人机由两部分组成:

*   **服务器**负责认证、存储库配置、用户、机密和接受 webhooks。
*   **代理**接收构建任务并实际运行您的工作流。

服务器和代理都是非常轻量级的服务，只使用大约 10-15MB 的 RAM。这基本上意味着你甚至感觉不到它在你的笔记本电脑、台式机甚至是树莓派上运行。

## 我的设置

我在 Raspberry PI 上运行无人机服务器，而代理在我的台式机上本地运行，我的台式机有大量 RAM 和 16 个 CPU 内核。这意味着构建比任何免费的公共服务都要快得多。此外，由于代理通过 Webhook 中继隧道连接到服务器，我可以将它们部署在 Kubernetes、备用笔记本电脑或其他任何地方运行的机器上:

![](img/39a948a20a1b68be36286fa68b07eeaf.png)

这种配置的好处:

*   无需配置 NAT，无人机服务器几乎不使用任何资源，因此它可以运行在廉价的硬件上。
*   代理是分离的，可以在您的家庭/办公室机器上运行。通过隧道连接到它们，代理配置保持不变，无论它们部署在哪里(外部云提供商或内部办公室)。

未来的计划包括从用于无人机服务器的 Raspberry Pi 过渡到英特尔 NUC(或其他一些低功耗替代产品)，并在同一台服务器上运行一些额外的无人机代理。

# 准备配置

初始配置非常简单。首先，我们在 Webhook Relay 上创建一个隧道来获取我们的公共子域。然后，我们使用该子域创建一个 Github OAuth 应用程序。

一旦服务器和隧道守护程序正在运行，您就可以毫不费力地连接/断开无人机代理。

# 为 Github 和远程访问设置隧道

我喜欢 dogfooding，因此我们在整个堆栈中使用我们自己的隧道(CI/CD、监控仪表板、各种其他自动化)。在这种情况下，Webhook 中继隧道提供了两个最重要的东西:

*   无论在哪里都可以访问无人机服务器(在您的笔记本电脑、台式机或某些 Raspberry Pi 上)，无需配置防火墙或路由器即可获得远程访问。
*   Github 的 Webhooks 将触发工作流。

去你的[隧道页面](https://my.webhookrelay.com/tunnels)创建一个新的隧道。如果你在一个自由层，你会得到一个生成的子域，这很好。

![](img/d71197aaa6ff1fd79021b45613a2ba5d.png)

这里:

*   隧道的名字可以是任何东西，但是它必须与你在 docker-compose.yaml 中设置的 webhookrelayd 容器相匹配。
*   **子域**是你在`*.webrelay.io`下的公共子域。如果您参加的是免费计划，这将是自动生成的。
*   **目的地**是无人机服务器容器的地址。因为 docker-compose 配置虚拟网络，所以它必须匹配 docker-compose.yaml 中的服务名。
*   **加密类型**可用于所有付费计划，基本上是 HTTP/TLS。我会推荐订阅获取加密！否则，请给我发短信，我可以安排免费试用。

# 配置 Github OAuth 应用程序

在本指南中，我们将使用 Github 设置无人机。可以关注[公文](https://docs.drone.io/installation/github/single-machine/)。简而言之:

创建 GitHub OAuth 应用程序。消费者密钥和消费者秘密用于授权对 Github 资源的访问。授权回叫 URL 必须匹配以下格式和路径，并且必须使用您的确切服务器方案和主机。因此，如果您的 Webhook 中继隧道地址是`my-drone-subdomain.webrelay.io`，那么授权回调 URL 将是`[https://my-drone-subdomain.webrelay.io/login](https://my-drone-subdomain.webrelay.io/login.)` [。](https://my-drone-subdomain.webrelay.io/login.)

# 部署无人机服务器和代理

我最初开始使用 Drone，服务器和代理都在我的桌面上运行，因为它只供我个人使用，很好。然而，后来我分离了服务器，并把它移到了一个树莓派。

一体式配置如下所示:

```
--- 
services: 
  drone-agent: 
    command: agent
    container_name: drone-agent
    depends_on: 
      - drone-server
    environment: 
      - "DRONE_RPC_SERVER=[https://my-drone-subdomain.webrelay.io](https://my-drone-subdomain.webrelay.io)"
      - "DRONE_RPC_SECRET=<shared secret>"
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME="local"
    image: "drone/agent:1.0.0-rc.5"
    restart: always
    volumes: 
      - "/var/run/docker.sock:/var/run/docker.sock"
  drone-server: 
    container_name: drone-server
    environment: 
      - DRONE_SERVER_HOST=my-drone-subdomain.webrelay.io
      - "DRONE_GITHUB_SERVER=[https://github.com](https://github.com)"
      - "DRONE_ADMIN=<your username>"
      - DRONE_GITHUB_CLIENT_ID=<github-client-id>
      - DRONE_GITHUB_CLIENT_SECRET=<github-client-secret>
      - DRONE_SERVER_PROTO=http
      - "DRONE_RPC_SECRET=<shared secret>"
    image: "drone/drone:1.0.0-rc.5"
    ports: 
      - "8080:80"
    restart: always
    volumes: 
      - "db-data:/var/lib/drone/"
  webhookrelay: 
    command: 
      - "--mode"
      - tunnel
      - "-t"
      - drone
    container_name: drone-webhookrelay
    depends_on: 
      - drone-server
    environment: 
      - KEY=<whr-key>
      - SECRET=<whr-secret>
    image: "webhookrelay/webhookrelayd:latest"
    restart: always
version: "3"
volumes: 
  db-data: ~
```

如果您需要另一台计算机上的另一个代理，仅代理配置:

```
--- 
services: 
  drone-agent: 
    command: agent
    container_name: drone-agent
    environment: 
      - "DRONE_RPC_SERVER=[https://my-drone-subdomain.webrelay.io](https://my-drone-subdomain.webrelay.io)"
      - "DRONE_RPC_SECRET=<shared secret>"
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME="local"
    image: "drone/agent:1.0.0-rc.5"
    restart: always
    volumes: 
      - "/var/run/docker.sock:/var/run/docker.sock"
version: "3"
```

一旦有了配置:

```
docker-compose up
```

它会要求您使用您的 Github 凭证进行认证。

# 使用无人机管道

无人机最好的一点是它们的管道配置。似乎在无人驾驶飞机展示了道路之后，行业紧随其后，现在大多数 CI/CD 服务通过 Docker 容器实现管道。这意味着您的构建和测试与物理主机分离，并且清理是通过设计进行的。一个示例流水线

1.  创建
2.  单元测试
3.  使用 Postgres 数据库运行端到端测试
4.  向时差发送一个网页挂钩
5.  构建图像并将其推送到 DockerHub

看起来像这样:

```
--- 
kind: pipeline
name: default
services: 
  - 
    environment: 
      POSTGRES_DB: pgdb
      POSTGRES_PASSWORD: pgpass
      POSTGRES_USER: pguser
    image: postgres
    name: database
steps: 
  - 
    commands: 
      - "cd cmd/app && go build"
    image: "golang:1.11.4"
    name: build
  - 
    commands: 
      - "go test -v `go list ./... `"
    image: "golang:1.11.4"
    name: unit-test
  - 
    commands: 
      - "make install-transponder"
      - "make e2e"
    environment: 
      POSTGRES_DB: pgdb
      POSTGRES_HOST: database
      POSTGRES_PASSWORD: pgpass
      POSTGRES_USER: pguser
    image: "golang:1.11.4"
    name: end-to-end
  - 
    image: plugins/slack
    name: slack
    settings: 
      channel: general
      icon_url: "[https://i.pinimg.com/originals/51/29/a4/5129a48ddad9e8408d2757dd10eb836f.jpg](https://i.pinimg.com/originals/51/29/a4/5129a48ddad9e8408d2757dd10eb836f.jpg)"
      username: drone
      webhook: 
        from_secret: slack_url
  - 
    image: plugins/docker
    name: docker
    settings: 
      auto_tag: true
      password: 
        from_secret: docker_password
      repo: namespace/repo
      username: 
        from_secret: docker_username
workspace: 
  base: /go
  path: src/github.com/webhookrelay/app
```

通过始终使用容器来运行管道，您将使您的构建和测试真正可移植。每当你按下 Github，一个 webhook 就会点击 Drone，它就会开始构建:

![](img/79da18e01d80e55c9a92842d5f9dc25e.png)

# 提示和技巧

我还想分享一些非常有用的命令。要安装`drone` CLI，请遵循[正式文档](https://docs.drone.io/cli/install/)。

## 直接执行管道

您还可以使用无人机 CLI 直接运行管道:

```
drone exec --secret-file drone_secrets.yaml .drone.yml
```

无人机秘密的模板:

```
slack_url: https://hooks.slack.com/services/xxxxxxxxxxxx
```

## 通过 CLI 添加机密

您可以通过 web 用户界面添加密码，也可以直接从终端使用`drone`:

```
drone secret add -repository username/repository-name --name foo --data bar --allow-pull-request
```

# 包扎

我广泛使用了 Jenkins、Gitlab runners、CircleCI 和 Google Cloud Builder。我想我还是喜欢很多与 GKE 相关的云构建器，因为它可以快速构建图像，并且无需任何额外的工作就可以认证到 GCR 注册中心。然而，您确实会被他们的管道配置文件所困，并且不能真正地转移到任何其他供应商。对于 Jenkins 来说，很难想象如何高效地运行 it 和更小的服务器。无人机，至少对我来说，是一个现代服务的理想例子，它专注于效率和正确地做一件事。

*原载于 2019 年 2 月 11 日*[*【webhookrelay.com】*](https://webhookrelay.com/blog/2019/02/11/using-drone-for-simple-selfhosted-ci-cd/)*。*