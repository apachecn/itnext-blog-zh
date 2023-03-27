# 处理 React Navigation v2 中的选项卡更改

> 原文：<https://itnext.io/handle-tab-changes-in-react-navigation-v2-faeadc2f2ffe?source=collection_archive---------0----------------------->

![](img/70d0e36103683ce613aa85dbe678ae7f.png)

本文涵盖 ***反应-导航 v2*** 。示例使用的是 **v2.9.0** 。
如果你使用的是 v1，请考虑 [*原文为 v1*](/handle-tab-changes-in-react-navigation-3717180cddb) 。

# 用例

1.  **切换到标签页时，复位到以前的状态**；
2.  **当用户聚焦标签时执行附加逻辑**；
3.  **阻止用户聚焦**特定标签；
4.  **处理选项卡上的双击**；

在 **v1** 中，你必须使用`tabBarOnPress`来处理这些用例。

但是 **v2** 引入了一个新特性，叫做`NavigationEvents`。我们将仔细研究这两种方法。

让我们考虑下面的 **TabNavigator** :

```
// App.js*import* { **createBottomTabNavigator** } *from* "react-navigation";*export* *default* createBottomTabNavigator({
  Home: HomeScreen,
  Settings: SettingsScreen
});
```

和具有以下内容的`**HomeScreen**`:

```
// HomeScreen.js*import* React *from* "react";
*import* { Text, View } *from* "react-native";class HomeScreen extends React.Component {
  render() {
    *return* (
      <View>
        <Text>Home</Text>
      </View>
    );
  }
}
```

现在，假设您想在用户聚焦 Home 选项卡时做一些事情。

# 方法 1:使用`NavigationEvents`

正如我之前提到的， *react-navigation v2* 引入了一个名为`[**NavigationEvents**](https://reactnavigation.org/docs/en/navigation-events.html)`的新组件，它提供了一个声明式 API 来订阅典型的导航事件，例如:

*   `onWillFocus`:聚焦标签页前；
*   `onDidFocus`:聚焦一个标签页后；
*   `onWillBlur`:在标签上失去焦点之前；
*   `onDidBlur`:在标签页上失去焦点后；

使用这个组件非常简单明了。您只需将它包含在需要由这些事件通知的屏幕中，并指定事件处理程序:

```
// HomeScreen.js...***import* { NavigationEvents } *from* "react-navigation";**class HomeScreen extends React.Component {
  render() {
    *return* (
      <View>
        **<NavigationEvents
          onWillFocus={payload => {** console.log("will focus", payload); **}}
        />** <Text>Home</Text>
      </View>
    );
  }
}
```

## 优点:

*   易于使用的声明式 API
*   在装载时立即订阅，在卸载时取消订阅；

## 缺点:

*   无法阻止聚焦在选项卡上；
*   无法检测双击；

所以这种方法只解决了我们最初用例中的 2/4。接下来，我们将看看另一种方法，这是必要的，但将为我们提供更多的灵活性。

# 方法 2:使用`tabBarOnPress()`

我们可以以两种不同的方式使用`[**tabBarOnPress()**](https://reactnavigation.org/docs/en/bottom-tab-navigator.html#tabbaronpress)`。我们可以在每个单独的屏幕上设置它:

```
// HomeScreen.jsclass HomeScreen extends React.Component { **static navigationOptions = () => {
    *return* {
      tabBarOnPress({ navigation, defaultHandler }) {** // perform your logic here // this is mandatory to perform the actual switch
        // don't call this if you want to prevent focus
 **defaultHandler();
      }
    };
  };** render() {
    ...
  }
}
```

或者，我们可以在导航器级别为所有选项卡设置它:

```
// App.js*export* *default* createBottomTabNavigator({
  Home: HomeScreen,
  Settings: SettingsScreen
}, {
 **navigationOptions: {
    tabBarOnPress: ({ navigation, defaultHandler }) => {** // perform your logic here // this is mandatory to perform the actual switch
      // don't call this if you want to prevent focus
 **defaultHandler();
    }
  }** });
```

如果您的逻辑可以直接在处理程序中执行，那么您就可以开始了。

但是如果你需要在组件上执行一些**方法**，或者访问组件的**状态**或者**属性**，你需要做一些额外的工作。

## 访问组件的状态、属性和方法

这是一种有点奇怪的方法。你必须:

*   使用`**setParams()**`方法在`navigation`对象上设置一个`param`；
*   从`**tabBarOnPress()**`打这个`param`；
*   从`param`内部访问任何`prop`、`state`或`method`，因为它是在组件的上下文中调用的；

```
// HomeScreen.jsclass HomeScreen extends Component {static navigationOptions = () => {
    *return* {
      tabBarOnPress({ navigation, defaultHandler }) { **navigation.state.params.onTabFocus();** defaultHandler();
      }
    };
  }; constructor(props) {
    *super*(props);
    **props.navigation.setParams({
      onTabFocus: *this*.handleTabFocus
    });**
  } **handleTabFocus = () => {
    // perform your logic here
  };** ...}
```

此外，您需要禁用选项卡导航器的延迟加载:

```
*export* *default* createBottomTabNavigator({
  ... // your screens
}, {
 **lazy: false** });
```

> **注意**:该设置目前在文档中不可用

所以我们不直接访问状态或道具。相反，我们:

*   使用`*this*.props.navigation.setParams({ *myParam*: this.*myHandler* })`在导航对象上设置自定义参数
*   所以这个自定义参数可以在 navigationOptions 中调用，使用`navigation.state.params.*myParam*`

好吧，这显然比使用声明性方法和`**NavigationEvents**`更样板化。但是通过这种方式，我们能够实现一些额外的功能。

## **防止标签焦点**

这个很简单。如果忽略调用`defaultHandler()`，选项卡将不会被聚焦:

```
// HomeScreen.jsclass HomeScreen extends Component { static navigationOptions = () => {
    *return* {
      tabBarOnPress({ navigation, defaultHandler }) { **if (/* some condition is met */) {** defaultHandler();
  **      }
**      }};
  };

  ...}
```

## 手柄双击

通常，移动应用程序会保存标签的内部状态。例如，如果在一个选项卡中有另一个`StackNavigator`,当您导航到另一个选项卡并返回时，状态会被持久化。

一旦用户再次点击同一个标签，重置标签的内部状态是一种常见的模式。这很容易实现:

```
// HomeScreen.jsclass HomeScreen extends Component { static navigationOptions = () => {
    *return* {
      tabBarOnPress({ navigation, defaultHandler }) {***if* (navigation.isFocused()) {**
          // same tab was tapped twice
          // reset inner state
          **return;**
        **}** // tab was not previously focused
        defaultHandler();
      }};
  };

  ...}
```

见完整工作示例:[*https://snack . Expo . io/@ andreipfeifffer/handle-tab-changes-in-react-navigation-v2*](https://snack.expo.io/@andreipfeiffer/handle-tab-changes-in-react-navigation-v2)