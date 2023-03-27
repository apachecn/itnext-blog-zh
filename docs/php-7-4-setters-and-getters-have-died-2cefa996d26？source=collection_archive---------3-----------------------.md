# PHP:setter 和 Getters 已经死亡

> 原文：<https://itnext.io/php-7-4-setters-and-getters-have-died-2cefa996d26?source=collection_archive---------3----------------------->

## 暗示的属性在这里停留

![](img/1c8e6b4fbe8119306064e3d61b9567be.png)

塞缪尔·泽勒在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

PHP 非常灵活，但有时开发人员为了正确使用他们的软件，不想让他们的库有太多的灵活性。不是因为他们想完全控制如何使用它，而是为了保护其他开发人员不会搞砸 Github 上的一个好问题并指责作者。

Getters 和 Setters 是控件的一部分。它们的目的非常明确:只允许在类中设置某种类型的变量，并检索开发人员(安全地)期望的东西。

```
<?phpclass Car
{
    /**
     * Car Engine
     *
     * @var CarEngine
     */
    protected $engine /**
     * Returns the Engine instance
     *
     * @return CarEngine
     */
    public function getEngine() : CarEngine
    {
        return $this->engine;
    } /**
     * Sets a Car Engine
     *
     * @param CarEngine $engine
     * @return void
     */
    public function setEngine(CarEngine $engine)
    {
        $this->engine = $engine;
    }}
```

现在他们已经死了。

# 他们是怎么死的？你在说什么？

PHP 7.4，[定于今年](https://wiki.php.net/todo/php74)11 月 21 日发布，[引入了类型化属性](https://wiki.php.net/rfc/typed_properties_v2)。简而言之，类型化属性只允许在 Class 属性中设置某种类型的变量。

例如，假设我们希望`Car`类允许任何实现`CarEngine`接口的类，比如`GasEngine`或`ElectricEngine`。到目前为止，我们需要为每个属性设置一个 getter 和 setter，以迫使开发人员只使用这些类型，正如您在上面的例子中看到的。

为了避免直接干预这些属性，我们使用了`protected`关键字。这意味着，只能从类方法或扩展它的其他类中访问这些属性，而不能从外部访问。

然而，如果我们使用类型化的属性，那么类将只允许将该类型设置为属性，从而扼杀了 getter 和 setter 的用途。我们不需要用它作为`protected`属性，只需设置为`public`即可。

```
<?phpclass Car
{
    /**
     * Car Engine
     *
     * @var CarEngine $engine
     */
    public CarEngine $engine; // ...}
```

而且我们不限于阶级。你也可以把它用于最常用的字符串、数组或浮点类型的变量。

这有两个好处:

*   将类精简到最基本的功能方法。
*   开发人员可以直接访问属性，而不是浪费时间来检索和设置它。
*   绕过使用 getter 或 setter 可能导致的任何方法开销。

现在，让我们使用我们的类。

```
<?php$car = new SportsCar(new GasEngine, new Wheels);$car->wheels->color = 'red';
$car->spoiler = null; // Comes with a default but we don't need itif ($car->engine->start()) {
    $car->lights->on = true;
}
```

让我们用老方法来比较一下:

```
<?php$car = new SportsCar(new GasEngine, new Wheels);$car->getWheels()->setColor('red');
$car->setSpoiler(null); // Comes with a default but we don't need itif ($car->getEngine()->start()) {
    $car->getLights()->setOn(true);
}
```

在那个例子中，我们只使用了一个方法调用，而不是七个。我们基本上**降低了 86%的开销**。

应用程序和框架需要一段时间来理解这一点，因为这意味着在正确地*键入*属性的同时公开一些属性，并删除类中的 getters 和 setters。由于这是一个突破性的改变，预计一些包只兼容 PHP 7.4 及以后的版本。

# 界面困境

还有另一个问题，或者说是两难的问题，需要更好的措辞。我们在没有接口的情况下使用了后面的例子，但是一些类必须使用它们，因为它们*需要*它们。

通常一些接口实现 getters 和 setters，以注意它应该有一个对象可用于设置和检索。我不认为这是一个好的实践，因为这些只是接口不打算*解决*的问题。接口是行为的契约，而不是形状，但是，嘿，这是 PHP，你可以自由地做你想做的。

```
<?phpinterface Car {
    public function getEngine();
    public function setEngine(CarEngine $engine); public function startEngine();
}
```

那么，为什么不在接口中使用公共属性呢？猜猜看:**接口没有属性**。如果实现一个接口的类可以为它自己的类型化属性释放 setter 和 getter，那么用这个类替换同一个接口的另一个类将是不可靠的，因为不能保证另一个类声明了相同的属性名。在这些情况下，您必须使用 getter 和 setter。

比如，`SportsCar`用`$engine`做引擎，而`ElectricCar`用两个引擎，分别保存在`$frontEngine`和`$backEngine`里。

```
<?php$sportsCar = new SportsCar;
$sportsCar->engine->start(); // true$electricCar = new ElectriCar;
$electricCar->engine->start(); // ERROR, the property doesn't exists
```

相反，我们应该这样做:

```
<?phpinterface Car
{
    public function startEngine();
}class ElectricCar implements Car
{
    protected Engine $frontEngine;
    protected Engine $backEngine; public function __construct(ElectricEngine $engine)
    {
        $this->frontEngine = $engine;
        $this->backEngine = clone($engine);
    } public function startEngine()
    {
        $this->frontEngine->start();
        $this->backEngine->start();
    }
}$car = new ElectricCar(new ElectricEngine);$car->startEngine();
```

同样，接口应该更关心它的公共行为，而不是属性的 getters 和 setters，这很好，但这意味着如果你想让开发人员处理它们，你不能(也不应该)依赖于直接访问对象属性。

> 就个人而言，我会避免访问实现某些东西的类的类型化属性。如果我把这个类换成另一个类，并且直接访问新类没有的属性，代码可能会出错，因为接口的全部意义就是使用它的方法来处理这个类应该提供的功能。

随着类型提示属性开放季节的到来，开发人员肯定会对此进行一些讨论，因为为了精简代码，它强制 PHP 7.4 或更高版本。但是如果你为你自己的类使用类型化属性，你应该是安全的。