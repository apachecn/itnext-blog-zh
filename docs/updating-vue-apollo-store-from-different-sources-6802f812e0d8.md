# 从不同来源更新 Vue Apollo 商店

> 原文：<https://itnext.io/updating-vue-apollo-store-from-different-sources-6802f812e0d8?source=collection_archive---------2----------------------->

自从 Vue-Apollo 第一次发布以来，我就一直在为每个 API 开发项目使用 GraphQL，就像任何其他新的工作环境一样，Vue-Apollo 也有自己的注意事项。其中之一是你不能简单地更新它自己的存储而不调用一个突变(*在实践中，实际上一个查询也会更新存储，但这个主题*却不是这样)。

显然，从开发者的角度来看，这是完全符合逻辑的。如果你想保持你的 [SSOT](https://en.wikipedia.org/wiki/Single_source_of_truth) 概念完好无损，这就是方法。

![](img/0992a069c48b387ab08a67e1c2366f43.png)

但是，而且总是有但是，可能在某些情况下，您可能想要更新客户端屏幕上获取的 GraphQL 数据( ***存储*** )。我的情况是这样的:

1.  在客户端，有一个简单的输入，用户将一些 Id 号输入数据库，
2.  在同一个页面上，有一个“最新的身份证号码”列表，所以用户可以在提交数据后看到一些结果，
3.  当第一次加载页面时，使用 GraphQL 查询获取 Id 号的最后 10 位，
4.  当用户提交一个新的 Id 号时，一个变异开始工作，并用新数据更新查询，
5.  在服务器端，Laravel 命令每分钟检查一个被标记为“无效”的 Id 号，
6.  这个 Id 号有一个第三方验证服务器，这个命令触发一个任务来发送 Id 号数据进行验证，
7.  同时，为了向用户显示这个验证过程的情况，而不需要每秒刷新页面或轮询 GraphQL 服务器，我使用 Pusher 服务，该作业触发一个事件来更新“最新 Id 号”列表，有趣的事情开始了。
8.  虽然用户仍然可以向数据库中插入新的 Id 号，但他仍然可以看到他的列表输入了他输入的数据，并用数据推送器对其进行更新。

这个概念的问题是，你可以更新 Vue 数据(来自 GraphQL 查询),你的页面仍然会很好。在上面的数字 8 中，Pusher 通过其通道发送更新，并通知更新的数据:

```
mounted () {
   this.channel = this.$pusher.subscribe(`private-User.Entries.${this.user.id}`) this.channel.bind('pusher:subscription_succeeded', (e) => { console.warn('Subscribed to Entries channel.') }) this.channel.bind('App\\Events\\EntryStatus', (data) => { this.onUpdateEntry(data) })
}
```

在这段代码中，您可以看到，无论何时在 Laravel 端触发 EntryStatus 事件，它都会更新 Pusher，Pusher 会更新我的 Vue 应用程序，然后使用其更新的数据触发 onUpdateEntry 方法。

```
data () {
    return {
        pendingUpdates: []
    }
},methods: {
   onUpdateEntry (entryUpdated) {
      // First part const entryRendered = this.$apolloData.data.entriesPaginate.find(entry => entry.id === entryUpdated.id) *if* (entryRendered !== undefined) {
         entryRendered.status = entryUpdated.status
     } // Second part const index = this.pendingUpdates.findIndex(entry => entry.id === entryUpdated.id) *if* (index !== -1) {
         this.$set(this.pendingUpdates, index, entryUpdated)
     } *else* {
         this.pendingUpdates.push(entryUpdated)
     }}
```

这段代码更新呈现的 GraphQL 数据，同时轮询更新的条目数据。请注意 entryRendered 对象。您可能会想，当可以更新 apolloData 时，为什么有人需要轮询 entryUpdated 数据，对吗？问题是，阿波罗数据公司并不是真正的阿波罗商店。它是数据集，您可以在查询的结果方法中更改它。

```
data () {
    return {
        pendingUpdates: [],
        loading: false,
        total: 0,
        pagination: {
            age: 1,
            rowsPerPage: 10,
            sortBy: 'id',
            descending: true
        },
        entriesPaginate: []
    }
},apollo: {
    entriesPaginate: () => ({
        query: ENTRY_PAGINATE,
        context: { uri: 'http://localhost/graphql' },
        variables () {
            *return* this.pagination
        },
        result ({ data: { entriesPaginate: { data, total } }, loading }) {
            *if* (!loading) {
                this.total = total
                this.entriesPaginate = data.slice().map((key, i) => ({
                   ...key,
                   __index: i
                }))
            }
        },
        watchLoading (isLoading) {
            this.loading = isLoading
        }
    })
}
```

使用 Vue-Apollo，您可以忽略查询中的 result 方法(以及数据中的 entriesPaginate 数组),让 Apollo 处理查询结果到 Vue 数据的分配。但在我的情况下，我需要单独的总数据，数据集上可能有一些其他的变化。因此我在这里使用结果方法。但是无论使用与否，entriesPaginate 数组都将是 Apollo store 原始查询结果的一部分。

所以我更新*这个也没关系。在 *onUpdateEntry* 方法中的$ Apollo data . data . entriespaginate*或 *this.entriesPaginate* 。当用户提交新的 Id 并触发变异时，Apollo 将触发查询结果方法本身(已定义或未定义)，并且这个. entriesPaginate 数组将使用 Apollo 存储的原始数据进行更新。这意味着来自 *onUpdateEntry* 方法的所有更新将在用户的屏幕上回滚。他/她将看到新的 Id 条目和旧的查询数据。

为了解决这个问题，我们会使用一个 *this。$apollo.store* 方法，但是没有这样一个。所以我想出了轮询更新数据的变通办法。在 *onUpdateEntry* 方法中，您可能会注意到有两部分代码带有注释，第一部分和第二部分。第二部分将 *updatedEntry* 数据插入 *pendingUpdates* 数组。当用户通过变异向数据库提交一个新的 Id 时，我首先保存这些记录来更新存储。这是:

```
data () {
    return {
        graphQLErrors: [],
        form: {
            idNumber: null
        },
        pendingUpdates: [], // Empty here for definition, actual data filled with multiple entryUpdated data.
    }
},methods: {
    submit () {
        this.loading = true this.$apollo.mutate({
             mutation: ENTRY_CREATE,
             variables: {
                 entry: {
                     id_number: this.form.idNumber
                 }
             },
             context: { uri: 'http://localhost/graphql' },
             update: (store, { data: { entryCreated } }) => {
                 const query = {
                     query: ENTRY_PAGINATE,
                     variables: this.pagination
                 }, data = store.readQuery(query)

                 if (data !== undefined) {
                      // First part
                      if (this.pendingUpdates.length) {
                         this.pendingUpdates.forEach(entryUpdated => {
                             const index = data.entriesPaginate.data.findIndex(entry => entry.id === entryUpdated.id)
                             if (index !== -1) {
                                 this.$set(data.entriesPaginate.data, index, entryUpdated)
                             }
                         }) this.pendingUpdates = [] // Reset
                     }

                     // Second part
                     if (data.entriesPaginate.data.findIndex(entry => entry.id === entryCreated.id) === -1) {
                         data.entriesPaginate.data.unshift(entryCreated)

                         store.writeQuery({
                             ...query,
                             data: data
                         })
                     }
                 }
             }
        }).catch((e) => {
            this.graphQLErrors = e
        }).then(() => {
            this.loading = false
        })
    }
}
```

你可能会注意到这里的*更新*方法的变异又分为两部分。第一部分，在用新创建的数据更新存储之前，迭代 *pendingUpdates* 数组并更新 Apollo 存储，然后第二部分在列表顶部插入 *entryCreated* 对象，最后 *store.writeQuery* 完成更新。

以上最重要的部分是，你必须向 Pusher 发布相同的 GraphQL 数据。我没有研究我的 Laravel GraphQL 包是否能够做到这一点，所以我只是模仿 GraphQL 服务数据的方式。更清楚地说，这是针对 *entriesPaginate* 的 GraphQL 查询结果示例:

```
{
  data: {
    entriesPaginate: [
      {id: 1, id_number: 12345678, created_at: SomeDate, validated_at: SomeDate, status: "queued", __typename: "Entry"},
      {id: 2, id_number: 34567890, created_at: SomeDate, validated_at: SomeDate, status: "validated", __typename: "Entry"},
      {id: 3, id_number: 45678901, created_at: SomeDate, validated_at: SomeDate, status: "validating", __typename: "Entry"},
      ...
    ],
    total: 41,
    __typename: "entriesPagination"
  }
}
```

而这个 my broadcating Event 的*broadcast 用*方法向 Pusher 发送数据:

```
public function broadcastWith()
{
    return [
        '__typename' => 'Entry',
        'id' => $this->entry->id,
        'id_number' => $this->entry->id_number,
        'status' => $this->entry->status,
        'created_at' => $this->entry->created_at,
        'validated_at' => $this->entry->validated_at
    ];
}
```

以相同的格式保存数据可以防止 Apollo 存储更新错误。

在 Vue 代码示例中，我使用了数据对象中最相关的数据项。所以请不要试图简单地复制粘贴上面的代码，因为它们可能缺少数据定义/方法。