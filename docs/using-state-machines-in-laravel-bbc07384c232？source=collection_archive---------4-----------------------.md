# 在 Laravel 中使用状态机

> 原文：<https://itnext.io/using-state-machines-in-laravel-bbc07384c232?source=collection_archive---------4----------------------->

“国家机器”

这会让你的身体对高级计算机科学理论感到恐惧吗？又是那种“总有一天我会深究一下”的感觉？

不要害怕！我将向您介绍一个非常简单但实用的用例，它会让您想知道没有它们您是如何度过的。

让我们先简单描述一下这是怎么回事。您是否曾经遇到过这样的问题单，即“该订单的状态为‘已取消’，但后来不知何故又变成了‘待处理’。请调查一下。”？

这就是状态机的设计目的。它们是**你内部状态变化的验证规则。**这就是你现在需要理解的全部内容！

我们所做的是，通过使用代码库，定义流程中所有有效的状态变化。然后，每当您想要更新该实体的状态时，首先通过您的规则集运行它，以检查它是否正常。通过这种方式，随着越来越多的业务逻辑被添加，您总是有一个潜在的安全检查，即您的业务工作流被遵循。

这里有一个非常简单的例子，您可以在默认的 Laravel 安装上试用，而无需太多代码。我们有一个例子，我们的`Users`是军人，每个人都有一个军衔。以下是业务规则:

1.  总军衔名单是:列兵、下士 A 级、下士 B 级、下士 C 级、中士 A 级、中士 B 级
2.  规则是:

A)二等兵可以成为三个职业中任何一个的下士。

b) A 级下士或 B 级下士只能成为 A 级中士；同样，下士 B 级或下士 C 级只能成为中士 B 级(因此下士 B 级可以成为任何一种类型的中士)。

c)没有降级。

要设置这个，添加一个字符串字段“rank”到你的标准安装`Users`表，然后 composer 安装[https://github.com/sebdesign/laravel-state-machine](https://github.com/sebdesign/laravel-state-machine)

我们刚刚安装的实际上只是一个真正逻辑上的 Laravel 的服务提供者，在[https://github.com/winzou/state-machine](https://github.com/winzou/state-machine)库中找到，但是我们现在不需要担心这个。如果您确实想在没有 Laravel 的情况下使用状态机，那么直接使用 git repo。

下一步是发布:` PHP artisan vendor:publish—provider = " SEB design \ SM \ service provider " `。这将在`config/state-machine.php`创建一个配置文件，我们将在这里设置“验证规则”。

```
return [
    'rank_graph' => [
        'class' => App\User::class,

        'property_path' => 'rank',

        'states' => [
            'Private',
            'Corporal A Class',
            'Corporal B Class',
            'Corporal C Class',
            'Sergeant A Class',
            'Sergeant B Class',
        ],

        'transitions' => [
            'promote_private_to_corporal_a_class' => [
                'from' => ['Private'],
                'to' => 'Corporal A Class',
            ],
            'promote_private_to_corporal_b_class' => [
                'from' => ['Private'],
                'to' => 'Corporal B Class',
            ],
            'promote_private_to_corporal_c_class' => [
                'from' => ['Private'],
                'to' => 'Corporal C Class',
            ],
            'promote_corporal_a_class_sergeant_a_class' => [
                'from' => ['Corporal A Class'],
                'to' => 'Sergeant A Class',
            ],
            'promote_corporal_b_class_to_sergeant_a_class' => [
                'from' => ['Corporal B Class'],
                'to' => 'Sergeant A Class',
            ],
            'promote_corporal_b_class_to_sergeant_b_class' => [
                'from' => ['Corporal B Class'],
                'to' => 'Sergeant B Class',
            ],
            'promote_corporal_c_class_sergeant_b_class' => [
                'from' => ['Corporal C Class'],
                'to' => 'Sergeant B Class',
            ],
        ],
    ],
];
```

我们来分析一下。对于状态机，单个规则集被称为“图”。我们的图叫做‘秩图’;对我们来说，它是数组中的键值，仅此而已。我们可能有其他集合来描述系统的其他部分，它们可以添加到下面的同一个配置数组中。

“类”是我们正在检查的模型；“property_path”我们保存各种等级、状态的字段，无论你想怎么称呼它们。顺便说一句，这些不一定是雄辩的模型。“States”是所有可能等级的完整列表——从某种意义上说，它只是一个枚举。

“过渡”是奇迹发生的地方。我们设置了所有*允许的*等级变化——“过渡路径”。每个验证规则都是一个嵌套的数组，它具有 1)一个标签(通常是对所检查的更改的清晰描述)和 2)一个“从/到”关系，该关系将允许哪些“从”状态更改为哪些“到”状态。例如，三个“promote _ private _ to _ 下士 _*_class”转换表明，如果用户当前是“private ”,他们可以被更改为三个下士军衔中的任何一个。然而，他们**不能**成为中士，因此这两个军衔不包括在“to”数组中。

既然我们已经建立了验证规则，我们需要确保我们的更新通过它们。有许多不同的方法来设置它(我将在最后提到一些其他的方法)，但是一个非常简单的方法是在你的模型中添加一个函数。有三种不同的语法可用于此目的:

```
// Style 1: **Get the factory from the interface**use SM\Factory\FactoryInterface;
public function stateMachine()
{
    $factory = app(CallbackFactoryInterface::class);
    return $factory->get($this, 'rank_graph');
}// Style 2: **Or get the factory from the alias**use SM\Factory\FactoryInterface;
public function stateMachine()
{
    $factory = app('sm.factory');
    return $factory->get($this, 'rank_graph');
}// Style 3: **Or get the state machine directly from the facade**use Sebdesign\SM\Facade as StateMachine;
public function stateMachine()
{
    return StateMachine::get($this, 'rank_graph');
}
```

这与我们在上面的配置中设置的名称是同一个数组键。这样，我们可以从用户实例调用状态机，并检查和更新逻辑:

```
$user->update($input);
$user->stateMachine()->apply('promote_private_to_corporal_a_class');
$user->save();
```

`update()`和`save()`是标准的 Laravel，用于任何其他可能更新的字段。中间一行是检查 state-machine.php 配置的地方，以查看是否允许这种特定的更改。在内部，它将检查当前的等级，查看我们试图更新到的等级，然后要么允许它通过，要么抛出一个 SMException 错误。

就是这样，真的！你能做的还有很多，但这将保证你的安全。接下来要做的是为您的各种状态设置一系列单元测试，例如:

```
/** @test */
public function it_can_promote_private_to_corporal_b_class()
{
    $user = factory(User::class)->create(['rank' => 'Private']);

    $user->update(['rank' => 'Corporal B Class']);
    $user->stateMachine()->apply('promote_private_to_corporal_b_class');
    $user->save();

    $updated_user = User::find($user->id);
    $this->assertEquals('Corporal B Class', $updated_user->rank);
}
```

就像我们在配置文件中所做的转换一样，这些转换彼此非常相似，不需要花费太多精力来设置。他们将帮助我们确保，如果我们确实需要改变我们的状态机逻辑，一切都继续工作。

**更多阅读和高级用法:**

上面的例子非常简单，但是在我们的一些站点上已经有了实际的产品(当然，有了实际的项目领域逻辑)。然而，一旦你设置了这个实验，你可能会发现你想更好地利用这个库。如果是这样，这里有一些更复杂的用例，您可以继续研究:

[https://github.com/sebdesign/laravel-state-machine](https://github.com/sebdesign/laravel-state-machine)我们用来和拉勒维尔合作的图书馆。您可以看到合并 Laravel Gates 和策略以及添加回调的更多示例。

[https://gist . github . com/iben 12/7 e 24 b 695421d 92 CBE 1 FEC 3 EB 5 f 32 fc 94](https://gist.github.com/iben12/7e24b695421d92cbe1fec3eb5f32fc94)Bence Ivady[的完整构建示例](https://medium.com/u/9382cd37c64c?source=post_page-----bbc07384c232--------------------------------)。他还创造了一个特征来代替我上面展示的`stateMachine`函数。

[https://github.com/winzou/state-machine](https://github.com/winzou/state-machine)底层状态机库由[https://twitter.com/a_bacco](https://twitter.com/a_bacco)这是框架不可知的；继续在一个简单的 php 文件中使用它！

[https://en.wikipedia.org/wiki/Finite-state_machine](https://en.wikipedia.org/wiki/Finite-state_machine)为了你们当中更勤奋好学的人；这是学习一般状态机的起点。

希望有所帮助！