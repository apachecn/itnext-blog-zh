# 地形:模块、输出和变量

> 原文：<https://itnext.io/terraform-modules-outputs-and-variables-dc157274fb5d?source=collection_archive---------5----------------------->

![](img/5f300cb28cb544277e4d7404b666b2d0.png)

最终，我到达了 Terraform 中的模块。

也就是说，我必须弄清楚如何在两个模块之间传递变量值。

所以在这篇文章中，最基本和最简单的例子就是使用模块及其值和输出。

更多信息请参见文档— [模块](https://developer.hashicorp.com/terraform/language/modules)。

# 根模块

首先，让我们创建一个根模块，它只是创建一个本地文件，稍后我们将在其中添加模块。

创建测试目录:

```
$ mkdir modules_example
$ cd modules_example/
```

在这个目录中添加一个带有`[local_file](https://registry.terraform.io/providers/hashicorp/local/latest/docs/data-sources/file)`类型的`[resource](https://developer.hashicorp.com/terraform/language/resources)`的文件`main.tf`，这将创建一个带有“*文件内容*文本的 *file.txt* :

```
resource "local_file" "file" {
  content  = "file content"
  filename = "file.txt"
}
```

运行`[terraform init](https://developer.hashicorp.com/terraform/cli/commands/init)`调出 Terraform 本身的必要模块:

```
$ terraform init
Initializing the backend…
Initializing provider plugins…
- Finding latest version of hashicorp/local…
- Installing hashicorp/local v2.2.3…
- Installed hashicorp/local v2.2.3 (signed by HashiCorp)
…
```

Terraform 已成功初始化！

然后，运行`[terraform plan](https://developer.hashicorp.com/terraform/cli/commands/plan)`来检查它是否能工作，以及它到底能做什么:

```
$ terraform plan
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
+ create
Terraform will perform the following actions:
local_file.file will be created
+ resource “local_file” “file” {
+ content = “file content”
+ directory_permission = “0777”
+ file_permission = “0777”
+ filename = “file.txt”
+ id = (known after apply)
}
Plan: 1 to add, 0 to change, 0 to destroy.
```

如果看起来正常，运行`[terraform apply](https://developer.hashicorp.com/terraform/cli/commands/apply)`:

```
$ terraform apply
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
+ create
Terraform will perform the following actions:
local_file.file will be created
+ resource “local_file” “file” {
+ content = “file content”
+ directory_permission = “0777”
+ file_permission = “0777”
+ filename = “file.txt”
+ id = (known after apply)
}
Plan: 1 to add, 0 to change, 0 to destroy.
Do you want to perform these actions?
Terraform will perform the actions described above.
Only ‘yes’ will be accepted to approve.
Enter a value: yes
local_file.file: Creating…
local_file.file: Creation complete after 0s [id=87758871f598e1a3b4679953589ae2f57a0bb43c]
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

并检查我们创建`main.tf`文件并启动命令的目录的内容:

```
$ ls -l
total 12
-rwxr-xr-x 1 setevoy setevoy 12 Nov 28 13:14 file.txt
-rw-r — r — 1 setevoy setevoy 90 Nov 28 13:09 main.tf
-rw-r — r — 1 setevoy setevoy 854 Nov 28 13:14 terraform.tfstate
```

而`file.txt`内容:

```
$ cat file.txt
file content
```

有用！让我们更进一步。

# 地形模块

接下来，让我们进入模块。

为两个模块创建两个目录:

```
$ mkdir -p modules/file_1
$ mkdir -p modules/file_2
```

在其中的每一个中，创建他们自己的`main.tf`文件——即`modules/file_1/main.tf`和`modules/file_2/main.tf`。

在`modules/file_1/main.tf`中使用同一个`local_file`资源创建一个 *file_1.txt* 文件:

```
resource "local_file" "file_1" {
  content  = "file_1 content"
  filename = "file_1.txt"
}
```

同样在`modules/file_2/main.tf`中为 *file_2.txt* :

```
resource "local_file" "file_2" {
  content  = "file_2 content"
  filename = "file_2.txt"
}
```

更新根模块，即`modules_example/main.tf`——删除`resource "local_file"`，取而代之的是用两个模块的目录路径来描述这两个模块:

```
module "file_1" {
  source = "./modules/file_1"
}

module "file_2" {
  source = "./modules/file_2"
}
```

再次运行`init`，以便 Terraform 创建其模块结构:

```
$ terraform init
Initializing modules…
- file_1 in modules/file_1
- file_2 in modules/file_2
Initializing the backend…
Initializing provider plugins…
- Reusing previous version of hashicorp/local from the dependency lock file
- Using previously-installed hashicorp/local v2.2.3
Terraform has been successfully initialized!
```

检查`.terraform/modules/`:

```
$ cat .terraform/modules/modules.json | jq
{
“Modules”: [
{
“Key”: “”,
“Source”: “”,
“Dir”: “.”
},
{
“Key”: “file_1”,
“Source”: “./modules/file_1”,
“Dir”: “modules/file_1”
},
{
“Key”: “file_2”,
“Source”: “./modules/file_2”,
“Dir”: “modules/file_2”
}
]
}
```

跑`plan`:

```
$ terraform plan
local_file.file: Refreshing state… [id=87758871f598e1a3b4679953589ae2f57a0bb43c]
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
+ create
- destroy
Terraform will perform the following actions:
local_file.file will be destroyed
(because local_file.file is not in configuration)
- resource “local_file” “file” {
- content = “file content” -> null
- directory_permission = “0777” -> null
- file_permission = “0777” -> null
- filename = “file.txt” -> null
- id = “87758871f598e1a3b4679953589ae2f57a0bb43c” -> null
}
module.file_1.local_file.file_1 will be created
+ resource “local_file” “file_1” {
+ content = “file_1 content”
+ directory_permission = “0777”
+ file_permission = “0777”
+ filename = “file_1.txt”
+ id = (known after apply)
}
module.file_2.local_file.file_2 will be created
+ resource “local_file” “file_2” {
+ content = “file_2 content”
+ directory_permission = “0777”
+ file_permission = “0777”
+ filename = “file_2.txt”
+ id = (known after apply)
}
Plan: 2 to add, 0 to change, 1 to destroy.
```

现在我们可以执行`apply`。

为了不每次都输入*“是*”，我们可以使用`-auto-approve`参数:

```
$ terraform apply -auto-approve
local_file.file: Refreshing state… [id=87758871f598e1a3b4679953589ae2f57a0bb43c]
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
+ create
- destroy
Terraform will perform the following actions:
local_file.file will be destroyed
(because local_file.file is not in configuration)
- resource “local_file” “file” {
- content = “file content” -> null
- directory_permission = “0777” -> null
- file_permission = “0777” -> null
- filename = “file.txt” -> null
- id = “87758871f598e1a3b4679953589ae2f57a0bb43c” -> null
}
module.file_1.local_file.file_1 will be created
+ resource “local_file” “file_1” {
+ content = “file_1 content”
+ directory_permission = “0777”
+ file_permission = “0777”
+ filename = “file_1.txt”
+ id = (known after apply)
}
module.file_2.local_file.file_2 will be created
+ resource “local_file” “file_2” {
+ content = “file_2 content”
+ directory_permission = “0777”
+ file_permission = “0777”
+ filename = “file_2.txt”
+ id = (known after apply)
}
Plan: 2 to add, 0 to change, 1 to destroy.
local_file.file: Destroying… [id=87758871f598e1a3b4679953589ae2f57a0bb43c]
module.file_2.local_file.file_2: Creating…
module.file_1.local_file.file_1: Creating…
local_file.file: Destruction complete after 0s
module.file_2.local_file.file_2: Creation complete after 0s [id=7225b36c22072cd558c23529d0d992c29cb873be]
module.file_1.local_file.file_1: Creation complete after 0s [id=82888a6ec6b05fc219759bd241ca5f0d6cba0e23]
Apply complete! Resources: 2 added, 0 changed, 1 destroyed.
```

检查文件是否已经出现:

```
$ ll
total 24
-rwxr-xr-x 1 setevoy setevoy 14 Nov 28 13:21 file_1.txt
-rwxr-xr-x 1 setevoy setevoy 14 Nov 28 13:21 file_2.txt
-rw-r — r — 1 setevoy setevoy 104 Nov 28 13:18 main.tf
drwxr-xr-x 4 setevoy setevoy 4096 Nov 28 13:15 modules
```

以及它们的内容:

```
$ cat file_1.txt
file_1 content
cat file_2.txt
file_2 content
```

作品？向前看。

# Terraform 模块中的变量

这里没有什么特别的，一切都和普通的 Terraform 变量一样。参见文档— [输入变量](https://developer.hashicorp.com/terraform/language/values/variables)。

在`modules_example/modules/file_1/main.tf`中声明一个变量`user_name`:

```
variable "user_name" {
  type = string
}    

resource "local_file" "file_1" {
  content  = "file_1 content from ${var.user_name}"
  filename = "file_1.txt"
}
```

第二个模块也是如此:

```
variable "user_name" {
  type = string
}    

resource "local_file" "file_2" {
  content  = "file_2 content from ${var.user_name}"
  filename = "file_2.txt"
}
```

更新根模块—为两个模块传递`user_name`变量的值:

```
module "file_1" {
  user_name = "user1"
  source = "./modules/file_1"
}

module "file_2" {
  user_name = "user2"
  source = "./modules/file_2"
}
```

应用更改:

```
$ terraform apply -auto-approve
module.file_1.local_file.file_1: Refreshing state… [id=82888a6ec6b05fc219759bd241ca5f0d6cba0e23]
module.file_2.local_file.file_2: Refreshing state… [id=7225b36c22072cd558c23529d0d992c29cb873be]
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement
Terraform will perform the following actions:
module.file_1.local_file.file_1 must be replaced
-/+ resource “local_file” “file_1” {
~ content = “file_1 content” -> “file_1 content from user1” # forces replacement
~ id = “82888a6ec6b05fc219759bd241ca5f0d6cba0e23” -> (known after apply)
(3 unchanged attributes hidden)
}
module.file_2.local_file.file_2 must be replaced
-/+ resource “local_file” “file_2” {
~ content = “file_2 content” -> “file_2 content from user2” # forces replacement
~ id = “7225b36c22072cd558c23529d0d992c29cb873be” -> (known after apply)
(3 unchanged attributes hidden)
}
…
Apply complete! Resources: 2 added, 0 changed, 2 destroyed.
```

而结果检查:

```
$ cat file_1.txt
file_1 content from user1
```

# 变量的模块和输出值

如何将变量的值从子模块传递到根模块？使用`outputs`。参见[输出值](https://developer.hashicorp.com/terraform/language/values/outputs)文档。

更新第一个模块—添加名为 *file_content* 的`output`:

```
variable "user_name" {
  type = string
}    

resource "local_file" "file_1" {
  content  = "file_1 content from ${var.user_name}"
  filename = "file_1.txt"
} 

output "file_content" {
  value = file("file_1.txt")
}
```

第二个也一样:

```
variable "user_name" {
  type = string
}    

resource "local_file" "file_2" {
  content  = "file_2 content from ${var.user_name}"
  filename = "file_2.txt"
} 

output "file_content" {
  value = file("file_2.txt")
}
```

然后在根模块中，使用使用`module.<MODULE_NAME>.<OUTPUD_NAME>`获得的值创建一个文件 *concat_file.txt* :

```
module "file_1" {
  user_name = "user1"
  source = "./modules/file_1"
}

module "file_2" {
  user_name = "user2"
  source = "./modules/file_2"
}

resource "local_file" "concat_file" {
  content  = "${module.file_1.file_content}\n${module.file_2.file_content}\n"
  filename = "concat_file.txt"
}
```

运行`apply`:

```
$ terraform apply -auto-approve
module.file_2.local_file.file_2: Refreshing state… [id=5771aa8b1046de4f4342d492faee20e3c289365d]
module.file_1.local_file.file_1: Refreshing state… [id=8a3552bfa2e46f311e7b718f8eed71b69bb8115f]
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
+ create
Terraform will perform the following actions:
local_file.concat_file will be created
+ resource “local_file” “concat_file” {
+ content = <<-EOT
file_1 content from user1
file_2 content from user2
EOT
+ directory_permission = “0777”
+ file_permission = “0777”
+ filename = “concat_file.txt”
+ id = (known after apply)
}
Plan: 1 to add, 0 to change, 0 to destroy.
local_file.concat_file: Creating…
local_file.concat_file: Creation complete after 0s [id=a4e1240fb9cdc96fffc1b0984392f6218422d194]
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

检查结果:

```
$ cat concat_file.txt
file_1 content from user1
file_2 content from user2
```

# 在模块之间传输变量值

最后，我们从如何将值从一个模块转移到另一个模块开始？

在第一个模块的文件`main.tf`中，添加一个名为`module_1_value`的变量，默认值为“*模块 1 值*”:

```
...
variable "module_1_value" {
  type = string
  default = "module 1 value"
}
...
```

在同一个地方，创建第二个变量`module_2_value`，然后我们将把来自第二个模块的值传递给它:

```
...
variable "module_2_value" {
  type = string
}
...
```

添加`output`将变量`module_1_value`的值返回给根模块，以便在第二个模块中进一步使用:

```
...
output "module_1_value" {
  value = var.module_1_value
}
...
```

完整的文件现在看起来像这样:

```
variable "user_name" {
  type = string
} 

variable "module_1_value" {
  type = string
  default = "module 1 value"
} 

variable "module_2_value" {
  type = string
}

resource "local_file" "file_1" {
  content  = "file_1 content from ${var.user_name} with value from file_2: ${var.module_2_value}"
  filename = "file_1.txt"
}

output "file_content" {
  value = file("file_1.txt")
}

output "module_1_value" {
  value = var.module_1_value
}
```

这里，在资源`local_file`中，我们将使用从第二个模块传输的值。

对模块`file_2`重复相同的操作:

```
variable "user_name" {
  type = string
} 

variable "module_2_value" {
  type = string
  default = "module 2 value"
} 

variable "module_1_value" {
  type = string
}

resource "local_file" "file_2" {
  content  = "file_2 content from ${var.user_name} with value from file_1: ${var.module_1_value}"
  filename = "file_2.txt"
}

output "file_content" {
  value = file("file_2.txt")
}

output "module_2_value" {
  value = var.module_2_value
}
```

返回到根模块，将变量`module_2_value`的转移添加到第一个模块，将变量`module_1_value`的转移添加到第二个模块:

```
module "file_1" {
  user_name = "user1"
  module_2_value = module.file_2.module_2_value
  source = "./modules/file_1"
}   

module "file_2" {
  user_name = "user2"
  module_1_value = module.file_1.module_1_value
  source = "./modules/file_2"
} 

resource "local_file" "concat_file" {
  content  = "${module.file_1.file_content}\n${module.file_2.file_content}\n"
  filename = "concat_file.txt"
}
```

现在在文件`file_1.txt`中，我们必须从`module "file_2"`中获取值，反之亦然。

应用:

```
$ terraform apply -auto-approve
module.file_1.local_file.file_1: Refreshing state… [id=d74b0e0b3296ab02e7b4791942d983b175d071b4]
local_file.concat_file: Refreshing state… [id=e0e88293b53693a484db146b15bd2ab3d8f4d250]
module.file_2.local_file.file_2: Refreshing state… [id=12faf027dc96d886c6022b54c878324a21d3d112]
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement
Terraform will perform the following actions:
local_file.concat_file must be replaced
-/+ resource “local_file” “concat_file” {
~ content = <<-EOT # forces replacement
- file_1 content from user1 with value from file_2: variable 2 value
- file_2 content from user2 with value from file_1: variable 1 value
+ file_1 content from user1 with value from file_2: module 2 value
+ file_2 content from user2 with value from file_1: module 1 value
EOT
~ id = “e0e88293b53693a484db146b15bd2ab3d8f4d250” -> (known after apply)
(3 unchanged attributes hidden)
}
Plan: 1 to add, 0 to change, 1 to destroy.
local_file.concat_file: Destroying… [id=e0e88293b53693a484db146b15bd2ab3d8f4d250]
local_file.concat_file: Destruction complete after 0s
local_file.concat_file: Creating…
local_file.concat_file: Creation complete after 0s [id=648854841581b273d26646fff776b33fdf060a19]
Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
```

检查结果:

```
$ cat concat_file.txt
file_1 content from user1 with value from file_2: module 2 value
file_2 content from user2 with value from file_1: module 1 value
```

完成了。

*最初发布于* [*RTFM: Linux、DevOps、系统管理*](https://rtfm.co.ua/en/terraform-modules-outputs-and-variables/) *。*