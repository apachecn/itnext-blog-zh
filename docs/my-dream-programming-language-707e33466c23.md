# 我梦想中的编程语言

> 原文：<https://itnext.io/my-dream-programming-language-707e33466c23?source=collection_archive---------1----------------------->

在使用不同的编程语言一段时间后，我开始考虑构建自己的语言。我就是这么想出来的！

![](img/11811575d55106ae5c1515204631dbeb.png)

照片由[安德里亚斯·瓦格纳](https://unsplash.com/@waguluz_)在 [Unsplash](https://unsplash.com) 拍摄

我一直对编程语言很着迷。每种编程语言都有自己的语法、语义和范例。

如果必须选择如何实现自己的编程语言，你会怎么做？这是我对 Inferi 的想法，一种编程语言。

# 索引

*   类型和语法
*   内置数据结构
*   函数！函数！函数！
*   不纯函数
*   故障处理
*   示例程序
*   项目状态

# 类型和语法

Inferi 是一种静态类型的编程语言。它使用亨德利·米尔纳类型系统进行类型签名。让我们定义一个简单的`add`函数，它将两个数字相加:

```
@spec add : Int -> Int -> Int
add (x y) ->
  x + y
```

括号是完全可选的，所以来自类似于 C 语言的开发人员会发现使用这种语言更容易:

```
@spec add : Int -> Int -> Int
add (x y) -> {
  x + y
}
```

此外，类型注释是完全可选的，可以在编译时推断出来( **REPL** 示例):

```
> let add x y -> x + y
> "add : Int -> Int -> Int"
```

**Inferi** 让您可以访问类型类，这样您就可以像下面这样重构`add`类型签名(就像在 **Haskell** 中一样):

```
@spec add Num a => a -> a -> a
add (x y) -> {
  x + y
}
```

函数将总是自动编译，以便(例如)T2 函数可以利用部分应用:

```
@spec add : Int -> Int -> Int
add (x y) -> {
  x + y
}

@spec add5 -> Int -> Int
add5 -> add 5

main -> {
  print add5(10)  -- 15
  print add(10 5) -- 15
}
```

每个 Inferi 程序都需要一个`main`函数，这个函数将是程序本身的入口点:

```
main -> println "Hello, World!"
```

# 内置数据结构

Inferi 提供了一组有用的内置数据结构，例如:

**地图**

```
data MapExample = {
  firstName : String,
  lastName  : String,
  nickname  : String
}@spec mapExample -> Map(MapExample)
mapExample -> {
  %{
    firstName: "Michele",
    lastName:  "Riva",
    nickname:  "Mitch"
  }
}
```

**列出了**

```
@spec listExample -> [Int]
listExample -> [10, 50, 20, 1010, 20391, 2039484]
```

**元组**

```
@spec tupleExample -> {String, Float, Bool}
tupleExample -> {"I am a string", -30.02, true}
```

# 函数！函数！函数！

阴尸从**哈斯克尔**和**仙丹**身上都拿了很多。

您可以通过创建不同的函数来对多个值进行模式匹配，就像在 **Haskell** 中一样(愚蠢的例子):

```
@spec greet : String -> String
greet ("Mitch") -> "Hello, Mitch!"
greet ("Jona")  -> "Hello, Jona!"
greet (_)       -> "Hello, stranger!"
```

当然，您可以将传递的参数绑定到变量:

```
@spec greet : String -> String
greet ("Mitch") -> "Hello, Mitch!"
greet ("Jona")  -> "Hello, Jona!"
greet (name)    -> "Hello, ${name}!"
```

您可以使用**管道**操作符来组合函数:

```
@spec scream : String -> String
scream (str) -> {
  str
    |> upcase
    |> (\x -> "${x}!!!") -- Lambda function, just like in Haskell
}
```

或者像 Haskell 中那样奉承:

```
@spec scream : String -> String
scream (str) -> str · upcase · (\x -> "${x}!!!")
```

此外，lambda 函数支持用于 lambda 函数的 Elixir **捕获运算符**:

```
@spec scream : String -> String
scream (str) -> str · upcase · (& "${&1}!!!")
--                              ^     ^ Here we say that we want to use the first argument of our lambda function
--                              ^ Here we say that this is a lambda functions
```

# 不纯函数

与 Haskell 不同， **Inferi** 不是一种纯粹的函数式编程语言，它允许我们编写一些支持不纯计算的函数。

**功能不纯:**

```
import http (call) -- import the "call" function from the "http" library
import list (push)data RESTPeople = {
  name : String `json:"name"` -- a little bit of Golang
  age  : Int    `json:"age"`
}@spec callRestAPI : Either(Error [String])
callRestAPI? -> {
  let mut people : [String]
  let mut callResult : [RESTPeople] &callResult <- await call("https://my-rest-api.com/get-people") for person in callResult -> {
    { name, age } = person
    if age >= 18 {
      push(people, name)
    }
  } return people
}
```

我们从`http`库导入`call`函数，从`list`库
导入`push`，我们定义一个`type`(在这种情况下，实际上类似于 **Golang** structs)来声明哪个值需要从 REST API 的 JSON 响应中解码。

之后，我们定义一个名为`callRestAPI`的函数，它返回一个单子(带有`mut`关键字或其他不纯计算的函数必须返回一个单子，参见失败处理段落)。

我们声明一个可变的`people`变量，它是一个字符串列表。我们声明可变的`callResult`变量，它将包含 REST 响应。

我们最终可以调用我们的 REST API，并将调用的结果放在`callResult`变量中。
之后，我们循环结果(这是一个`RESTPeople`列表),并使用`push`函数更新之前创建的列表`people`。

作为我们函数的最后一条语句，我们需要使用`return`关键字，它将结果包装成一个单子。
如果出现错误，它会将错误封装在`Left`结果中，否则它会填充单子的`Right`结果。

**纯对应:**

```
import http (call?)data RESTPeople = {
  name : String `json:"name"`
  age  : Int    `json:"age"`
}@spec callRestAPI : Either(Error [String])
callRestAPI? -> {
  return $
    (await call?("https://my-rest-api.com/get-people") : [RESTPeople])
      |> filter(& &1.age >= 18 )
      |> map(& drop(&1, "age"))
}
```

您还可以通过使用管道操作符更纯粹地编写相同的函数。
操作符`$`用在`return`之后，用于替换圆括号，就像在 Haskell 中一样。

# 故障处理

功能总是会失败。这就是为什么 **Inferi** 想要提供三种不同的方法来处理和避免运行时错误，如下所示:

**1)模式匹配:**

```
divide (x y) -> x / y
divide (x 0) -> error "You can't divide by 0"main -> {
  divide (10 5) -- 2
  divide (10 0) -- Runtime error: You can't divide by 0
}
```

**2)一元故障处理:**
可以在函数名后面加上`?`，这样它会把函数输出封装成一个`Either`一元:

```
@spec divide : Either (Num a => a -> a -> a)
divide? (x y) -> x / ymain -> {
  divide?(10 5) -- Right (10)
  divide?(10 0) -- Left (Error{reason: "Division by 0", args: {10, 0}})
}
```

如果出现运行时错误， **Inferi** 将通过返回一个包含错误原因和传递的参数的 struct 让您知道(便于调试)。

您也可以自己实现**一元故障处理**:

```
@spec divide : Either (Num a => a -> a -> a)
divide? (x y) -> {
  try {
    result = x / y
    Right(result)
  } catch error -> {
    println (error) -- you can debug the error, send it to Sentry,
                    -- do whatever you want
    Left(Error{reason: "You can't divide by 0!"}) -- The arguments will 
                                                  -- automatically be attached runtime
  }
}
```

**3)使用状态元组:**
就像在 Elixir/Erlang 中一样，可以返回一些表示你的计算结果的元组。这不能防止运行时错误，但这是定义不同类型函数的最佳实践:

```
@spec divide! : Num a => a -> a -> Status(a)
--          ^ it is best practice to set `!` after the function name for telling
--            that you're returning a Status tupledivide! (x y) (when y == 0) -> {:error, 0, "Division by 0"}
--           ^ Please note this Erlang-style guardsdivide! (x y) -> {
  result = x / y
  {:ok, result, ""}
}main -> {
  print divide(10 5) -- {:ok, 2, ""}
  print divide(10 0) -- {:error, 0, "Division by 0"}
}
```

Status 是一个私有类型，表示`:ok`或`:error`原子(取自 Elixir/Erlang atoms):

```
type Status a   -> {Atom, a, String}
type Status a b -> {Atom, a, b}
```

正如你在上面的定义中看到的，我们可以将参数传递给我们的类型。
这意味着我们可以根据我们编写的函数定义不同的`Status`类型。

例如，这将会失败:

```
@spec divideFloat! : Float -> Float -> Status(Float)
divideFloat! (x y) (when y == 0.0) -> {:error, 0, "Division by 0"}-- The 0 passed to the Status tuple is of Int type, but the Status Tuple
-- wants a Float (so, 0.0)!
```

但这是可行的:

```
@spec divideFloat! : Float -> Float -> Status(Float)
divideFloat! (x y) (when y == 0.0) -> {:error, 0.0, "Division by 0"}
```

正如您在`Status`定义中看到的，我们还可以通过显式传递第三个 tuple 元素的类型来自定义它:

```
@spec doSomethingCool : String -> String -> Status(String Maybe(Error))
doSomethingCool (str1 str2) -> {
  returningString       = "${str1} ${str2}"
  {:ok, returningString, Nothing}
}main -> {
  print doSomethingCool ("Michele" "Riva") -- {:ok, "Michele Riva", Nothing}
}
```

# 示例程序

让我们只使用 **Inferi** 标准库编写一个非常简单的 web 服务器:

```
import webserver
import system (env){--
  Here we get the port number that we want to expose
  from our webserver.
--}@spec port : Int
port -> {
  case env("PORT") -> {
    | Just(env_port) -> read env_port -- automatic unwrap, is this OCaml?
    | Nothing        -> 8000 
  }
}{--
  We then specify a list of tuples which represents
  our webserver routes.
--}@spec routes : [webserver.Route]
routes -> {
  [
    {"/",            indexController}
  , {"/about",       aboutController}
  , {"/greet/:name", greetController}
  ]
}{-- Here we write down some controllers --}@spec indexController : [webserver.Req] -> [webserver.Res]
indexController (_ res) -> {
  res.json(%{        -- hey! Is this Express.js?
    success: true
  })
}@spec aboutController : [webserver.Req] -> [webserver.Res]
aboutController (_ res) -> {
  res.render("about.html") -- oh Express.js, here we meet again
}@spec greetController : [webserver.Req] -> [webserver.Res]
greetController (req res) -> {
  %{ name } = Req.path -- destructure the request path and get the "name" variable case name -> {
    | ""   -> res.json(%{success: false, message: "Who should I greet?"})
    | name -> res.json(%{success: true, message: "Hello, ${name}!"})
  }
}{-- our main function --}main -> {
  webserver.start(%{
    routes -- Hey is this JavaScript? 
  , port
  })
}
```

# 项目状态

整个项目只是一个想法。
没有编译器，没有规范，就这篇文章。

现在你可能会疑惑:“Inferi 真的很像 Haskell，Rust，Elixir，OCaml 你为什么要写这个？”。

我相信软件开发的未来是功能性的。但是即使这样，纯粹的函数式编程对许多人来说还是很难，我们需要一种语言，它既是命令式的又是函数式的，从两个世界中取长补短。由于这个原因，我认为未来的编程语言将会是翻转的，而 Inferi 将会是我未来构建这种语言的努力方向。