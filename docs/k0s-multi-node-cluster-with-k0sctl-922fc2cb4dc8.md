# 使用 K0sctl 的 K0s 多节点集群

> 原文：<https://itnext.io/k0s-multi-node-cluster-with-k0sctl-922fc2cb4dc8?source=collection_archive---------3----------------------->

## 轻松运行单节点或多节点 k0s 集群

![](img/b50b637fd35e3483afe4bd73d2cb1b07.png)

由 [Vladimir Mokry](https://unsplash.com/@vmokry?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/s/photos/cluster?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

自从几个月前我写了关于 k0s 的第一篇文章以来，k0s 环境中发生了很多事情。在这篇新文章中，我们将首先展示单节点集群的简化设置，接下来我们将展示 k0s 的配套工具 [k0sctl](https://github.com/k0sproject/k0sctl) 的用法，它以一种非常简单的方式设置多节点集群。

## 创建单节点集群

由于在 0.12 版本中引入了一个`--single`标志，k0s 使得单节点集群的创建变得更加简单。基本上，一旦下载了 k0s 二进制文件，只需两个命令就可以运行单节点 k0s 集群:

```
$ k0s install controller --single
$ systemctl start k0scontroller
```

*   第一个创建 systemd 单元文件
*   第二个开始这个过程

如果您想在本地机器上运行 k0s，一个简单的解决方案是创建一个 VM 并在其中安装 k0s。我们将使用[流浪者](https://vagrantup.com)来说明这些步骤，这是来自 [Hashicorp](https://hashicorp.com) 的一个很棒的工具。vagger 专注于开发环境，它可以轻松地供应和配置虚拟机。让我们按照以下步骤进行操作:

*   按照各自网站上的说明安装[vagger](https://vagrantup.com)和 [Virtualbox](https://virtualbox.org)
*   创建一个新文件夹和一个名为*的文件，内容如下:*

```
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.network "private_network", ip: "192.168.33.11"
  config.vm.provision "shell", inline: <<-SHELL
    curl -sSLf [https://get.k0s.sh](https://get.k0s.sh) | sudo sh
    sudo k0s default-config > /tmp/k0s.yaml
    sudo k0s install controller --single -c /tmp/k0s.yaml
    sudo systemctl start k0scontroller && sleep 10
    sudo cp /var/lib/k0s/pki/admin.conf /vagrant/kubeconfig
    sed -i "s/localhost/192.168.33.11/" /vagrant/kubeconfig
  SHELL
end
```

这个文件告诉 vagger 创建一个 VM 并在其中安装 k0s。

*   在同一文件夹中，运行以下命令:

```
$ vagrant up
```

完成安装只需要几十秒钟，群集启动并运行只需要多一点时间。`kubeconfig`文件已被复制到当前文件夹中，可以像往常一样用于访问集群:

```
$ export KUBECONFIG=$PWD/kubeconfig$ kubeclt get nodes
NAME      STATUS   ROLES    AGE     VERSION
vagrant   Ready    <none>   2m57s   v1.20.5-k0s1
```

我们看到创建本地单节点 k0s 集群非常容易。现在，如何创建一个高可用性多节点集群呢？这就是我们将在下一部分阐述的内容。

## 创建多节点集群

除了 k0s 0.10 [之外，Mirantis](https://medium.com/u/fedcaa2e9074?source=post_page-----922fc2cb4dc8--------------------------------) 还推出了 [k0sctl](https://github.com/k0sproject/k0sctl) ，这是一款使用一个命令部署多节点集群的工具。在这一部分中，我们将看到 k0sctl 的实际应用，并使用它来创建我们自己的 HA 集群，该集群由 3 个控制器节点和 3 个 workers 组成。我们将遵循以下步骤:

*   基础设施的创建
*   k0sctl 的安装
*   创建集群的配置文件
*   集群的创建
*   清除

让我们从创建虚拟机开始，稍后将在这些虚拟机上安装集群。

1.  **创建基础设施**

在本文中，我们将在欧洲云提供商 [Exoscale](https://exoscale.com) 上部署一个集群。为了部署我们的 6 个 ubuntu 虚拟机，我们可以使用 Exoscale 的 web 界面，因为它简洁且非常直观，但我们也可以使用 [Terraform provider](https://registry.terraform.io/providers/exoscale/exoscale/latest) 。

注意:如果您想尝试 Exoscale Terraform 提供程序，您可以使用 [this GitLab repository](https://gitlab.com/lucj/infra/-/tree/master/terraform/exoscale-instances) 中的配置文件。

在本例中，我使用 Terraform 创建虚拟机，并获得每个虚拟机的名称和 IP 地址作为输出。

```
instances_names = {
  "exo-0" = "89.145.162.84"
  "exo-1" = "89.145.162.193"
  "exo-2" = "194.182.171.240"
  "exo-3" = "89.145.160.110"
  "exo-4" = "89.145.161.64"
  "exo-5" = "89.145.162.204"
}
```

我们将在群集的配置步骤中使用这些 IP。

**2。安装 k0sctl**

k0sctl 二进制文件可以从 GitHub 发布页面下载。有适用于 MacOS、Linux 和 Windows 的软件包，但也可以从其 Go 源代码安装。

不带任何参数运行 k0sctl 会列出所有可用的命令:

```
**$ k0sctl** NAME:
   k0sctl - k0s cluster management toolUSAGE:
   k0sctl [global options] command [command options] [arguments...]COMMANDS:
   version     Output k0sctl version
   apply       Apply a k0sctl configuration
   kubeconfig  Output the admin kubeconfig of the cluster
   init        Create a configuration template
   reset       Remove traces of k0s from all of the hosts
   help, h     Shows a list of commands or help for one commandGLOBAL OPTIONS:
   --debug, -d  Enable debug logging (default: false) [$DEBUG]
   --help, -h   show help (default: false)
```

我们将在以下步骤中演示每个命令。

**3。创建集群的配置文件**

在这一步中，我们将创建集群的配置文件，该文件包含 k0sctl 创建和配置集群所需的所有信息。

由于我们不知道这个配置文件是什么样子，我们可以使用下面的命令让 k0sctl 为我们生成一个示例文件:

```
$ k0sctl init > cluster.yaml
```

这个文件提供了一个有用的例子。它基本上定义了一个 2 节点集群，一个控制器和一个工作器。对于每个节点，都提供了 IP 地址和 ssh 密钥(k0sctl 连接到节点时需要)。

```
**$ cat cluster.yaml** apiVersion: k0sctl.k0sproject.io/v1beta1
kind: Cluster
metadata:
  name: k0s-cluster
spec:
  hosts:
  - ssh:
      address: 10.0.0.1
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/id_rsa
    role: controller
    os: ""
  - ssh:
      address: 10.0.0.2
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/id_rsa
    role: worker
    os: ""
  k0s:
    version: 0.13.1
```

我们将对此文件稍加修改，使其使用我们自己的虚拟机:

*   `exo-0`、`exo-1`和`exo-2`将作为控制器
*   `exo-4`、`exo-5`和`exo-6`将充当工人

以下是该配置文件的新版本:

```
apiVersion: k0sctl.k0sproject.io/v1beta1
kind: Cluster
metadata:
  name: k0s-cluster
spec:
  hosts:
  - ssh:
      address: 89.145.162.84
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/training
    role: controller
    os: ""
  - ssh:
      address: 89.145.162.193
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/training
    role: controller
    os: ""
  - ssh:
      address: 194.182.171.240
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/training
    role: controller
    os: ""  
  - ssh:
      address: 89.145.160.110
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/training
    role: worker
    os: ""
  - ssh:
      address: 89.145.161.64
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/training
    role: worker
    os: ""
  - ssh:
      address: 89.145.162.204
      user: root
      port: 22
      keyPath: /Users/luc/.ssh/training
    role: worker
    os: ""
  k0s:
    version: 0.13.1
```

注意事项:

*   */Users/luc/。ssh/training* 是我的本地私钥，对应于在创建过程中自动添加到每个实例的公钥(参数提供给 Terraform 配置文件)
*   在这个阶段，可以指定许多附加的配置选项，但是这些选项对于了解全局来说是很好的

在下一步中，我们将使用此配置文件在我们的基础架构上创建集群。

**4。集群的创建**

要使用 k0sctl 创建集群，我们只需向`apply`子命令提供我们的配置文件:

```
$ k0sctl apply --config cluster.yaml
```

K0sctl 将连接到每个节点并相应地配置它们

```
⠀⣿⣿⡇⠀⠀⢀⣴⣾⣿⠟⠁⢸⣿⣿⣿⣿⣿⣿⣿⡿⠛⠁⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠀█████████ █████████ ███
⠀⣿⣿⡇⣠⣶⣿⡿⠋⠀⠀⠀⢸⣿⡇⠀⠀⠀⣠⠀⠀⢀⣠⡆⢸⣿⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀███          ███    ███
⠀⣿⣿⣿⣿⣟⠋⠀⠀⠀⠀⠀⢸⣿⡇⠀⢰⣾⣿⠀⠀⣿⣿⡇⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠀███          ███    ███
⠀⣿⣿⡏⠻⣿⣷⣤⡀⠀⠀⠀⠸⠛⠁⠀⠸⠋⠁⠀⠀⣿⣿⡇⠈⠉⠉⠉⠉⠉⠉⠉⠉⢹⣿⣿⠀███          ███    ███
⠀⣿⣿⡇⠀⠀⠙⢿⣿⣦⣀⠀⠀⠀⣠⣶⣶⣶⣶⣶⣶⣿⣿⡇⢰⣶⣶⣶⣶⣶⣶⣶⣶⣾⣿⣿⠀█████████    ███    ██████k0sctl v0.8.2 Copyright 2021, k0sctl authors.
Anonymized telemetry of usage will be sent to the authors.
By continuing to use k0sctl you agree to these terms:
[https://k0sproject.io/licenses/eula](https://k0sproject.io/licenses/eula)
**INFO ==> Running phase: Connect to hosts** INFO [ssh] 89.145.162.204:22: connected
INFO [ssh] 89.145.162.84:22: connected
INFO [ssh] 89.145.160.110:22: connected
INFO [ssh] 89.145.161.64:22: connected
INFO [ssh] 194.182.171.240:22: connected
INFO [ssh] 89.145.162.193:22: connected
**INFO ==> Running phase: Detect host operating systems** INFO [ssh] 89.145.160.110:22: is running Ubuntu 20.04.2 LTS
INFO [ssh] 89.145.162.84:22: is running Ubuntu 20.04.2 LTS
INFO [ssh] 89.145.162.204:22: is running Ubuntu 20.04.2 LTS
INFO [ssh] 194.182.171.240:22: is running Ubuntu 20.04.2 LTS
INFO [ssh] 89.145.161.64:22: is running Ubuntu 20.04.2 LTS
INFO [ssh] 89.145.162.193:22: is running Ubuntu 20.04.2 LTS
**INFO ==> Running phase: Prepare hosts
INFO ==> Running phase: Gather host facts** INFO [ssh] 89.145.162.193:22: discovered eth0 as private interface
INFO [ssh] 89.145.162.84:22: discovered eth0 as private interface
INFO [ssh] 194.182.171.240:22: discovered eth0 as private interface
**INFO ==> Running phase: Download k0s on hosts** INFO [ssh] 89.145.162.84:22: downloading k0s 0.13.1
INFO [ssh] 89.145.160.110:22: downloading k0s 0.13.1
INFO [ssh] 194.182.171.240:22: downloading k0s 0.13.1
INFO [ssh] 89.145.162.204:22: downloading k0s 0.13.1
INFO [ssh] 89.145.161.64:22: downloading k0s 0.13.1
INFO [ssh] 89.145.162.193:22: downloading k0s 0.13.1
**INFO ==> Running phase: Validate hosts
INFO ==> Running phase: Gather k0s facts
INFO ==> Running phase: Validate facts
INFO ==> Running phase: Configure k0s**
WARN [ssh] 89.145.162.84:22: generating default configuration
INFO [ssh] 89.145.162.84:22: validating configuration
INFO [ssh] 89.145.162.84:22: configuration was changed
INFO [ssh] 89.145.162.193:22: validating configuration
INFO [ssh] 89.145.162.193:22: configuration was changed
INFO [ssh] 194.182.171.240:22: validating configuration
INFO [ssh] 194.182.171.240:22: configuration was changed
**INFO ==> Running phase: Initialize the k0s cluster** INFO [ssh] 89.145.162.84:22: installing k0s controller
INFO [ssh] 89.145.162.84:22: waiting for the k0s service to start
INFO [ssh] 89.145.162.84:22: waiting for kubernetes api to respond
**INFO ==> Running phase: Install controllers** INFO [ssh] 89.145.162.84:22: generating token
INFO [ssh] 89.145.162.193:22: writing join token
INFO [ssh] 89.145.162.193:22: installing k0s controller
INFO [ssh] 89.145.162.193:22: starting service
INFO [ssh] 89.145.162.193:22: waiting for the k0s service to start
INFO [ssh] 89.145.162.193:22: waiting for kubernetes api to respond
INFO [ssh] 89.145.162.84:22: generating token
INFO [ssh] 194.182.171.240:22: writing join token
INFO [ssh] 194.182.171.240:22: installing k0s controller
INFO [ssh] 194.182.171.240:22: starting service
INFO [ssh] 194.182.171.240:22: waiting for the k0s service to start
INFO [ssh] 194.182.171.240:22: waiting for kubernetes api to respond
**INFO ==> Running phase: Install workers** INFO [ssh] 89.145.162.84:22: generating token
INFO [ssh] 89.145.161.64:22: writing join token
INFO [ssh] 89.145.160.110:22: writing join token
INFO [ssh] 89.145.162.204:22: writing join token
INFO [ssh] 89.145.160.110:22: installing k0s worker
INFO [ssh] 89.145.162.204:22: installing k0s worker
INFO [ssh] 89.145.161.64:22: installing k0s worker
INFO [ssh] 89.145.160.110:22: starting service
INFO [ssh] 89.145.162.204:22: starting service
INFO [ssh] 89.145.161.64:22: starting service
INFO [ssh] 89.145.160.110:22: waiting for node to become ready
INFO [ssh] 89.145.162.204:22: waiting for node to become ready
INFO [ssh] 89.145.161.64:22: waiting for node to become ready
**INFO ==> Running phase: Disconnect from hosts** INFO ==> Finished in 1m35s
INFO k0s cluster version 0.13.1 is now installed
INFO Tip: To access the cluster you can now fetch the admin kubeconfig using:
INFO      k0sctl kubeconfig
```

我留下了整个输出，因为看到设置过程中涉及的不同阶段非常有趣(粗体文本)。

然后，我们可以使用以下命令获得集群的`kubeconfig`:

```
$ k0sctl kubeconfig -c cluster.yaml > kubeconfig
```

像往常一样，我们设置 KUBECONFIG 环境变量，使它指向这个`kubeconfig`:

```
$ export KUBECONFIG=$PWD/kubeconfig
```

我们的本地 kubectl 客户机现在可以与集群通信了。列出集群的节点，我们只能看到工作节点，这是 k0s 的特性之一，确保控制器和工作节点是隔离的。

```
**$ kubectl get nodes** NAME    STATUS   ROLES    AGE     VERSION
exo-3   Ready    <none>   4m54s   v1.20.5-k0s1
exo-4   Ready    <none>   4m53s   v1.20.5-k0s1
exo-5   Ready    <none>   4m54s   v1.20.5-k0s1
```

**5。清理**

k0sctl 附带了一个方便的`reset`命令，可以从基础设施中删除所有与 k0s 相关的组件。当使用相同的基础设施来测试多种配置时，这非常有用。

```
$ k0sctl reset -c cluster.yaml
```

## 摘要

k0s 仍然很新，但它发展得非常快。几个版本之前引入的额外的 k0sctl 工具无疑是一个伟大的举动，因为它隐藏了许多内部复杂性。k0s 和 k0sctl 绝对是两个值得密切关注的有前途的项目。请继续关注未来版本的评论。