# 使用 Terraform 管理 Azure RBAC 角色和任务

> 原文：<https://itnext.io/managing-azure-rbac-roles-and-assignments-with-terraform-1a1cdb2f8eb6?source=collection_archive---------2----------------------->

![](img/5a5b3588e4c86ec9fd8e3cc6457cbb6c.png)

火星表面。我们还没有把这个地球化。Yuliya Kosolapova 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

这是一个我迫不及待想分享的快速指南。这是一种使用 Terraform 在 Azure 中管理自定义角色和角色分配的方法。我使用的 Terraform、AzureRM 和 AzureAD 提供程序的版本如下:

```
➜ terraform version 
Terraform v0.12.24
+ provider.azuread v0.7.0
+ provider.azurerm v2.0.0
```

在本例中，我创建了一个自定义角色，允许一些用户在我们的 Azure 订阅中查看共享仪表板。用户应该能够查看 Terraform 已经创建的仪表板，它由 terraform 资源`azurerm_dashboard.insights-dashboard`引用:

```
resource "azurerm_role_definition" "support_dash_read" {
  name        = "dashboard-${var.environment}-support"
  scope       = data.azurerm_subscription.current.id
  description = "Custom Role for viewing Dashboards"
  permissions {
    actions = [
      "Microsoft.Portal/dashboards/read",
      "Microsoft.Insights/*/read",
      "Microsoft.OperationalInsights/workspaces/search/action",
    ]
    not_actions = []
  }
  assignable_scopes = [
    azurerm_dashboard.insights-dashboard.id,
  ]
}
```

该角色授予以下用户读取权限:

*   在 Terraform 中的其他地方创建的特定共享仪表板，方法是将此角色的范围限定在仪表板上
*   与内置的监视读取器角色具有相同的读取权限，但不具备提出支持票证的能力。由于 Insights 查询大量数据——每个数据都显示在不同的权限条目中，例如`Microsoft.Insights/metricAlerts/read`、`Microsoft.Insights/alertRules/read`、`Microsoft.Insights/components/*/read` ——我发现更容易使读取权限更加宽松，并模仿其中一个内置角色。该角色的范围仍然只能分配给仪表板，因此我们有效地授予了仪表板显示的任何洞察数据的读取权限。

向用户分配角色时，您需要他们在 Azure AD 中的主体 ID(也称为对象 ID)来执行分配。就我个人而言，我不希望在运行 terraform 之前，通过一些手动过程或使用 CLI 来找出每个用户的对象 ID。相反，我们可以创建一个变量来存储 Azure 中与用户相关联的所有电子邮件地址(也是他们的 UPN，或用户的主要名称):

```
support_users = [
  "user1@contoso.com",
  "user2@contoso.com",
  "user3@contoso.com",
  "user4@contoso.com",
]
```

然后，我们将该变量传递给 AzureAD 提供者，并使用`for_each`参数遍历用户:

```
*# Query the Principal ID for each user in support_users* data "azuread_user" "user" {
  for_each            = toset(var.support_users)
  user_principal_name = format("%s", each.key)
}
```

最后，当我们想要将所有这些用户分配给我们上面创建的自定义角色时，我们再次使用`for_each`来完成，这次提供我们上面创建的数据资源(`azuread_user`):

```
resource "azurerm_role_assignment" "example" {
  for_each           = data.azuread_user.user
  scope              = azurerm_dashboard.insights-dashboard.id
  role_definition_id = azurerm_role_definition.support_dash_read.id
  principal_id       = data.azuread_user.user[each.key].object_id
}
```

就是这样！现在，您可以使用 Terraform 将一批用户分配到 Azure 中的 RBAC 角色。