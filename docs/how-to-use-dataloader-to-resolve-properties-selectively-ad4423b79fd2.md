# 如何使用 DataLoader 有选择地解析属性

> 原文：<https://itnext.io/how-to-use-dataloader-to-resolve-properties-selectively-ad4423b79fd2?source=collection_archive---------2----------------------->

![](img/189115698f952a6a26a91b34140791de.png)

罗伯特·v·鲁杰罗在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄的照片

我的团队最近试图构建一个 GraphQL 服务器，它可以从各种其他 REST 服务中提取和连接数据。出于某种历史原因，其中一个服务在响应中展平了父数据中的部分而非全部子数据。

```
 GET /user[
  {
    "id": "123",
    "name": "...",
    **"addresses": [
      {
        "id": "222",
        "city": "San Francisco"
      }
    ]**
  },
  ...
]GET /address/222{
  "id": "222",
  "line1": "...",
  "line2": "...",
  "city": "San Francisco",
  "state": "CA"
}
```

例如，用户可以拥有一个地址数组。除了包含地址`id`之外，`city`也作为用户地址的一部分返回。然而，如果我们想要得到`state`，它并不包含在内，我们别无选择，只能用另一个 REST API 调用来获取地址 222。

# 问题是

调用获取地址并加入用户是合理的做法，因为外键是为地址提供的。在大多数情况下，解析器看起来像这样。

```
const resolvers = {
  User: {
    addresses: user => {
      // For each address, load the address by its ID.
      return user.addresses.map(address => loadAddress(address.id));
    }
  },
}
```

即使在加载用户列表时，解析器也能很好地工作，但是由于 [N+1 问题](/how-to-avoid-n-1-problem-in-apollo-federation-8b4f37729fc4)，性能非常糟糕。通过使用[数据加载器](https://github.com/graphql/dataloader)将所有单独的`loadAddress()`调用批处理为单个`loadAddresses()`调用，可以*可能*修复 N+1 问题。我说*可能*是因为这取决于你的 REST 服务是否提供批量 API。

```
query GetUsers {
  getUsers {
    id
    name
    **addresses** {
      **id**
      **city**
    }
  }
}
```

不管怎样，即使我们已经知道了`city`是什么，上面的查询仍然必须至少调用`loadAddress()`或`loadAddresses()`一次。还记得 REST API 响应中包含的`city`吗？

那么，推迟调用`loadAddress()`到`Address`的属性呢？如果我们这样做，我们只在必须解析属性时调用`loadAddress()`。

```
const resolvers = {
  User: {
    addresses: user => {
      // For each address, load the address by its ID.
      return user.addresses.map(address => loadAddress(address.id));
    }
  },
  Address: {
    id: address => address.id,
    line1: address => **loadAddress(address.id)**.line1,
    line2: address => **loadAddress(address.id)**.line2,
    city: address => address.city,
    state: address => **loadAddress(address.id)**.state,
  }
}
```

太好了。当我们查询`id`或`city`时，我们不必调用`loadAddress()`。然而，如果我们在查询中包含`line1`、`line2`或`state`，我们仍然会多次调用`loadAddress()`。因此，该解决方案解决了一个问题，但引入了另一个问题。

# 解决方案

那么我们如何在引用`line1`、`line2`或者`state`的时候最多调用一次`loadAddress()`？这就是我们可以利用数据加载器来合并`loadAddress()`的地方。

```
const DataLoader = require('dataloader');
const uniqWith = require('lodash/uniqWith');
const isEqual = require('lodash/isEqual');

async function getAddresses(addressIDs) {
  // *Remove duplicate address IDs.*
  const uniqueAddressIDs = uniqWith(addressIDs, isEqual); // *Each address is loaded once.*
  const addresses = await Promise.all(
    uniqueAddressIDs.map(async addressID => loadAddress(addressID))
  ); // *Return addresses in the same order of the addressIDs.*
  return addressIDs.map(addressID => {
    const index = uniqueAddressIDs.findIndex(
      uniqueAddressID => isEqual(uniqueAddressID, addressID)
    );
    return addresses[index];
  });
}

const addressDataLoader = new DataLoader(getAddresses);const resolvers = {
  Address: {
    id: address => address.id,
    line1: address => **addressDataLoader.load(address.id)**.line1,
    line2: address => **addressDataLoader.load(address.id)**.line2,
    city: address => address.city,
    state: address => **addressDataLoader.load(address.id)**.state,
  },
  // ...
}
```

这就是了。通过几行代码和使用 DataLoader，我们可以重用来自 REST API 响应的数据(`id`和`city`),并且只在必要时对每个唯一 ID 最多调用一次`loadAddress()`。

[](/how-to-avoid-n-1-problem-in-apollo-federation-8b4f37729fc4) [## 如何避免阿波罗联邦中的 N+1 问题

### 如果不小心的话，联邦中也会有同样的陷阱。

itnext.io](/how-to-avoid-n-1-problem-in-apollo-federation-8b4f37729fc4)