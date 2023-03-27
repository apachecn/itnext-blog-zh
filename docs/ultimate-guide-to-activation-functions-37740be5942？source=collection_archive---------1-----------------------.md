# 激活功能终极指南

> 原文：<https://itnext.io/ultimate-guide-to-activation-functions-37740be5942?source=collection_archive---------1----------------------->

寻找和学习激活功能的一个好地方是[维基百科](https://en.wikipedia.org/wiki/Activation_function)；然而，这些年来，激活函数的表波动很大，函数被一次又一次地添加和删除。你可以在这里查看这个特殊的维基百科页面[的历史变化列表](https://en.wikipedia.org/w/index.php?title=Activation_function&offset=20220309014907%7C1076034609&limit=9999&action=history)。用户[laughing the stocks](https://en.wikipedia.org/wiki/User:Laughsinthestocks)于 2015 年 11 月在[首次介绍了“激活功能表”。从那以后，在我写这篇文章的时候，维基百科页面已经有了 391 处修改。在这篇文章中，我写了一个算法，从这个维基百科页面截至 2022 年 4 月 22 日的历史中挖掘出每一个唯一的激活函数，这样我就可以在这里的一个综合文档中列出它们。我还提供了额外的链接，链接到没有激活功能的相关研究论文，或者在没有具体研究论文的情况下，在适当的位置提供感兴趣的相关论文。](https://en.wikipedia.org/w/index.php?title=Activation_function&oldid=689235273)

一般来说，人们会用 tanh 来代表 [FNN](https://en.wikipedia.org/wiki/Feedforward_neural_network) ，用 ReLU 来代表 [CNN](https://en.wikipedia.org/wiki/Convolutional_neural_network) 。

*如果我们包括身份激活函数，这个列表将包含 42 个激活函数，尽管你可以说如果包括双极性 sigmoid，它确实是 42 个。我没有读过《银河系漫游指南》。说真的。*

如果可能，衍生产品由 f(𝑥提供，但在某些情况下可能不是这样。那就是𝑥.了

## [二进制步骤](https://en.wikipedia.org/wiki/Heaviside_step_function)

![](img/0f3d00a3f09ac016a280163c0fc6b66f.png)![](img/ceb21779d235d7f725a8dad2bbd1e2e5.png)

二元阶跃激活

![](img/d0181c4add7197f69e7bda69737a5ed6.png)

二元阶跃导数

## [后勤](https://en.wikipedia.org/wiki/Logistic_function)，[乙状结肠](https://en.wikipedia.org/wiki/Sigmoid_function)，或者[软步](https://arxiv.org/pdf/1607.01691.pdf)

![](img/b8d5a4dc9fe414090085160ea3f823a7.png)![](img/14e56d2933c0d704bb63e0ad462714a4.png)

乙状结肠激活

![](img/979b3e480cb4241ff204aa0202effe08.png)

Sigmoid 导数

还有双极性的 sigmoid `(1.f-expf(-x)) / (1.f + expf(-x))`也许 [Wolfram 可以帮助你推导出](https://www.wolframalpha.com/input?i=%281.f-expf%28-x%29%29+%2F+%281.f+%2B+expf%28-x%29%29+derivative)。

## [ElliotSig](http://citeseerx.ist.psu.edu/viewdoc/download;jsessionid=603A43F0BC096B83BC7C42D605EBED99?doi=10.1.1.46.7204&rep=rep1&type=pdf) 或[软标记](https://arxiv.org/pdf/2003.00547.pdf)

![](img/684258ee9ff17ab2b9b7ba2c9b935f53.png)

ref[https://www.ire.pw.edu.pl/~rsulej/NetMaker/index.php?pg=n01](https://www.ire.pw.edu.pl/~rsulej/NetMaker/index.php?pg=n01)

![](img/e5ee9ba395ecd7210acfcaa2a17502b5.png)

ElliotSig 激活

![](img/bda1f0e5c0720970d63b51fb51103ea7.png)

ElliotSig 衍生物

## 双曲正切([正切](https://en.wikipedia.org/wiki/Hyperbolic_functions#Hyperbolic_tangent))

![](img/f6c6d8f7d059600421db5337d04603d5.png)![](img/bb84d8eb7d26ecc99ae35d46ce1359aa.png)

tanh 激活

![](img/75d6241d1d00d6723c9931829b98f549.png)

双曲正切导数

## [反正切](https://en.wikipedia.org/wiki/Arctangent) /反正切/反正切

![](img/b02f1594856c6cc5457d54d44c976f38.png)![](img/bb0b582c5d38d87a7b9e2e0d43cc13ec.png)

Arctan 激活

![](img/88bff920a0e598124c664c7310d3214d.png)

反正切导数

## [Softplus](https://arxiv.org/pdf/2009.03863.pdf)

![](img/0308448642093a692e2bd65be122a80b.png)![](img/ec5281eb1698402274b711dddf71b114.png)

Softplus 激活

![](img/bd598c1b9f9b69ae7e845f6a12576672.png)

软加衍生工具

## [整流线性单元](https://en.wikipedia.org/wiki/Rectifier_(neural_networks)) (ReLU) ( [ReLU6](http://www.cs.utoronto.ca/~kriz/conv-cifar10-aug2010.pdf) )

![](img/e3ab609c2f19d3e0053df80139eec0b3.png)![](img/3b0f4acaab3fbf086470be5e619d3c29.png)

ReLU 激活

![](img/fd4e83d4175c9d4a3811b940c75b377f.png)

ReLU 衍生物

## [指数线性单位(ELU)](https://arxiv.org/pdf/1511.07289.pdf)

![](img/bfd2e8592178a9aae9b26f15314dc453.png)![](img/1c36715b00b698b88815a7514b1488ac.png)

ELU 激活

![](img/34964863173cb960677fa9b265ffe121.png)

ELU 导数

## [高斯误差线性单元(GELU)](https://arxiv.org/pdf/1606.08415.pdf)

![](img/ff82ff7a05038bf6f620ea39afe557f8.png)

ref[https://medium . com/synced review/Gaussian-error-linear-unit-activates-neural-networks-beyond-relu-121d 1938 a1f 7](https://medium.com/syncedreview/gaussian-error-linear-unit-activates-neural-networks-beyond-relu-121d1938a1f7)

![](img/d71c0dac489b8f322833ad1e6e81590d.png)

GELU 激活

![](img/a881e4168afed37f967bfd8afeae9825.png)

格鲁衍生物

## [比例指数线性单位(SELU)](https://arxiv.org/pdf/1706.02515.pdf)

![](img/bbf39c399b428156e4e93f6850b129e4.png)

ref[https://arxiv.org/pdf/1807.10117.pdf](https://arxiv.org/pdf/1807.10117.pdf)

![](img/66eb1dab9351dc698122f0e5268d9356.png)

SELU 激活

![](img/8d74c0557a52b04594ce5c4be0810aa8.png)

SELU 导数

## [米什](https://arxiv.org/pdf/1908.08681.pdf)

![](img/6b6be319f1f92b14b5404afd769078fe.png)

ref【https://github.com/digantamisra98/Mish 

![](img/391e67554a7677d062e400dc7d99055b.png)

Mish 激活

![](img/caeb9122e40b5a6ad69b8c2cc2b7006c.png)

Mish 导数

## [漏泄整流线性单元(漏泄 ReLU)](https://arxiv.org/pdf/1505.00853.pdf)

![](img/863c9fe67de2002c012ee977923022f1.png)![](img/2b257ee21af7c34beb995ca447b632a6.png)

LReLU 激活

![](img/a7c16e2b27e87ae3f94b10d3bce73ab6.png)

LReLU 衍生物

## [参数整流线性单元(PReLU)](https://arxiv.org/pdf/1502.01852.pdf)

![](img/1d5aeb0b63540edd009a6085aa79cd2f.png)![](img/0f411d0790e60841f77c1a868a3efae9.png)

预激活

![](img/39d8c78355301ffa0e0d1e11e10c93aa.png)

预衍生

## [参数指数线性单元(PELU)](https://arxiv.org/pdf/1605.09332.pdf)

![](img/719cd381afe7572a0b18e89121243de5.png)![](img/5de3982b700497f7abf87030ce3b67f3.png)

PELU 激活

![](img/eb06c2b49eb65e0ae7ec298acb4b2e62.png)

PELU 导数

## [S 形整流线性激活单元(SReLU)](https://arxiv.org/pdf/1512.07030.pdf)

![](img/9cf93476f27504325c0ddc2ff6d0a57d.png)

ref[https://arxiv.org/pdf/1512.07030.pdf](https://arxiv.org/pdf/1512.07030.pdf)

![](img/238dbd02215c22ee857bc49b927373df.png)

SReLU 激活

![](img/dbbb96e042c7d72b70d4e39fe050303e.png)

斯雷卢衍生物

## [双极整流线性单元(BReLU)](https://arxiv.org/pdf/1709.04054.pdf)

![](img/8c860a95d6e361a061e7045b3bab3a99.png)

ref[https://arxiv.org/pdf/1709.04054.pdf](https://arxiv.org/pdf/1709.04054.pdf)

![](img/cec1501995eeefb19734bea893420b23.png)

布雷卢激活

![](img/1dcae26f194ca1210b954a7063de058a.png)

布雷卢衍生物

## [随机漏整流线性单元](http://www.gabormelli.com/RKB/Randomized_Leaky_Rectified_Linear_Activation_(RLReLU)_Function)

![](img/1d5aeb0b63540edd009a6085aa79cd2f.png)![](img/2cf11d887590774113d8f18077fcb928.png)

RReLU 激活

![](img/21758d50bcf1be49b26941c4e3b6974e.png)

RReLU 衍生物

## [乙状结肠线性单元(路斯)](https://arxiv.org/pdf/1702.03118.pdf)或[唰](https://arxiv.org/pdf/1710.05941v1.pdf)

![](img/3465abd2cfbb60631bf4681719c95cea.png)![](img/9bfc5479b26f6a9a517907413f559478.png)

嗖嗖激活

![](img/3939baf5b2586e924f6cad603f7570f1.png)

Swish 导数

## [高斯](https://en.wikipedia.org/wiki/Gaussian_function)

![](img/1a9c60de49bf836034f15ce6643d4f52.png)![](img/0d4702e569f1c1551fa7b98767279702.png)

高斯激活

![](img/8eeb516edec6b56ad8fadc151b93ffa5.png)

高斯导数

## [增长余弦单位(GCU)](https://arxiv.org/pdf/2108.12943.pdf)

![](img/c893bd97224f4a3e9d35985ca9d038c9.png)![](img/fa2ef2b49c9f9b4c864fa01cb6473ba7.png)

GCU 激活

![](img/3e6ab91990c672ea7b0524bf7685643b.png)

GCU 导数

## [移位二次单位(SQU)](https://arxiv.org/pdf/2111.04020.pdf)

![](img/b00cf2d0d8ab427dc3ff8149c487a646.png)![](img/c1694783cf92474733f831128b1f5ba8.png)

SQU 激活

![](img/5591ede07a4baff530050a5b3bbb479a.png)

SQU 导数

## [非单调立方单位(NCU)](https://arxiv.org/pdf/2111.04020.pdf)

![](img/1cf0b4927a401aa4810f8106116d62a1.png)![](img/32a05aebd8397efc15e704403d72148a.png)

NCU 激活

![](img/dc39108529e73ba74cc21eca072f27d0.png)

NCU 衍生物

## [移位正弦单位(SSU)](https://arxiv.org/pdf/2111.04020.pdf)

![](img/4b1199dcd5cc1c0f369fe69322120eaf.png)![](img/c2928a80b94a8981b8303508bc3484e2.png)

SSU 激活

*没有提供导数，参考* [*Wolfram*](https://www.wolframalpha.com/input?i=pi*sinc%28x-pi%29+derivative) *或* [*论文*](https://arxiv.org/pdf/2111.04020.pdf) *。*

## [衰减正弦单位(DSU)](https://arxiv.org/pdf/2111.04020.pdf)

![](img/2eff2a85aa9b9c9c7549c307b1d0da56.png)![](img/0ba45b648a7bf26a072f5bb674993216.png)

DSU 激活

*未提供导数，参考*[*Wolfram*](https://www.wolframalpha.com/input?i=pi%2F2+*+%28sinc%28x-pi%29+-+sinc%28x%2Bpi%29%29++derivative)*或* [*论文*](https://arxiv.org/pdf/2111.04020.pdf) *。*

## [费西合唱团](https://www.techrxiv.org/articles/preprint/Phish_A_Novel_Hyper-Optimizable_Activation_Function/17283824#:~:text=Phish%20is%20a%20novel%20non,constructed%20using%20different%20activation%20functions.)

![](img/77714db3525d812869e6de6ba5e0b1cb.png)![](img/f70224cdcef418cf9f8914a9bcad36c3.png)

费西合唱团激活

*未提供导数，参考*[*Wolfram*](https://www.wolframalpha.com/input?i=x+*+tanh%28+1%2F2*x*%281+%2B+%28x%2Fsqrt%282%29%29%29+%29+derivative)*或* [*论文*](https://www.techrxiv.org/ndownloader/files/33227273/2) *。*

## [SQ-RBF](https://en.wikipedia.org/wiki/Radial_basis_function)

![](img/f3b36033e3598ec0dc18b81eb45faf08.png)

SQ-RBF 激活

![](img/1de927b4bf0b03c7329ec179cc8131dc.png)

SQ-RBF 导数

## [平方根倒数单位(ISRU)](https://arxiv.org/pdf/1710.09967.pdf)

![](img/333e7a6fc18588f540354e7c5556e797.png)![](img/1d2f8026b1cdb990cd51d549a62d382d.png)

ISRU 激活

![](img/b84e75f51d244de22b2e7a9a0ab5f105.png)

ISRU 导数

## [反平方根线性单元(ISRLU)](https://arxiv.org/pdf/1710.09967.pdf)

![](img/b1744a915f18e9e227e26ae36e0aedd7.png)![](img/fc8884190db701642165d3ea3c69f5c5.png)

ISRLU 激活

![](img/68be24fdb81960487b2633b69cf2021d.png)

ISRLU 衍生物

## [平方非线性(SQNL)](https://raw.githubusercontent.com/awur978/SQNL_Paper/master/slides.pdf)

![](img/5f8990e0ea71fbf5738e8415d1bf95f4.png)![](img/b7f771547caa64cfd12d22db601647d6.png)

SQNL 激活

![](img/ff7a68f13b250db2c2935fca379e1fee.png)

SQNL 导数

## [乙状结肠收缩](https://hal.archives-ouvertes.fr/hal-02136546/document)

![](img/8e495c97e71aa4f3c9e568fbd70b1ed2.png)![](img/29e68fafd027b8f5a449da5dfa521e69.png)

激活

![](img/ae7f16e8c87295d8cd02779f9c987bb3.png)

导数

## [“挤压功能”](https://reader.elsevier.com/reader/sd/pii/S0950705120302896?token=65179BE047942C07C29012A1ECC13F16A7053F324AF9135822974715558837C2EAF447061E19518F8673EE9DA34ECBA2&originRegion=eu-west-1&originCreation=20220423000237) ( [基准](https://arxiv.org/pdf/2010.08760.pdf))

![](img/5de0ffa6a426aa9fd03de977b272e83e.png)![](img/c9dc7c28122135501fdeb9156c13720c.png)

激活

![](img/3070abebd5a04f5088fd19eb86353092.png)

w.r.t 激活

![](img/b56f53bf48151c54dccb5f26d022dc41.png)

导数

## [Maxout](https://arxiv.org/pdf/1302.4389.pdf)

![](img/c9586807bbd1e79ecd5d276288b5e338.png)

最大激活

![](img/14f4cce9cf108faefa0d9c133cc2ce5b.png)

“衍生产品”

## [弯曲的身份](https://arxiv.org/pdf/1611.05827.pdf)

![](img/95f6ee08d3de305d30bc5247da11f88b.png)![](img/dac5c4488a25fd9c68bc06b4e156b00b.png)

弯曲激活

![](img/3b52579e02f6fa65678b404327a5ad04.png)

弯曲导数

## [正弦曲线](https://en.wikipedia.org/wiki/Sine_wave)

![](img/bb0e749c003757ef96079c97051a952f.png)![](img/b04f3829eccff3b6a7835c7b3bc7d95a.png)

正弦激活

![](img/481338d6b2381f16c37108cf95ddbb3d.png)

正弦导数

## [Sinc](https://en.wikipedia.org/wiki/Sinc_function) ( [驯服海浪](https://openreview.net/pdf?id=Sks3zF9eg))

![](img/14d63c8f27c13a12758d1121d574203a.png)![](img/9bcd1df9435e539a65c6d3d9ef98d6cc.png)

Sinc 激活

![](img/c6f2fe6b88905ebdeda94213b39eade9.png)

正弦导数

## [阿尔辛赫](https://en.wikipedia.org/wiki/Inverse_hyperbolic_functions#Inverse_hyperbolic_sine)

![](img/08f37abfa47ce5d296f45fbc2b3a31f5.png)

ref[https://en.wikipedia.org/wiki/Inverse_hyperbolic_functions](https://en.wikipedia.org/wiki/Inverse_hyperbolic_functions)

![](img/0bf4dbb829041668bfe894bc94fc0201.png)

ArSinH 激活

![](img/3e058bf47727b16763e9c4bad4aa2710.png)

阿辛衍生物

## [软剪裁](https://arxiv.org/pdf/1810.11509.pdf) ( [金发女郎](https://arxiv.org/pdf/2002.05059.pdf))

![](img/b9b7e1cd972c9165ac6cb9163a833db0.png)![](img/59991196f9bef46bfca06b0afb894c27.png)

激活

![](img/d47b889fa3ef0fdbfe8da7373fefd117.png)

导数

## [分段线性单元(PLU)](https://arxiv.org/pdf/1809.09534.pdf)

![](img/b026096245696467ded528640061be3c.png)

ref[https://arxiv.org/pdf/1809.09534v1.pdf](https://arxiv.org/pdf/1809.09534v1.pdf)

![](img/85a3e0a3b01f0af04ac813deca27b288.png)

PLU 激活

![](img/4f9635140cf3f86cf5470f8a522fb5b4.png)

PLU 导数

## [自适应分段线性(APL)](https://arxiv.org/pdf/2108.00700.pdf)

![](img/f3d970c505ede5f9570570a2745ad5a4.png)

ref[https://arxiv.org/pdf/1512.07030.pdf](https://arxiv.org/pdf/1512.07030.pdf)

![](img/302151818b5478c5bc3e0576618b93a4.png)

APL 激活

![](img/371d060f07e654c7cadac2b2b5fbdc0b.png)

APL 衍生物

## [逆立方](https://en.wikipedia.org/w/index.php?title=Activation_function&oldid=760259047#Comparison_of_activation_functions)

![](img/71f68bfe5ba16e996d54ccde96695bab.png)

激活

![](img/06ffd278be7aeca0705be208066e6e62.png)

导数

## [软指数](https://arxiv.org/pdf/1602.01321.pdf)

![](img/06ffe8dcbbfb4e03a590a25e09dc6b24.png)![](img/abeecafab0bdaf64163aec9975912e5e.png)

激活

![](img/44dba16f5ad0e99a0535973cfa26a3ec.png)

导数

## [(42？)LeCun 双曲正切](http://yann.lecun.com/exdb/publis/pdf/lecun-98b.pdf)

![](img/cf7290a59baba91172ebe239f623301f.png)

ref【https://datascience.stackexchange.com/a/107616 号

## 进一步阅读

[预测金融时间序列的神经网络中新激活函数的比较](https://www.cin.ufpe.br/~tbl/artigos/NeuralComput&Applic-gecy-Final.pdf) ( [Logit & Probit](https://tutorials.methodsconsultants.com/posts/what-is-the-difference-between-logit-and-probit-models/)

[比例指数正则线性单元(SERLUs)的有效性](https://arxiv.org/pdf/1807.10117v2.pdf)

[](https://www.semanticscholar.org/paper/Comparison-of-new-activation-functions-in-neural-Gomes-Ludermir/9b37079041bdaca4248ab4f62f1a63013a50f067/figure/1) [## 金融预测神经网络中新激活函数的比较

### 表 1 激活函数及其导数标签激活函数对应的导数函数…

www.semanticscholar.org](https://www.semanticscholar.org/paper/Comparison-of-new-activation-functions-in-neural-Gomes-Ludermir/9b37079041bdaca4248ab4f62f1a63013a50f067/figure/1) [](https://www.semanticscholar.org/paper/Activation-Functions-for-Generalized-Learning-A-Villmann-Ravichandran/04d54996bcbe44b3547da889d7eab8aab3660990/figure/0) [## 表 1 来自广义学习矢量量化的激活函数-性能…

### 表 1。根据[11]的 MLP 及其衍生物的成功活化函数。-“激活…

www.semanticscholar.org](https://www.semanticscholar.org/paper/Activation-Functions-for-Generalized-Learning-A-Villmann-Ravichandran/04d54996bcbe44b3547da889d7eab8aab3660990/figure/0) [](https://www.semanticscholar.org/paper/A-comparative-performance-analysis-of-different-in-Farzad-Mashayekhi/bcfdfe54796c501a90c3b353661a19e9c161d2c8/figure/0) [## 表 1 来自 LSTM 网络中不同激活函数的比较性能分析…

### 表 1 每个激活函数的标签、定义、相应导数和范围-“比较性能…

www.semanticscholar.org](https://www.semanticscholar.org/paper/A-comparative-performance-analysis-of-different-in-Farzad-Mashayekhi/bcfdfe54796c501a90c3b353661a19e9c161d2c8/figure/0) [](https://www.semanticscholar.org/paper/Searching-for-Activation-Functions-Ramachandran-Zoph/c8c4ab59ac29973a00df4e5c8df3773a3c59995a/figure/2) [## 表 2 来自搜索激活功能|语义学者

### 表 2: CIFAR-100 精度。-“搜索激活功能”

www.semanticscholar.org](https://www.semanticscholar.org/paper/Searching-for-Activation-Functions-Ramachandran-Zoph/c8c4ab59ac29973a00df4e5c8df3773a3c59995a/figure/2) [](https://www.semanticscholar.org/paper/Flatten-T-Swish%3A-a-thresholded-ReLU-Swish-like-for-Chieng-Wahid/126e6a2f6d3e1cf1f793b9567a3c06509f01d329/figure/0) [## 图 1 来自 Flatten-T Swish:一个用于深度学习的阈值 ReLU-Swish 样激活函数…

### 图一。FTS (T = 0.00)与 ReLU - "Flatten-T Swish:一个阈值化的类似 ReLU-Swish 的激活函数，用于深度…

www.semanticscholar.org](https://www.semanticscholar.org/paper/Flatten-T-Swish%3A-a-thresholded-ReLU-Swish-like-for-Chieng-Wahid/126e6a2f6d3e1cf1f793b9567a3c06509f01d329/figure/0)