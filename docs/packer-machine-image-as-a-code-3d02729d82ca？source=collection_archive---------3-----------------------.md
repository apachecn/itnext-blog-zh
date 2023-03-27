# 打包机:作为代码的机器图像

> 原文：<https://itnext.io/packer-machine-image-as-a-code-3d02729d82ca?source=collection_archive---------3----------------------->

![](img/5600c32e93be9ba7ae5ea33363203b9a.png)

[rawpixel](https://unsplash.com/@rawpixel?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的“画灰色小猫的人的照片”

在这个云计算时代，许多公司都采用不可变的基础设施，将基础设施作为代码来简化配置管理和可靠性。在不可变的基础设施中，创建新的服务器，而不是在运行的服务器上进行更改。构建不可变的基础设施是困难的，并且需要一个复杂的过程来构建和测试它们。实现不可变基础设施的最佳方式是通过代码完成所有事情，包括服务器映像、作为代码的基础设施、使用代码的任何特定配置。我们已经在[基础设施中看到了使用 terraform](https://medium.com/@mitesh_shamra/infrastructure-as-a-code-with-terraform-e7021bf28d7d) 的代码，使用 ansible 的[配置管理。在本帖中，我们将重点放在使用](https://medium.com/@mitesh_shamra/introduction-to-ansible-e5b56ee76b8c) [Hashicorp Packer](https://packer.io/) 的机器图像上。

当我们启动一组应用程序和服务来实现实例上的特定目的时，转到实例并设置它们可能需要一些时间。为了解决这个问题，我们可以使用机器映像，它作为一个静态单元，包含已安装的软件、应用程序和操作系统。在 AWS 环境中，AMI 或 Amazon 机器映像是一个主映像，包含在云中启动实例服务器所需的信息。AMI 就像启动实例所需的根卷的模板。它们包括操作系统、应用程序服务、应用程序以及其他文件或配置。一旦我们创建了一个 AMI 并注册了它，就可以用它来启动一个新的实例。

创建机器映像的过程包括以下步骤:

1.  使用基本映像引导实例。
2.  运行 ansible 或脚本之类的配置工具，将实例配置到所需的状态。
3.  从该服务器创建新的机器映像

一旦创建了一个新的机器映像，那么从这个新的映像启动一个新的服务器将给出在这个实例上已经完成的所有配置。这有助于我们顺利完成部署过程。这也有助于快速扩展我们的服务。

Hashicorp Packer 是一个轻量级、易于使用的工具，它可以自动为多种平台创建任何类型的机器映像。Packer 不是 Ansible 这样的配置管理工具的替代品。Packer 与 ansible 等工具一起工作，在创建映像时安装软件。Packer 使用配置文件创建机器映像。Packer 使用构建器的概念来启动实例，运行供应器来配置应用程序或服务。安装完成后，它会关闭实例并保存新的烘焙机器实例以及任何所需的后处理。Packer 只构建图像。

使用封隔器的优点:

1.  快速基础架构部署:机器映像使我们能够更快地启动配置好的机器。
2.  可扩展:Packer 在映像创建期间为机器安装和配置所有软件，因此我们不需要运行任何配置管理工具。使用相同的 AMI，我们可以生成任意数量的实例，而无需进行额外的配置。
3.  多提供商支持:Packer 可用于为多个云提供商创建图像，如 AWS、GCP、数字海洋等。
4.  提供可测试性:机器映像可以被测试以验证它们正在工作。

使用封隔器的缺点:

1.  可管理性:packer 不提供 AMI 可管理性。您需要自己使用标签或版本来管理它们。继续删除旧的未使用的 ami。

封隔器配置文件有三个主要部分:

**用户变量**:用户变量用于参数化 packer 配置文件模板，这样我们就可以将机密、环境变量和其他参数排除在模板之外。它们有助于配置文件的移植。它们有助于分离出可以在模板中修改的部分。变量可以通过命令行、环境变量、Vault 或文件传递。变量部分是一个键值映射，变量名分配给一个默认值。

**构建器**:构建器部分包含打包器用来生成机器映像的构建器列表。构建器负责创建一个实例并从中生成机器映像。一个构建器映射到一个机器映像。构建器包含的信息包括作为构建器名称的类型、构建器连接到 AWS 等平台所需的访问密钥。

**Provisioners**:Provisioners 部分包含一个 Provisioners 列表，packer 在创建机器映像之前使用它来安装和配置运行实例中的软件。置备程序是可选的。置备程序包含指定置备程序名称的类型，如 Shell、Ansible 等。

让我们通过一个例子来理解，在 AWS 提供的基础 Amazon Linux 2 AMI 之上安装 nginx 来创建一个新的 AMI。我们将在用户变量部分移动所有将来可以更改的参数。我们将使用环境变量作为访问和密钥。我们使用用户变量部分中的 env 函数将默认值设置为环境变量值。

```
"aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
"aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
```

其他变量用默认值定义。您可以根据需要更改这些值。

```
"variables": {
    "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
    "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
    "region": "us-east-1",
    "ssh_username": "ec2-user",
    "base_ami": "ami-0e6d2e8684d4ccb3e",
    "instance_type": "t2.micro"
}
```

我们将生成 EBS 支持的机器映像，因此需要在构建器部分将类型定义为“amazon-ebs”。因为我们要创建单个 AMI，所以这里只有一个 JSON 部分。需要定义支持新图像所需的其他参数，如源 AMI 名称、访问密钥、密钥、区域等。

```
"builders": [
    {
      "type": "amazon-ebs",
      "access_key": "{{user `aws_access_key`}}",
      "secret_key": "{{user `aws_secret_key` }}",
      "region": "{{user `region` }}",
      "source_ami": "{{user `base_ami`}}",
      "instance_type": "{{user `instance_type` }}",
      "ssh_username": "{{user `ssh_username`}}",
      "ami_name": "packer-base-{{timestamp}}",
      "associate_public_ip_address": true
    }
]
```

一旦构建器准备好了，我们就跳到供应器来定义需要运行什么脚本或配置管理。我们将运行 shell 命令来安装 nginx。在 Amazon Linux 2 中，我们首先需要使用 amazon-linux-extras 来启用 nginx。一旦启用，我们就可以使用 yum 命令安装并安装 nginx。

```
"provisioners": [
    {
      "type": "shell",
      "inline": [
        "echo ENABLE NGINX",
        "sudo amazon-linux-extras enable nginx1.12",
        "echo INSTALL NGINX",
        "sudo yum -y install nginx",
        "echo START NGINX",
        "sudo systemctl start nginx"
      ]
    }
]
```

我们需要使用 validate 命令来验证我们的 packer 文件是否有效。

```
packer validate example.json
```

我们需要使用 build 命令执行 packer JSON 文件来生成一个新的 AWS AMI。

```
packer build example.json
```

如果在 provisioner 文件的执行过程中出错，我们最终会得到一条错误消息。

```
**Build 'amazon-ebs' errored: Script exited with non-zero exit status: 1**
```

如果一切正常，那么我们会得到下面提到的消息。

```
**amazon-ebs output will be in this color.****==> amazon-ebs: Prevalidating AMI Name: packer-base-1539107879**amazon-ebs: Found Image ID: ami-0e6d2e8684d4ccb3e**==> amazon-ebs: Creating temporary keypair: packer_5bbcec27-767b-384c-c5d7-c047ebc099c6****==> amazon-ebs: Creating temporary security group for this instance: packer_5bbcec2a-37e9-602c-9fdc-fd504f4d17a1****==> amazon-ebs: Authorizing access to port 22 from 0.0.0.0/0 in the temporary security group...****==> amazon-ebs: Launching a source AWS instance...****==> amazon-ebs: Adding tags to source instance**amazon-ebs: Adding tag: "Name": "Packer Builder"amazon-ebs: Instance ID: i-0b738eeecacf78658**==> amazon-ebs: Waiting for instance (i-0b738eeecacf78658) to become ready...****==> amazon-ebs: Using ssh communicator to connect: 54.80.150.83****==> amazon-ebs: Waiting for SSH to become available...****==> amazon-ebs: Connected to SSH!****==> amazon-ebs: Provisioning with shell script: /var/folders/bz/90_0_cjj71b9vmsl6qsp8dwc0000gp/T/packer-shell802152345**amazon-ebs: ENABLE NGINXamazon-ebs:   0  ansible2                 available    [ =2.4.2  =2.4.6 ]amazon-ebs:   1  emacs                    available    [ =25.3 ]amazon-ebs:   2  httpd_modules            available    [ =1.0 ].................
Some More Message
.................amazon-ebs:   nginx-mod-mail.x86_64 1:1.12.2-1.amzn2.0.2amazon-ebs:   nginx-mod-stream.x86_64 1:1.12.2-1.amzn2.0.2amazon-ebs:   stix-fonts.noarch 0:1.1.0-5.amzn2amazon-ebs:amazon-ebs: Complete!amazon-ebs: START NGINX**==> amazon-ebs: Stopping the source instance...**amazon-ebs: Stopping instance, attempt 1**==> amazon-ebs: Waiting for the instance to stop...****==> amazon-ebs: Creating unencrypted AMI packer-base-1539107879 from instance i-0b738eeecacf78658**amazon-ebs: AMI: ami-0c185c9c51c96be52**==> amazon-ebs: Waiting for AMI to become ready...****==> amazon-ebs: Terminating the source AWS instance...****==> amazon-ebs: Cleaning up any extra volumes...****==> amazon-ebs: No volumes to clean up, skipping****==> amazon-ebs: Deleting temporary security group...****==> amazon-ebs: Deleting temporary keypair...****Build 'amazon-ebs' finished.**==> Builds finished. The artifacts of successful builds are:--> amazon-ebs: AMIs were created:us-east-1: ami-0c185c9c51c96be52
```

一旦 packer 文件被成功执行，我们就会在最后打印出新生成的 AMI 名称。在这种情况下，我们最新的 AMI 名称是“ami-0c185c9c51c96be52”。

希望这篇文章能帮助你找到足够的动机来使用 packer 来支持 AMI 使用代码。

完整的代码可以在这个 git 资源库中找到:[https://github.com/MiteshSharma/PackerAmi](https://github.com/MiteshSharma/PackerAmi)

***PS:如果你喜欢这篇文章，请鼓掌支持。欢呼***