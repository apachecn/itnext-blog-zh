# 多击，多击，多击

> 原文：<https://itnext.io/laravel-multitap-multitap-multitap-2496f2d58174?source=collection_archive---------3----------------------->

## 因为敲打不应该在第一轮就结束

![](img/af2d34c458e11428d90b58262ae919b8.png)

由 [Taras Chernus](https://unsplash.com/@chernus_tr?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

Laravel 包括的一个简洁的助手是`[tap()](https://laravel.com/docs/master/helpers#method-tap)`[方法](https://laravel.com/docs/master/helpers#method-tap)。简而言之，它允许您调用一个对象方法并返回相同的对象而不是方法结果，允许您链接另一个方法。

```
tap($user)->save()->setAttribute('done', true);
```

我不会给你解释它的工作原理和使用方法。更好的是，我将指出泰勒关于 `[t](https://medium.com/@taylorotwell/tap-tap-tap-1fc6fc1f93a6)ap()` [方法](https://medium.com/@taylorotwell/tap-tap-tap-1fc6fc1f93a6)的文章。

我多次面临的问题是`tap()`只对**有效一次**。我的意思是，一旦你调用的方法返回了对象，下一个方法调用就会给出它的结果而不是同一个对象。如果下一个方法结果是`void`，你将得到一个`void`结果。

```
// This will return the method resulttap($object)->doSomething()->getResult();
```

有时我想链接多个方法，而不必一次又一次地调用对象。有时我不能添加 fluent 方法，因为对象来自外部包。

然后我发明了 multitap。

# 无限制攻丝

我决定扩展`HigherOrderTapProxy`类，它是在`tap()`助手下工作的魔法，覆盖 _call 方法。我把它保存为`InfiniteHigherOrderTapProxy`,因为虽然它很长，但有意义。

有了这个类，你可以不断地调用你的类方法，它总是透明地传递给它。如果你想得到一个方法结果，或者完全摆脱敲击，你可以调用任何附加了“AndUntap”的方法，你将得到结果。整洁！

当然，我喜欢有表现力的一行程序，所以我们要在某个地方添加一个函数助手。我更喜欢`/helpers/helpers.php`，但是你可能想使用任何其他的技术来添加你自己的根函数到你的项目中。

```
if (! function_exists('multitap')) {
    /**
     * Makes something infinitely tapped
     *
     * @param mixed $object
     * @return \App\Helpers\InfiniteHigherOrderTapProxy
     */
    function multitap($object)
    {
        return new InfiniteHigherOrderTapProxy($object);
    }
}
```

因此，有了它，你就可以用一个 liners 来处理简单乏味的对象，并流畅地使用它们，即使它的每个方法都只返回对象本身。然后，我将调用最后一个方法，附加“AndUntap”来退出点击。

```
echo multitap($boring)
    ->set(10)
    ->add(2)
    ->remove(5)
    ->getResultAndUntap();// 13
```

因为它扩展了`HigherOrderTapProxy`，您可以选择在任何时候通过指向`target`属性来到达底层对象。

```
echo multitap($boring)
    ->set(20)
    ->add(4)
    ->remove(10)
    ->target->getResult();// 14
```

这差不多就是我如何将我力所不及的任何类转换成一个可多次点击的对象，我可以流畅地链接它，直到我需要为止。