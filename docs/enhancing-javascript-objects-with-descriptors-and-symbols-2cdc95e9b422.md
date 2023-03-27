# 用描述符和符号增强 JavaScript 对象

> 原文：<https://itnext.io/enhancing-javascript-objects-with-descriptors-and-symbols-2cdc95e9b422?source=collection_archive---------2----------------------->

![](img/f1162a1d6f53617d8b90d664ca87454f.png)

[Thor Alvis](https://unsplash.com/@terminath0r?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上发布的“棕色和黑色城市景观 3D 地图”

不久前，我和我的同事遇到了一个相当有趣的问题，但我们无法立即解决。我们非常广泛地使用[不可变库](https://facebook.github.io/immutable-js/)——尤其是记录，因为它们允许点存取器和对象析构。

假设我们有以下形状的记录:

```
const record = Record({
    a: 1,
    b: 2,
    c: 3
})();
```

我们希望从该记录中提取一个属性，并将其余的属性组织到一个单独的对象中，因此我们像往常一样进行了析构:

```
const { a, ...rest } = record;
```

你知道我们所做的有什么问题吗？我们没有。直到大量的挖掘之后，我们才能够想出为什么`rest`总是`undefined`。

我们发现，答案是，虽然不可变记录允许点访问和析构，但是记录上的属性没有一个是*可枚举的*，猜猜`...` object rest 操作符将哪种属性组合在一起？没错，可枚举属性。所以我们永远不会用这个方法得到一个不是`undefined`的对象。

但是在整个调试过程中，我陷入了一个相当大的兔子洞，围绕着如何使用 vanilla JS 配置对象的属性和定制其功能。有了这些知识，我们可以重新创建许多不可变记录的行为，甚至可以极大地增加对象默认行为的基本功能。

为了使用一个具体的示例，我们将创建一个可能的数据库条目，其中包括计算的属性、访问和设置属性的预处理和后处理，以及一些元编程特性，以使我们的条目对其执行上下文有一些自我意识。

# 入门指南

首先，我们将考察我们的条目的特性。最好能实现一个(n):

*   身份
*   名字
*   上次访问日期
*   上次修改日期
*   出生年月日
*   severely subnormal 智力严重逊常

因此，让我们创建数据库条目:

```
const dbEntry = Object.create( null );
```

我选择使用`Object.create`而不是更熟悉的`{}`语法，因为前一个条目允许我们指定对象的原型应该是什么，通过传入`null`，我解除了这个对象与任何原型的链接。实际上，这个对象除了我们在它上面定义的功能之外，没有任何其他功能。

# 属性数据描述符

属性数据描述符是分配给对象属性的对象(每个属性一个描述符),它指示 JavaScript 引擎将如何处理该属性。数据描述符可以有四个键:

*   `value`:我们希望属性的实际值(默认值`undefined`)
*   `enumerable`:属性是否应该出现在枚举对象键的操作中，比如`for...in`循环或者`Object.keys()`(默认为`false`)
*   `configurable`:指示我们是否可以在以后更改描述符设置或删除对象的属性(默认值`false`)
*   `writable`:告知属性值是否可以更改(默认`false`)

现在，当我们通常设置一个对象的属性时，这些值是自动建立的，从而使我们能够完全控制该属性。

```
const obj = {};
obj.a = 1;Object.getOwnPropertyDescriptor(obj, 'a');
// { value: 1,
//   writable: true,
//   enumerable: true,
//   configurable: true }
```

那么，当我们设置一个属性时，我们如何着手限制可以做的事情呢？使用`Object.defineProperty`并传入我们想要的配置。

所以让我们在我们的`dbEntry`对象上设置一个 ID。因为这代表了它在数据库中的主键，一旦它被设置，我们就不能更改它，但是如果我们遍历条目的键，我们可能希望能够访问它。因此，我们将执行以下操作:

```
Object.defineProperty(
    dbEntry,              // Object we're defining property on
    'id',                 // Property name
    {                     // Property descriptor
        value: 1,
        enumerable: true
    }
);
```

我们将`dbEntry.id`的值设置为 1(当然，不管它的实际主键是什么，它都会是 1)，并指定我们希望能够枚举它。我们不需要传入`configurable`或`writable`标志，因为当我们定义一个属性并且没有指定描述符设置时，JavaScript 将使用上面指定的各自的默认值(在这些情况下是`false`)。从现在开始，我将指定所有的描述符选项，即使缺省值是我想要的。

让我们也为 SSN 和生日添加属性:

```
Object.defineProperties(
    dbEntry,                      // Object to define props on
    {                             // Object of props and descriptors
        ssn: {
            value: '123-45-6789',
            writable: false,
            enumerable: true,
            configurable: false
        },
        birthdate: {
            value: '01/21/1980',
            writable: false,
            enumerable: true,
            configurable: false
        }
    }
);
```

这一次我们使用了`Object.defineProperties`方法来同时添加多个属性，而不是必须调用`Object.defineProperty`两次。第二个参数只是一个对象，其键将成为指定对象的属性，其值是描述符。

# 属性访问器描述符

虽然我们已经可以通过使用数据描述符对对象进行一些很好的控制(*例如*，增强不变性，确定一个键有多“可访问”)，但是如果我们从数据描述符(相当静态)转移到访问器描述符(允许我们拥有相当动态的对象属性)，我们仍然可以发挥相当多的额外能力。

访问器描述符和数据描述符之间的主要区别在于，访问器用`get`和`set`函数替换了早期的`value`和`writable`配置标志。

让我们用它们来定义一个名称属性。在后台，我们希望分别存储名、中间名和姓，但向用户显示实际的全名。这允许数据库有选择地搜索我们想要的名字的任何部分，同时允许用户看到更可读的形式。这也是使用`get`和`set`的完美方式。

在我们开始使用它们之前，我们需要指出一个要点。从某种意义上说，getters 和 setters 是*虚*。这意味着，如果我们有一个用`get`和`set`访问器定义的`obj.prop`，那么`get`和`set` *必须使用不同的存储位置来保存我们正在获取/设置的实际数据。*

这样做的原因是，如果我们在代码中的任何地方调用`obj.prop`，JavaScript 引擎将调用在其上定义的`get`函数，但是因为`get`从`obj.prop`提取数据，我们进入了`obj.prop`的另一个循环，调用它的 getter，然后 getter 再次调用自己，*无限地调用*。当我们进入一个无限递归的函数调用集时，这导致了一个`Maximum call stack size exceeded`错误。

根据上面的警告，让我们在`dbEntry`上定义一个新的属性来存放“后端”数据，供我们所有的 getter/setter 使用。

```
Object.defineProperty(
    dbEntry,
    '_',
    {
        value: {},
        writable: false,
        enumerable: false,
        configurable, false
    }
);
```

在这里，我们将创建`firstname`、`middlename`和`lastname`属性来实际存储名称数据。

```
Object.defineProperties(
    dbEntry._,
    {
        firstname: {
            value: 'John',
            writable: true,
            enumerable: false,
            configurable: false
        },
        middlename: {
            value: 'Jack',
            writable: true,
            enumerable: false,
            configurable: false
        },
        lastname: {
            value: 'Doe',
            writable: true,
            enumerable: false,
            configurable: false
        }
    }
);
```

做完这些，我们终于可以整合我们的财产了。为了保持代码简单，我们假设每个传入的名字都有一个名、中间名和姓。我们还会设置`configurable: true`；这一点以后会很重要。

```
Object.defineProperty(
    dbEntry,
    'name',
    {
        enumerable: true,
        configurable: true,
        get() {
            const { firstname, middlename, lastname } = this._;
            return `${firstname} ${middlename} ${lastname}`;
        },
        set(newName) {
            const [
                firstname,
                middlename,
                lastname
            ] = newName.split(' ');
            this._.firstname = firstname;
            this._.middlename = middlename;
            this._.lastname = lastname;
        }
    }
);
```

现在，如果我们检索`dbEntry.name`，我们将得到“约翰·杰克·多伊”,如果我们设置`dbEntry.name = 'John Stuart Mill'`,我们可以看到我们在`dbEntry._`中的姓名数据确实发生了变化。

关于访问器描述符的一个非常好的事情是它们考虑到了副作用。在`get`或`set`函数中，我们可以执行 API 调用，记录到控制台，或者在每次调用时对输入/输出(仅举几个例子)进行某种处理。

让我们开始吧。每当设置或检索`dbEntry.name`时，我们将在该对象上设置各自的`lastAccessed`和`lastModified`属性。因为我们说过`dbEntry.name`仍然是可配置的，所以添加这个功能只是重新定义它的属性描述符。

```
Object.defineProperties(
    dbEntry,
    {
        lastAccessed: {
            value: Date.now(),
            writable: true,
            configurable: false,
            enumerable: true
        },
        lastModified: {
            value: Date.now(),
            writable: true,
            configurable: false,
            enumerable: true
        },
        name: {
            enumerable: true,
            configurable: false,
            get() {
                this.lastAccessed = Date.now(); const { firstname, middlename, lastname } = this._;
                return `${firstname} ${middlename} ${lastname}`;
            },
            set(newName) {
                this.lastModified = Date.now(); const [
                    firstname,
                    middlename,
                    lastname
                ] = newName.split(' ');
                this._.firstname = firstname;
                this._.middlename = middlename;
                this._.lastname = lastname;
            }
        }
    }
);
```

现在，每当我们检索或设置条目的名称时，我们将启动一个副作用来更新上次修改或访问记录的时间。另外，我们在 name 属性上设置了`configurable: false`,所以现在我们不能再修改它了。此外，如果我们没有使`birthdate`和`ssn`不可配置，我们可以将它们改为 getters 和 setters，以类似地更新`lastAccessed`和`lastModified`。

# 用符号添加元编程

我们已经展示了许多有趣的方法，可以让我们的对象做比我们通常只使用典型赋值所做的更多的事情(*例如* `obj.a = 3`)，事实上我们可以在 getter 或 setter 中做任何我们想做的事情，这是一个非常强大的特性。

但是我们可以通过添加众所周知的符号来进一步改善我们的数据库条目。我不会深入什么是符号，但是众所周知的(*即*，全局)符号允许对象的行为根据它所在的代码的上下文而改变。

## 符号迭代器

在之前的文章中，我谈到了如何使用`Symbol.iterator`使对象可迭代。这将是一个方便的功能，所以让我们打开可迭代性:

```
Object.defineProperty(
    dbEntry,
    Symbol.iterator,
    {
        value: function* () {
            yield this.id;
            yield this.name;
            yield this.birthdate;
            yield this.ssn;
            yield this.lastAccessed;
            yield this.lastModified;
        }
    }
);
```

现在，我们可以通过编程以明确定义的顺序循环遍历我们的键。注意，我们也可以使用`Object.keys`而不是定义`Symbol.iterator`，但是因为`Object.keys`不保证顺序，所以当涉及到我想在这个基础上构建的特性时，它对我们来说用处不大。注意，基于`Symbol`的属性是不可写、不可枚举、不可配置的——即使您在描述符中将这些标志作为`true`传递。

我想拥有 iterability 的真正原因是因为有一个非常简洁的`Symbol`,它允许我们根据 JavaScript 引擎提供的“提示”(由对象上正在执行的操作类型决定的提示)将对象强制转换为字符串或数字原语。

## 符号. toPrimitive

在使用 Python 和 Ruby 的过程中，我一直喜欢能够定义自定义行为来将两个类添加在一起，这是朝着同一个方向迈出的一步。所以，让我们定义如何初始化我们的对象。

```
Object.defineProperty(
    dbEntry,
    Symbol.toPrimitive,
    {
        value: function (hint) {
            if (hint === 'string') {
                return [ ...this ].join(', ');
            } return NaN;
        }
    }
);
```

现在，当我们的`dbEntry`发现自己处于一个“字符串化”环境中时，它将返回一个逗号分隔的列表，其中包含我们在`Symbol.iterator`属性中定义的所有属性。据我所知，唯一这样的环境就是字符串插值(*即*，执行类似``${dbEntry}``的东西)。

所有其他情况将返回`NaN`。这是有意义的，因为为像`dbEntry + 1`这样的东西定义行为实际上没有多大意义，尽管如果有意义，我们会想为`hint === 'number'`添加一些东西。

# 兜了一圈

我们最初开始这次跋涉是因为我们不能在对象析构中使用`...` rest 操作符。现在让我们看看当我们尝试使用它时会得到什么:

```
const { id, ...rest } = dbEntry;id;   // 1
rest; // {
      //     birthdate: '01/21/1980',
      //     lastAccessed: 1540940075708,
      //     lastModified: 1540940079708,
      //     name: 'John Stuart Mill',
      //     ssn: '123-45-6789'
      // }
```

注意，我们的`_`属性和我们熟知的符号都不在我们的`rest`对象中，因为它们都是不可枚举的。然而，有可能把它们从我们的物体上专门摘下来。

```
const { [Symbol.iterator]: iterator, _ } = dbEntry;iterator; // f* () { ... }
_;        // {
          //      firstname: 'John',
          //      middlename: 'Stuart',
          //      lastname: 'Mill'
          // }
```

# 最后几句话

关于属性描述符的使用，还有几件事情需要提及。

*   如果您定义了一个可写但不可配置的属性，那么您*可以*实际返回并更新该属性的配置，关闭可写性(以及最后一次设置它的值)。
*   如果你试图给一个属性重新赋值，而这个属性的`writable`标志被设置为`false`，那么操作会无声地失败，除非你处于严格模式下——在这种情况下，你会得到一个`TypeError`。
*   我们可以直接在我们的对象上定义我们众所周知的符号属性，而不是使用`Object.defineProperty`，我们将得到完全相同的属性描述符配置。我们这样做只是为了保持一致性。

在实现我们自己的自定义属性描述符(尤其是访问器)方面有很大的自由度，并且您可以用它们做的事情实际上只受您的想象力的限制。我们没有时间实现的功能将是在我们对属性进行`set`后对数据进行更加精细的后处理(*例如*，数据净化/验证)或对数据进行`get`的预处理(*例如*，检查用户的安全凭证，如果他们没有被授权查看该数据，则屏蔽它)。

但是我们确实看到了足够的东西来展示我们如何用比我们通常在教程中学到的更健壮的东西来升级我们的对象的典型功能。毕竟，很多库都利用了这种功能，那么我们为什么不把它用在自己身上呢？

[这个要点](https://gist.github.com/ryandabler/e9c5f757813894e752f7153f35091657)有一个我们定义的对象及其功能的完整例子，尽管它的形式比我们在文章中给出的稍微有所改变。