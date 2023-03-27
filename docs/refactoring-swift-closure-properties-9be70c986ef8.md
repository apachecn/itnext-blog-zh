# Swift 中的重构:闭包回调

> 原文：<https://itnext.io/refactoring-swift-closure-properties-9be70c986ef8?source=collection_archive---------5----------------------->

## 当使用闭包时，闭包的内容通常只包含一行:对使用参数执行工作的函数的调用。这需要三行代码，对于它的功能来说，太多了两行。

![](img/e3adbf859170fc2effce149a851b52cd.png)

是的，我们会谈到 currying，但不是这些成分…

你以前见过这个吗，可能是在`viewDidLoad`里？

```
self.model.intakeValueChanged = { [weak self] value in
    self?.updateIntake(to: value)
}

self.model.calorieValueChanged = { [weak self] calories in
    self?.updateCalories(to: calories)
}

self.model.dataIsValid = { [weak self] valid in
    self?.updateSaveButton(active: valid)
}

self.model.saveActionCompleted = { [weak self] in
    self?.resetData()
}
```

虽然这并不可怕，但我所能看到的是一个重复的模式，它将被这个正则表达式匹配。当我们在代码中看到重复的模式时，我们该怎么办？我们重构。

开箱即用，Swift 允许您将一个功能分配给一个闭包属性。因此，我们可以这样做来进行重构:

```
self.model.intakeValueChanged = self.updateIntake(to:)self.model.calorieValueChanged = self.updateCalories(to:)self.model.dataIsValid = self.updateSaveButton(active:)self.model.saveActionCompleted = self.resetData
```

这是合法的，然而警钟应该在你的脑海中响起:我们的`weak`怎么办？

****

在第一个例子中，在代码运行后调用`print(CFGetRetainCount(self))`会导致`7`被打印到控制台，然而从第二个例子的底部调用它会打印`8`。我们创造了一个保留周期。

输入我的同事告诉我的所谓的“函数 currying ”,我不相信他，所以我谷歌了一下(这是几年前…)。在这种情况下，currying 意味着我们可以将我们的实例函数转换为一个以实例为参数并返回原始函数的函数。

> `**self.someFunc(arg:) == type(of: self).someFunc(self)**`

例如，假设我们的`updateIntake(to:)`函数位于`ViewController`内部:

```
class ViewController: UIViewController {

    func updateIntake(to value: Int) {
        ...
    }
}
```

要从`ViewController`内部调用`updateIntake(to:)`，我们通常只需使用`self.updateIntake(to: someInt)`。然而，这种说法的“俗套”方式是:

```
let function: ((Int) -> ()) = ViewController.updateIntake(self)
function(someInt)
```

这是因为静态版本的`updateIntake(to:)`接受了一个`ViewController`的实例，并返回一个与其实例版本相同的函数。为了清楚起见，我在上面的代码片段中添加了类型声明——您所需要的就是`ViewController.updateIntake(self)(someInt)`。

因此，可以使用一个简单的函数在实例函数和闭包之间进行映射，而不会增加`self`的引用数。这是一个适用于只有一个参数而没有返回值的函数的版本:

```
protocol Bindable: class {}
extension Bindable {

    typealias Function = Self

    func weak<Args>(_ method: @escaping ((Function) -> ((Args) -> Void))) -> ((Args) -> Void) {
        return { [weak self] arg in
            guard let `self` = self else { return }
            method(self)(arg)
        }
    }
}
```

为了使它更加可重用，我定义了一个名为`Function`的 typealias，它只是映射到类名。让你的班级符合`Bindable`，你就可以开始了:

```
self.model.intakeValueChanged = weak(Function.updateIntake(to:))self.model.calorieValueChanged = weak(Function.updateCalories(to:))self.model.dataIsValid = weak(Function.updateSaveButton(active:))self.model.saveActionCompleted = weak(Function.resetData)
```

您需要在`Bindable`中添加另一个不带`Args`的函数，以便支持最后一行，因为`saveActionCompleted` / `resetData`没有任何参数。您还可以为具有返回值的闭包创建一个`unowned`版本。

再次运行之前的`print(CFGetRetainCount(self))`测试，注意这次保留计数没有增加！