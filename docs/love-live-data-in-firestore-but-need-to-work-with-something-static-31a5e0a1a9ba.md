# 喜欢 Firestore 中的实时数据，但需要处理静态数据？

> 原文：<https://itnext.io/love-live-data-in-firestore-but-need-to-work-with-something-static-31a5e0a1a9ba?source=collection_archive---------4----------------------->

![](img/8978ad4a26b5dea3bd8a29bb73cc829d.png)

奥斯卡·伊尔迪兹在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

我正在开发一个应用程序，我想在单击时向列表项添加一个简单的动画，并应用一些幕后代码将该项添加到 Firestore 的一个组中。

我考虑这样做的方法是在列表项上切换一个类，然后像这样激活该项(注意我使用的是 Angular，但本文适用于所有框架):

```
onClickItem(): void {
   item.isActive = true;
}animations: [
   trigger("isActive", [
      state("true", style({
         backgroundColor: "#08aeea",
         color: "#fff"
      })),
      transition("* => true", animate("150ms ease-in"))
   ])
]
```

好吧，太简单了。我在我的列表上试着这样做，瞧，像魔法一样工作，然后我注意到一些有趣的事情，偶尔我会点击列表项目，动画会运行一半，闪烁，然后恢复正常——这是怎么回事？？？

不打算说谎这确实难倒了我大约一个小时。然后我看了看我是如何在 Firestore 上呈现我的清单的:

```
 this.groups$ = this.getGroupsAsObservable(); 
// groups$ to indicate is an ObservablegetGroupsAsObservable(): Observable<Group[]> {
   return this.afs.collection<Group>("groups").valueChanges();
}
```

> 注意 valueChanges()部分——这是允许你的应用从 firestore 直播数据的秘方

现在发生的是，当点击该项目时，它会在后台检查该项目是否属于某个组，如果不属于，它会将该项目添加到该组，并在 Firestore 中列出该组(一个对象)的相关项目的小块数据。

这意味着该组的文档发生了变化，并将在我们的应用程序中触发重新呈现！

这是我意识到我需要我的组列表是纯静态的地方。

*这实际上有点棘手，文档中没有明确提到“静态”或“读取一次”数据*。

我需要数据的第一件事是返回数据的快照。文件中有一点是这样的:

```
getUsersGroupsOnce(userId): Promise<QuerySnapshot> { 
// return a static snapshot of the users groups return this.afs.collection("groups", ref =>      
      ref.where(`users.${userId}`, "==", true)
   )
   .ref.get(); // this is the static part}
```

很好，现在我有了一个静态引用—我需要为它构建一个数组，按照我做的文档:

```
this.fb.getUsersGroupsOnce(user.uid).then(q => {
   q.forEach(doc => {
     console.log(doc.data())
   });
});
```

好了，现在我有了数据，我只需要把它推到一个数组中，对吗？我试过这个:

```
q.forEach(doc => {
   this.groups.push(doc.data());
});
```

但是 typescript 在抱怨，所以我就像这样强迫它:

```
q.forEach(doc => {
   this.groups.push(doc.data() as Group);
});
```

瞧，我现在有了一个静态的— *或者我喜欢称之为‘read once’*列表。现在我知道 99/100 的实时数据流很棒，但是知道如何不时地获取静态数据也很方便。

有更好的方法吗？欢迎在下方留言评论。