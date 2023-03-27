# 使用 Docker 实现零停机部署

> 原文：<https://itnext.io/a-simple-solution-for-zero-downtime-on-deployments-with-docker-bdb71f3101d0?source=collection_archive---------2----------------------->

![](img/b6c314a4089266aa9692708e6eb566ae.png)

用简单的部件建造美妙的建筑

两个月前，在对我们的应用程序进行分类后，我接到了一项任务，要为我们的部署实现零宕机。不使用 Docker 的时候，为了 0-down，可以使用 Puma HTTP server 的`phased-restart`功能。有了 Docker 基础设施，你也可以使用`Swarm`或`Kubernetes`，但是大多数时候，`Swarm`或`Kubernetes`对我们来说都是多余的。

在本文中，我们将使用一些简单的工具来实现 0 停机部署，您可以用其他工具来替代它们，因此我们不想拘泥于某些特定的工具。请记住，实现一种适当的容器部署方式，需要对 Docker 技术有很好的了解，因此在阅读本文之前，请确保您对 Docker 有足够的了解。

首先，我们将实现一个具有单个端点的简单 Rails 应用程序，在将其 docker 化之后，我们将使用 Capistrano 进行一个基于 docker 容器的简单部署。然后我们将设置 Nginx 反向代理，为了做到这一点，我们将使用`nginx-proxy` docker 图像。在最后两步中，我们将改进我们的部署脚本以支持零停机时间，最后，我们将使用`ApacheBench`工具在部署上测试零停机时间。

根据这篇文章，我还准备了一个示例项目，您可以通过下面的链接查看:

[](https://github.com/AliSepehri/zero_downtime) [## AliSepehri/zero_downtime

### 此时您不能执行该操作。您已使用另一个标签页或窗口登录。您已在另一个选项卡中注销，或者…

github.com](https://github.com/AliSepehri/zero_downtime) 

# 初始化应用程序并实现简单的端点

为了有一个简单的应用程序，我们想使用 Ruby-on-Rails 框架。如果您使用的是其他语言/框架，那么对于这个部分，我们唯一需要的就是一个返回带有`200` HTTP 状态的响应的`/ping`端点。

在本文中，我们不想讨论安装 Ruby、Ruby on Rails 以及其他先决条件。我假设我们已经在机器上安装了 Ruby on Rails，为了初始化 Rails 应用程序，我们执行下面的命令:

```
rails new zero-downtime --api
```

为了实现`/ping`端点，使用 Rails 生成器生成`ping`控制器:

```
rails g controller ping
```

将简单测试添加到`test/controllers/ping_controller_test.rb`文件，将路线添加到`config/routes.rb`文件，将相关动作添加到`app/controllers/ping_controller.rb`:

。/test/controllers/ping _ controller _ test . Rb

。/config/routes.rb

。/app/controllers/ping _ controller . Rb

最后，运行测试以确保一切工作正常:

```
rails test test/controllers/ping_controller_test.rb
```

# 将申请归档

现在我们将为我们的 Rails 应用程序编写一个简单的`Dockerfile`。如果你正在使用另一个框架，你肯定可以在网上找到很多关于如何将它 dockerize 的文章。

。/Dockerfile

现在通过运行`docker build`命令构建 Docker 映像:

```
docker build -t zero-downtime .
```

基于我们的 Docker 映像创建并运行一个容器:

```
docker run --rm --name zero-downtime-app -p 3000:3000 zero-downtime
```

并测试它:

```
curl [http://localhost:3000/ping](http://localhost:3000/ping)# ===> {"message":"pong!"}
```

如果你得到了`{"message":"pong!"}`，这意味着你的容器工作了；停止正在运行的容器(`Ctrl+C`)，跟随文章。在下一节中，我们将通过 Capistrano 工具将容器部署到我们的本地机器上。

# 使用 Capistrano 实现简单部署

现在我们想使用 Capistrano 为我们的应用程序实现一个简单的部署流程。当然，Capistrano 有许多替代方案，您可以轻松地使用它们，因为我们在 Capistrano 任务中使用的命令非常接近原始 shell 命令。

首先，我们要将 Capistrano 添加到我们的 Gemfile 中，安装并初始化它。为此，我们可以执行以下命令:

```
*# Add and install the Gem*
**bundle add capistrano***# Generate default config files*
**bundle exec cap install**
```

去掉不必要的线条后，我展示了我的`Capfile`:

。/Capfile

到`./config/deploy.rb`文件中，我们需要设置`repo_url`来引用我们的代码。为此，您应该使用 git 服务器。例如，您可以使用 Github 存储库，然后将 ssh 远程 URL 从存储库页面复制到我们的`deploy.rb`文件中。

。/config/部署. rb

如您所见，我们还有一些任务将在默认部署步骤之后执行。我们将实现`docker.rake`文件中的任务。

使用 Capistrano，您可以针对不同阶段进行不同的配置。我已经展示了我们在开发阶段的配置。

。/config/deploy/开发. rb

如您所见，我们已经为`server`属性设置了`localhost`，这意味着我们将使用本地机器进行部署，而不需要使用虚拟机或真实的服务器。

正如我们之前提到的，我们有一些定制任务要在 Capistrano 的默认步骤之后运行，以完成部署流程的主要部分。

我们已经将这些定制任务放在了`./lib/capistrano/tasks/docker.rake`文件中。

。/lib/capistrano/tasks/docker . rake

我将简要地解释每个任务，当然我们应该对 Docker 有一些经验，以找出在这些任务中到底发生了什么。`create_network`创建`backend`网络，我们将使用它来运行同一网络中的所有容器，以便它们能够相互访问。`create_volumes`创建必备的 Docker 卷。`db_setup`运行`rails db:setup`来初始化数据库(当然，对于一个实际的应用程序，您不应该在每个部署上执行数据库设置)。最后`start_container`将使用指定的环境变量运行应用程序容器。

现在我们已经准备好部署我们的应用程序了。`bundle exec cap development deploy`命令运行开发阶段的 Capistrano 部署流程。运行部署命令后，您应该会看到以下错误:

```
...Errno::ECONNREFUSED: Connection refused — connect(2) for 127.0.0.1:22...
```

我们的本地机器上没有任何 ssh-server，这就是我们得到这个错误的原因。要在 Ubuntu 机器上安装 ssh-server 并配置 ssh-key 对，您可以遵循以下命令:

```
*# ------- Install ssh-server, enable, and run its servic -------* **sudo apt-get install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh***# ------- Create a new key-pair -------*
**ssh-keygen -t rsa -b 4096 -C "Capistrano Local" -f ~/.ssh/capistrano_local***# ------- Add the new private-key for ssh-client -------*
**ssh-add ~/.ssh/capistrano_local***# ------- Add the new public-key for ssh-server -------*
**cat ~/.ssh/capistrano_local.pub >> ~/.ssh/authorized_keys**
```

所以我们将在我们的机器上同时拥有`ssh-server`和`ssh-client`。现在，我们可以开始部署了:

```
bundle exec cap development deploy
```

当它成功完成时，应用程序容器应该正在运行并负责`curl`请求:

```
**curl** [**http://localhost:3000/ping**](http://localhost:3000/ping)*# ===> {"message":"pong!"}*
```

如果我们得到了`{"message":"pong!"}`响应，这意味着您的应用程序容器已经启动并正在运行。

# 添加 Nginx

在我们现有的结构中，运行在应用程序容器中的 Puma 直接接收请求。在这一节中，我们将设置一个 Nginx 容器，并将应用程序容器放在它的后面。为此，我们将使用 [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy) 图像，而不是[官方 nginx](https://hub.docker.com/_/nginx) 图像。当一个新的容器运行时，`nginx-proxy`会识别它，生成反向代理配置，并自动重新加载 Nginx 进程。

我们只需要运行`nginx-proxy`一次，最好不要让它出现在您的部署流程中，但是为了简单起见，我们将使用 Capistrano 来运行`nginx-proxy`容器。此外，我们需要向应用程序容器添加一个新的环境变量(`VIRTUAL_HOST`)。由于有了`VIRTUAL_HOST`环境变量，新容器将被自动识别，并在 Nginx 配置中用作服务器名。

。/lib/capistrano/tasks/docker . rake

`start_nginx`检查正在运行的容器，如果没有运行，则运行`nginx-proxy`容器。对于`start_container`任务，我们也有两个小的变化，正如你所看到的，我们不再需要端口映射，取而代之的是，我们将`VIRTUAL_HOST` env 传递给应用程序容器——正如我们之前所描述的。

我们需要更新`config/deploy.rb`文件，将新任务添加到我们的部署流程中:

。/config/部署. rb

现在我们需要再次运行 Capistrano 部署命令来查看我们的更改:

```
bundle exec cap development deploy
```

对于测试，请记住，如果请求来自通过`VIRTUAL_HOST`环境变量指定的指定域，那么`nginx-proxy`会将请求导航到我们的应用程序容器。所以我们需要通过为`curl`请求设置`Host`头来伪造域:

```
**curl --header "Host: app-domain.test:8080" localhost:8080/ping***# ===> {"message":"pong!"}*
```

# 通过蓝绿部署实现零停机(关键时刻)

我们的想法是向我们的应用程序容器添加一个健康检查，在不停止旧容器的情况下运行新版本的容器，同时运行旧版本和新版本的应用程序，直到新版本负责请求，并最终停止旧容器。

通过向应用程序容器添加健康检查，我们将能够发现容器是否对请求负责。为我们的容器实现健康检查的一个简单方法是向`/ping`端点发送`curl`请求——这是我们在第一节中实现的。

。/lib/capistrano/tasks/docker . rake

因此，根据我们的想法，我们需要在`start_container`任务中添加 health-check。`curl -f localhost:3000/ping`将每 10 秒执行一次，并将更新容器的状态。`wait_for_container`将等待新容器(`zero-downtime-app-new`)变得健康，之后将执行`stop_container`来停止旧容器，最后`rename_container`任务将把新容器的名称从`zero-downtime-app-new`重命名为`zero-downtime-app`。

确保您已经将新步骤添加到`config/deploy.rb`文件中:

。/config/部署. rb

# 使用 ApacheBench 测试零停机时间

我们想使用 [ApacheBench 工具](https://httpd.apache.org/docs/2.4/programs/ab.html)来测试我们的零停机部署实现。我们将使用它在指定的时间限制内发送大量请求，并同时运行部署。ApacheBench 将告诉我们`Non-2xx responses`的计数，这个值应该为零，以便我们确认我们的解决方案。

在机器上打开两个独立的终端。在第一个终端中运行部署:

```
bundle exec cap development deploy
```

在第二个示例中运行 ApacheBench 如果这是您第一次运行这个命令，那么从注册表中获取`ab`映像需要一些时间:

```
docker run --rm --network=backend httpd ab -s5000 -t50 -n1000000 -H "Host: app-domain.test" -c1 [http://nginx/ping](http://nginx/ping)
```

完成部署后，您可以停止 ApacheBench 容器(`Ctrl + C`)，它将生成报告:

```
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, [http://www.zeustech.net/](http://www.zeustech.net/)
Licensed to The Apache Software Foundation, [http://www.apache.org/](http://www.apache.org/)Benchmarking nginx (be patient)Server Software:        nginx/1.19.3
Server Hostname:        nginx
Server Port:            80Document Path:          /ping
Document Length:        19 bytesConcurrency Level:      1
Time taken for tests:   52.124 seconds
**Complete requests:      24705**
**Failed requests:        0**
Total transferred:      13488930 bytes
HTML transferred:       469395 bytes
Requests per second:    473.97 [#/sec] (mean)
Time per request:       2.110 [ms] (mean)
Time per request:       2.110 [ms] (mean, across all concurrent requests)
Transfer rate:          252.72 [Kbytes/sec] receivedConnection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       9
Processing:     1    1   0.3      1      13
Waiting:        1    1   0.3      1      13
Total:          1    1   0.3      1      13Percentage of the requests served within a certain time (ms)
  50%      1
  66%      1
  75%      2
  80%      2
  90%      2
  95%      2
  98%      2
  99%      3
 100%     13 (longest request)
```

`Failed requests`应该是`0`并且你也不应该看到报告中的`Non-2xx responses`部分。

# 托多斯

*   在服务器或 CI 上使用 Capistrano 构建应用程序 docker-image
*   在部署工作流之外配置并运行`nginx-proxy`