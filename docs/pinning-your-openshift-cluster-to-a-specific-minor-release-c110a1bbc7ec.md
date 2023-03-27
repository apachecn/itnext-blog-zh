# 将 OpenShift 集群绑定到特定的次要版本

> 原文：<https://itnext.io/pinning-your-openshift-cluster-to-a-specific-minor-release-c110a1bbc7ec?source=collection_archive---------4----------------------->

![](img/6e66317939d882d031d2bd3964cf910c.png)

迈克尔·克里斯滕森在 [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

大约一个月前，Kubernetes 项目发布了一个 [**安全补丁**](https://kubernetes.io/blog/2018/04/04/fixing-subpath-volume-vulnerability/) ，不幸地破坏了我们的 OpenShift 集群。这个问题表现在[无法为容器卷挂载](https://github.com/kubernetes/kubernetes/issues/61076#issuecomment-372554309)创建卷子路径，这最终使我们的管道陷入停顿。

在我们的 3.7.22 集群中配置新节点后，我们就受到了影响，这在过去是无缝运行的，没有任何问题。然而，这一次，要么是新的次要包被破坏，要么是该错误已经进入了所有 OpenShift 3.x rpm 包。

幸运的是，到目前为止，我们没有失去任何支持我们度过这个问题的主要节点。幸运的是，RedHat (3.7.44)最近发布了对该错误的修复，但这一次我们将努力确保我们能够控制应用于集群中节点的 rpm 包版本。

## **强制特定的 OpenShift 包版本。**

根据文档，安装程序使用以下可用清单事实来强制安装特定的软件包版本:

```
[OSEv3:vars]openshift_release=v3.7.44# Value [appended to the yum package](https://github.com/openshift/openshift-ansible/blob/0ffb616c01a37862da73a80bdfd2ffc826ef808a/roles/openshift_cli/tasks/main.yml#L3) install
openshift_pkg_version=-3.7.44# Prevents an unsupported docker version from being installed
enable_docker_excluder=true# Still figuring out [why this is needed](https://github.com/openshift/openshift-ansible/blob/release-3.7/roles/openshift_excluder/README.md): 
enable_openshift_excluder=true
```

然而，Ansible installer playbook 中存在一个问题，即预安装例程通过检查可用的软件包版本而不是已安装的版本而失败。换句话说，如果**rhel-7-server-ose-3.7-rpms**repo 中有比 **openshift_pkg_version** 中指定的版本更高的软件包，集群安装将不会运行。

为了克服这一点，我们只需通过 **/etc/yum.conf** 文件排除更高的包版本。安装之前，每个节点上都必须有此设置:

```
[main]#  One entry for every upper version (*-openshift*3.7.47 *-openshift*3.7.48)exclude= *openshift*3.7.46
```

最后，我们必须从运行安装程序的主机上的 ansi ble[**open shift _ health _ checker**](/usr/share/ansible/openshift-ansible/roles/openshift_health_checker/library/aos_version.py)模块中的 [**aos_version.py**](/usr/share/ansible/openshift-ansible/roles/openshift_health_checker/library/aos_version.py) 文件中删除以下行；否则，安装程序将忽略我们刚刚在上一步中配置的排除设置。

```
# /usr/share/ansible/openshift-ansible/roles/openshift_health_checker/library/aos_version.py# yb.conf.disable_excludes = ['all']
```

同样，我不清楚这种可疑逻辑的原因，但最终结果是删除该行允许安装程序继续工作。

一旦安装程序完成，跨节点运行一个 **yum list | grep openshift** 来确认软件包安装在正确的版本上。

```
$ ansible all -m shell -a 'yum list | grep openshift'
10.90.66.117 | SUCCESS | rc=0 >>
atomic-openshift.x86_64         3.7.44-2.git.0.6b061d4.el7
atomic-openshift-clients.x86_64 3.7.44-2.git.0.6b061d4.el7
atomic-openshift-clients-redistributable.x86_64
atomic-openshift-cluster-capacity.x86_64
atomic-openshift-descheduler.x86_64
atomic-openshift-docker-excluder.noarch
atomic-openshift-dockerregistry.x86_64
atomic-openshift-excluder.noarch
atomic-openshift-federation-services.x86_64
atomic-openshift-master.x86_64  3.7.44-2.git.0.6b061d4.el7
atomic-openshift-node.x86_64    3.7.44-2.git.0.6b061d4.el7
atomic-openshift-node-problem-detector.x86_64
atomic-openshift-pod.x86_64     3.7.44-2.git.0.6b061d4.el7
atomic-openshift-sdn-ovs.x86_64 3.7.44-2.git.0.6b061d4.el7
atomic-openshift-service-catalog.x86_64
atomic-openshift-template-service-broker.x86_64
atomic-openshift-tests.x86_64   3.7.44-2.git.0.6b061d4.el7
atomic-openshift-utils.noarch   3.7.44-1.git.9.684c638.el7
golang-github-openshift-oauth-proxy.x86_64
golang-github-openshift-prometheus-alert-buffer.x86_64
hawkular-openshift-agent.x86_64 1.2.2-1.el7           rhel-7-server-ose-3.7-rpms
jenkins-plugin-openshift-client.x86_64
jenkins-plugin-openshift-login.x86_64
jenkins-plugin-openshift-pipeline.x86_64
jenkins-plugin-openshift-sync.x86_64
nodejs-openshift-auth-proxy.noarch
openshift-ansible.noarch        3.7.44-1.git.9.684c638.el7
openshift-ansible-callback-plugins.noarch
openshift-ansible-docs.noarch   3.7.44-1.git.9.684c638.el7
openshift-ansible-filter-plugins.noarch
openshift-ansible-lookup-plugins.noarch
openshift-ansible-playbooks.noarch
openshift-ansible-roles.noarch  3.7.44-1.git.9.684c638.el7
openshift-elasticsearch-plugin.noarch
openshift-eventrouter.x86_64    0.1-1.git5bd9251.el7  rhel-7-server-ose-3.7-rpms
openshift-external-storage-efs-provisioner.x86_64
openshift-external-storage-local-provisioner.x86_64
openshift-external-storage-snapshot-controller.x86_64
openshift-external-storage-snapshot-provisioner.x86_64
python2-openshift.noarch        1:0.3.4-2.el7         rhel-7-server-ose-3.7-rpms
tuned-profiles-atomic-openshift-node.x86_64
```

## 包扎

我认为我们可以认为自己是幸运的，这次我们能够在没有任何重大并发症的情况下幸存下来。话虽如此，但它确实触发了一个警报信号，提醒我们在托管这些平台时，不应该忽视适当的内务管理实践。

我们的最终目标是更深入地挖掘 OpenShift playbook 安装程序，并找出一种创建黄金 ami 的方法，以便我们能够提供真正相同的节点，即使我们闭着眼睛也可以信任这些节点。

在那之前，动手干活吧。

干杯