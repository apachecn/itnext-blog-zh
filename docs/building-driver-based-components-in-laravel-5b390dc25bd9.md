# 在 Laravel 中构建基于驱动程序的组件

> 原文：<https://itnext.io/building-driver-based-components-in-laravel-5b390dc25bd9?source=collection_archive---------0----------------------->

## 我们来谈谈在 laravel 中构建基于驱动的组件。

![](img/351421fd1269fe32cc2ebcb70e8c92e8.png)

照片由[奥斯汀·尼尔](https://unsplash.com/@arstyy?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

组件化是构建可扩展和可靠的软件系统的好方法。它允许我们构建由解耦的、独立的和可重用的组件组成的大型系统。它为我们提供了一种构建软件系统的即插即用方法。

Laravel 作为一个框架，由大量可重用的组件组成——其中一些是第三方 Symfony 组件——这些组件都是定义良好的，拼凑在一起构成了系统。

# 成分

大多数现代软件系统都是通过组装小型的、独立的、可重用的实体来构建的，这些实体为系统提供特定的服务和功能。从本质上讲，软件组件是一个小单元，通常具有定义良好的接口，这些接口构成了更大系统的组成基础。它将一组相关的功能(或数据)封装到一个可重用的单元中。

## 基于驱动程序的组件

组件通常是在软件系统中实施关注点分离的实体。它们是模块化的，负责向应用程序交付特定的服务，例如，处理 web 应用程序中的状态的会话组件。有趣的是，您可以以某种方式构建组件，允许它们以不同的方式交付服务，同时仍然提供它承诺的相同契约。这是基于驱动程序的组件设计方法。

最核心的是，您在设计组件时考虑到了可扩展性，以某种方式允许其默认行为被实现组件契约的对象所替换。

驱动程序是软件系统的组件契约的具体实现。它提供了一个到底层基础设施的接口，组件的服务构建在底层基础设施上。

这种驱动程序和基于驱动程序的组件的思想被构建到 laravel 中，并且它被框架开箱即用地支持。这是我们想要探索的框架的一个方面，看看我们如何在我们的应用程序中使用这个模式来构建基于驱动的组件。

![](img/d3c6e85bcbcb3bcef86a87a709cb4fbc.png)

**进入管理者行列！**

# 经理

当我们构建基于驱动的组件时，我们需要一种方法来管理它们。我们希望能够创建多个预定义的驱动程序，甚至在应用程序生命周期的后期创建它们。我们希望能够请求特定驱动程序的实例，并且有一个后备驱动程序，当我们没有指定驱动程序时，调用可以代理到这个后备驱动程序。这是一个经理的工作。

> 管理器是管理基于驱动程序的组件的创建的实体。它负责根据应用程序的配置创建特定的驱动程序实现。

管理器的设计思想是组件可以有多个驱动程序(以不同方式实现的组件实例)。使用管理器，组件可以定义创建它支持的驱动程序所需的逻辑。管理器充当组件的创建和定制驱动程序的中枢，是组件的入口。

如前所述，laravel 附带了对管理器的支持，我们希望利用这一点来创建基于驱动程序的组件。让我们更详细地了解一下经理是如何工作的。

# 经理阶层

Laravel 在支持名称空间(`Illuminate\Support\Manager`)中提供了一个抽象管理器类。这个类定义了有用的方法来帮助我们管理我们的驱动程序。首先，您需要`extend`manager 类，并在子类(组件的 manager 类)中定义驱动程序创建方法。

```
use Illuminate\Support\Manager;class FooManager extends Manager
{
    //
}
```

## 创建驱动因素

当然，创建一个基于驱动的组件需要我们能够创建驱动。管理器类定义了一个`createDriver($driver)`方法，它完全按照 tin 上所说的那样，创建一个新的驱动程序实例。该方法接受单个参数；要创建的驱动程序的名称。它假设扩展类已经定义了创建驱动程序的方法。这些创建方法应该具有以下签名:

```
create[Drivername]Driver()
```

其中`Drivername`是经过学习后的驱动程序的名称。

您在 manager 类中定义的驱动程序创建方法应该返回驱动程序的一个实例。

## 获取驱动程序

这就像订购一个优步，你得到一个管理器的实例并在其上调用`driver($driver = null)`方法。基本管理器提供了这个方法，它接受一个可选参数；要获取其实例的驱动程序的名称。然后，管理器通过调用您在管理器类中定义的适当的驱动程序创建方法来为您创建驱动程序。如果您没有将想要获得的驱动程序的名称传递给`driver($driver = null)`方法，它将返回默认驱动程序的一个实例。

## 后备驱动程序

管理器是一个抽象类，它声明了一个必须由扩展管理器类定义的抽象`getDefaultDriver()`方法。当未指定驱动程序时，此方法应返回组件应使用的默认驱动程序的名称。这个后备驱动程序应该作为主要驱动程序。

## 扩展组件

您可以通过调用管理器上的`extend()`方法来添加不是由基于驱动程序的组件预定义的自定义驱动程序。该方法为您提供了一种使用闭包注册自定义驱动程序创建者的方法。当您向管理器请求驱动程序时，它会检查该驱动程序是否存在自定义驱动程序创建者，并调用自定义创建者。注册为定制创建者的闭包在被调用时会收到一个`\Illuminate\Foundation\Application`的实例。

```
protected function callCustomCreator($driver)
{
    return $this->customCreators[$driver]($this->app);
}
```

> 如果尚未创建，这些自定义驱动程序可以覆盖管理器中同名的预定义驱动程序。

如果您想了解基本管理器类的完整实现，请访问 Github。(本文发布时的 Laravel 5.7)

[](https://github.com/laravel/framework/blob/5.7/src/Illuminate/Support/Manager.php) [## laravel/框架

### 在 GitHub 上创建一个帐户，为 laravel/framework 开发做贡献。

github.com](https://github.com/laravel/framework/blob/5.7/src/Illuminate/Support/Manager.php) 

好了，说得够多了，让我们通过在 Laravel 中构建一个简单的基于驱动程序的 SMS 组件来看看经理们的表现。

![](img/077e1438c164095b4f68e823cc2a5aaa.png)

[Joanna Kosinska](https://unsplash.com/@joannakosinska?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

# SMS 组件

我们想建立一个简单的具有多个驱动程序的 SMS 组件。开箱即用，该组件将支持三个驱动程序:一个 Nexmo 驱动程序、一个 Twilio 驱动程序和一个 Null 驱动程序。正如我们将在本文后面看到的，我们还可以扩展组件并为其创建自定义驱动程序。

我们的组件将存在于应用程序的`App\Components\Sms`名称空间中。首先，让我们创建组件的 ServiceProvider:

这将我们的`sms`组件注册为服务容器中的单例组件，并返回组件管理器的实例(`App\Components\Sms\SmsManager)`)。让我们在`config/app.php`快速向 Laravel 注册我们的服务提供商:

```
'providers' => [
    // Other service providers...

    App\Providers\SmsServiceProvider::class,
],
```

接下来，让我们继续定义经理:

首先要注意的是我们的 manager 类如何扩展 Laravel 的 Manager ( `Illuminate\Support\Manager`)。这是在 laravel 中创建基于驱动的组件的第一步。基础管理器类定义了帮助创建和管理我们的驱动程序的逻辑。因为它是一个抽象类，并且声明了一个必须实现的`getDefaultDriver()`方法，所以我们在 manager 类中定义了该方法，并返回了默认的驱动程序:

```
/**
*
** Get the default SMS driver name.
*
** @returnstring
*/
*public* function getDefaultDriver()
{
   return $this->app['config']['sms.default'] ?? 'null';
}
```

您会注意到配置来自于一个配置文件(`config/sms.php`)，我们定义它来保存关于组件的信息。这是一个非常简单的文件，包含每个驱动程序的凭证以及要使用的默认驱动程序。如果没有设置默认驱动程序，我们将退回到`NullDriver`。

为了方便地选择使用哪个驱动程序，manager 类定义了一个返回指定驱动程序实例的`driver($name)`方法。我们为名为`channel($name)`的组件创建了一个方便的方法，该方法调用基本的`driver($name)`方法，并将我们想要获得的驱动程序的名称传递给它。

```
/**
** Get a driver instance.
*
** @paramstring|null *$name
** @returnmixed
*/
*public* function channel($name = null)
{
   return $this->driver($name);
}
```

我们的驱动程序位于`App\Components\Sms\Drivers`名称空间中。为了创建我们的驱动程序，我们定义了 3 个创建方法来创建组件支持的每个驱动程序。所有驱动程序`extends`一个基类`Driver`实现我们组件的契约(`App\Components\Sms\Contracts\SMS`)。该契约声明了一个`send` 方法，该方法必须由组件的所有驱动程序实现。这是组件对系统的服务合同，它承诺能够发送 SMS。

```
<?phpnamespace App\Components\Sms\Contracts;interface SMS
{
  /**
   ** Send the given message to the given recipient.
   *
   ** @returnmixed
   */
   *public* function send();
}
```

让我们看一下`NexmoDriver`，看看我们的组件是如何在内部工作的:

驱动程序实现了`send`方法，并使用 Nexmo PHP 客户端传递消息。

一旦我们设置好我们的组件，也就是说，为我们的组件注册一个 facade 为`SMS`，设置配置文件，并安装我们的依赖项，我们就可以通过获得一个`SmsManager`的实例并在其上调用`send`方法来快速使用组件。

```
SMS::to($phoneNumber)
    ->content('Building driver-based components in Laravel')
    ->send();
```

`to($phoneNumber)`和`content($message)`方法由组件中所有驱动程序扩展的基类`Driver`定义。

这里，我们没有指定要使用的驱动程序，所以它默认为我们的`Nexmo`驱动程序，因为这是我们在组件中设置的默认驱动程序。要指定一个驱动程序，我们可以调用`channel($name)`方法或者基础`driver($name)`方法。

```
SMS::channel('twilio')
    ->to($phoneNumber)
    ->content('Using twilio driver to send SMS')
    ->send();
```

现在，我们已经在 laravel 中成功地创建了一个基于驱动程序的组件。如果你有兴趣看完整的实现，我已经把源代码添加到 GitHub 中了。

[](https://github.com/orobogenius/building-driver-based-components) [## orobogenius/基于建筑驱动因素的组件

### 关于在 laravel 中构建基于驱动的组件的文章的演示报告。-orobogenius/基于建筑驱动因素的组件

github.com](https://github.com/orobogenius/building-driver-based-components) 

现在我们已经创建了组件，我们可以通过扩展组件和添加自定义驱动程序创建者来添加更多的驱动程序。为此，我们获取了一个`SmsManager`的实例，并在其上调用了`extend`方法。但是首先，我们将创建一个实现组件契约的驱动程序:

```
<?phpuse App\Components\Sms\Contracts\SMS;class FooSmsDriver implements SmsContract
{
    protected $someDependency; public function __construct($dependency)
    {
        $this->someDependency = $dependency;
    } *public* function send()
    {
       // Define send logic
    }
}
```

然后，我们将调用 extend 方法，并为其提供定制的驱动程序创建逻辑。

```
SMS::extend('foo', function ($app) {
    return new FooSmsDriver($dependency);
});
```

一旦我们注册了自定义驱动程序创建器，我们现在就可以像使用该组件支持的其他现成驱动程序一样使用它了。

```
SMS::channel('foo')
    ->send();
```

通过测试，我们可以验证该组件能够使用多个驱动程序，包括自定义创建的驱动程序。

# 结论

Laravel 使得使用管理器类创建基于驱动程序的组件变得很容易。我应该很快指出，在构建基于驱动程序的组件时，总是使用这个管理器类并不是一成不变的，您可以构建自己的管理器来处理组件的驱动程序创建和管理。

我希望这篇文章是有帮助的，感谢阅读。