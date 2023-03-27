# JSF·CK——代码古怪会影响执行性能吗？

> 原文：<https://itnext.io/jsf-ck-does-code-weirdness-affect-execution-performance-779816003ef1?source=collection_archive---------1----------------------->

## 检查我们如何编写 JS 是否可以改变其执行的性能，即使是相同的代码？

![](img/0c4f800f94be8bfa0c9524a7621d75ed.png)

来源:https://media.boingboing.net/

> *这篇文章是基于马丁·克莱普(* [*@aemkei*](https://twitter.com/aemkei) *)关于将 JS 转换成仅六个基本字符并保持其可执行性的工作而创建的。是对来自* [*的评论的回应乔纳森的意思是*](https://www.youtube.com/watch?v=sRWE5tnaxlI&lc=Ugw8lYvWtPQReYaahmZ4AaABAg) *。*

对于那些断章取义的读者，在开始测试之前，我将花点时间解释一下这个问题。

# 什么是 JSF*ck？

除了是 NSFW，这是一种只用六个字符编写和执行 JS 代码的编程风格。我们之所以能这样做，是因为有一种叫做**型强制**的东西。JS 强制偶尔会在 JS 会议上出现(主要是为了制作一些有趣的视频)，它的目的是允许用户操作不同类型的类型，而不是显式地将它们转换成一种类型。

```
const a = "1" + 5 // "15"
```

这个想法很棒，但有时(或者更可能是经常)人们不明白它实际上是如何工作的。对于那些对这个想法感兴趣的人，这里有一个规范的链接[http://www . ECMA-international . org/ECMA-262/# sec-type-conversion](http://www.ecma-international.org/ecma-262/#sec-type-conversion)。

大多数演讲都选择了一些非直觉的例子，比如

```
const a = [] + [] // ""
```

还是最有名的一个(`+ +"a"`回报**“楠”**)

```
// "banana"
const yellowFruit = ("b" + "a" + +"a" + "a").toLocaleLowerCase()
```

# 我们能用它做什么？

由于强制，我们应该能够在 JavaScript 中创建任何有效的句子(包括求值)。如果我们能做到这一点，那么我们应该能够将我们的代码转换成如下形式:

```
// 5+4
[!+[]+!+[]+!+[]+!+[]+!+[]]+(+(+!+[]+(!+[]+[])[!+[]+!+[]+!+[]]+[+!+[]]+[+[]]+[+[]])+[])[!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]
```

实现这一点并不像你想象的那么简单。

得到一个字符串形式的数字

```
+[] // "0"
+!+[] // "1"
[+!+[]]+[+[]] // "10
```

获得字符串更加困难

```
[][[]][!+[]+!+[]] // "d" or "undefined"[2]
```

它利用了字符串可以作为字符数组被访问的事实，你可以通过调用`[][[]]`很容易地生成 **"undefined"** 字符串。

我不打算详细解释其中的每一个。https://www.youtube.com/watch?v=sRWE5tnaxlI 的 T2 发布了一个很棒的视频，涵盖了这个话题。如果你想玩它，请去 http://www.jsfuck.com/的[试试你的代码。](http://www.jsfuck.com/)

# 对性能的影响

您可能会注意到，以这种方式写下整串代码非常占用空间。我已经设法将一些测试代码解析成 JSF 约定和代码，如下所示

```
const startT1 = Date.now();

const N = 10000;

let f = { tmp: 3, tmp2: 3, tmp3: 3, tmp4: 3, tmp5: 3, a: 'Gandalf', b: 'The Grey' };
let f2 = { tmp: 3, tmp2: 3, tmp3: 3, tmp4: 3, tmp5: 3, a: 'Jack', b: 'Sparrow' };
let f3 = { tmp: 3, tmp2: 3, tmp3: 3, tmp4: 3, tmp5: 3, a: 'Charles', b: 'Xavier' };
let f4 = { tmp: 3, tmp2: 3, tmp3: 3, tmp4: 3, tmp5: 3, a: 'Frodo', b: 'Baggins' };
let f5 = { tmp: 3, tmp2: 3, tmp3: 3, tmp4: 3, tmp5: 3, a: 'Legolas', b: 'Thranduilion' };
let f6 = { tmp: 3, tmp2: 3, tmp3: 3, tmp4: 3, tmp5: 3, a: 'Indiana', b: 'Jones' };

function test(obj) {
  let result = '';
  for (let i = 0; i < N; i += 1) {
    result += obj.a + obj.b;
  }
  return result;
}

for(let i = 0; i < N; i += 1) {
	test(f);
	test(f2);
	test(f3);
	test(f4);
	test(f5);
	test(f6);
}
console.log("test with one shape:", Date.now() - startT1, "ms.");
```

解析成 847192 行长的字符串。因此，我们现在有 828KB 的文件，而不是 1KB 以上。您可以在这里获取代码[，并通过调用`node index.js`来执行它。](https://gist.github.com/burnpiro/5a177d8bb307005c4d8f5672fe9ff0a3)

## 是时候开始测试我们的代码了！！！

测试环境:

```
Ubuntu 18.04 
Node 12.13.0 
Intel i7-7820X
```

**测试示例:**

*   标准呼叫—[https://gist . github . com/burnpiro/56be 270 AC 032924 faf 48824 e 08995687](https://gist.github.com/burnpiro/56be270ac032924faf48824e08995687)
*   string eval—[https://gist . github . com/burn piro/ffaa 8 edac 33370 E1 aa 5126 c 83 FB 728 bb](https://gist.github.com/burnpiro/ffaa8edac33370e1aa5126c83fb728bb)
*   JSF 评估—[https://gist . github . com/burnpiro/5a 177 D8 bb 307005 C4 D8 f 5672 Fe 9 ff 0 a 3](https://gist.github.com/burnpiro/5a177d8bb307005c4d8f5672fe9ff0a3)

我测试了两次标准函数，以便能够发现`eval`执行中的差异。

**测试结果:**

标准呼叫(50 个样本)

```
AVG: 3274ms 
Std. Dev: 7ms 
Heap: 8.06MB
```

字符串评估(50 个样本)

```
AVG: 3272ms 
Std. Dev: 6ms 
Heap: 8.08MB
```

**JSF** 评估(50 个样本):

```
AVG: 3241ms 
Std. Dev: 8ms 
Heap: 18.88MB
```

> *注意！您机器上的值可能不同，因为您使用的是不同版本的 nodeJS 或不同的处理器。*

# 结论

V8 (node)中同一代码不同版本的执行性能没有区别。这里看到的是 V8 使用的内存量，这是意料之中的，但也是非常重要的。求值和标准函数调用几乎没有区别，但是将代码解析为 JSF 比原始代码需要更多的内存。

你看了也没那么惊讶。在 JS Engine 中存储和解析 1MB 的文件要比 1KB 的文件占用更多的内存。

即使 JSF 代码不实用，人们也可以通过阅读它的一部分来更好地理解 JS 是如何工作的。类型强制并不像许多人在推特上说的那样神奇。我知道 ***“任何足够先进的技术都无法与魔法区分开来”*** (克拉克第一定律)但我喜欢重新表述一下: ***“任何无法理解的技术都无法与魔法区分开来”*** 。如果你能理解它，它就不再是魔法了:)

*最初发布于*[*https://erdem . pl*](https://erdem.pl/2019/11/jsf-ck-does-code-weirdness-affect-execution-performance)*。*