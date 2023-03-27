# 代码形式的基础设施:使用 Pulumi 提供和引导 GCP 实例

> 原文：<https://itnext.io/infrastructure-as-code-using-pulumi-to-provision-and-bootstrap-a-gcp-instance-318f06c61a03?source=collection_archive---------4----------------------->

![](img/7b1305e7604ec5f48e973c107b1ce382.png)

这篇文章将向您展示如何使用 [Pulumi](https://www.pulumi.com/docs/index.html) 作为在 Google 云平台中提供实例的基础设施即代码工具。对于 Pulumi 代码示例，我将使用 Python。我在 MacOS 上运行了下面的例子。

我使用`brew`来安装 pulumi CLI:

```
$ brew install pulumi
```

如果已经安装了 pulumi，可以将其升级到最新版本:

```
$ brew upgrade pulumi
```

下一步是下载 [gcloud SDK 和 CLI](https://cloud.google.com/sdk/) ，并按照此处的设置说明进行设置:

[https://www . pulumi . com/docs/intro/cloud-providers/GCP/setup/](https://www.pulumi.com/docs/intro/cloud-providers/gcp/setup/)

```
**$ gcloud auth login** You are now logged in as [myuser@example.com].Your current project is [myproject]. You can change this setting by running:$ gcloud config set project PROJECT_ID**$ gcloud auth application-default login** Credentials saved to file: [/Users/ggheo/.config/gcloud/application_default_credentials.json]These credentials will be used by any library that requestsApplication Default Credentials.To generate an access token for other uses, run:gcloud auth application-default print-access-token
```

然后我在 GCP 为 Python 创建了一个新的 Pulumi 项目。我将这个项目命名为`gcpinfra`，并保留了默认的栈名`dev`。

```
**$ pulumi new gcp-python** This command will walk you through creating a new Pulumi project.Enter a value or leave blank to accept the (default), and press <ENTER>.Press ^C at any time to quit.**project name**: (**gcpinfra**)**project description**: (A minimal Google Cloud Python Pulumi program) **Provision GCP infrastructure resources****Created project ‘gcpinfra’**Please enter your desired stack name.To create a stack in an organization, use the format <org-name>/<stack-name> (e.g. `acmecorp/dev`).**stack name**: (dev) **dev****Created stack ‘dev’****project: The Google Cloud project to deploy into: myproject**Saved configYour new project is ready to go! ✨To perform an initial deployment, run the following commands:1\. virtualenv -p python3 venv2\. source venv/bin/activate3\. pip3 install -r requirements.txtThen, run ‘pulumi up’
```

我创建并激活了一个基于 Python 3 的虚拟环境，然后通过`pip3 install -r requirements.txt`安装了 Pulumi 需求:

```
**$ virtualenv -p /usr/local/bin/python3 venv** Running virtualenv with interpreter /usr/local/bin/python3Using base prefix ‘/usr/local/Cellar/python/3.7.4/Frameworks/Python.framework/Versions/3.7’
New python executable in /Users/ggheo/code/mycode/codepraxis/mymooc/pulumi/gcpinfra/venv/bin/python3.7
Also creating executable in /Users/ggheo/code/pulumi/gcpinfra/venv/bin/python
Installing setuptools, pip, wheel…
done.**$ source venv/bin/activate****$ pip3 install -r requirements.txt**
```

此时，我可以看到创建了以下文件:

```
**$ ls -la**total 40drwxr-xr-x 8 ggheo staff 256 Nov 16 15:53 .drwxr-xr-x 4 ggheo staff 128 Nov 16 15:51 ..
-rw — — — — 1 ggheo staff 12 Nov 16 15:51 .gitignore
-rw-r — r — 1 ggheo staff 35 Nov 16 15:52 Pulumi.dev.yaml
-rw — — — — 1 ggheo staff 78 Nov 16 15:51 Pulumi.yaml
-rw — — — — 1 ggheo staff 203 Nov 16 15:51 __main__.py
-rw — — — — 1 ggheo staff 32 Nov 16 15:51 requirements.txt
drwxr-xr-x 6 ggheo staff 192 Nov 16 15:53 venv
```

Pulumi 中的 gcp-python 示例项目创建了以下 Pulumi 代码:

```
**$ cat __main__.py** import pulumi
from pulumi_gcp import storage
# Create a GCP resource (Storage Bucket)
bucket = storage.Bucket(‘my-bucket’)
# Export the DNS name of the bucket
pulumi.export(‘bucket_name’, bucket.url)
```

我的下一步是设置一系列 [Pulumi 配置变量](https://www.pulumi.com/docs/intro/concepts/config/)，我将在我的代码中使用它们:

**GCP 具体:**

```
$ pulumi config set gcp:project myproject
$ pulumi config set gcp:region us-central1
$ pulumi config set gcp:zone us-central1-a
```

**特定应用:**

```
$ pulumi config set instance_name dev
$ pulumi config set instance_type n1-highmem-2
$ pulumi config set instance_image ubuntu-1604-xenial-v20191010
$ pulumi config set instance_disk_size 50
```

然后，我修改了示例 __main__。py 代码，以便它创建一个 GCP 实例，并通过 init/startup 脚本引导它。我的代码基于另一个 [Pulumi 例子](https://github.com/pulumi/examples/blob/master/gcp-py-instance-nginx/__main__.py)。

```
**$ cat __main__.py** import pulumi
from pulumi_gcp import compute
with open(‘init_script.txt’, ‘r’) as init_script:
    data = init_script.read()
script = data
config = pulumi.Config()
instance_name = config.require(‘instance_name’)
instance_type = config.require(‘instance_type’)
instance_image = config.require(‘instance_image’)
instance_disk_size = config.require(‘instance_disk_size’)addr = compute.address.Address(instance_name)network = compute.Network(instance_name)
firewall = compute.Firewall(
  instance_name,
  network=network.self_link,
  allows=[
    {
      “protocol”: “tcp”,
      “ports”: [“22”]
    }
  ]
)instance = compute.Instance(
  instance_name,
  name=instance_name,
  machine_type=instance_type,
  boot_disk={
    “initializeParams”: {
       “image”: instance_image,
       “size”: instance_disk_size
     }
  },
  network_interfaces=[
    {
      “network”: network.id,
      “accessConfigs”: [{“nat_ip”: addr.address}]
    }
  ],
  metadata_startup_script=script,
)# Export the DNS name of the bucket
pulumi.export(“instance_name”, instance.name)
pulumi.export(“instance_network”, instance.network_interfaces)
pulumi.export(“external_ip”, addr.address)
```

注意，我将数据从一个名为 init_script.txt 的文件传递到计算机。实例构造函数，将其赋给变量 metadata_startup_script。

下面是 init 脚本，我在远程实例上添加了我个人的 ssh 公钥，然后安装 docker 和 docker-compose:

```
**$ cat init_script.txt** # add ssh pubkeymkdir -p /home/ggheo/.ssh
chmod 700 /home/ggheo/.ssh
echo “ssh-rsa contents of my ssh public key” >> /home/ggheo/.ssh/authorized_keys
chown -R ggheo:ggheo /home/ggheo/.ssh# install dockersudo apt update
sudo apt install -y make apt-transport-https ca-certificates curl software-properties-common
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo apt-key add -
sudo add-apt-repository “deb [arch=amd64] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) bionic stable”
sudo apt update
sudo apt-cache policy docker-ce
sudo apt install -y docker-ce# download docker-composesudo curl -L [https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname](https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname) -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

为了提供`dev`堆栈，我运行了:

```
**$ pulumi up** Previewing update (dev):Type Name Plan+ pulumi:pulumi:Stack gcpinfra-dev create+ ├─ gcp:compute:Address dev create+ ├─ gcp:compute:Network dev create+ ├─ gcp:compute:Firewall dev create+ └─ gcp:compute:Instance dev createResources:+ 5 to create**Do you want to perform this update? yes**Updating (dev):Type Name Status+ pulumi:pulumi:Stack gcpinfra-dev created+ ├─ gcp:compute:Network dev created+ ├─ gcp:compute:Address dev created+ ├─ gcp:compute:Instance dev created+ └─ gcp:compute:Firewall dev createdOutputs:external_ip : “35.223.75.5”instance_name : “dev”instance_network: [[0]: {accessConfigs : [[0]: {natIp : “35.223.75.5”network_tier : “PREMIUM”}]name : “nic0”network : “https://www.googleapis.com/compute/v1/projects/myproject/global/networks/dev-6de02e1"networkIp : “10.128.0.2”subnetwork : “https://www.googleapis.com/compute/v1/projects/myproject/regions/us-central1/subnetworks/dev-6de02e1"subnetworkProject: “myproject”}]Resources:+ 5 createdDuration: 58sPermalink: [https://app.pulumi.com/griggheo/gcpinfra/dev/updates/6](https://app.pulumi.com/griggheo/gcpinfra/dev/updates/6)
```

我现在能够使用 gcloud CLI 列出新的 GCP 实例和 ssh:

```
**$ gcloud compute instances list** NAME ZONE MACHINE_TYPE PREEMPTIBLE INTERNAL_IP EXTERNAL_IP STATUS
dev us-central1-a n1-highmem-2 10.128.0.2 35.223.75.5 RUNNING**$ gcloud compute ssh dev** Warning: Permanently added ‘compute.1672722981827980594’ (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0–1046-gcp x86_64)
* Documentation: [https://help.ubuntu.com](https://help.ubuntu.com)
* Management: [https://landscape.canonical.com](https://landscape.canonical.com)
* Support: [https://ubuntu.com/advantage](https://ubuntu.com/advantage)
40 packages can be updated.
18 updates are security updates.
The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
**ggheo@dev:~$**
```

此时，我验证了 docker、docker-compose 和 tutor 安装正确。我可以在/var/log/auth.log 中看到 init 脚本安装命令。

我还能够直接通过 ssh 访问远程 GCP 实例，因为我的 ssh 密钥被添加到 authorized_files 中。

作为更新现有 Pulumi 堆栈的一个例子，让我们修改 init 脚本，将我们的用户添加到 docker 组中。我将这一行添加到 init_script.txt 的末尾:

```
usermod -a -G docker ggheo
```

当运行`pulumi up`时，它将替换实例，因为它检测到启动脚本已经改变:

```
**$ pulumi up** Previewing update (dev):**Type Name Plan Info****pulumi:pulumi:Stack gcpinfra-dev****+- └─ gcp:compute:Instance dev replace [diff: ~metadataStartupScript]**Outputs:~ instance_name : “dev” => output<string>- instance_network: [- [0]: {- accessConfigs : [- [0]: {- natIp : “35.223.75.5”- network_tier : “PREMIUM”}]- name : “nic0”- network : “https://www.googleapis.com/compute/v1/projects/myproject/global/networks/dev-6de02e1"- networkIp : “10.128.0.2”- subnetwork : “https://www.googleapis.com/compute/v1/projects/myproject/regions/us-central1/subnetworks/dev-6de02e1"- subnetworkProject: “myproject”}]+ instance_network: output<string>Resources:+-1 to replace4 unchangedDo you want to perform this update? yesUpdating (dev):Type Name Status Infopulumi:pulumi:Stack gcpinfra-dev+- └─ gcp:compute:Instance dev replaced [diff: ~metadataStartupScript]Outputs:external_ip : “35.223.75.5”instance_name : “dev”~ instance_network: [~ [0]: {accessConfigs : [[0]: {natIp : “35.223.75.5”network_tier : “PREMIUM”}]name : “nic0”network : “https://www.googleapis.com/compute/v1/projects/myproject/global/networks/dev-6de02e1"~ networkIp : “10.128.0.2” => “10.128.0.3”subnetwork : “https://www.googleapis.com/compute/v1/projects/myproject/regions/us-central1/subnetworks/dev-6de02e1"subnetworkProject: “myproject”}]Resources:+-1 replaced4 unchangedDuration: 2m12s
```

我的最后一个例子将展示如何通过元数据将值传递给实例来定制远程实例上的环境。例如，当您希望在远程实例的引导过程中创建包含某些自定义值的文件时，这可能会很有用。

下面举例说明如何根据作为元数据传递的自定义值，在远程实例上自定义“当日消息”(又名 MOTD)问候语。

我首先在 pulumi 中设置了一个名为`motdgreeting`的配置变量:

```
**$ pulumi config set motdgreeting ‘Hello from dev instance’**
```

我在 __main__py 中读取这个配置变量的值，并将其作为`metadata`字典中`motdgreeting`键的值传递给`compute.Instance`构造函数:

```
# example of metadata variablemotd_greeting = config.require(‘motdgreeting’)instance = compute.Instance(
  instance_name,
  name=instance_name,
  machine_type=instance_type,
  boot_disk={
    “initializeParams”: {
       “image”: instance_image,
       “size”: instance_disk_size
     }
  },
  network_interfaces=[
    {
      “network”: network.id,
      “accessConfigs”: [{“nat_ip”: addr.address}]
    }
  ],
  metadata_startup_script=script,
  metadata={
    “motdgreeting”: motd_greeting,
  },
)
```

最后，我在 init 脚本中检索了`motdgreeting`元数据变量的值，并使用它来设置 MOTD 消息:

```
MOTD_GREETING=$(curl [http://metadata.google.internal/computeMetadata/v1/instance/attributes/motdgreeting](http://metadata.google.internal/computeMetadata/v1/instance/attributes/motdgreeting) -H “Metadata-Flavor: Google”)
echo “$MOTD_GREETING” > /etc/motd
```

如果您一直在跟踪，现在您已经有了一个在 GCP 运行的实例，您可以 ssh 到它，它已经在运行 docker，您知道如何根据您的需要定制它。如果你不打算使用这个实例，不要忘记删除它，这样你就不会为此付费。为此，您可以运行:

```
**$ pulumi destroy**
```