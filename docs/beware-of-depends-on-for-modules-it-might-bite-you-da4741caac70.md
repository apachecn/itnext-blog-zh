# 小心 Terraform 模块的依赖关系。它可能会咬你！

> 原文：<https://itnext.io/beware-of-depends-on-for-modules-it-might-bite-you-da4741caac70?source=collection_archive---------0----------------------->

![](img/b1d731aeb49eea05fb7be22a381aff1a.png)

由[汤姆·威尔森](https://unsplash.com/@pastorthomasbwilson?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

# TLDR

由于 [Terraform 0.13](https://www.hashicorp.com/blog/announcing-hashicorp-terraform-0-13) ，可以在模块中使用 Terraform 的 [depends_on](https://www.terraform.io/language/meta-arguments/depends_on) 元参数。起初，这似乎是表达模块间依赖关系并确保资源按一定顺序应用的好方法。

然而，`depends_on`对 Terraform 如何创建计划有着[重大影响](https://www.terraform.io/language/meta-arguments/depends_on#processing-and-planning-consequences)，尤其是在与数据源结合使用时。这甚至可能导致意外的资源重新创建！

除了意识到这种行为，你还可以做几件事来避免不受欢迎的意外。主要思想是尽可能地依赖隐式依赖关系，同时保持数据源靠近工作空间的根。这些将避免对`depends_on`的需求或大大减轻其不受欢迎的副作用。

> 这些例子使用了 Azure，但是它们说明的问题和模式一般适用于 Terraform 代码

# 为什么模块的依赖行为会导致不受欢迎的意外

一个不知情的 Terraform 用户可能会认为`depends_on`只不过是一种指定模块和/或资源之间顺序的方式。虽然这是事实，但这只是故事的一部分。

`depends_on`的使用对 terraform 如何计算计划也有很大的影响。虽然每个人都可以在文档[中看到这种影响](https://www.terraform.io/language/meta-arguments/depends_on#processing-and-planning-consequences)，但它的真实含义可能不会被注意到(*粗体字是我的*)

> `*depends_on*`元参数指示 Terraform 在对声明依赖关系的对象执行操作之前，完成对依赖关系对象的所有操作(**，包括读取操作**)。当依赖对象是一个完整的模块时，`*depends_on*`影响 Terraform 处理与该模块相关的所有资源和数据源的顺序。更多细节请参考[资源依赖](https://www.terraform.io/language/resources/behavior#resource-dependencies)和[数据资源依赖](https://www.terraform.io/language/data-sources#data-resource-dependencies)。

而在链接的[数据资源依赖关系](https://www.terraform.io/language/data-sources#data-resource-dependencies)文档中，我们读到:

> […]在`*data*`块**中设置`*depends_on*`元参数会将数据源的读取**推迟到**应用完对依赖关系的所有更改**之后。

还不确定它的含义吗？我们举个例子试试。想象你的主要地形如下图所示。

*   我们正在创建 AKS 集群以及 VNET/子网和其他网络元素。
*   因为像 VNET 和子网这样的资源名称是预先知道的，所以我们将它们传递给网络和集群模块。
*   集群模块将检索带有数据源的子网的 id，并且为了确保它在被创建后是只读的，在模块之间添加了一个显式的`depends_on`依赖。

```
# in main.tf
module "my_aks_network" {
  vnet_name = local.vnet_name
  subnet_name = local.subnet_name
  resource_group_name = local.resource_group_name
  ...
}module "my_aks_cluster" {
  **depends_on = [module.my_aks_network]**

  vnet_name = local.vnet_name
  subnet_name = local.subnet_name
  resource_group_name = local.resource_group_name
  ...
}# inside "my_aks_cluster" module:
**data "azurerm_subnet" "sn"** {
  name                 = var.subnet_name
  virtual_network_name = var.vnet_name
  resource_group_name  = var.resource_group_name
}resource "azurerm_kubernetes_cluster" "cluster" {
  ...
  default_node_pool {
    **vnet_subnet_id = data.azurerm_subnet.sn.id
    ...** }
}
```

有了这个 Terraform 代码，每次 my_aks_network 模块内**有任何变化**，就会替换 **AKS 集群**！

这是为什么呢？因为`depends_on`使 Terraform 在计划阶段不评估`data.azurerm_subnet`。由于默认节点池的`vnet_subnet_id`参数未知(并且是一个在不替换集群的情况下无法修改的参数)，这导致了替换 AKS 集群的计划。当您看到计划输出时，这一点很明显:

```
# module.my_aks_cluster.data.azurerm_subnet.sn **will be read during apply**
  # (config refers to values not yet known)
 <= data "azurerm_subnet" "sn"  {
  ...      
  **+ id = (known after apply)**
  ...
}# module.my_aks_cluster.azurerm_kubernetes_cluster.cluster **must be replaced**
-/+ resource "azurerm_kubernetes_cluster" "cluster" {
  ...      
~ default_node_pool {
    ...
    **~ vnet_subnet_id = "/subscriptions/..." -> (known after apply) # forces replacement**
  }
}
```

这就是为什么讨论计划中`depends_on`含义的同一[文档](https://www.terraform.io/language/meta-arguments/depends_on#processing-and-planning-consequences)也**建议您将它作为最后手段**的原因。内容如下，虽然在我看来这应该是一个大红色警告在页面顶部:

> 你应该把`***depends_on***`作为最后的手段，因为它会导致 Terraform 创建更保守的计划，替换更多不必要的资源。[……]当您使用`*depends_on*`作为模块时，这尤其可能。
> 
> 我们建议尽可能使用[表达式引用](https://www.terraform.io/language/expressions/references)来暗示依赖关系，而不是`*depends_on*`。

文章的其余部分讨论了备选方案，可以总结为:

*   首选隐式依赖关系
*   避免模块内的数据源声明显式的`depends_on`依赖

如果您必须将`depends_on`用于模块，我们将以一些简短的提示结束。

# 你能做什么来代替

## 将数据源移到模块之外，将它们带到工作空间的根

由于模块中最令人担忧的问题是由数据源引起的，一个显而易见的方法是将数据源移出依赖模块，并将读取值注入模块。

即，而不是:

```
# in main.tf
module "my_aks_network"{
  ...
}
module "my_aks_cluster" {
  **depends_on = [module.my_aks_network]**
  ...
}# inside "my_aks_cluster" module. 
# Load resource group with a data source and use its id in some resources
**data "azurerm_resource_group" "rg" {**
  name = ...
}# For example, when assigning a role to the cluster principal
resource "azurerm_role_assignment" "ra" {
  **scope                = data.azurerm_resource_group.rg.id**
  role_definition_name = ...
  principal_id         = ...
}
```

将代码重构为如下形式:

```
# in main.tf
**data "azurerm_resource_group" "rg" {
  name = ...
}**module "my_aks_network"{
  ...
}
module "my_aks_cluster" {
  depends_on = [module.my_vnet_and_subnets]
  **resource_group = data.azurerm_resource_group.rg**
  ...
}# inside "my_aks_cluster" module:
resource "azurerm_role_assignment" "ra" {
  **scope                = var.resource_group.id**
  role_definition_name = ...
  principal_id         = ...
}
```

`depends_on`仍然会导致更保守的计划，但是现在这不会影响数据源的读取。这样，您就避免了更严重的副作用，这些副作用会导致不必要的更新或更糟的资源重建。

## 定义资源之间的隐式相关性

有时你有两个互相依赖的资源，但是这种依赖是基于一个预先知道的参数，就像它的`name`。

在这些情况下，很容易在两个资源中使用相同的`name`参数/local。例如，您可以创建 Azure 网络安全组和一些规则，代码如下:

```
resource "azurerm_network_security_group" "nsg" {
  name = var.network_security_group_name
  ...
}resource "azurerm_network_security_rule" "my_rule" {
  ...
  network_security_group_name = var.network_security_group_name
}
```

然而，使用这段代码，Terraform 无法知道这两个资源之间存在依赖关系，因此它会试图同时创建这两个资源，这会带来麻烦。您可能想使用`depends_on`来明确定义这种关系:

```
resource "azurerm_network_security_group" "nsg" {
  name = var.network_security_group_name
  ...
}resource "azurerm_network_security_rule" "my_rule" {
  ...
  network_security_group_name = var.network_security_group_name
  **depends_on = [azurerm_network_security_group.nsg]**
}
```

由于`depends_on`，这将引导你走向惊喜之路。幸运的是，有一种简单的方法可以避免`depends_on`，并且仍然定义资源之间的隐含关系。

代替`depends_on`，使用依赖项的输出属性(上面的 NSG)作为依赖项的输入参数(上面的 NSG 规则)

```
resource "azurerm_network_security_group" "nsg" {
  name = var.network_security_group_name
  ...
}resource "azurerm_network_security_rule" "my_rule" {
  ...
  network_security_group_name = **azurerm_network_security_group.nsg.name**
}
```

## 通过菊花链模块输出和输入，定义模块资源之间的隐式依赖关系

前面的模式可以跨模块推广。当在模块 A 中创建了模块 B 需要的东西时，将其细节(名称、id 等)作为模块 A 的输出公开，并作为输入参数注入模块 B。

当依赖关系基于设计时未知的参数(比如云提供商分配的 id)时，这直接避免了可能的资源重建。这正是本文开头的示例中发生的情况，它导致 AKS 集群被重新创建。解决问题的方法是:

```
# in main.tf
module "my_aks_network" {
  ...
}module "my_aks_cluster" {
  ... 
  # this way cluster module implicit depends on cluster_subnet
  **subnet_id = module.my_aks_network.id**
}# inside my_aks_network module
output "id" { 
 **value = azurerm_subnet.snet.id** 
}# inside my_aks_cluster module
resource "azurerm_kubernetes_cluster" "cluster" {
  ...
  default_node_pool {
    **vnet_subnet_id = var.subnet_id
    ...** }
}
```

然而，即使依赖关系是基于设计时已知的参数(比如用户分配的参数，比如子网的 CIDR)，使用它们的输出和输入而不是`depends_on`来链接它们也是值得的。

这个依赖项不会直接导致重新创建，但是添加一个`depends_on`可能会由于其他依赖项/资源而导致重新创建！在此过程中，您还将显式地链接各个资源，从而允许 Terraform 更好地理解依赖关系图，并按照预期的顺序创建/更新/删除它们。

```
# in main.tf
module "app_gw_subnet" {
  subnet_cidr = local.app_gw_cidr
  ...
}module "app_gw" {
  ... # **do not do this** 
  # subnet_cidr = local.app_gw_cidr 
  # depends_on = [module.app_gw_subnet]
  # **Instead do this, so app_gw implicitly depends on app_gw_subnet**
  subnet_cidr = **module.app_gw_subnet.cidr**
}# inside app_gw_subnet module
output "id" { 
 **value = azurerm_subnet.snet.address_prefixes.0**
}
```

> 将资源的实际输出属性作为模块输出公开是很重要的，即使它的值是从输入参数中得知的！

```
# given a subnet where cidr is a user assigned parameter
resource "azurerm_subnet" "snet" {
  address_prefixes = [var.subnet_cidr]
}# Do not expose the cidr as an output like this
output "cidr" { 
value **= var.subnet_cidr** 
}# Instead, do this
output "cidr" { 
value **= azurerm_subnet.snet.address_prefixes.0**
}
```

## 用一个*假的*输入变量定义隐式依赖关系

有时没有参数可以用来链接一个模块的输出和另一个模块的输入。

当这种情况发生时，您可以创建一个*假的*输入参数，并把它作为一个无害的东西使用，比如一个标签。通过添加和使用这个参数，您现在已经向您的模块添加了一种创建隐式依赖关系的方法(如果您只是添加了参数而没有使用它，那么就没有真正的隐式依赖关系)。

```
# main.tf
module "module_a" {
  ...
}module "module_b" {
  ...
  **phony_depends_on_a = module_a.some_output**
}# inside module_b code
variable "phony_depends_on_a" {
  ...
}resource "azurerm_some_resource" "my_resource" { 
  ...
  # use the fake input as something harmless, like a tag
  **tags** = {
    ... 
    module_a_dep = **var.phony_depends_on_a**
  }
}
```

Hashicorp 的以下博客文章详细阐述了“假模块依赖”的思想，以防您需要走这条路(我必须说，以一种更简洁的方式，避免了像我示例中的标签那样的强制使用)。参见:

[https://medium . com/hashi corp-engineering/creating-module-dependencies-in-terra form-0-13-4322702 DAC 4a](https://medium.com/hashicorp-engineering/creating-module-dependencies-in-terraform-0-13-4322702dac4a)

# 如果您仍然需要用 depends_on 显式定义依赖关系

尽量将`depends_on`的使用限制在同一个模块内的单个资源上，而不是模块之间。

但是如果你必须在模块之间使用`depends_on`呢？然后尝试确保声明`depends_on`依赖关系的模块(即*依赖*)或者:

*   内部没有数据源
*   有数据源，但是由于从这些数据源读取的输入参数，不会重新创建使用它们的资源。例如，从数据源获取资源组名称可能会替换资源，但是从数据源获取标记值只会导致不必要的标记更新。

仅此而已。在我得到一些令人惊讶的计划后，我艰难地学会了这些陷阱和技巧。因此，我希望这能帮助其他人找到了解他们的简单方法！