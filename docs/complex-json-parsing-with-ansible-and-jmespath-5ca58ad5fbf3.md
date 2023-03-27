# 用 Ansible 和 JMESPath 进行复杂的 JSON 解析

> 原文：<https://itnext.io/complex-json-parsing-with-ansible-and-jmespath-5ca58ad5fbf3?source=collection_archive---------1----------------------->

![](img/98f4ed2a80009910fbe8a16ceb3f75b3.png)

JSON 已经成为计算机世界中应用程序间共享信息的通用语言。几乎所有编写代码与 Web APIs 交互或检索应用程序结果的人都需要知道如何解析 JSON。幸运的是，由于 JSON 的流行，它得到了广泛的支持，并且有许多包，比如 [JMESPath](https://jmespath.org) ，可以用来帮助解析复杂的 JSON 结构。

当我经常使用 [Ansible](https://www.ansible.com) 部署或更新基础设施时，我必须解析来自[云提供商](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html)的 JSON 结果，或者在与 [Kubernetes](https://kubernetes.io) 交互时解析类似`kubectl`的命令输出。这些来源的输出通常包含大量信息，解析所有这些信息以获得所需的内容并将其转换为可用的格式通常很困难。

在下面来自`kubectl get node node-name -o json`的示例输出中，没有一种简单的方法可以在不遍历列表的情况下使用 Ansible 的本地 JSON 解析来获取`type` `Ready`的`status`。

```
{
  "conditions": [
              {
                  "status": "False",
                  "type": "NetworkUnavailable"
              },
              {
                 "status": "False",
                  "type": "MemoryPressure"
              },
              {
                  "status": "False",
                  "type": "DiskPressure"
              },
              {
                  "status": "False",
                  "type": "PIDPressure"
              },
              {
                 "status": "True",
                  "type": "Ready"
              }
  ]
}
```

然而，Ansible 通过使用 [json_query](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#json-query-filter) 过滤器公开 JMESPath JSON 解析库解决了这个问题。

在这个例子中，我将演示如何使用 Ansible 和 JMESPath 解析复杂的 JSON 输出，并通过使用一个键过滤器和压缩两个 JSON 列表将结果转换成更有用的格式。

对于这个例子，我使用 [OpenStack Heat](https://www.openstack.org/software/releases/ussuri/components/heat) 创建一个 8 节点(3 个 MON 节点和 5 个 OSD 节点)的 Ceph 集群，该集群将使用 [ceph-ansible](https://docs.ceph.com/ceph-ansible/master/) 安装 Ceph。因为我处于这个项目的开发和测试阶段，所以我经常创建和销毁集群。OpenStack Heat 和 Ansible 在自动化大多数创建和销毁步骤方面做得很好，但是仍然有一个手动步骤，我必须将 OpenStack Heat 创建的节点的主机名和 ip 地址复制到 Ansible 的库存文件中。为了完全自动化这个过程，我必须在 Ansible 中捕获 OpenStack Heat 的输出，这样我就可以使用 Ansible 模板自动生成库存文件。

要创建库存文件，这意味着要转换这个 JSON:

```
{
    "stack_create.stack.outputs": [
        {
            "description": "Ceph osd management addresses",
            "output_key": "ceph_osd_management_addresses",
            "output_value": [
                "192.168.0.95",
                "192.168.0.101",
                "192.168.0.155",
                "192.168.0.161",
                "192.168.0.23"
            ]
        },
        {
            "description": "Ceph osd server names",
            "output_key": "ceph_osd_server_names",
            "output_value": [
                "ceph-osd-0",
                "ceph-osd-1",
                "ceph-osd-2",
                "ceph-osd-3",
                "ceph-osd-4"
            ]
        },
        {
            "description": "Ceph mon management addresses",
            "output_key": "ceph_mon_management_addresses",
            "output_value": [
                "192.168.0.117",
                "192.168.0.240",
                "192.168.0.44"
            ]
        },
        {
            "description": "Ceph mon server names",
            "output_key": "ceph_mon_server_names",
            "output_value": [
                "ceph-mon-0",
                "ceph-mon-1",
                "ceph-mon-2"
            ]
        }
    ]
}
```

到以下库存文件中:

```
[mons]
ceph-mon-0 ansible_host=192.168.0.117 
ceph-mon-1 ansible_host=192.168.0.240 
ceph-mon-2 ansible_host=192.168.0.44[osds]
ceph-osd-0 ansible_host=192.168.0.95 
ceph-osd-1 ansible_host=192.168.0.101 
ceph-osd-2 ansible_host=192.168.0.155
ceph-osd-3 ansible_host=192.168.0.161
ceph-osd-4 ansible_host=192.168.0.23
```

一般来说，Ansible 有很好的 JSON 原生解析，但是 Ansible 的原生 JSON 解析不能处理这种情况，就像上面一样，当你需要根据一个 JSON 对象中的键值过滤一系列 JSON 对象时。例如，在上面的 JSON 输出中，我想要键`output_value`中的 ip 地址列表，其中`output_key` = `ceph_mon_management_addresses`。Ansible 的原生 JSON 解析所能做到的最好的是`stack_create.stack.outputs[2].output_value`，但是这需要`ceph_mon_management_addresses`总是列表中的第三项，这是无法保证的。

这就是 Ansible 的 [json_query 过滤器](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#json-query-filter)的用武之地。使用 JMESPath，我们可以在对象列表中搜索一个键值对，但返回同一对象中另一个键值。实际上，对于这个例子，我们可以在对象列表中搜索对象 where `output_key` = `ceph_mon_management_addresses`并返回`output_value`的值。下面是一个 Ansible `set_fact`任务，它使用一个 JMESPath 查询来获得结果:

```
- name: Create a list of mon ip addresses
  set_fact:
     mon_ips: "{{ stack_create | json_query(\"**stack.****outputs[?output_key == ‘ceph_mon_management_addresses’].output_value**\") }}"
```

在本例中，对包含`output_key == ‘ceph_mon_management_addresses’`的对象的搜索是通过上面的语句使用 JMESPath [过滤器投影](https://jmespath.org/tutorial.html#filter-projections) ( `?`)完成的。然后我们追加`.output_value`来返回`output_value`键的值。结果将会是这样的:

```
[
  [
    "192.168.0.117",
    "192.168.0.240",
    "192.168.0.44"
  ]
]
```

因为 JMESPath 保留了 JSON 的原始格式，所以有两个嵌套列表，对象列表和 ip 地址列表。我们只想要一个 ip 地址列表，因此我们可以应用 JMESPath[flatten projection](https://jmespath.org/tutorial.html#flatten-projections)来获得我们想要的输出。简单地把`[]`加到声明的末尾，就像这样:

```
- name: Create a list of mon ip addresses
  set_fact:
     mon_ips: "{{ stack_create | json_query(\"stack.outputs[?output_key == ‘ceph_mon_management_addresses’].output_value**[]**\") }}"
```

结果是我们想要的:

```
[                                               
  "192.168.0.117",                                    
  "192.168.0.240",
  "192.168.0.44"
]
```

顺便说一下，如果我们想将所有 ip 地址放入一个列表中，我们可以使用 JMESPath 的 OR ( `||`)操作符和 **filter** 和 **flatten projections** 来获得所有 mon 和 osd ip 地址的列表。

```
- name: Create a list of all ip addresses
  set_fact:
     mon_ips: "{{ stack_create | json_query(\"stack.outputs[?output_key == ‘ceph_mon_management_addresses’ **|| output_key ==’ceph_osd_management_addresses’**].output_value[]\") }}"
```

结果是:

```
[
  "192.168.0.95",
  "192.168.0.101",
  "192.168.0.155",
  "192.168.0.161",
  "192.168.0.23",
  "192.168.0.117",
  "192.168.0.240",
  "192.168.0.44"
]
```

为了在顶部生成清单文件，我们需要两个不同 JSON 对象中的所有 mon 名称和 ip 地址。我们可以在`ceph_mon_management_addresses`和`ceph_mon_server_name`上使用 JMESPath 或运算符以及过滤器投影，如下所示:

```
- name: Create a list of all ip addresses
  set_fact:
     mon_ips: "{{ stack_create | json_query(\"stack.outputs[?output_key == ‘ceph_mon_management_addresses’ **|| output_key == ‘ceph_mon_server_names’**].output_value\") }}"
```

结果是:

```
[
  [
    "192.168.0.117",
    "192.168.0.240",
    "192.168.0.44"
  ],
  [
    "ceph-mon-0",
    "ceph-mon-1",
    "ceph-mon-2"
  ]
]
```

结果给出了我们需要的所有信息，但不幸的是，我们得到了一个列表的列表，如果不进行复杂的循环，我们就不能用它来完成一个模板。

我们希望信息以下列格式返回:

```
{
    "ceph-mon-0": "192.168.0.117",
    "ceph-mon-1": "192.168.0.240",
    "ceph-mon-2": "192.168.0.44"
}
```

在这种格式下，我们可以使用一个简单的 [Jinja](https://jinja.palletsprojects.com/en/2.11.x/) 模板来创建 ceph-ansible 所需的库存文件。`key`是主机名，`value`是 ip 地址:

```
[mons]
{% for key, value in mons.items() %}
{{ key }} ansible_host={{ value }}
{% endfor %}[osds]
{% for key, value in osds.items() %}
{{ key }} ansible_host={{ value }}
{% endfor %}
```

为了完成这个任务，我们需要创建两个列表，一个是主机名列表，一个是 ip 地址列表，然后将这两个列表合并。组合两个列表以产生 Jinja 模板所需格式的最简单方法是使用 [zip(或 zipper 或 Convolution](https://en.wikipedia.org/wiki/Convolution_(computer_science)) )函数。不幸的是，JMESPath 没有 [zip 方法](https://github.com/jmespath/jmespath.py/issues/152)，但是 [Ansible 有](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#zip-and-zip-longest-filters)。

现在，让我们一起来看看。第一个可回答的任务将创建四个可回答的事实，包含列表`mon_ips`、`mon_name`、`osd_ips`和`osd_names`。然后，第二个可行任务将把`mon_names`和`osd_names`列表转换成字典，然后把`mon_ips`和`osd_ips`压缩成字典。

```
- name: Set mon and osd facts
  set_fact:
     mon_ips: "{{ stack_create | json_query(\"stack.outputs[?output_key == 'ceph_mon_management_addresses'].output_value[]\") }}"
     mon_names: "{{ stack_create | json_query(\"stack.outputs[?output_key == 'ceph_mon_server_names'].output_value[]\") }}"
     osd_ips: "{{ stack_create | json_query(\"stack.outputs[?output_key == 'ceph_osd_management_addresses'].output_value[]\") }}"
     osd_names: "{{ stack_create | json_query(\"stack.outputs[?output_key == 'ceph_osd_server_names'].output_value[]\") }}"

- name: Zip names and ips
  set_fact:
     mons: "{{ **dict**(mon_names | **zip**(mon_ips)) }}"
     osds: "{{ **dict**(osd_names | **zip**(osd_ips)) }}"
```

`mons`和`osds`变量将如下所示，并完全适合 Jinja 模板，以创建所需的 ceph-ansible 库存文件。

```
mons:
{
    "ceph-mon-0": "192.168.0.117",
    "ceph-mon-1": "192.168.0.240",
    "ceph-mon-2": "192.168.0.44"
}osds:
{
    "ceph-osd-0": "192.168.0.95",
    "ceph-osd-1": "192.168.0.101",
    "ceph-osd-2": "192.168.0.155",
    "ceph-osd-3": "192.168.0.161",
    "ceph-osd-4": "192.168.0.23"
}
```

希望这个例子有帮助。JSON 在计算中无处不在，找到解析它的捷径对于简单易管理的代码至关重要。JMESPath 是用于简单 JSON 解析的强大工具，我们可以用循环和复杂的 Jinja 模板获得相同的结果，但是我们有一些简短、简单和易于查看的东西。