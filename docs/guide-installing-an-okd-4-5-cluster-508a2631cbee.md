# 指南:安装 OKD 4.5 集群

> 原文：<https://itnext.io/guide-installing-an-okd-4-5-cluster-508a2631cbee?source=collection_archive---------0----------------------->

***2020 年 7 月 29 日更新***

[OKD](https://okd.io/) 是红帽 OpenShift 容器平台(OCP)的上游和社区支持版本。OpenShift 将 vanilla Kubernetes 扩展成一个为企业大规模使用而设计的应用程序平台。从 [OpenShift 4](https://blog.openshift.com/introducing-red-hat-openshift-4/) 发布开始，默认的操作系统是 [Red Hat CoreOS](https://docs.openshift.com/container-platform/4.1/architecture/architecture-rhcos.html) ，它提供了不可变的基础设施和自动更新。 [Fedora CoreOS](https://getfedora.org/en/coreos/) 和 OKD 一样，是红帽 CoreOS 的上游版本。

![](img/4fa96facb049f8009c242e85a0c37ae0.png)

通过使用 OKD 和 FCOS 的开源上游组合(Fedora CoreOS)来构建您的集群，您可以在您的家庭实验室中体验 OpenShift。如今，运行 OKD 集群的家庭实验室的二手硬件相对便宜([$ 250-$ 350](https://www.ebay.com/sch/i.html?_from=R40&_trksid=m570.l1313&_nkw=server+96gb&_sacat=0&LH_TitleDesc=0&_osacat=0&_odkw=server+96gb))，尤其是与每月花费超过 250 美元的云托管解决方案相比。

本指南的目的是帮助您成功构建 OKD 4.5 集群，以便您能够开发并学习使用 OpenShift 和 OKD。

**注意:**在本指南中，我使用 VMWare ESXi 作为虚拟机管理程序，但您也可以使用 Hyper-V、libvirt、Proxmox、VirtualBox，甚至裸机。

# 虚拟机概述:

对于我的安装，我使用了一台 ESXi 6.5 主机，具有 96GB 的 RAM 和一个为 OKD 配置的独立 VLAN。以下是虚拟机的分类:

![](img/e8a9b172e2785ee161441a0686a56b6e.png)![](img/2996252f784e339aa8bb0f743b2bb007.png)

# 在 VMWare 中为 OKD 创建一个新网络:

登录您的 VMWare 主机。选择网络→端口组→添加端口组。在一个未使用的 VLAN 上建立一个 OKD 网络，在我的例子中是 VLAN 20。

![](img/86d12b85659b874f456a61f1d9d3bbac.png)

命名您的群并设置您的 VLAN ID。

![](img/c0dc2d55650c1d89701350f2100937b9.png)

# 创建 ok D4-服务虚拟机:

[下载](https://www.centos.org/download/)CentOS 8 ISO DVD。示例:[CentOS-8 . 2 . 2004-x86 _ 64-DVD 1 . iso](http://isoredirect.centos.org/centos/8/isos/x86_64/CentOS-8.2.2004-x86_64-dvd1.iso)并将其上传到您的 ESXi 主机数据存储。

![](img/7b9bde61b72a897a5b0ad0062d905022.png)

创建新的虚拟机。选择来宾操作系统为 *Linux* ，选择 CentOS 7 (64 位)。

![](img/d789a3186e6730d67ac5ffbde3e5732d.png)

选择数据存储，并将设置自定义为 4 个 vCPU、4GB RAM、100GB HD。向 OKD 网络添加第二个网络适配器。附上 CentOS ISO。

![](img/56b11e1d92425b272a6378965b391bf5.png)

检查您的设置，然后单击完成。

![](img/80065e283b7f477b217677d41968f79e.png)

# 在 okd4-services 虚拟机上安装 CentOS 8:

运行 CentOS 8 安装。

我更喜欢使用“标准分区”存储配置，而不为/home 安装存储。在“安装目标”页面上，单击存储配置下的自定义，然后单击完成。

![](img/edb4ad4e550791dc36076fad46f028ca.png)

在手动分区页面上，选择标准分区，然后单击“单击此处自动创建它们”

![](img/60ea9aad8629b98785470e71117f2bbe.png)

选择“/home”分区，然后单击“-”将其删除。

![](img/46eb8edc040ff29ac1eabcb87fc9b1f8.png)

删除“Desired Capacity”字段的内容，使其为空，并单击 Done，然后接受更改。

![](img/f5141a326619294e82b74ab874098e86.png)

对于软件选择，使用带 GUI 的服务器并添加来宾代理。

![](img/0e040b207cb7a9bb3064b019c17c8e8c.png)

启用连接到虚拟机网络的 NIC，并将主机名设置为 okd4-services，然后单击 Apply 和 Done。

![](img/21b3a1876d34076fe0ee79074c1abd10.png)

单击“开始安装”开始安装。

![](img/002f4de0b8f39efdb3e4891e8afeaadf.png)

设置 Root 密码，并创建一个管理员用户。

![](img/c729f813ed3063a4390f86b77cb5cec6.png)

安装完成后，登录并更新操作系统。

```
sudo dnf install -y epel-release
sudo dnf update -y
sudo systemctl restart
```

为从家庭网络远程访问设置 XRDP

```
sudo dnf install -y xrdp tigervnc-server
sudo systemctl enable --now xrxp
sudo firewall-cmd --zone=public --permanent --add-port=3389/tcp
sudo firewall-cmd --reload
```

下载 Google Chrome rpm 并和 git 一起安装

![](img/47c9c0385a5eaa7da7ea202cce3596d1.png)

```
sudo dnf install -y ~/Downloads/google-chrome-stable_current_x86_64.rpm git
```

# 创建 okd4-pfsense 虚拟机:

[下载 pfSense ISO](https://www.pfsense.org/download/) 并将其上传到您的 ESXi 主机的数据存储区。

![](img/a5df5f696877e3b00c7d6e3cc2acc2ea.png)

创建新的虚拟机。选择 Guest OS 为 *Other* ，选择 FreeBSD 64 位。

![](img/8326e6d8cd50564fa8da3a1f632d9317.png)

使用资源的默认模板设置。

为网络适配器 1 选择您的家庭网络，并使用 OKD 网络添加新的网络适配器。

![](img/9dba9979a7cc204f81b5ec72de841b5e.png)

# 设置 okd4-pfsense 虚拟机:

启动 pfSense 虚拟机，并使用所有默认值运行安装。完成后，您的虚拟机控制台应该如下所示:

![](img/5a49406e302c5c9596d83751393f2e9d.png)

通过 okd4-services 虚拟机上的网络浏览器登录到 pfSense。默认用户名是“admin”，密码是“pfsense”。

![](img/801d5a6918930e9a4733d3145f3af407.png)

登录后，单击 next，对主机名使用“okd4-pfsense ”,对域使用“okd.local ”,并将 192.168.1.210 添加为主 DNS 服务器。

![](img/a0eaeb132ea35e058df33d4f97d9020c.png)

选择您的时区。下一个。

使用 WAN 配置的默认值。取消选中“阻止 RFC1918 专用网络”,因为您的家庭网络是此设置中的“WAN”。下一个。

![](img/4e56e5648ceaf0c0c7a82e141a19b767.png)

使用默认的局域网 IP 和子网掩码。在下一个屏幕上设置管理员密码。

![](img/ba35feac4df46d6cdb8bce66c9c9704d.png)

# 创建引导节点、主节点和工作节点:

下载 [Fedora CoreOS 裸机 ISO](https://getfedora.org/coreos/download/) 并上传到您的 ESXi 数据存储。

撰写本文时最新的稳定版本是 32.20200715.3.0

![](img/df006a1c1eb60fc8d2cbdc8d71101fc0.png)

使用本文开头的电子表格中的值，在您的 ESXi 主机上创建六个 ODK 节点(引导、主、工作节点):

![](img/f58e40d8950ff72c6cc971206ec0fe0f.png)![](img/4907b117541001da98f1d49b1c52c181.png)

您应该最终拥有以下虚拟机:

![](img/900f254cfb0363dc07e8b026589b6141.png)

# 设置 DHCP 保留:

通过查看虚拟机的硬件配置，编译 OKD 节点 MAC 地址列表。

![](img/dd69b0b6e31494d02394d04f06df7d8f.png)

登录 pfSense。转到服务→ DHCP 服务器，将您的结束范围 IP 更改为 192.168.1.99，并将主 DNS 服务器设置为 192.168.1.210，然后单击保存。

![](img/d18989cfa39af97c952ea2dc1336f69b.png)

在 DHCP 服务器上，单击页面底部的添加。

![](img/d14139b07b66e1184dec6f91789e1142.png)

填写 MAC 地址、IP 地址和主机名，然后单击保存。为每个 ODK 虚拟机执行此操作。同时，在 okd 网络中加入 okd4-services MAC。完成后，单击页面顶部的应用更改。S

![](img/73c388d9778236a0e09efa721e9f0868.png)

# 配置 ok D4-服务虚拟机以托管各种服务:

okd 4-服务虚拟机用于提供 DNS、NFS 导出、web 服务器和负载平衡。

在虚拟机硬件配置页面上复制连接到 OKD 网络的 NIC 的 MAC 地址，并使用 IP 地址 192.168.1.210 为该虚拟机设置 DHCP 预留。

![](img/5169e409150ef67d73a35a1e067b9066.png)

完成后点击 DHCP 页面顶部的“应用更改”。

![](img/bc5e1d20a399583cde75844fa78a78b3.png)

在 okd4-services 虚拟机上打开一个终端，克隆包含 DNS、HAProxy 和 install-conf.yaml 示例文件的 okd4_files repo:

```
cd
git clone [https://github.com/cragr/okd4_files.git](https://github.com/CountPickering/okd4_files.git)
cd okd4_files
```

# 安装绑定(DNS)

```
sudo dnf -y install bind bind-utils
```

复制命名的配置文件和区域:

```
sudo cp named.conf /etc/named.conf
sudo cp named.conf.local /etc/named/
sudo mkdir /etc/named/zones
sudo cp db* /etc/named/zones
```

启用并启动命名:

```
sudo systemctl enable named
sudo systemctl start named
sudo systemctl status named
```

创建防火墙规则:

```
sudo firewall-cmd --permanent --add-port=53/udp
sudo firewall-cmd --reload
```

将连接到虚拟机网络(非 okd)的 okd4 服务 NIC 上的 DNS 更改为 127.0.0.1。

![](img/7eb6c36f758afa2d4d6878ef73b8b086.png)

在 okd4-services 虚拟机上重新启动网络服务:

```
sudo systemctl restart NetworkManager
```

在 okd4-services 上测试 DNS。

```
dig okd.local
dig –x 192.168.1.210
```

DNS 正常工作时，您应该会看到以下结果:

![](img/80181fcb069c475a3bc040d8269a3c5c.png)![](img/a32159a6a09900ce9fac31341a48899e.png)

# 安装 HAProxy:

```
sudo dnf install haproxy -y
```

从 git okd4_files 目录中复制 haproxy 配置:

```
sudo cp haproxy.cfg /etc/haproxy/haproxy.cfg
```

启动、启用和验证 HA 代理服务:

```
sudo setsebool -P haproxy_connect_any 1
sudo systemctl enable haproxy
sudo systemctl start haproxy
sudo systemctl status haproxy
```

添加 OKD 防火墙端口:

```
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=22623/tcp
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

# 安装阿帕奇/HTTPD

```
sudo dnf install -y httpd
```

将 httpd 监听端口更改为 8080:

```
sudo sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf
```

在防火墙上启用并启动 httpd 服务/允许端口 8080:

```
sudo setsebool -P httpd_read_user_content 1
sudo systemctl enable httpd
sudo systemctl start httpd
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

测试 web 服务器:

```
curl localhost:8080
```

成功的卷发应该是这样的:

![](img/0cadbbbd27fe4de9b78555da00b636cb.png)

# 恭喜你，你已经成功了一半！

恭喜你。现在，您应该有了一个单独的家庭实验室环境设置，并为 ODK 做好了准备。现在我们可以开始安装了。

# 下载 openshift-installer 和 oc 客户端:

到 okd 4-服务虚拟机的 SSH

要下载最新的 oc 客户端和 openshift-install 二进制文件，您需要使用 oc 客户端的现有版本。

从 [OKD 发布页面](https://github.com/openshift/okd/releases)下载 4.5 版本的 oc 客户端并打开 shift-install。示例:

```
cd
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-07-29-070316/openshift-client-linux-4.5.0-0.okd-2020-07-29-070316.tar.gz
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-07-29-070316/openshift-install-linux-4.5.0-0.okd-2020-07-29-070316.tar.gz
```

解压 oc 客户端的 okd 版本，打开 shift-install:

```
tar -zxvf openshift-client-linux-4.5.0-0.okd-2020-07-29-070316.tar.gz
tar -zxvf openshift-install-linux-4.5.0-0.okd-2020-07-29-070316.tar.gz
```

将 kubectl、oc 和 openshift-install 移动到/usr/local/bin 并显示版本:

```
sudo mv kubectl oc openshift-install /usr/local/bin/
oc version
openshift-install version
```

最新版本可在 https://origin-release.svc.ci.openshift.org[获得](https://origin-release.svc.ci.openshift.org/)

# 设置 openshift-installer:

在 install-config.yaml 中，您可以使用来自 RedHat 的 pull-secret 或默认的" { " auths ":{ " fake ":{ " auth ":" bar " } } "作为 pull-secret。

如果您还没有 SSH 密钥，请生成一个。

```
ssh-keygen
```

创建安装目录并复制 install-config.yaml 文件:

```
cd
mkdir install_dir
cp okd4_files/install-config.yaml ./install_dir
```

在 install_dir 中编辑 install-config.yaml，插入您的 pull secret 和 ssh 密钥，并备份 install-config.yaml，因为它将在下一步中被删除:

```
vim ./install_dir/install-config.yaml
cp ./install_dir/install-config.yaml ./install_dir/install-config.yaml.bak
```

为集群生成 Kubernetes 清单，忽略警告:

```
openshift-install create manifests --dir=install_dir/
```

修改 cluster-scheduler-02-config . YAML 清单文件，以防止在控制平面计算机上调度 pod:

```
sed -i 's/mastersSchedulable: true/mastersSchedulable: False/' install_dir/manifests/cluster-scheduler-02-config.yml
```

现在，您可以创建点火配置:

```
openshift-install create ignition-configs --dir=install_dir/
```

**注意:**如果重用 install_dir，确保它是空的。隐藏文件是在生成配置后创建的，在第二次尝试使用同一文件夹之前，应该删除这些文件。

# 在 web 服务器上托管点火和 Fedora CoreOS 文件:

在/var/www/html 中创建 okd4 目录:

```
sudo mkdir /var/www/html/okd4
```

将 install_dir 内容复制到/var/www/html/okd4 并设置权限:

```
sudo cp -R install_dir/* /var/www/html/okd4/
sudo chown -R apache: /var/www/html/
sudo chmod -R 755 /var/www/html/
```

测试 web 服务器:

```
curl localhost:8080/okd4/metadata.json
```

下载 [Fedora CoreOS](https://getfedora.org/coreos/download/) 裸机 bios 镜像和 sig 文件，并缩短文件名:

```
cd /var/www/html/okd4/
sudo wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/32.20200715.3.0/x86_64/fedora-coreos-32.20200715.3.0-metal.x86_64.raw.xz
sudo wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/32.20200715.3.0/x86_64/fedora-coreos-32.20200715.3.0-metal.x86_64.raw.xz.sig
sudo mv fedora-coreos-32.20200715.3.0-metal.x86_64.raw.xz fcos.raw.xz
sudo mv fedora-coreos-32.20200715.3.0-metal.x86_64.raw.xz.sig fcos.raw.xz.sig
sudo chown -R apache: /var/www/html/
sudo chmod -R 755 /var/www/html/
```

# 启动引导节点:

打开 odk4 引导虚拟机的电源。按 TAB 键编辑内核引导选项，并添加以下内容:

```
coreos.inst.install_dev=/dev/sda coreos.inst.image_url=http://192.168.1.210:8080/okd4/fcos.raw.xz coreos.inst.ignition_url=http://192.168.1.210:8080/okd4/**bootstrap.ign**
```

![](img/764dcb07d8b5362a8718c04a1b681402.png)

您应该看到 fcos.raw.gz 图像和签名正在下载:

![](img/b607f38cd40f7e80eea71ecc1a91b89d.png)

# 启动控制平面节点:

打开控制面板节点的电源，按 TAB 键编辑内核引导选项并添加以下内容，然后按 enter 键:

```
coreos.inst.install_dev=/dev/sda coreos.inst.image_url=http://192.168.1.210:8080/okd4/fcos.raw.xz coreos.inst.ignition_url=http://192.168.1.210:8080/okd4/**master.ign**
```

![](img/1fdb80239cd2fa9332982b6e1b39cf20.png)

您应该看到 fcos.raw.gz 图像和签名正在下载:

![](img/3aec62b87a1dea48fd7c4a7f98c7c9da.png)

# 启动计算节点:

打开控制面板节点的电源，按 TAB 键编辑内核引导选项并添加以下内容，然后按 enter 键:

```
coreos.inst.install_dev=/dev/sda coreos.inst.image_url=http://192.168.1.210:8080/okd4/fcos.raw.xz coreos.inst.ignition_url=http://192.168.1.210:8080/okd4/**worker.ign**
```

![](img/c3b943c333f39556624880e3fbb7e496.png)

您应该看到 fcos.raw.gz 图像和签名正在下载:

![](img/18ef70c85dbf1f05dc2bc74d1f90c717.png)

在引导过程完成之前，工作节点通常会显示以下内容:

![](img/c69a0a60291e74574cbe8dc95caab5b5.png)

# 监控引导安装:

您可以从 okd4-services 节点监控引导过程:

```
openshift-install --dir=install_dir/ wait-for bootstrap-complete --log-level=info
```

![](img/f39a6a7922c6b7bf848e9106ca61ff6c.png)

引导过程可能需要 30 分钟以上，完成后，您可以关闭引导节点。现在是编辑/etc/haproxy/haproxy.cfg、注释掉引导节点并重新加载 haproxy 服务的好时机。

```
sudo sed '/ okd4-bootstrap /s/^/#/' /etc/haproxy/haproxy.cfg
sudo systemctl reload haproxy
```

# 登录集群并批准 CSR:

现在主服务器已经联机，您应该能够登录 oc 客户端了。使用以下命令登录并检查集群的状态:

```
export KUBECONFIG=~/install_dir/auth/kubeconfig
oc whoami
oc get nodes
oc get csr
```

![](img/0a4f18af736477c9bb0764ce34c1f8df.png)

您应该只看到主节点和几个等待批准的 CSR。安装 jq 包以帮助一次批准多个 CSR。

```
wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
chmod +x jq
sudo mv jq /usr/local/bin/
jq --version
```

![](img/cefa2d55eba4127fb54913d59cf90b6e.png)

批准所有待定证书并检查您的节点:

```
oc get csr -ojson | jq -r '.items[] | select(.status == {} ) | .metadata.name' | xargs oc adm certificate approve
```

![](img/529f987aa3ac7903c44159fb614a8eea.png)

检查集群操作符的状态。

```
oc get clusteroperators
```

![](img/63f52d4ee946a3a95246f8ae8b6f6cb1.png)

控制台刚刚在我上面的图片中可用。从 install_dir/auth 文件夹中获取您的 kubeadmin 密码，然后登录到 web 控制台:

```
cat install_dir/auth/kubeadmin-password
```

![](img/959d9e5e9611d75cad5b0e77e9e61fec.png)

打开您的网络浏览器到 https://console-openshift-console.apps.lab.okd.local/的，使用上面的密码以 kubeadmin 的身份登录:

![](img/3b5a7ea17970ec269fcb0d42b62f6790.png)

群集状态可能仍然显示正在升级，并且它会继续完成安装。

![](img/b0c8082b23b5bbd3afbf37d7283c9d5d.png)

# 持久存储:

在我们完成这个项目之前，我们需要为我们的注册表创建一些持久存储。让我们将 okd4-services VM 配置为 NFS 服务器，并将其用于持久存储。

登录到您的 okd4-services 虚拟机，并开始设置一个 NFS 服务器。以下命令安装必要的包，启用服务，并配置文件和文件夹权限。

```
sudo dnf install -y nfs-utils
sudo systemctl enable nfs-server rpcbind
sudo systemctl start nfs-server rpcbind
sudo mkdir -p /var/nfsshare/registry
sudo chmod -R 777 /var/nfsshare
sudo chown -R nobody:nobody /var/nfsshare
```

创建 NFS 导出

在新的/etc/exports 文件“/var/nfsshare 192.168.1.0/24(rw，sync，no_root_squash，no_all_squash，no_wdelay)”中添加这一行

```
echo '/var/nfsshare 192.168.1.0/24(rw,sync,no_root_squash,no_all_squash,no_wdelay)' | sudo tee /etc/exports
```

![](img/7b667dc56abc7861fd40bb51ba8d96f5.png)

重新启动 nfs 服务器服务并添加防火墙规则:

```
sudo setsebool -P nfs_export_all_rw 1
sudo systemctl restart nfs-server
sudo firewall-cmd --permanent --zone=public --add-service mountd
sudo firewall-cmd --permanent --zone=public --add-service rpc-bind
sudo firewall-cmd --permanent --zone=public --add-service nfs
sudo firewall-cmd --reload
```

# 注册表配置:

在 NFS 共享上创建一个永久卷。使用 git repo 中 okd4_files 文件夹中的 registry_py.yaml:

```
oc create -f okd4_files/registry_pv.yaml
oc get pv
```

![](img/f3e6fe0c62f5414a5fd941f3f84406a1.png)

编辑图像注册表操作符:

```
oc edit configs.imageregistry.operator.openshift.io
```

将管理状态从“已删除”更改为“已管理”。在存储下:添加 pvc:和 claim: blank 以附加 PV 并自动保存您的更改:

```
managementState: Managedstorage:
    pvc:
      claim:
```

![](img/248ee86b150163a81b82c723c8978ee7.png)

检查您的持久卷，现在应该声明:

```
oc get pv
```

![](img/efa7b77c8f85f133ac92c4201c2c41b5.png)

检查导出大小，它应该为零。在下一节中，我们将推送至注册表，文件大小不应为零。

```
du -sh /var/nfsshare/registry
```

![](img/4a118c67e2474b91b6340cea67fbf314.png)

在下一节中，我们将创建一个 WordPress 项目并将其推送到注册表中。推送后，NFS 导出应该显示 200+ MB。

# 创建 WordPress 项目:

创建一个新的 OKD 项目。

```
oc new-project wordpress-test
```

![](img/d8055d74234add128be7e46dde6f194b.png)

使用 docker hub 中的 centos php73 s2i 映像创建一个新应用程序，并使用 WordPress GitHub repo 作为源代码。公开服务以创建路由。

```
oc new-app centos/php-73-centos7~[https://github.com/WordPress/WordPress.git](https://github.com/WordPress/WordPress.git)
oc expose svc/wordpress
```

![](img/cf83a28f8a4678da6e7c9152d86afc3f.png)

使用 centos7 MariaDB 映像和一些环境变量创建一个新应用程序:

```
oc new-app centos/mariadb-103-centos7 --name mariadb --env MYSQL_DATABASE=wordpress --env MYSQL_USER=wordpress --env MYSQL_PASSWORD=wordpress
```

![](img/a40724e03a20faeec646b7cee2e9ae0b.png)

打开 OpenShift 控制台，浏览到 WordPress-test 项目。一旦 WordPress 图像被构建并准备好，它将是深蓝色的，就像 MariaDB 实例所示:

![](img/a1d1c6df8e7837d4b2f449b3e27cf28d.png)

点击 WordPress 对象，然后点击路线，在你的网络浏览器中打开它:

![](img/8e4deecde44549f4cfefd874861a9913.png)

你应该看到 WordPress 设置配置，点击让我们开始。

![](img/e2e15c75c84aaea8e573138b57ef19f4.png)

如图所示，填写数据库、用户名、密码和数据库主机，并运行安装程序:

![](img/de63ec776c976a0100d76c5f5ff7c7cc.png)

填写欢迎信息并点击安装 WordPress。

![](img/27756e61a87508aa12563882d9e97614.png)

登录，你应该有一个工作的 WordPress 安装:

![](img/bb34dfe7037d5ba378b7a4694e639968.png)![](img/c443b5692a505ac52bca71dc1aa36ce4.png)![](img/9ecdba524ff6e6faa6c45082b9a4d8a1.png)

在 okd4-services 虚拟机上检查 NFS 导出的大小。大小应该在 300MB 左右。

```
du -sh /var/nfsshare/registry/
```

![](img/db131b2b8fd0989a5807970f65c469f1.png)

您刚刚验证了您的永久卷正在工作。

# HTPasswd 安装程序:

kubeadmin 是一个临时用户。设置本地用户最简单的方法是使用 htpasswd。

```
cd
cd okd4_files
htpasswd -c -B -b users.htpasswd testuser testpassword
```

![](img/cc5a74825b6eaac21670a81f151ca511.png)

使用您生成的 users.htpasswd 文件在 openshift-config 项目中创建一个秘密:

```
oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd -n openshift-config
```

添加身份提供者。

```
oc apply -f htpasswd_provider.yaml
```

![](img/01a70f2d662788df3e70708db762fa95.png)

注销 OpenShift 控制台。然后选择 htpasswd_provider，用 testuser 和 testpassword 凭证登录。

![](img/54b420490ae5fab4a84781d5976b03aa.png)![](img/e6f422237c2acb3d2caf1c7c2fb406c7.png)

如果您访问管理员页面，您应该看不到任何项目:

![](img/e9c5c290d57f0b47e080522d5e850cb1.png)

授予您自己集群管理权限，项目应该立即填充:

```
oc adm policy add-cluster-role-to-user cluster-admin testuser
```

您的用户现在应该拥有集群管理员级别的访问权限:

![](img/1523d0ccc6c0234b045e82e5f0af8b5e.png)

# 恭喜你。您已经创建了一个 OKD 集群！

希望您已经创建了一个 OKD 集群，并在这个过程中学到了一些东西。至此，您应该有了修补 OpenShift 继续学习的良好基础。

这里有一些资源可以帮助你的旅程:

要报告问题，请使用[https://github.com/openshift/okd Github Repo:](https://github.com/openshift/okd)T2

如需支持，请查看 [k8s Slack](https://slack.k8s.io/) 上的#openshift-users 频道

OKD 工作组每两周召开一次会议，讨论发展和下一步措施。会议日程和地点在[open shift/社区报告](https://github.com/openshift/community/projects/1#card-28309038)中记录。

okd-wg 的谷歌小组:[https://groups.google.com/forum/#!forum/okd-wg](https://groups.google.com/forum/#!forum/okd-wg)