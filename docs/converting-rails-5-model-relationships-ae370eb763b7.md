# 转换 Rails 5 模型关系

> 原文：<https://itnext.io/converting-rails-5-model-relationships-ae370eb763b7?source=collection_archive---------3----------------------->

![](img/7b3844ebc3e1e16a3ded98eaa26032b0.png)

这是如此普遍，我很惊讶，它在文件中是不正确的。

## 从一对多到多对多

假设您希望用户属于一家公司，但是您的客户告诉您，“哦，是的，我们有这些在不止一个地方工作的人。”

这些模型有一些像这样的 Ruby 代码:

```
**class *User*** < ***ApplicationRecord*** *belongs_to* **:company
end

class *Company*** < ***ApplicationRecord*** *has_many* **:users
end**
```

事实证明重构非常简单。

生成一个迁移，目标是为公司和用户创建一个新的连接表，从`Users`表中迁移现有的`company_id`数据，然后相应地更新 Rails 模型。

```
**class *AddCompanyUsers*** < ***ActiveRecord***::***Migration***[5.2]
  **def** *change
    create_join_table* **:companies**, **:users do** |*t*|
      *t*.index **%i[company_id user_id]**, **unique**: **true
    end** *# migrate users to new table* up_only **do
      *User***.all.each **do** |*u*|
        **unless** *u*.company_id.nil?
          *u*.update(**companies**: [***Company***.*find*(*u*.company_id)])
        **end
      end** *# drop company_id from users
      remove_column* **:users**, **:company_id
    end
  end
end**
```

其中一个重要的标志是`unique: true`,因为它清除了表两端的连接表中的任何重复。这样，一个用户可以属于许多公司，但每个公司只能有一个用户。

更新模型以支持它们之间的多对多关系:

```
**class *User*** < ***ApplicationRecord*** *has_and_belongs_to_many* **:companies
end

class *Company*** < ***ApplicationRecord*** *has_and_belongs_to_many* **:users
end**
```

最后一步是运行迁移:

`rails db:migrate`

这将负责基础，但现在控制器需要处理新的信息。以前我们只允许一个`company_id`参数，然后直接在他们的行中设置用户的公司。但是现在我们有了一个公司 id 数组(或者一个用户 id 数组)来接受我们的参数。

```
**class *UserController*** < ***ApplicationController* def** *update
    # woop no changes probably* **end

  def** *user_params
    # How it was* params.*require*(**:user**).permit(**:name**, **company_ids**: [])
  **end
end**
```

现在只需在您的 UI 中设置 company_id，并将它们与您的其他参数一起发送。Rails 会处理剩下的部分。

## 从多对多到一对多

除非连接表会对性能产生重大影响，否则就让它成为多对多连接吧，这种情况不太可能发生。这种变化可能需要抛出一个客户想要的选择，他们会知道哪一个是该保留的。

这与上面的相反，但是我们希望保留旧的连接表一段时间，迁移只选择数组中的第一个值作为新列值的默认值。

```
**class *AddCompanyIdToUsers*** < ***ActiveRecord***::***Migration***[5.2]
  **def** *change
    add_column* **:company_id**, **:users**, **optional**: **true** *# migrate users to new table* up_only **do
      *User***.all.each **do** |*u*|
        *u*.update(**company**: user.companies.first)
      **end
    end
  end
end**
```

那么您的模型将会更新如下:

```
**class *User*** < ***ApplicationRecord*** *belongs_to* **:company**, **optional**: **true
end

class *Company*** < ***ApplicationRecord*** *has_many* **:users
end**
```

可选性在这里很重要，因为在一段时间内，应用程序有两个成员身份的真实来源，但我们接受我们发现的第一个值可能是正确的。UI 需要通过`company_ids`停止更新，并开始接受`user.company_id`是正确的值。所有真正的重构都将发生在那里。

想要我覆盖更多迁移路径吗？对 Rails 有疑问吗？在下面留言，我会更新文章。

干杯！