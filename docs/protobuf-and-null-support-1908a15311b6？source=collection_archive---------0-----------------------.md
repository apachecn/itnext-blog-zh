# Protobuf 和 Null 支持

> 原文：<https://itnext.io/protobuf-and-null-support-1908a15311b6?source=collection_archive---------0----------------------->

![](img/60701c57eb492c253f3a3c2ef980fe6d.png)

图片由[大卫·马克](https://pixabay.com/users/tpsdave-12019/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=92358)从[皮克斯拜](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=92358)拍摄

假设您有以下协议缓冲区/ gRPC 定义:

```
service MyDataService {
  rpc UpateMyData (UpdateMyDataRequest) 
     returns (UpdateMyDataResponse);
}message MyData {
  int32 id = 1;
  string stringValue = 2;
  SubData subData = 3;
}

message SubData {
 int64 bigValue = 1;
}message UpdateMyDataRequest {
  MyData update = 1;
}
```

现在假设您想要删除 *MyData.stringValue* 的数据库条目

您的第一种方法可能是这样的:

```
UpdateMyDataRequest request = UpdateMyDataRequest.*newBuilder*()
  .setUpdate(MyData.*newBuilder*()
    .setId(id)
    .setStringValue(null)
  )

serviceFutureStub.update(request)
```

只有当你开始运行时，你才会得到一个 NullPointerException。

默认情况下，在协议生成的 MessageTypes 中设置任何值都会引发 NullPointerException。另一方面，所有 get 方法都不会返回 null。如果它们未被设置，get 将返回一个默认值。

```
UpdateMyDataRequest.*newBuilder*().build().getStringValue() == ""
```

## 如何用协议缓冲区发送空值？

让我用我自己的问题来回答你的问题。

**空是什么意思？**

问题是 null 在不同的上下文中可能有不同的含义:

*   空就是空
*   Null 是未设置的/可选的
*   Null 是默认值
*   Null 与其他值相混淆

为了避免这种混淆，Protobuf 团队决定不序列化空值。相反，protobuf 迫使您使用几种显式策略，从而避免 Protobuf / gRPC API 中的任何语义混乱。

在接下来的小节中，我们将讨论上面概述的每一个空用例，以及如何用 Protobuf 来表示它们。

我们将专注于原型 3。Proto2 还有其他的语义，我们在这里就不赘述了。

## 首先是一些基本知识

所有字段都是:

1.  可选择的
2.  从不为空
3.  用默认值(0、空字符串等)初始化

# Null 是 Null:Null 值模式之一

有时，null 是一个有效值。例如，null 可用于从数据库列中删除值。在本例中，假设我们希望允许消费者将 *MyData.stringValue* 设置为 null。

Json 等效 MyData 对象:

```
{
  "id": 123
  "stringValue": null
}
```

正如我们前面提到的，我们不能将一个值设置为 null。因此，我们需要通过其他方式来跟踪空信息。我们可以通过引入可空类型来做到这一点。熟悉[科特林的人会认出这种模式](https://kotlinlang.org/docs/reference/null-safety.html)。

**原型定义:**

```
syntax = "proto3";

package io.github.efenglu.protobuf.examples.oneof;

option java_multiple_files = true;

import "google/protobuf/struct.proto";

service MyDataService {
  rpc UpateMyData (UpdateMyDataRequest) 
     returns (UpdateMyDataResponse);
}

message MyData {
  int32 intValue = 1;
  NullableString stringValue = 2;
  NullableSubData subData = 3;
}

message SubData {
  int64 bigValue = 1;
}

message NullableSubData {
  oneof kind {
    google.protobuf.NullValue null = 1;
    SubData data = 2;
  }
}

message NullableString {
  oneof kind {
    google.protobuf.NullValue null = 1;
    string data = 2;
  }
}

message UpdateMyDataRequest {
 MyData data = 1;
}

message UpdateMyDataResponse {

}
```

您会注意到两种新的“可空”类型:

*   可空字符串
*   NullableSubData

这些类型由一个[或一个](https://developers.google.com/protocol-buffers/docs/proto#oneof)组成，有两个可能的值:

*   空
*   非空对象

oneof 帮助我们强制数据不能同时为空和非空。

下面是 java 客户端如何使用生成的代码:

**发送空值:**

```
UpdateMyDataRequest request = UpdateMyDataRequest.*newBuilder*()
  .setData(MyData.*newBuilder*()
    .setStringValue(NullableString.*newBuilder*()
      .**setNull(NullValue.*NULL_VALUE*)
**      .build()
    )
    .setSubData(NullableSubData.*newBuilder*()
      .**setNull(NullValue.*NULL_VALUE*)**
      .build()
    ).build()
).build();

service.upateMyData(request);
```

注意这里我们调用了 *setNull* 来表明我们有意发送一个空值。

**客户端发送非空值:**

```
UpdateMyDataRequest request = UpdateMyDataRequest.*newBuilder*()
  .setData(MyData.*newBuilder*()
    .setStringValue(NullableString.*newBuilder*()
      **.setData("hello")**
      .build()
    )
    .setSubData(NullableSubData.*newBuilder*()
      .**setData**(SubData.*newBuilder*()
        .setBigValue(1234567)
      .build()
    ).build()
  ).build()
).build();

service.upateMyData(request);
```

注意在这个客户端代码中，我们如何调用 *setData* 来发送实际数据。

**服务器实现:**

```
if (request.hasData()) {

  if (request.getData().hasStringValue()) {
    final String nullableString;
    if (request.getData().getStringValue().hasNull()) {
      nullableString = null;
    } else {
      nullableString = request.getData()
        .getStringValue()
        .getData();
      }
  }

  if (request.getData().hasSubData()) {
    final SubData nullableSubData;
    if (request.getData().getSubData().hasNull()) {
      nullableSubData = null;
    } else {
      nullableSubData = request.getData()
       .getSubData()
       .getData();
    }
  }

}
```

请注意我们如何确保该值为 null/non null，以及客户端实际上已经设置了该值。

**优点:**

*   可空值的类型安全，为可空值创建不同的 MessageType
*   非常明确，确保 null 是一个设定值

**缺点:**

*   需要空值消息类型
*   对许多类型来说不太好

## Null 作为可选:字段掩码模式

这在客户端只需要更新对象的一部分时，或者在创建返回部分填充对象的查询/搜索参数时非常有用。

这里，null 用于表示不应被解释的缺失信息。也就是说，值为空不是因为我们希望它为空，它为空是因为我们不在乎。您通常会在 json 字段的遗漏中看到这一点。

```
{
 "id": 123
 -- ommited "stringValue" --
}
```

我们将对 proto 做类似的事情，只是我们也将明确地告诉服务器我们实际上忽略了哪些字段。

**原型定义:**

```
service MyDataService {
  rpc Update (UpdateMyDataRequest) returns (UpdateMyDataResponse);
  rpc List (ListMyDataRequest) returns (ListMyDataResponse);
}

message MyData {
  int32 id = 1;
  string stringValue = 2;
  SubData subData = 3;
}

message SubData {
  int64 bigValue = 1;
}

message UpdateMyDataRequest {
  MyData update = 1;
  **google.protobuf.FieldMask field_mask = 2;**
}

message UpdateMyDataResponse {
  MyData new_data = 1;
}

message ListMyDataRequest {
  int32 id = 1;
  **google.protobuf.FieldMask field_mask = 2;**
}

message ListMyDataResponse {
  repeated MyData data = 1;
}
```

请注意， *UpdateMyDataRequest* 和 *ListMyDataRequest* 有一个 FieldMask 字段。这是一种特殊的类型，它将传达数据中的哪些字段应该受到关注。

**示例客户端用法:**

```
MyData **sendUpdate**(int id, String value) {
  UpdateMyDataRequest request = UpdateMyDataRequest.*newBuilder*()
    .setUpdate(MyData.*newBuilder*()
      .setId(id)
      .setStringValue(value)
    )
    .setFieldMask(FieldMaskUtil.*fromFieldNumbers*(
      MyData.class, 
      MyData.*STRINGVALUE_FIELD_NUMBER*)
    )
    .build();

  return serviceFutureStub.update(request).getNewData();
}

List<MyData> **listOnlySubData**(int id) {
  ListMyDataRequest request = ListMyDataRequest.*newBuilder*()
    .setId(id)
    .setFieldMask(FieldMaskUtil.*fromFieldNumbers*(
      MyData.class, 
      MyData.*SUBDATA_FIELD_NUMBER*)
    )
    .build();

  return serviceFutureStub.list(request).getDataList();
}
```

**示例服务器实现:**

```
@Override
public void update(
  UpdateMyDataRequest request,
  StreamObserver<UpdateMyDataResponse> responseObserver
) {

  MyData updateData = request.getUpdate();
  FieldMask fieldMask = request.getFieldMask();

  **// Fetch exiting Values**
  MyData existing = repo.readData(updateData.getId());
  MyData.Builder builder = existing.toBuilder();

  **// Update only the fields listed in the fieldmask**
  FieldMaskUtil.*merge*(fieldMask, updateData, builder);

  **// Store the result**
  repo.writeData(builder.build());

  **// Send the new state back**
  responseObserver.onNext(UpdateMyDataResponse.*newBuilder*()
    .setNewData(builder)
    .build()
  );
}
```

**更新中的通知:**

1.  获取我们想要更新的对象的现有值
2.  转变为建设者
3.  使用字段掩码 Util 将输入数据合并到构建器中
4.  存储新状态
5.  返回新值

*FieldMaskUtil* 将只从输入请求中复制字段掩码中列出的字段，并保留任何其他字段的现有值。

```
@Override
public void **list**(
  ListMyDataRequest request,
  StreamObserver<ListMyDataResponse> responseObserver
) {
  int id = request.getId();
  FieldMask fieldMask = request.getFieldMask(); **// Fetch the list**
  List<MyData> result = repo.listData(id);

  ListMyDataResponse.Builder response = 
    ListMyDataRespons.*newBuilder*(); MyData.Builder builder = MyData.*newBuilder*(); for (MyData data : result) {
    builder.clear();

    **// Use the field mask to send back ONLY the data requested**
    FieldMaskUtil.*merge*(fieldMask, data, builder);

    response.addData(builder);
  }

  **// Send the filtered list back**
  responseObserver.onNext(response.build());
}
```

这里有很多相同的，只是我们返回一个过滤值。

1.  获取列表
2.  对于每个列表元素，过滤元素以仅返回请求的字段
3.  返回过滤列表

**优点:**

1.  简明代码
2.  更容易测试

**缺点:**

1.  字段掩码的概念可能很难理解
2.  要求客户手动将字段调出到字段掩码中，这看起来可能是重复的
3.  字段的语义契约可以被打破

## Null 为可选:有模式

最后一种模式是大多数人在谈到 protobuf 时的出发点。非原语消息类型中的每个字段都生成一个“ *has* 方法，该方法返回一个**布尔值**。如果已经设置了值*，则该方法返回 true。我们可以利用这个特性来查看消费者何时“设定了一个值”。然后我们可以推断未设置的字段不重要。*

*现在这只适用于非原始类型，即消息类型。如果您需要原语的这种行为，Proto3 为所有原语类型提供了包装器。*

```
*...

**import "google/protobuf/wrappers.proto";**

service MyDataService {
  rpc Update (UpdateMyDataRequest) returns (UpdateMyDataResponse);
}...message UpdateMyDataRequest {
  int32 id = 1;
  **google.protobuf.StringValue stringValue = 2;**
  UpdateSubData subData = 3;
}

message UpdateSubData {
  **google.protobuf.Int64Value bigValue = 1;**
}...*
```

*注意*Google/proto buf/wrappers . proto*和*Google . proto buf . string value*和*Google . proto buf . int 64 value*的导入。这些字段不再是原语，因此将生成一个“ *has* ”方法。*

***客户端使用:***

```
*void update() {
  service.update(UpdateMyDataRequest.*newBuilder*()
    .setStringValue(**StringValue.*of*("customValue")**)
    .build()
  );
}*
```

*在这里，客户端设置他们想要使用的字段。一个警告是字符串值字段必须用一个 *StringValue* 对象填充，如上所示。*

*熟悉 Java 预自动装箱的人会认识到这种模式。*

***服务器实现:***

```
*@Override
public void update(
  UpdateMyDataRequest request,
  StreamObserver<UpdateMyDataResponse> responseObserver) {

  **// Fetch exiting Values**
  MyData existing = repo.readData(request.getId());
  MyData.Builder builder = existing.toBuilder();

  **// Update Fields as necessary**
  if (request.hasStringValue()) {
    builder.setStringValue(request.getStringValue().getValue());
  }

  if (request.hasSubData()) {
    if (request.getSubData().hasBigValue()) {
      builder.setSubData(
        builder.getSubData().toBuilder()
          .setBigValue(request.getSubData()
            .getBigValue()
            .getValue()
          )
        );
      }
  }

  repo.writeData(builder.build());

  responseObserver.onNext(UpdateMyDataResponse.*newBuilder*()
    .setNewData(builder)
    .build()
  );
}*
```

*服务器实现也非常相似。然而，与将字段合并委托给**字段相反，我们必须手动合并字段。***

1.  *获取现有对象*
2.  *转换为构建器*
3.  *对于每个字段和递归字段，检查 has 并根据需要进行赋值*
4.  *存储值*
5.  *返回新值*

***优点:***

1.  *概念上容易理解*
2.  *小型客户端代码*

***缺点:***

1.  *服务器实现容易中断:错过一个已经，或添加一个字段和合并被打破*
2.  *有许多分支的大型服务器代码*

# *空反模式:默认值*

*我们已经讨论了您“应该”使用的几种模式及其优缺点。现在我们来讨论一个你不应该使用的模式！*

*对照默认值检查该值。*

*你可能会说。*

> *哦，如果值是 default，那么我知道这个值没有被设置，因此是 null。*

***假的！***

*   *消费者可能已经将该值设置为默认值*
*   *Proto3 不允许你提供缺省值，它们只是典型的缺省值(0，"")，因此有点模糊*

*不要自作聪明，把值当值就行了。*

*不要试图从缺省值中为原始类型创建自己的" *has"* "。使用“ *has* ”方法和原始包装器。*

# *空反模式:空字符串*

*看到此代码的图像:*

```
*String value;
if (value != null) {
  // insert value into database
}*
```

*看这里有什么不对。*

*   *空字符串“”的值是什么？*
*   *或者如果值都是空白" "，该怎么办？*

*Protobuf 将字符串视为原始类型，因此它们不能为空。不要检查字符串是否为空，而是使用标准库，如 apache commons，来检查字符串是否为空。*

```
*String value;
if (StringUtils.isNotBlank(value)) {
  // insert value into database
}*
```

*很明显，如果值不为空，将会插入该值。*

# *高级用例*

*我们的团队希望通过我们的“零支持”更进一步。我们创建了定制的协议代码生成插件来定制生成的代码以适应我们的目的。*

*我们增加了对以下内容的支持:*

*   *可选退货类型*

*我们还分叉了协议生成器，以允许:*

*   *get 上的空检查*
*   *允许设置空值来清除该值*
*   *创建了一个"*有"*的方法用于原语类型*

*看看我的另一篇关于如何创建自己的协议插件的文章。*

*[](/customizing-grpc-generated-code-5909a2551ca1) [## 定制 gRPC 生成的代码

### 如何定制 gRPC 生成的代码

itnext.io](/customizing-grpc-generated-code-5909a2551ca1)* 

*总而言之。是的，这是真的，协议缓冲区不支持空值。然而，从大局来看，事情并没有那么糟糕。*

*协议缓冲迫使你问:*

*   *你是如何使用价值的？*
*   *我在这里用 null 想表达什么？*
*   *我可以不用 null 的模糊性来表达这个吗？*

*查看与本文相关的完整示例:*

*[](https://github.com/efenglu/protobuf_null) [## efenglu/protobuf_null

### Protobuf 和空值序列化的示例用例我的 Medium 文章中引用了这些示例。

github.com](https://github.com/efenglu/protobuf_null)*