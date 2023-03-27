# 在 SwiftUI 中使用 UIView 和 UIViewController

> 原文：<https://itnext.io/using-uiview-in-swiftui-ec4e2b39451b?source=collection_archive---------6----------------------->

![](img/4c8c16d0749959e3b5527fa61e233ffd.png)

由 [Jesper Aggergaard](https://unsplash.com/@aggergakker?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

我不时会遇到一个不使用 SwiftUI 的 SDK，而是使用带有`UIView`和/或`UIViewController`的“旧”UIKit。

因为我正在写一个关于 DJI 无人机编程的[系列](https://twissmueller.medium.com/programming-dji-drones-97974c18af06)，我使用的是基于 UIKit 的 DJIs SDKs。因此，我必须找到一种方法，在我基于 SwiftUI 的应用程序中使用它们。

这篇文章解释了如何在 SwiftUI-apps 中使用`UIView`和`UIViewController`，并提供了一些“剪切粘贴”的蓝图。

如果你喜欢这篇文章，请随意给我买杯咖啡。

## 集成 UIView

如果 SwiftUI 中需要一个基于`UIView`的类，那么创建一个实现`UIViewRepresentable`协议的 SwiftUI-view 结构就很简单了。

这是通过提供两种方法来实现的:

*   `makeUIView()`和
*   `updateUIView()`

在`UIKit`类需要委托的情况下，`Coordinator`类可以充当委托。

基本上，必须采取三个步骤来调整该蓝图:

1.  将名称`MyView`更改为视图的名称。
2.  将类型`SomeUIKitView`更改为应使用的类型`UIView`。
3.  如果需要委托，请删除注释并替换委托的类型。

```
struct MyView: UIViewRepresentable {

    typealias UIViewType = SomeUIKitView

    func makeUIView(context: Context) -> UIViewType {
        let myView = UIViewType()
        // myView.delegate = context.coordinator
        return myView
    }

    func updateUIView(_ uiView: UIViewType, context: Context) {
        // left blank
    }

    func makeCoordinator() -> MyView.Coordinator {
        return Coordinator(self)
    }
}

extension MyView {
    class Coordinator /*: SomeUIKitViewDelegate */ {
        var parent: MyView

        init(_ parent: MyView) {
            self.parent = parent
        }

        // Implement delegate methods here
    }
}
```

## 集成 UIViewController

类似于`UIView`我们可以通过创建一个实现`UIViewControllerRepresentable`方法的结构来集成一个`UIViewController`

*   `makeUIViewController`还有
*   `updateViewController`

同样，在使用这个蓝图时，必须完成同样的三个更改:

1.  将名称`MyView`更改为视图的名称。
2.  将类型`SomeUIKitViewController`更改为应使用的类型`UIViewController`。
3.  如果需要委托，请删除注释并替换委托的类型。

```
struct MyView: UIViewControllerRepresentable {

        typealias UIViewControllerType = SomeUIKitViewController

    func makeUIViewController(context: Context) -> UIViewControllerType Controller {
        let myViewController = UIViewControllerType()
        // myView.delegate = context.coordinator
        return myViewController
    }

    func updateUIViewController(_ uiViewController: UIViewControllerType, context: Context) {
            // left blank
    }

    func makeCoordinator() -> MyView.Coordinator {
        return Coordinator(self)
    }
}

extension MyView {
    class Coordinator /*: SomeUIKitViewDelegate */ {
        var parent: MyView

        init(_ parent: MyView) {
            self.parent = parent
        }

        // Implement delegate methods here
    }
}
```

如果你喜欢这篇文章，请给我买杯咖啡。。

## 资源

*   [UIViewRepresentable](https://developer.apple.com/documentation/swiftui/uiviewrepresentable)
*   [UIViewControllerRepresentable](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable)
*   [如何为 SwiftUI 包装自定义 ui view](https://www.hackingwithswift.com/quick-start/swiftui/how-to-wrap-a-custom-uiview-for-swiftui)
*   [在 SwiftUI 视图中包装 uiview controller](https://www.hackingwithswift.com/books/ios-swiftui/wrapping-a-uiviewcontroller-in-a-swiftui-view)
*   [在 SwiftUI 中使用 UIView 和 UIView controller](https://www.vadimbulavin.com/using-uikit-uiviewcontroller-and-uiview-in-swiftui/)