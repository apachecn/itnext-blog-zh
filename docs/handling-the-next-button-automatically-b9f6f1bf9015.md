# 自动处理下一个按钮

> 原文：<https://itnext.io/handling-the-next-button-automatically-b9f6f1bf9015?source=collection_archive---------1----------------------->

![](img/0924350466b62a01f122f14acdf5330c.png)

照片由 eleven x 在 Unsplash 上拍摄

在多个文本字段中输入文本是如此常见的模式——在任何地方，不仅仅是 iOS——应该有一种方法可以轻松地从一个字段导航到下一个字段，最好是“正确的”字段。遗憾的是，iOS 没有提供这一功能，但让我们看看如何自己实现这一功能。

首先，快速回顾一下我们需要什么:

*   如果当前字段之后有任何字段，请查看“下一步”按钮；最好是那些还空着的。
*   点击“下一步”按钮，我们将进入正确的字段。
*   如果当前字段之后没有更多的字段，请查看返回按钮；最好考虑空的。
*   点击“返回”按钮会退出当前响应者。
*   不管是什么视图，也不管有多少字段，一切都应该自动工作。
*   一切都应该封装在一个协议中。

太好了，我们开始吧，从协议开始。我们在这里需要什么？

*   文本字段列表(1)。
*   一个处理文本字段编辑的方法开始，显示正确的按钮(2)。
*   一种处理文本字段的编辑结束/回车的方法，以执行正确的操作(3)。

```
protocol NextTextFieldHandler { var textFields: [UITextField] { get } // 1 
    func setupReturnKeyType(for textField: UITextField) // 2 
    func handleReturnKeyTapped(on textField: UITextField) // 3 }
```

`textFields`属性(1)可以保存视图中的所有字段，也可以只保存我们感兴趣的字段——协议并不关心这一点，当遵守协议时，我们的工作就是决定需要什么。这两个函数用于满足我们的另外两个需求(2，3)。

```
extension NextTextFieldHandler {     private var _textFields: [UITextField] { 
        return textFields.filter { !$0.isHidden && $0.alpha != 0 && $0.isEnabled } // 1 
    }}
```

我们可以添加一些安全措施，以防我们不小心，这也是一种便利:这些条件将不得不一遍又一遍地编写，因此我们可以自动过滤掉隐藏的字段和禁用的字段(1)。

但是，如上所述，协议不应该关心`textFields`中的内容；不管出于什么原因，我们可能想要包含一个禁用的文本字段，但是没有办法做到这一点。如果你*确实*确定这个项目没有怪异的场景，你可以添加上面的属性并用它代替`textFields`。

接下来，我们需要一种方法来提取当前字段之后的所有字段:

```
extension NextTextFieldHandler {     private func fields(after textField: UITextField) -> ArraySlice<UITextField> { 
        let textFields = self.textFields // 1         guard let currentIndex = textFields.index(of: textField) else { return [] } // 2 

        return textFields.suffix(from: min(currentIndex + 1, textFields.endIndex - 1)) // 3 
    }

}
```

我们首先将数组保存在一个局部常量中，以略微提高效率(1)，因为`textFields`属性可能是计算出来的。在少数文本字段中，这种提升可以忽略不计，但是不遍历计算出的属性仍然是一个好习惯——特别是在迭代时——除非我们实际上需要反复重新计算属性，而我们并不需要。

然后我们找到传入的文本字段的索引，如果找不到就退出。最后，我们返回一个`ArraySlice`,其中所有文本字段都从当前文本字段之后的字段开始。

为什么要用切片？因为它保留了原始索引。因此，举例来说，如果我们有`[field1, field2, field3, field4]`并且我们从索引 2 开始做切片，我们会得到`[field3, field4]`，但是每个元素会有以下索引:`[2, 3]`。我们一会儿就会明白为什么需要这样做。

在我们转到这两个函数之前，还有最后一件事:一个简单的助手来检查上面的切片中是否有任何空字段:

```
extension NextTextFieldHandler { private func emptyFieldsExist(after textField: UITextField) -> Bool { 
        return fields(after: textField)
            .filter { $0.text?.isEmpty != false }
            .isEmpty == false 
    } }
```

它只接受当前之后的字段，筛选其中有文本的字段，并检查结果是否为空。

最后，我们可以开始实施:

```
extension NextTextFieldHandler {     func setupReturnKeyType(for textField: UITextField) { 
        let textFields = self.textFields // 1         guard let currentIndex = textFields.index(of: textField) else { return } // 2         let emptyFieldsExistAfterCurrent = emptyFieldsExist(after: textField) if currentIndex < textFields.endIndex - 1, emptyFieldsExistAfterCurrent { // 2 
            textField.returnKeyType = .next 
        } 
        else { // 3 
            textField.returnKeyType = .done 
        } 
    } }
```

和以前一样，我们将`textFields`数组保存在一个本地常量(1)中，如果我们找不到传递给`textField` (2)的索引，我们就退出。

然后我们要做出一个决定:

*   如果我们不在最后一个`textField`上，并且我们在当前字段之后有空字段，那么返回键应该是`.next` (2)。
*   如果我们在最后一个`textField`，或者当前字段之后的所有字段都被填充，则回车键应为`.done` (3)。

最后，点击 Next/Return 键的逻辑稍微复杂一点，我们最终会明白为什么要使用上面的`ArraySlice`:

```
extension NextTextFieldHandler {     func handleReturnKeyTapped(on textField: UITextField) { 
        let textFields = self.textFields // 1         guard let currentIndex = textFields.index(of: textField) else { return } // 2         let fieldsAfterCurrent = fields(after: textField) // 3 
        let nextEmptyIndex = fieldsAfterCurrent
            .firstIndex { $0.text?.isEmpty != false } // 4 
            ?? textFields.index(currentIndex, offsetBy: 1, limitedBy: textFields.endIndex - 1) // 5 
            ?? textFields.endIndex - 1 // 6         let emptyFieldsExistAfterCurrent =	emptyFieldsExist(after: textField) if currentIndex == textFields.endIndex - 1 || !emptyFieldsExistAfterCurrent { // 7 
            textField.resignFirstResponder() 
        } 
        else { // 8 
            textFields[nextEmptyIndex].becomeFirstResponder() 
        } 
    } }
```

和往常一样，我们首先将`textFields`数组保存在一个本地常量(1)中，如果我们找不到传递给`textField` (2)的索引，我们就退出。然后，我们获取作为切片传入的字段之后的下一个字段(3)，并找到第一个空字段的索引(4)，如果失败，则返回到下一个文本字段(5)，如果也失败，则再次返回到最后一个文本字段(6)。

我们现在要做另一个决定:

*   如果我们在最后一个`textField`或当前字段填写后的所有字段，请退出第一响应者(7)。
*   如果我们不在最后一个`textField`上，并且我们在当前字段之后有空字段，聚焦当前字段(8)之后的第一个空`textField`。

这就是我们需要`ArraySlice`的地方。让我们看一下前面的例子:

*   `textFields = [field1, field2, field3, field4]`。
*   我们在`field1`上，第一个空文本字段是`field3`，它的索引为 2。
*   `fields(after: field1)`返回`[field2, field3, field4]`带指标`[1, 2, 3]`。
*   `firstIndex { $0.text?.isEmpty != false }` (4)将返回索引`2`，这是应该的。
*   `textFields[nextEmptyIndex].becomeFirstResponder()`将注意力集中到正确的领域，`field3`。

如果我们不使用切片，`fields(after: field)`仍然会返回`field2, field3, field4]`，但是带有索引`[0, 1, 2]`。此外，我们没有在`fields(after:)`中使用`filter`，因为那样会重置索引，使它们从`0`开始。

如果我们不使用切片，我们也可以通过在(4)处使用`first { $0.text?.isEmpty != false }`来找到`field3`，然后通过调用`textFields.index(of: nextField)`来找到它的索引，但是我认为切片稍微好一些，因为我们可以编写更容易的回退(5，6)。

我们如何使用它？实际上很简单:

```
extension LoginViewController: UITextFieldDelegate, NextTextFieldHandler {     var textFields: [UITextField] { // 1 
        return aStackView.arrangedSubviews
            .compactMap { $0 as? UITextField }
            .filter { !$0.isHidden && $0.alpha != 0 && $0.isEnabled } // 2         /* 3 
        if mode == .login { 
            return [emailTextField, passwordTextField] 
        } 
        else { 
            return [nameTextField, emailTextField, passwordTextField] 
        } 
        */ 
    }     func textFieldShouldBeginEditing(_ textField: UITextField) -> Bool { 
        setupReturnKeyType(for: textField) // 4         return true 
    }     func textFieldShouldReturn(_ textField: UITextField) -> Bool { 
        handleReturnKeyTapped(on: textField) // 5 return true 
    } }
```

首先，我们设置我们需要的`textFields`(1)。我们从假想的`UIStackView` (2)中提取所有文本字段(或者手动返回— 3)，然后从`textFieldShouldBeginEditing(:_)` (4)中调用`setupReturnKeyType(for:)`，从`textFieldShouldReturn(:_)` (5)中调用`handleReturnKeyTapped(on:)`。

一如既往，我很想知道你的想法，或者是否有任何可以改进的地方。

【rolandleth.com】原载于[](https://rolandleth.com/handling-the-next-button-automatically)**。**