# 从本机到 JavaScript，并在 React Native 中返回或触发本机组件

> 原文：<https://itnext.io/from-native-to-javascript-and-back-or-trigger-native-components-in-react-native-b19992bb822d?source=collection_archive---------1----------------------->

![](img/39457f3d53953809e6f94a93d1a0cd87.png)

如果您正在寻找如何在 React Native 中将本地组件与 JavaScript 桥接，您可以查看[如何构建 react-native 桥并获取 PDF Viewer](/how-to-build-react-native-bridge-and-get-pdf-viewer-44614f11e08e) 文章，这在大多数情况下已经足够了。但是如果你需要用另一种方式过桥呢？当然，你可以使用声明的方式和改变属性，但这不是我们要找的，我们不想重新呈现反应组件。此外，它限制了缺乏功能的应用程序，如果加载失败(*可能是一些网络问题或您需要刷新文档等)，则很难实现像`reload`文档这样的功能。*)，或者其他动作如`save`、`refresh`等。实际上，我们可以把它与单向过桥相比较，因为没有回头路

![](img/8c8deaeffb7bed69840e1dffa0b92784.png)

希望 React Native 足够强大，也允许从 JavaScript 调用本地组件。假设您已经有了一个项目和一个桥接到 JavaScript 的组件，让我们从 JavaScript 调用本地组件并扩展桥接功能成为可能。可以通过以下方式完成:

*   实施 android 变更
*   实现 JavaScript 更改和测试
*   实施 iOS 变更

# Android 实现

打开视图管理器，实现`getCommandsMap`和`receiveCommand`方法:

*   `getCommandsMap` -应返回命令名和 id 之间的映射，然后在`receiveCommand`方法中使用
*   `receiveCommand` -直接从 JavaScript 接收事件/命令的方法

实现如下:

```
private static final int COMMAND_RELOAD = 1;// the code of your view manager
...@Override
public Map<String,Integer> getCommandsMap() {
  return MapBuilder.of("reload", COMMAND_RELOAD);
}public void receiveCommand(final PDFView view, int command, final ReadableArray args) {
  switch (command) {
    case COMMAND_RELOAD: {
      // The code you are going to execute when the command is called
      break;
    }
    default: {
      break;
    }
  }
}
```

正如我们所看到的，它实现起来非常简单，您也可以轻松地用其他命令扩展它。

# JavaScript 实现

实现需要[直接操作](https://facebook.github.io/react-native/docs/direct-manipulation)和访问组件(可以使用 `*ref*`访问*):*

```
<RNPDFView
  ref={ref => {
    this._viewerRef = ref;
  }}
  {/* other attributes */}
/>
```

下一步是定义将触发命令的`reload`函数，它对于 android 和 iOS 略有不同:

*   在 android 上，我们可以使用`UIManager`和`dispatchViewManagerCommand`
*   在 iOS 上，我们访问视图管理器并调用它的方法来触发事件

实施将是:

```
// Imports and other codeclass PDFView extends React.Component<Props, *> {
  // component implementation

  reload() {
    if (this._viewerRef) {
      const handle = findNodeHandle(this._viewerRef); if (!handle) {
        throw new Error('Cannot find node handles');
      } await Platform.select({
        android: async () => {
          return UIManager.dispatchViewManagerCommand(
            handle,
            UIManager.PDFView.Commands.reload,
            [],
          );
        },
        ios: async () => {
          return NativeModules.PDFViewManager.reload(handle);
        },
      })();
    } else {
      throw new Error('No ref to PDFView component, check that component is mounted');
    }
  }
}
```

并且在您的应用中，在接收到`ref`到`PDFView`组件后，触发`reload`命令:

```
// Event that can be triggered by some button
onButtonClick = () => {
  this.reload();
}// A method that triggers reload
reload = () => {
  if (this._pdfRef) {
    this._pdfRef.reload();
  }
}// Get ref to component
onRef = (ref: ?PDFView) => {
  this._pdfRef = ref;
}render() {
  // other code
  return (
    {/* App implementation */}
    <PDFView
      {/* Other properties */}
      onRef={this.onRef}
    />
  );
}
```

仅此而已。现在`reload`方法可以在应用程序的任何地方被调用，只要需要。不需要强制更改属性或重新加载 react 组件。

# iOS 实施

现在，当 JavaScript 到 android 的桥梁准备好并工作时，是时候在 iOS 上实现它了。iOS 的实现略有不同，需要组件公开必须触发的方法。`RCT_EXPORT_METHOD`用于说明:

```
RCT_EXPORT_METHOD(reload: (nonnull NSNumber *)reactTag resolver: (RCTPromiseResolveBlock)resolve rejecter: (RCTPromiseRejectBlock)reject) {
  // implementation
}
```

并且导出的方法提供了`reactTag`参数并承诺。使用`uiManager`和 received `reactTag`可以获得查看器的实例:

```
PDFView *view = (PDFView *)[self.bridge.uiManager viewForReactTag: reactTag];
```

此后，查看器的任何方法都可以被触发(当前情况下*重新加载*)。最终实现将是:

```
RCT_EXPORT_METHOD(reload: (nonnull NSNumber *)reactTag resolver: (RCTPromiseResolveBlock)resolve rejecter: (RCTPromiseRejectBlock)reject) {
    dispatch_async(dispatch_get_main_queue(), ^{
        PDFView *pdfView = (PDFView *)[self.bridge.uiManager viewForReactTag: reactTag];
        if (!pdfView) {
            reject(ERROR_INVALID_REACT_TAG, [NSString stringWithFormat: @"ReactTag passed: %@", reactTag], nil);
            return;
        }
        [pdfView reload];
        resolve(nil);
    });
}
```

# 结论

![](img/0ff41dbcc155d988619d4074bddc12d3.png)

在 JavaScript 和 native 之间建立桥梁并不需要做很多工作。拥有这样的桥梁使我们能够显著扩展 React Native 的功能，例如提高应用程序中一些需要更好性能的关键部分的性能，实现缺失的功能或重用一些已经开发的组件。此外，它再次证明了 React Native 的可能性真的很惊人，React Native 允许在 iOS 和 android 两个平台上以更少的努力开发成熟的应用程序。

所有描述的技术都在 [PDF Viewer](https://github.com/rumax/react-native-PDFView) react native 组件中使用，该组件还包括一个[演示项目](https://github.com/rumax/react-native-PDFView/tree/master/demo)，您可以在其中检查桥是如何工作的，等等。