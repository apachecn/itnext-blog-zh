# MVVM 在使用供应商

> 原文：<https://itnext.io/mvvm-in-flutter-using-providers-409c3c7e654?source=collection_archive---------1----------------------->

在设计应用程序时，MVVM 随时都是热门话题。

![](img/6e772a76a06a99e5b867c3c9dee585eb.png)

动荡中的 MVVM

在这篇文章中，我们将看到使用 MVVM 学习和构建一个简单的 Flutter 应用程序的最简单的方法。

# **观看视频教程**

MVVM 在颤振中使用提供商

这里我们将使用包“Provider”进行状态管理。

# **添加依赖关系**

```
provider: ^6.0.1
http: ^0.13.3
```

# **MVVM**

模型视图视图模型模式。

MVVM 的主要目的是分离关注点。

也就是说，让组件尽可能地松散耦合，这样你就可以单独测试它，并尽可能高效地维护代码。这就是你需要知道的一切。

好吧，那我们开始吧…

这里我们将使用下面的服务来获取数据。

[*https://jsonplaceholder.typicode.com/users*](https://jsonplaceholder.typicode.com/users)

这将响应一个用户记录列表。我们将在 flutter 中使用 MVVM 模式来显示这个列表。

所以你必须明白视图模型只是你的视图和服务/数据层之间的中间人。

**查看← →查看型号← →回购**

因此，我们将有一个视图，这是我们的 *UI* ，然后是一个视图模型类，它保存我们所有的*业务逻辑*和一个服务类，它从数据库、文件或任何其他方式获取*数据*。

我们将在项目中创建三个文件夹。

1.  *视图*
2.  *视图 _ 模型*
3.  *回购*
4.  *型号*

先从回购说起。

# 被卖方收回的汽车

在这里我们将获取我们的数据。

首先，我们需要创建模型类。

到达[*https://jsonplaceholder.typicode.com/users*](https://jsonplaceholder.typicode.com/users)并复制响应。

现在转到 [app.quicktype.io](http://app.quicktype.io/) ，在页面右侧的下拉列表中选择 dart 作为语言。

现在将响应粘贴到左边的编辑器中。

给这个类命名并生成它。我将命名为“***UsersListModel***”，并复制生成的类定义。

让我们在“模型”文件夹中创建一个名为“ *users_list_model.dart* 的文件并保存它。模型类可以在[这里](https://bitbucket.org/vipinvijayan1987/tutorialprojects/src/MVVM/FlutterTutorialProjects/flutter_demos/lib/users_list/models/)找到。

现在创建另一个名为“ *api_status.dart* ”的类，它将包含两个类

" ***成功*** "和" ***失败*** "这是我们要写的服务返回的。

```
class Success {
  int code;
  Object response;
  Success({this.code, this.response});
}class Failure {
  int code;
  Object errorResponse;
  Failure({this.code, this.errorResponse});
}
```

现在让我们编写服务。

创建一个名为" *user_services.dart* 的类

下面的服务将从服务中获取用户列表。

```
import 'dart:io';
import 
'package:flutter_demos/users_list/models/users_list_model.dart';
import 'package:flutter_demos/users_list/repo/api_status.dart';
import 'package:flutter_demos/utils/constants.dart';
import 'package:http/http.dart' as http;class UserServices {     static Future<Object> getUsers() async {
    try {
      var response = await http.get(Uri.parse("[https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users)"));
      if (SUCCESS == response.statusCode) {
        return Success(response: usersListModelFromJson(response.body));
      }
      return Failure(
              code: USER_INVALID_RESPONSE, 
              errorResponse: 'Invalid Response');
    } on HttpException {
      return Failure(
              code: NO_INTERNET, 
              errorResponse: 'No Internet          Connection');
    } on SocketException {
      return Failure(
               code: NO_INTERNET, 
               errorResponse: 'No Internet Connection');
    } on FormatException {
      return Failure(
            code: INVALID_FORMAT, 
            errorResponse: 'Invalid Format');
    } catch (e) {
      return Failure(
             code: UNKNOWN_ERROR, 
             errorResponse: 'Unknown Error');
    }
  }}
```

这里是完整的[代码](https://bitbucket.org/vipinvijayan1987/tutorialprojects/src/MVVM/FlutterTutorialProjects/flutter_demos/lib/users_list/repo/)的链接。

我们的回购实现现在已经完成。

# 视图模型

现在让我们来创造这场表演的“明星”吧。

在“ *view_models* ”文件夹中创建一个名为“***UsersViewModel***”的类。

这个想法是在视图模型中处理所有与状态相关的事情和业务逻辑，这样我们的屏幕将保持无状态、独立、易于测试和维护。

```
import 'package:flutter/material.dart';
import 'package:flutter_demos/users_list/models/user_error.dart';
import 'package:flutter_demos/users_list/models/users_list_model.dart';
import 'package:flutter_demos/users_list/repo/api_status.dart';
import 'package:flutter_demos/users_list/repo/user_services.dart';class UsersViewModel extends ChangeNotifier {

  bool _loading = false;
  List<UserModel> _userListModel = [];
  UserError _userError;  bool get loading => _loading;
  List<UserModel> get userListModel => _userListModel; UserError get userError => _userError;  UsersViewModel() {
    getUsers();
  }  setLoading(bool loading) async {
    _loading = loading;
    notifyListeners();
  }    setUserListModel(List<UserModel> userListModel) {
    _userListModel = userListModel;
  }     setUserError(UserError userError) {
    _userError = userError;
  }    getUsers() async {
    setLoading(true);
    var response = await UserServices.getUsers();
    if (response is Success) {
      setUserListModel(response.response as List<UserModel>);
    }
    if (response is Failure) {
      UserError userError = UserError(
        code: response.code,
        message: response.errorResponse,
      );
      setUserError(userError);
    }
    setLoading(false);
  }
}
```

这里，我们在 ViewModel 中设置 loading、UserListModel 和 UserError，并在 UI 中读取它。

# 视角

让我们看看如何访问状态和更新 UI。

我们显示用户列表的界面如下。

为了访问***user viewmodel***中的状态，我们在 build 方法中使用下面的代码。

```
UsersViewModel usersViewModel = context.watch<UsersViewModel>();
```

*我们在 build 方法中调用它，因为每当 ViewModel 调用 notifyListeners()时，相应 UI 中的 build 方法将被触发，它将被重新呈现，我们将获得最新的状态值。*

```
import 'package:flutter/material.dart';
import 'package:flutter_demos/components/app_error.dart';
import 'package:flutter_demos/components/app_loading.dart';
import 'package:flutter_demos/components/user_list_row.dart';
import 'package:flutter_demos/users_list/models/users_list_model.dart';
import 'package:flutter_demos/users_list/view_models/users_view_model.dart';
import 'package:flutter_demos/utils/navigation_utils.dart';
import 'package:provider/provider.dart';class HomeScreen extends StatelessWidget { [@override](http://twitter.com/override)
  Widget build(BuildContext context) {
    UsersViewModel usersViewModel = context.watch<UsersViewModel>();
    return Scaffold(
      backgroundColor: Colors.white,
      appBar: AppBar(
        title: Text('Users'),
        actions: [
          IconButton(
            onPressed: () async {
              openAddUser(context);
            },
            icon: Icon(Icons.add),
          ),
          IconButton(
            onPressed: () async {
              usersViewModel.getUsers();
            },
            icon: Icon(Icons.refresh),
          )
        ],
      ),
      body: Container(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          children: [
            _ui(usersViewModel),
          ],
        ),
      ),
    );
  } _ui(UsersViewModel usersViewModel) {
    if (usersViewModel.loading) {
      return AppLoading();
    }
    if (usersViewModel.userError != null) {
      return AppError(
        errortxt: usersViewModel.userError.message,
      );
    }
    return Expanded(
      child: ListView.separated(
        itemBuilder: (context, index) {
          UserModel userModel = usersViewModel.userListModel[index];
          return UserListRow(
            userModel: userModel,
            onTap: () async {
              usersViewModel.setSelectedUser(userModel);
              openUserDetails(context);
            },
          );
        },
        separatorBuilder: (context, index) => Divider(),
        itemCount: usersViewModel.userListModel.length,
      ),
    );
  }
}
```

最后一件重要的事情是用提供者视图模型类包装监听状态的屏幕。

*比如。*

```
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_demos/users_list/view_models/users_view_model.dart';
import 'package:provider/provider.dart';
import 'users_list/views/home_screen.dart';void main() {
  runApp(MyApp());
}class MyApp extends StatelessWidget {
  [@override](http://twitter.com/override)
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => UsersViewModel()),
      ],
      child: MaterialApp(
        title: 'MVVM',
        debugShowCheckedModeBanner: false,
        theme: ThemeData(
          primarySwatch: Colors.blue,
          brightness: Brightness.light,
          visualDensity: VisualDensity.adaptivePlatformDensity,
        ),
        home: HomeScreen(),
      ),
    );
  }
}
```

这里我们用 MultiProvider 包装了主根小部件。

```
MultiProvider(
   providers: [
        ChangeNotifierProvider(create: (_) => UsersViewModel()),
],
```

您可以拥有任意数量的提供商。

所以我们把状态从一个屏幕提升到一个顶级的通知类。因此，当通知类中发生变化时，我们所有的屏幕都会得到通知，我们不必在每个屏幕中维护任何状态逻辑。也不需要在屏幕构造函数中发送参数。

查看下面链接的 ***源代码中的整个实现。***

# [比特桶](https://bitbucket.org/vipinvijayan1987/tutorialprojects/src/MVVM/FlutterTutorialProjects/flutter_demos/)

 [## 比特桶

### 编辑描述

bitbucket.org](https://bitbucket.org/vipinvijayan1987/tutorialprojects/src/MVVM/FlutterTutorialProjects/flutter_demos/) 

***你需要知道的就这些。请在下面留下您的宝贵意见。***