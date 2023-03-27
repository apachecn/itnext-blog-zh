# React-Native-Maps 的高性能自定义地图标记

> 原文：<https://itnext.io/performant-custom-map-markers-for-react-native-maps-ddc8d5a1eeb0?source=collection_archive---------0----------------------->

*TL；dr-使用标记属性* `*tracksViewChanges*` *控制标记在自定义标记加载后何时重新渲染，方法是将其设置为* `*false*` *。*

# 添加自定义标记

```
<Marker
    coordinate={marker.coordinate}
>
    <CustomMarkerComponent />
</Marker>
```

典型的方法是简单地将您的自定义标记组件或`<Image/>`包装在`<Marker>`标签中。在你有多个标记之前，这都没问题。即使像这样向地图添加 10 个自定义标记，也会导致地图渲染出现严重的性能问题。

# 问题是

与几乎所有 React 原生性能问题一样，问题的根源在于大量不必要的重新渲染。每次地图更新时，所有自定义标记都会被渲染，这是非常低效的，因为标记大头针通常不会改变(图像和坐标)。

# 提高性能

为了解决这个问题，我们将告诉地图准确的时间来跟踪标记针的视图变化。我们用`marker`上的`tracksViewChanges`道具来做这个。

```
import React, { PureComponent } from 'react';
import { Image } from 'react-native';
import { Marker } from 'react-native-maps';export default class CustomMarker *extends* PureComponent { constructor() {
    super();
    this.state = {
      tracksViewChanges: true,
    };
  } stopTrackingViewChanges = () => {
    this.setState(() => ({
      tracksViewChanges: false,
    }));
  } render() {
    const { tracksViewChanges } = this.state;
    const { marker } = this.props; return (
      <Marker
        coordinate={marker.coordinate}
        tracksViewChanges={tracksViewChanges}
      >
        <Image
          onLoad={this.stopTrackingViewChanges}
          fadeDuration={0}
        />
      </Marker>
    );
  }
}
```

让我们来分析一下这里发生了什么:

1.我们用`tracksViewChanges = true`初始化，所以当自定义标记图像加载时，它将呈现。
2。我们设置了图像的`onLoad`，这样我们就不会不必要地重新渲染自定义标记。
3。我们应该设置`fadeDuration = 0`，否则当标记停止跟踪视图变化时，图像可能会卡在部分淡入状态。

使用这个组件，你可以添加大量的引脚，而不会降低你的应用程序。

但是请记住，标记不会在初始图像加载后跟踪变化。这意味着**对标记的更新不会触发重新渲染**。

![](img/4f5e87d3a8643c81da4c88e81c82f7b5.png)

除非我们处理适当的变化，否则不可能创建像这样的效果，其中单个标记图像基于传送带的位置被更新。

# 处理道具变化

您所要做的就是添加这个`componentWillRecieveProps`逻辑并用您自己的实现来实现`shouldUpdate`方法:

```
 componentWillReceiveProps(nextProps) {
    if (this.shouldUpdate(nextProps)) {
      this.setState(() => ({
        tracksViewChanges: true,
      }));
    }
  } shouldUpdate = (nextProps) => { //TODO implement }
```

地图图钉现在可以动态更新，而不会影响性能。