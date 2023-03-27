# HMock:哈斯克尔一流的模拟

> 原文：<https://itnext.io/hmock-first-rate-mocks-in-haskell-e59d7c3b066c?source=collection_archive---------3----------------------->

在今年的 Zurihac 年底，我发布了 HMock 的预览版[，这是一个用 Haskell 进行模拟测试的新库。我们来谈谈这是什么，我为什么写它，以及你如何使用它。](https://hackage.haskell.org/package/HMock-0.1.0.1)

![](img/469f9e95e86bb49f705c1a5131ca7e1b.png)

# 玩具聊天机器人

假设我想用 Haskell 写一个聊天机器人。我可能会从几个类型开始，像这样…

```
newtype **User** = User String deriving (Eq, Show)
data **PermLevel** = Guest | NormalUser | Admin deriving (Eq, Show)
newtype **Room** = Room String deriving (Eq, Show)
data **BannedException** = BannedException deriving (Show)instance Exception BannedException
```

现在我需要登录聊天服务器，连接到正确的房间，阅读和发送消息…真是一团糟！我可以深入研究并开始用套接字和 TCP 在`IO`中实现，但是构建一个抽象层也不错。我会用一些 MTL 风格的类型类来做这件事。下面是我的机器人需要做的一些事情。

```
class Monad m => **MonadAuth** m where
  **login** :: String -> String -> m ()
  **logout** :: m ()
  **hasPermission** :: PermLevel -> m Boolclass MonadAuth m => **MonadChat** m where
  **joinRoom** :: String -> m Room
  **leaveRoom** :: Room -> m ()
  **sendChat** :: Room -> String -> m ()
  **pollChat** :: Room -> m (User, String)
  **ban** :: Room -> User -> m ()class Monad m => **MonadBugReport** m where
  **reportBug** :: String -> m ()
```

这样好多了！考虑到正确的原语，我将编写机器人。我不会解释代码，但它在这里。我们的机器人监听聊天，禁止任何淘气的用户，并接受和传递错误报告。

```
type MonadChatBot m = (MonadChat m, MonadBugReport m)**chatbot** :: (MonadMask m, MonadChatBot m) => String -> m ()
chatbot roomName = do
  login "HMockBot" "secretish"
  handleRoom roomName `finally` logout**handleRoom** :: (MonadMask m, MonadChatBot m) => String -> m ()
handleRoom roomName = do
  room <- joinRoom roomName
  listenAndReply room `finally` leaveRoom room**listenAndReply** :: MonadChatBot m => Room -> m ()
listenAndReply room = do
  (user, msg) <- pollChat room
  finished <- case words msg of
    ["!hello"] -> sendChat room "Nice to meet you."
    ["!leave"] -> return True
    ("!bug" : ws) -> reportBug (unwords ws) >> return False
    ws | any isFourLetterWord ws -> do
      banIfAdmin room user
      return False
    _ -> return False
  unless finished (listenAndReply room)**isFourLetterWord** :: [Char] -> Bool
isFourLetterWord = (== 4) . length . filter isLetter**sendBugReport** :: MonadChatBot m => Room -> String -> m ()
sendBugReport room bug = do
  reportBug bug
  sendChat room "Thanks for the bug report!"**banIfAdmin** :: MonadChat m => Room -> User -> m ()
banIfAdmin room user = do
  isAdmin <- hasPermission Admin
  when isAdmin $ do
    ban room user
    sendChat room "Sorry for the disturbance!"
```

太好了，都完成了！但是等等…我怎么知道这面代码墙实际上是正确工作的呢？如果没有这三个类型类的实现，我甚至无法尝试。

# 测试有效代码

Haskell 在构建高度可靠软件的一些技术方面领先于编程行业。Haskellers 通常依靠类型系统来验证代码的属性，维护无状态代码和不可变数据以消除许多出现错误的机会，利用纯功能代码快速轻松地进行测试，依靠 QuickCheck 进行随机属性测试，甚至使用检查测试来检查编译器的属性。Haskell 程序员也将尽可能多的逻辑从有效的代码中转移到纯粹的功能性代码中。(上面我真的做得不太好…)

但是有效的代码最终是必要的，这是 Haskell 使用起来有点困难的测试领域的一个方面。上面的聊天机器人与许多外部系统对话:认证服务器、聊天服务器、错误报告系统，甚至其他人！很难将这些部分(尤其是人)放置在自动化测试的适当位置，所以我们必须找到一种没有它们的测试方法。

这里基本上有两种选择:

*   *假货*是 API 的假实现，试图表现得像一个真实的系统，这样你就可以用它们进行测试。
*   *模拟*是愚蠢的实现，对系统的预期行为一无所知，但是可以通过*测试本身*告诉它如何行为。

如果你有一个高质量的假实现，那么你一定要用它来测试！然而，制造一个高质量的赝品需要大量的工作。假货更适合非常稳定的界面。让一个伪代码与快速变化的 API 保持同步可能需要做大量的工作。这些都是全新的界面，我们预计随着时间的推移会大幅调整。当事情不简单时，您可能会努力编写测试来验证您的行为。网络瘫痪怎么办？如果事务提交失败怎么办？很少用假货掩盖这些案子。

所以我们盘子里的另一个答案是模仿。这就是我们接下来要找的地方。

# 关于模仿的一切

mock 是一个只做你让它做的事情的对象。它根本没有任何内部逻辑，只是简单地遵循指令，将您正在测试的代码中的调用与脚本中的行进行匹配，并按照要求做出响应。

下面是如何用 HMock 编写一个简单的测试。

```
makeMockable ''MonadAuth
makeMockable ''MonadChat
makeMockable ''MonadBugReportrunMockT $ do
  expect $ Login "HMock" "secretish"
  expect $ JoinRoom "#haskell" |-> Room "#haskell"
  expect $ PollChat (Room "#haskell")
    |-> (User "Alice", "!hello")
    |-> (User "Bob", "!leave")
  expect $ SendChat (Room "#haskell") "Nice to meet you."
  expect $ LeaveRoom (Room "#haskell")
  expect $ Logout chatbot "#haskell"
```

我们可以解剖这些部分。

*   前三行是模板 Haskell，生成一些样板文件，使这些接口可以模仿。
*   `runMockT`运行`MockT` monad transformer，它管理测试中所有预期的行为。
*   下一个模块设定了期望值。在这里，我们预计会发生几件事:机器人将登录，加入一个房间，两次轮询聊天消息，发送消息，离开房间，然后注销。当一个方法有一个有趣的返回值时，我们用`|->`来说明它应该是什么。
*   最后，我们调用正在测试的实际代码块。代替实际的实现，实现将使用它所使用的接口的模拟。正确的实例由`makeMockable`生成。
*   当被测代码执行动作时，`MockT`将在设定的期望值中查找它们，并按照要求做出响应。当`runMockT`结束时，它将检查是否满足了所有期望。

# 模仿会出什么问题

Haskell 对模拟测试的问题有一些答案。Alexis King 写了一个名为 [monad-mock](https://hackage.haskell.org/package/monad-mock) 的库，这是我在开始这项工作时研究的。这个周末我还了解到阿克谢·曼卡尔写了[多义词模拟](https://hackage.haskell.org/package/polysemy-mocks)来处理这个效果系统。但我创造了不同的东西。

原因是我认为一个虚弱的嘲讽框架是一个危险的工具。模拟测试可能会在几个方面出错。

*很容易断言太多。*假设你的代码通过网络获取两个文件。你关心它们是按什么顺序取来的吗？事实上，你在乎它们被取了多少次吗？也许是这样，但是您真的希望您的测试套件中的每一个测试都失败，仅仅因为您两次获取相同的文件吗？很好地使用 mocks 需要一种语言，让你对文件说足够多的*(比如:“如果这个文件被获取，这是将会得到的响应”)，但不要说太多的*导致脆弱的测试，当对代码进行无害的更改时，测试会中断。**

***嘲讽太多容易*。您需要模拟您不能用于测试的东西:外部系统、用户等等。用 mock 建立一个奇特的测试是很常见的，只是意识到你已经告诉 mock 精确地做你应该测试的事情。很好地使用 mocks 有时需要精确控制什么被剔除，什么不被剔除。**

***很容易打破可组合性。主流语言中常用的模拟系统以容易出错而闻名。您可能会添加一个期望，希望它在某个时间触发，结果却意外地在错误的时间触发，将错误的行为注入到您的测试中，随之而来的就是混乱！为了做好测试，你需要一种很好的可组合的语言来描述行为。模拟框架通常依赖于全局状态、总计数等。从测试的一部分向另一部分泄露细节。***

**HMock 试图解决这些问题。**

**像主流编程语言的最佳框架一样，HMock 为您提供了一种强大而灵活的语言来设置期望值。您负责:动作是否需要按顺序执行，动作发生的次数是否有限制，参数是被精确检查，完全忽略，还是对照谓词检查。**

**HMock 还可以让您控制 API 被模仿的程度，以及这些模仿有多聪明。使用 HMock 通常是值得的，即使只是设置一个 fake，因为用 HMock 委托方法不需要像新的`MonadChat`实现那样定义新的 monad 类型或 transformers，还因为很容易注入故障和测试 fake 无法处理的其他困难情况。**

**最后，与主流语言中的大多数模仿不同，HMock 基于基本的可组合的原语语言，如 Svenningsson 等人在论文 [*中描述的一种模仿*](https://link.springer.com/content/pdf/10.1007%2F978-3-642-54804-8_27.pdf) 的表达性语义。事实上，它通过向论文中的集合添加交错重复，实现了一组更具表达性的原语。像选择和重复序列这样的操作可以让你更准确地说出每个操作应该在什么时候发生，这样就不会留下可能在错误的时间意外匹配的期望。你可以很容易地说这样的话:“检查后视镜和系好安全带的顺序并不重要，但在两者都完成之前不要启动汽车，”或者“1040 表格或 1040EZ 表格必须提交，但不能两者都提交”。**

**如果你想看到我上面介绍的聊天机器人的更多测试，我从 HMock 测试套件的[演示中获得了代码，其中有更多的测试和技术！](https://github.com/cdsmith/HMock/blob/main/test/Demo.hs)**

# **案例研究**

**我的第一个 HMock 用例是…HMock 本身的测试套件！我真的不希望这样，因为我希望在测试与用户或复杂的外部服务交互的有效系统时需要模拟。但是 HMock 确实与一个复杂的外部服务交互:GHC！它使用模板 Haskell API 来实现这一点。**

**在测试 HMock 的过程中，我意识到我想测试一些模板 Haskell 代码。但是模板 Haskell 在构建时运行，这有一些缺点:**

*   **你只能测试成功的案例。未能构建的测试是不可接受的状态，这意味着不可能在构建时测试失败的使用。**
*   **我非常喜欢 hpc，GHC 的测试覆盖率报告系统。然而，hpc 看不到在构建时运行过的代码，所以产生的覆盖率报告认为第四段代码完全没有经过测试。**
*   **有时测试内部方法而不仅仅是顶级声明是有用的。这些在构建时不容易测试。**

**所以我想在 IO 中运行模板 Haskell。嗯，假设有这样一种方法:IO 是模板 Haskell 使用的准类型类的一个实例。但是这是一个相当糟糕的实例，当你试图做一些有趣的事情时，比如查找一个类型或实例，就会抛出错误。我需要更多。事实证明，我能够使用 HMock 为 *Quasi* 类型类构建一个 Mock，并使用它来测试 HMock 自己的模板 Haskell 代码。我可以在构建时运行查询，然后用 th 的 lift 类将它们的响应提升到运行时，并使用 HMock 清除准 monad 来返回那些保存的响应，而不是手工编写每个 TH 查询的响应。**

**结果如下:**

*   **通过运行我已经开始编写的简单测试，我发现了三个 bug。这些大多是错误案例中的错误。**
*   **我可以查看 hpc 输出，它突出显示了没有被测试的代码部分。通过考虑何时需要该代码，我生成了更多的测试用例，导致在同一天又发现了四个 bug。**
*   **后来，我能够更快地开发模板 Haskell 代码，因为我可以编写测试用例来验证 TH 的行为，运行它们，并断言或检查结果，这种方式在编译时代码中通常很难做到。**

**总的来说，这是一个压倒性的成功案例。**

# **状态**

**HMock 现在作为预发布版在 Hackage 上发布。我称之为预发布的主要原因是它目前只为 MTL 风格的类型类实现。我想扩展到最常见的效果系统，以及 servant，haxl 等等。作为这一过程的一部分，我预计会有一些重大的突破性变化。然而，只要您的版本上限适当，现在没有理由不能使用 HMock。**

**除了上面提到的内容，HMock 还实现了以下功能:**

*   **一种将参数与方法相匹配的谓词的扩展语言，让您可以轻松地隔离可以附加响应的有趣调用。这些谓词适用于各种类型，包括强大的技术，如正则表达式和不区分顺序的容器匹配。**
*   **可配置默认值。默认情况下，没有显式响应的方法将根据`Default`类返回其默认值。但是您可以为某些或所有调用设置不同的缺省值。**
*   **每类设置。您可以配置默认行为，并通过编写一个方便的类型类的实例将它们与类型捆绑在一起。**
*   **广泛支持大多数种类的类(包括具有函数依赖关系的多参数类)和大多数方法(包括多态方法，除非返回值是由方法绑定的类型变量)。**