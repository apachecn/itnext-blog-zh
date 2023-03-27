# 更好的开发体验的严格存根

> 原文：<https://itnext.io/strict-stubs-for-better-dev-experience-bb455f733753?source=collection_archive---------3----------------------->

大多数现代的双测试库创建具有默认行为的存根。现在，如果被测试的代码以不同的方式使用存根，测试将会失败——更重要的是，会出现一个难以解释的错误。严格的存根有助于避免这个问题，从而提供更好的体验。[test double-only——当](https://github.com/alexbepple/testdouble-only-when)是与 [testdouble.js](https://github.com/testdouble/testdouble.js) 一起使用的严格存根的助手时。

## 又有什么问题？

大多数存根都有默认行为。在 Java 领域， [Mockito](http://mockito.org/) 存根返回 *null、false、*等。当没有为呼叫排练特定行为时。在 JavaScript 领域， [testdouble.js](https://github.com/testdouble/testdouble.js) 存根和 [Sinon](http://sinonjs.org/) 存根都返回 *undefined，*如果你以未经训练的方式调用它们。

```
import td from 'testdouble'
const stub = td.func()
td.when(stub(0)).thenReturn('foo')
assertThat(stub(0), is('foo'))
assertThat(stub(), is(undefined))
```

这有时甚至会很方便。然而，通常如果你对返回值做了什么，处理它或者传递它， *undefined* 很可能会给你带来问题。

在下面的示例中，被测试的函数 *(fut)* 调用不带任何参数的存根，然后对存根的返回值调用 *substring(1)* 。我们设置断言 *assertThat(fut(stub)，is(' oo ')*，假设存根将返回 *foo。然而，它不会，因为，不管出于什么原因，我们以不同的方式排练了它的行为。因此它将返回*未定义的*并且我们最终得到一个比其他任何东西都更令人困惑的错误消息，因为它没有指出问题的根本原因:*

> TypeError:无法读取未定义的属性“substring”

在可执行的术语中:

```
describe('Problem', () => {
  it('unrehearsed usage fails late because of the consequences of default stub behavior', () => {
    const stub = td.function()
    td.when(stub(0)).thenReturn('foo') const fut = (collaborator) => collaborator().substring(1) assertThat(
      () => assertThat(fut(stub), is('oo')),
      throws(
        typedError(TypeError, "Cannot read property 'substring' of undefined")
      )
    )
  })
})
```

您观察到的错误并不是测试代码行为不当的结果。相反，这个错误是完全不同的事情的结果。—想象你额头发热。你是刚跑完 10 公里还是因为感冒而发烧？

## 如果存根告诉我们它正以意想不到的方式被使用，这不是很好吗？

进入 [testdouble-only-when](https://github.com/alexbepple/testdouble-only-when) 。让我们仅使用而不是 *td.when.* 来排练存根的行为

```
import { onlyWhen } from 'testdouble-only-when'
const stub = td.function()
onlyWhen(stub(0)).thenReturn('foo')
```

现在，使用排练过的存根的行为与前面一样:

```
assertThat(stub(0), is('foo'))
```

然而，以不同的方式调用存根，例如 *stub()* 不再返回 *undefined。*现在结果是:

> 错误:您以意外的方式调用了 test double。这个测试 double 有 1 个存根和 1 个调用。
> 
> Stubbings:
> —用`( 0)调用时，返回`" foo " `。
> 
> 调用:
> —用`()'调用。
> 在…

测试很早就失败了，你可以更准确地知道要寻找什么。

[testdouble-only-when](https://github.com/alexbepple/testdouble-only-when) 几乎可以替代 *td.when(…)。*唯一的例外:多次存根。由于 testdouble.js 存根的定义方式，当您定义多个行为时， *onlyWhen* 无法工作。在那些罕见的情况下，你可以使用 *failOnOtherCalls:*

```
import { failOnOtherCalls } from 'testdouble-only-when'
const stub = td.function()
td.when(stub(0)).thenReturn('foo')
td.when(stub(1)).thenReturn('bar')
failOnOtherCalls(stub)
```

PS:有些人可能还记得，在 Mockito 流行之前，Java 领域的一个主要的双测试库是 EasyMock。它的存根在未经练习的情况下会失败。人们必须明确地[要求 EasyMock stub 变得更好](http://easymock.org/user-guide.html#mocking-nice),这样它才能像其他库中常见的那样表现出默认行为。

如果你愿意，你可以在草稿上阅读这个版本，语法高亮[作为要点](https://gist.github.com/alexbepple/2fdfacbaa58bff914a98dd1b96a7d77d) / [。](http://alexbepple.roughdraft.io/2fdfacbaa58bff914a98dd1b96a7d77d-stricter-stubs-for-better-dev-experience)