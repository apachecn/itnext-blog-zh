# Web3:管理不同网络中的地址差异

> 原文：<https://itnext.io/web3-managing-address-differences-in-different-networks-d4f3cc0e1ae5?source=collection_archive---------3----------------------->

![](img/53f5708c79a522c716934a6449df8bfa.png)

寻找正确的地址有时会很棘手

*   开发人员经常 1)在本地，2)在测试网上，3)在主网上测试他们的代码
*   智能合同的地址在每个网络上通常是不同的
*   管理和切换这些地址有时会很麻烦和混乱

在开发智能合同或 dApp 时，您可能会在不同的网络上进行测试，为部署到 MainNet 做准备。一个典型的周期可能包括测试

1.  在本地
2.  在测试网上
3.  在主网上

如果您的协定与其他智能协定交互，则您可能需要根据您所在的网络来更改代码中引用的地址。

比如说…

*   如果您的合同与 UniSwap 或 PancakeSwap 之类的交换进行交互，您可能需要使用不同的地址，这取决于您是从 TestNet 还是 MainNet 进行测试。
*   也许你正在建造一个带有 ERC20 令牌的 NFT。本地部署和部署到真实网络时，令牌的地址会有所不同。

本文就如何简化代码库中这些不同地址的管理提供了一些提示和建议。我们将关注 Solidity 中的智能契约开发和 JavaScript 中的单元测试/dApp 开发。

# 不要硬编码地址；将它们传递给构造函数

尽可能避免硬编码地址。当开发一个与另一个智能合约交互的智能合约时(假设是一个 ERC20 令牌)，您的第一个想法可能是，“*这个令牌已经部署在 MainNet 上，所以我将把地址设置为* `*constant*` *，以稍微简化我的代码。”*

然而，您确实 ***而不是*** 想要这样做，因为您将设置您的代码在 MainNet 之外的任何地方都不工作。

有时，开发人员会在测试时进入他们的智能合约代码并注释/取消注释地址，这取决于他们测试的网络。

```
contract MyToken is ERC20 {
   // TestNet
   address public uniswapRouter = 0x…456; 
   address public devAddress = 0x…de2;      // MainNet
   //address public uniswapRouter = 0x…789; 
   //address public devAddress = 0x…de3;

   constructor() ERC20(“MyToken”, “MTOK”) { }
}
```

如果你养成了像这样修改代码的习惯，你就有可能忘记注释掉正确的代码，并在你的智能契约中引入一个 bug。这种方法还有其他令人头疼的问题，比如需要重新编译代码，每次都必须进入文件，滚动到正确的位置并更改代码。

相反，通过构造函数将您需要的所有地址传递到智能协定中。这将允许您将代码部署到本地计算机或 TestNet，并设置您需要使用的地址，而无需更改您的智能协定代码。

```
contract MyToken is ERC20 {
   address public uniswapRouter;
   address public devAddress; constructor(address _uniswapRouter, address _devAddress) 
        ERC20(“MyToken”, “MTOK”) {
      uniswapRouter = _uniswapRouter;
      devAddress = _devAddress; 
   }
}
```

# 如果您必须硬编码地址，请提供一种方法来更改它们

如果您已经决定必须对地址进行硬编码，请尝试提供一个函数来允许以编程方式更改它们。这样，当您在本地或 TestNet 上进行测试时，您可以调用该函数来适当地更新地址。

如果选择这种方法，一定要对函数应用适当的安全性，比如只允许契约的所有者调用它。

```
function setUniswapRouter(address _uniswapRouter) external onlyOwner {
   uniswapRouter = _uniswapRouter;
}function setDevAddress(address _devAddress) external onlyOwner 
{
   devAddress = _devAddress;
}
```

# 将地址存储在配置文件中

地址实际上是配置数据，因此它们属于配置文件。

如果您使用 Hardhat/JavaScript 来执行单元测试或部署您的智能契约，那么一个`.json`文件是存储地址的一个好选择。您可以这样格式化它:

```
module.exports = {
   “LOCAL”: {
      uniswapRouter: “0x…123”,
      devAddress: “0x…de1”
   },      “TEST”: {
      uniswapRouter: “0x…456”,
      devAddress: “0x…de2”
   }, “MAIN”: {
      uniswapRouter: “0x…789”,
      devAddress: “0x…de3”
   }
}
```

然后，在部署您的智能合同时，您可能会执行以下操作:

```
const NETWORK = "LOCAL";
const uniswapRouter = require("./config")[NETWORK].uniswapRouter;
const devAddress = require("./config")[NETWORK].devAddress;
//
// ...
//
const myToken = await MyToken.deploy(uniswapRouter, devAddress);
```

现在你需要做的就是切换顶部的`NETWORK`常量来使用指定网络中的地址。您也可以设置您的代码接受命令行参数，类似于 Hardhat 接受参数的方式。

如果您有 dApp，您可以使用相同的技术在测试期间轻松地在网络之间切换。

通过修改上面的`NETWORK`常数，你仍然在改变代码。然而，这种方法与在智能契约或 dApp 代码中更改代码有几个重要的区别。

1.您没有修改*生产代码*。与修改部署后无法更改的智能联系人相比，修改帮助您运行单元测试的脚本的风险要小得多。

2.您正在一个更常见的地方进行更改。或者，至少比在代码文件中间更常见，可能有数百行代码深。

3.您正在进行*一项*更改(在“本地”、“测试”、“主”等之间切换。).而不是对每个地址进行单独的修改(在我们的例子中，1)注释掉`uniswapRouter`和 2)注释掉`devAddress`。

# 结论

尽可能避免硬编码地址。虽然一旦部署了智能契约，它的地址就永远不会改变，因此是“常数”，但它不是常数，就像`PI`是常数一样；该地址仅在特定网络范围内保持不变。

通过将地址设置为可配置的，使您的代码更容易使用，更重要的是，更容易测试。

__

如果你需要开发下一个 Web3 应用程序的帮助，请查看 10x development([https://10xdevelopment.com/](https://10xdevelopment.com/))。