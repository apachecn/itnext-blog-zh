# 用实际例子理解颤振中的零安全

> 原文：<https://itnext.io/understanding-null-safety-in-flutter-with-real-world-example-851f010c16c3?source=collection_archive---------0----------------------->

![](img/5730988be4a45e1fff75c3230676dfc0.png)

在本文中，我们将通过一个真实的服务示例来了解如何在不同条件下在 Flutter 中实现*。*

****观看视频教程****

***第一部分***

*用实际例子理解颤振中的零安全性*

***第二部分***

*用实际例子理解颤振中的零安全性*

**首先创建一个显示数据的屏幕。**

*让我们创建一个类主屏幕，看起来像这样*

```
*class HomeScreen extends StatefulWidget { const HomeScreen({Key? key}) : super(key: key); [@override](http://twitter.com/override)
  _HomeScreenState createState() => _HomeScreenState();}class _HomeScreenState extends State<HomeScreen> { // .... your code}*
```

*您可能会也可能不会在下面看到错误，这取决于您的项目是如何创建的。您的项目可能有 Null 安全支持，也可能没有。否则，您将在“Key？”中得到一个错误，这意味着您的项目不支持空安全，让我们开始修复它。*

# ***设置零安全项目***

*要支持空安全，请转到您的' ***pubspec.yaml*** '文件，并将环境更新为*

```
*environment:
  sdk: ">=2.12.0 <3.0.0"*
```

*调用'**'的*颤振包得到*** 来安装所有的依赖项。*

*项目最小支持的 sdk 版本必须是' *2.12.0* '。*

****太好了，现在你的项目支持空安全。****

*现在主屏幕上的错误是“键？”应该会消失。*

*太棒了，第一步成功完成。*

# ***解析来自服务的数据***

*让我们使用下面的服务。*

*[*https://jsonplaceholder.typicode.com/users*](https://jsonplaceholder.typicode.com/users)*

*它将以下面的记录格式返回 10 条记录*

```
*{
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "suite": "Apt. 556",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": {
        "lat": "-37.3159",
        "lng": "81.1496"
      }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
      "name": "Romaguera-Crona",
      "catchPhrase": "Multi-layered client-server neural-net",
      "bs": "harness real-time e-markets"
    }
  },*
```

*让我们为上面的 json 生成模型类。*

*转到[*app . quick type . io*](https://app.quicktype.io)并粘贴来自上述服务的整个响应，并将该文件命名为“User ”,您应该会得到如下内容。*

```
*import 'dart:convert';List<Users> usersFromJson(String str) => List<Users>.from(json.decode(str).map((x) => Users.fromJson(x)));String usersToJson(List<Users> data) => json.encode(List<dynamic>.from(data.map((x) => x.toJson())));class Users {
    Users({
        this.id,
        this.name,
        this.username,
        this.email,
        this.address,
        this.phone,
        this.website,
        this.company,
    }); final int id;
    final String name;
    final String username;
    final String email;
    final Address address;
    final String phone;
    final String website;
    final Company company;factory Users.fromJson(Map<String, dynamic> json) => Users(
        id: json["id"] == null ? null : json["id"],
        name: json["name"] == null ? null : json["name"],
        username: json["username"] == null ? null : json["username"],
        email: json["email"] == null ? null : json["email"],
        address: json["address"] == null ? null : Address.fromJson(json["address"]),
        phone: json["phone"] == null ? null : json["phone"],
        website: json["website"] == null ? null : json["website"],
        company: json["company"] == null ? null : Company.fromJson(json["company"]),
    );Map<String, dynamic> toJson() => {
        "id": id == null ? null : id,
        "name": name == null ? null : name,
        "username": username == null ? null : username,
        "email": email == null ? null : email,
        "address": address == null ? null : address.toJson(),
        "phone": phone == null ? null : phone,
        "website": website == null ? null : website,
        "company": company == null ? null : company.toJson(),
    };
}class Address {
    Address({
        this.street,
        this.suite,
        this.city,
        this.zipcode,
        this.geo,
    });final String street;
    final String suite;
    final String city;
    final String zipcode;
    final Geo geo;factory Address.fromJson(Map<String, dynamic> json) => Address(
        street: json["street"] == null ? null : json["street"],
        suite: json["suite"] == null ? null : json["suite"],
        city: json["city"] == null ? null : json["city"],
        zipcode: json["zipcode"] == null ? null : json["zipcode"],
        geo: json["geo"] == null ? null : Geo.fromJson(json["geo"]),
    );Map<String, dynamic> toJson() => {
        "street": street == null ? null : street,
        "suite": suite == null ? null : suite,
        "city": city == null ? null : city,
        "zipcode": zipcode == null ? null : zipcode,
        "geo": geo == null ? null : geo.toJson(),
    };
}class Geo {
    Geo({
        this.lat,
        this.lng,
    });final String lat;
    final String lng;factory Geo.fromJson(Map<String, dynamic> json) => Geo(
        lat: json["lat"] == null ? null : json["lat"],
        lng: json["lng"] == null ? null : json["lng"],
    );Map<String, dynamic> toJson() => {
        "lat": lat == null ? null : lat,
        "lng": lng == null ? null : lng,
    };
}class Company {
    Company({
        this.name,
        this.catchPhrase,
        this.bs,
    });final String name;
    final String catchPhrase;
    final String bs;factory Company.fromJson(Map<String, dynamic> json) => Company(
        name: json["name"] == null ? null : json["name"],
        catchPhrase: json["catchPhrase"] == null ? null : json["catchPhrase"],
        bs: json["bs"] == null ? null : json["bs"],
    );Map<String, dynamic> toJson() => {
        "name": name == null ? null : name,
        "catchPhrase": catchPhrase == null ? null : catchPhrase,
        "bs": bs == null ? null : bs,
    };
}*
```

*在上面的模型中，我们没有实现任何空安全。但我们现在就要做。*

# *实现零安全的不同方法*

*让我们看看下面对象中的一个变量，' ***名*** '。*

*所以对于每个声明的变量，你都会得到一堆错误。*

*但是，不要担心，我们会解决所有这些问题。*

```
*Users({
        this.id,
        this.name,
        this.username,
        this.email,
        this.address,
        this.phone,
        this.website,
        this.company,
    });*
```

*这里， ***名称*** 声明为*

```
*final String name;*
```

*要修复这个错误，您需要为变量' ***name*** '提供一个初始值。*

*那么我们该怎么做呢？*

*因为这来自服务，并且服务可以具有空值*或空值*或适当的名称。有些情况下，服务可能根本不返回' ***name*** '键。***

**所以我们这里有 3 个问题…**

***就零安全而言，我们如何管理这一点？***

**如果值是 ***null*** ，我们显示名称值的文本小部件可能会导致错误，因为文本小部件不会接受一个 ***null*** 值。**

****选项 1:****

**使名称变量 ***可空*** 。在这种情况下，确保您有适当的逻辑来处理 UI 中文本小部件的空值*。***

***为了使它可空，我们将添加'**？**'在变量的类型前面，像这样***

```
***final String? name;***
```

***这意味着名字变量可以是 ***null*** 。***

**名称变量构造函数中的错误现在应该消失了。**

> ***在 Flutter 中你不能显式初始化一个变量到* **null** *。***

**现在，在您试图使用这个变量的地方，您可能需要打开包装以从可空变量中获取值。如果你试图只使用' ***name*** '，你的文本小部件将会抱怨它不能接受空值。**

```
**Text(name!)**
```

**注意到了'**！**'符号结束于 ***名称*** 变量。这将打开可为 will 的 name 变量，以获取值并显示在 UI 中。**

> **警告！！！**

**如果 name 的值实际上是来自服务的' ***null*** '怎么办，这会导致你的 UI 坏掉。所以你应该避免使用“！”在您的生产代码中。**

> *****用'！'只有当您完全确定该值永远不会为空时。*****

***如果值实际上是'****null****'它会导致'* ***null 检查一个空值*** *'的错误。***

**所以尽量避免。**

****选项二:****

**我不希望我的名字在任何时候都有一个 ***null*** 值。然后呢？？？**

**如果变量 ***为空*** 退出服务，我们可以给它一个*默认值*。如果来自服务的值是 ***null*** ，那么让我们将它设置为空。通过这种方式，name 变量永远不能为空值*，即使服务的值为空值*。****

```
***factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json["id"] == null ? null : json["id"],
      name: json["name"] == null ? '' : json["name"]***
```

***这意味着，如果 ***json[【名称】]*** 为***null***from service，我们会将该值设置为' '(空)。这样我们就可以避免不必要的逻辑来防止 ***在 UI 中为空*** 。***

**让我们在解析的时候把它变得更短一点…**

```
**name: json["name"] ?? ''**
```

**‘**？？**可以代替 ***空值*** 检查。**

**默认值必须是一个 ***常量*** *值*。**

*****如果没有来自服务的‘name’键，但是我们在期待着呢？*****

**在这种情况下，我们可以在*构造函数*本身中给出一个默认值。**

```
**User({
    this.id,
   * this.name = '',*
    this.username,
    this.email,
    this.address,
    this.phone,
    this.website,
    this.company,
  });**
```

**最后，如果这是您正在构建的模型，您可以像下面这样将参数设置为' ***必需的*** '，并且如果您没有添加'**，也可以让您已经接受了一个 ***空值*** 或不接受？**结尾，也就是说，即使是 ***必需的*** 参数，它也可以接受 ***null*** 。这里我把' ***id'*** 字段作为' ***必填的*** '参数而没有添加'**？**’结尾。因此，每当我们创建一个'*用户*'对象时，我们被迫提供一个值为' ***非空*** '的参数。**

```
**User({
    required this.id,
    this.name = '',
    this.username,
    this.email,
    this.address,
    this.phone,
    this.website,
    this.company,
});final int id;
final String name;
final String? username;
final String? email;
final Address? address;
final String? phone;
final String? website;
final Company? company;factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json["id"] == null ? null : json["id"],
      name: json["name"] ?? '',
      username: json["username"] == null ? null : json["username"],
      email: json["email"] == null ? null : json["email"],
      address:
          json["address"] == null ? null : Address.fromJson(json["address"]),
      phone: json["phone"] == null ? null : json["phone"],
      website: json["website"] == null ? null : json["website"],
      company: json["company"],
    );
}**
```

**在上面的代码中，我们已经将' ***id*** ' a **必需的**参数和其他参数设置为可空，但是除了' ***Company*** '本身是一个对象之外，我们对空进行了适当的检查。**

**现在让我们看看如何能为 ***公司*** 提供一个默认值。**

**这就是公司类的样子**

```
**class Company {
  Company({
        this.name,
        this.catchPhrase,
        this.bs,
  }); final String name;
  final String catchPhrase;
  final String bs;
}**
```

**就像 ***name*** 一样，我们既可以从服务接受一个 ***null*** 值，也可以提供一个默认值。**

1.  **假设我们已经准备好接受来自服务的 null，那么上面的代码没有变化。**

```
**final Company? company;**
```

**这个管用。**

**我们如何使用它？**

**由于' ***公司*** '可以为空，我们需要将其解包。**

```
**Text(user.company!.name!), // not recommended**
```

**2.我们不接受 ***null*** 的服务，在这种情况下我们需要一个 ***默认值*** 。**

**对于颤振，默认值 ***必须是一个常量*** ，就像我们设置**为 ***名*** 一样。****

****为了做到这一点，我们必须让' ***公司*** '构造函数保持不变。****

```
****class Company { const Company({
    this.name = '',
    this.catchPhrase,
    this.bs,
  }); final String name;
  final String? catchPhrase;
  final String? bs;}****
```

****于是我们做了 const 构造函数，还将 ***名称*** 设置为**“**作为默认值(不需要)。****

****让我们创建一个 const 默认公司值。****

```
****const defaultCompany = Company(bs: '', catchPhrase: '', name: '');****
```

****这里我们将所有的值都设置为空。****

****现在，如果服务器的响应中没有公司密钥，并且服务器为' ***公司*** '密钥发送了一个 ***null*** 值，我们可以将构造函数中的值设置为默认值。****

```
****class User {
  User({
    required this.id,
    this.name = '',
    this.username,
    this.email,
    this.address,
    this.phone,
    this.website,
    this.company = defaultCompany,
  });final int id;
  final String name;
  final String? username;
  final String? email;
  final Address? address;
  final String? phone;
  final String? website;
  final Company company;factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json["id"] == null ? null : json["id"],
      name: json["name"] ?? '',
      username: json["username"] == null ? null : json["username"],
      email: json["email"] == null ? null : json["email"],
      address:
          json["address"] == null ? null : Address.fromJson(json["address"]),
      phone: json["phone"] == null ? null : json["phone"],
      website: json["website"] == null ? null : json["website"],
      company: json["company"] == null
          ? defaultCompany
          : Company.fromJson(json["company"]),
    );
  }Map<String, dynamic> toJson() => {
        "id": id,
        "name": name,
        "username": username == null ? null : username,
        "email": email == null ? null : email,
        "address": address == null ? null : address?.toJson(),
        "phone": phone == null ? null : phone,
        "website": website == null ? null : website,
        "company": company.toJson(),
      };
}****
```

****整个用户对象将如上所示。****

****这里，我们在构造函数中设置默认值，如下所示****

```
****this.company = defaultCompany,****
```

****我们也在这里解析来自服务器的 json，所以如果服务器发送空值*，我们将它设置为“ ***默认公司*** ”。*****

```
****company: json["company"] == null
          ? defaultCompany
          : Company.fromJson(json["company"]),****
```

****我们现在如何使用它？****

```
****Text(user.company.name),****
```

****干净多了，不是吗？****

****如果只是 ***公司*** 为空而 ***名称*** *公司对象* 不为空呢？然后****

```
****// not recommended because company can be
// null and textview won't accept it.
Text(user.company!.name),****
```

****如果将它设置为可空并在 UI 中处理，那么****

```
****Text(user.company!.name! ?? ''),****
```

****这样，即使值是 ***null*** ，我们在 UI 中做一个 ***null*** 检查，并设置为空。****

****我们也可以这样做****

```
****Text('Address ${user.address?.city}'),****
```

****在这种情况下，如果地址对象为 ***null*** ，那么 flutter 将停止访问城市变量，并返回 ***null*** 。通过这种方式，我们可以避免导致对空值的“*空检查”错误。但是请记住，在文本小部件中，该值仍然是空的。*****

****现在让我们在 HomeScreen 小部件中声明一些变量，用于显示来自 service 的值。****

```
****class _HomeScreenState extends State<HomeScreen> {

  List<User> _userList;
  String _error;...****
```

****这将导致错误，因为我们还没有初始化这两个变量。****

****让我们解决这个问题。****

```
****// Make it nullable
List<User>? _userList;
String? _error;// or give default values
List<User> _userList = [];
String _error = '';// or use lateinit
late List<User> _userList;
late String _error;****
```

****如果您正在使用 ***lateinit*** ，请确保在访问它之前将其初始化。****

****这里我们可以在主屏幕的 ***initState*** 中初始化它。****

```
****class _HomeScreenState extends State<HomeScreen> {

  List<User> _userList = [];
  late String _error;[@override](http://twitter.com/override)
  void initState() {
    super.initState();
    _error = '';
  }...****
```

****如果我们设置默认值，我们的整个主屏幕将会是这个样子。****

```
****class HomeScreen extends StatefulWidget {
  const HomeScreen({Key? key}) : super(key: key);[@override](http://twitter.com/override)
  _HomeScreenState createState() => _HomeScreenState();
}class _HomeScreenState extends State<HomeScreen> {

  List<User> _userList = [];
  String _error = '';[@override](http://twitter.com/override)
  void initState() {
    super.initState();
    _getUsers();
  }_getUsers() async {
    var result = await UserServices.getUsers();
    if (result is Success) {
      _userList = result.response as List<User>;
      setState(() {});
      return;
    }
    if (result is Failure) {
      _error = result.errorResponse as String;
      setState(() {});
    }
  }[@override](http://twitter.com/override)
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Users'),
      ),
      body: Container(
        child: Column(
          children: [
            Visibility(visible: _error.isNotEmpty, child: Text(_error)),
            ListView.separated(
              padding: const EdgeInsets.all(20.0),
              shrinkWrap: true,
              itemBuilder: (context, index) {
                return UserListRow(
                  user: _userList[index],
                  onTap: (User user) async {
                    user.address!.city = null;
                    Navigator.push(
                      context,
                      MaterialPageRoute(
                        builder: (context) => UserDetailsScreen(user: user),
                      ),
                    );
                  },
                );
              },
              separatorBuilder: (context, index) => Divider(),
              itemCount: _userList.length,
            ),
          ],
        ),
      ),
    );
  }
}****
```

****请注意，如果您将 ***_error*** 设置为*可空值*或*空值*，那么可见性小部件仍然会呈现其中的子部件。Visibility 小部件意味着即使 child 的值为空，也只是使小部件可见和不可见。如果子代为空，将导致错误。****

****请务必查看我的 youtube 频道，了解更详细的练习说明。****

*****觉得我的文章有用的可以拍起来* ***50*** *。*****

****或者[给我买杯咖啡](https://www.buymeacoffee.com/vipinvijayan)。****

# ******源代码******

 ****[## 比特桶

### 编辑描述

bitbucket.org](https://bitbucket.org/vipinvijayan1987/tutorialprojects/src/Null_Safety_Demo/FlutterTutorialProjects/flutter_demos/)**** 

****谢谢****