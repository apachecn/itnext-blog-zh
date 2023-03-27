# URL 收缩

> 原文：<https://itnext.io/url-shortening-explained-2e524b1f8d4f?source=collection_archive---------8----------------------->

![](img/72359318891de717b614abddbcb24585.png)

url 缩短允许你把一个很长的 URL 缩短成一小段很好的文本，这样更容易分享。新的缩短的 URL 然后链接到真实的 URL。当我开始为这个做算法时，我的第一个直觉是从长 URL 中做一个唯一的散列，因为我知道长 URL 将是唯一的。但是后来我了解到最好的方法是使用一个数字 id 作为它的唯一标识符。为了做到这一点，我重新熟悉了碱基，以便从它们的 id 中生成独特的缩写 URL。

因此，挑战在于将 1 或 37，849，823，742 这样的数字映射到包含 URL 安全字符的字符串。我们举个例子，让 3GY 等于 id 10002。在其核心，我们需要使 3+G+Y = 10002。听起来不可能，对吧？g 和 Y 不是数字。这是真的，但是我们可以用基数来表示一个数值。现在我们来谈谈碱基。

我们知道最常见的基数是十进制。那是我们使用的数字系统。它被称为以 10 为基数，因为每个数字位置都有 10 个数字，直到它达到“最大值”并必须转到下一个位置。在我们的例子中，一个位置可以有 0，1，2，3，4，5，6，7，8，9 个数字。一旦超过 9，左侧递增，右侧重新开始，得到 10。

所以现在我们可以通过它的位置得到每个数字。在 10 的例子中，我们在第一个索引中有一个 1，可以表示为 1*(10 ) + 0*(10⁰).9823 可以表示为 9*10 + 8*10 + 2*10 + 3*10⁰.我们也可以通过除法“遍历”基数，通过不断地将数除以它的基数，将每一步写成“余数* 10^n'.”来得到这种格式我们这样做，直到被除的数等于 0

```
let num = 100 
num%10 = 3  => 3*10⁰
num = Math.floor(num/10) 
num%10 = 2 => 2*10¹ + 3*10⁰
num = Math.floor(num/10) 
num%10 = 8 => 8*10² + 2*10¹ + 3*10⁰
num = Math.floor(num/10) 
num%10 = 9 => 9*10³ + 8*10² + 2*10¹ + 3*10⁰
num/10 is now 0 so we stop
```

这些例子都是以 10 为基数的，但是以 13 为基数的会是什么样子呢？让我们试着得到 101 的 13 进制表示。

为了找到这个，让我们使用除法技巧。

```
let num = 101
console.log(num%13) => 9*13⁰
num = Math.floor(num / 13);
console.log(num%13) => 7*13¹ + 10*13⁰
```

所以我们可以看到 91 + 10 = 101。问题是 10 占据了 2 个数字位置，所以我们需要创造一个新的字符来代替 10。让我们有一个相等的 10。所以现在不是 7*13 + 10*13⁰，这将使我们写无效的 710。我们将它表示为 7*13 + A*13⁰，其中 a 表示我们所理解的 10。所以我们新的 13 进制数是 7A。

这是一个小例子，但是通过让等于 10，我们将 101 缩短为 7A。这是我们将如何使用字母来表示代表我们唯一 id 的其他数字的先驱。

因此，我们的基数为 13 的计数器可能是 0123456789ABC，其中 A、B 和 C 分别是 10、11 和 12。但是如果我们使用所有的大写字母、小写字母和数字呢？这将为我们提供 2 组字母和 9 个数字(我们排除 0，因为大多数数据库自动从 1 开始 ids)，这将等于一个基数为 61 的数字系统(26+26+9 = 61)。

所以让我们回到最初的挑战；看 3GY 怎么能等于 10002。使用我们的 61 进制数字系统会是这样的

```
const alphabet = "123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
const base = alphabet.length
```

长度将是我们的基数，由于字符串可以通过索引访问字符，我们可以说 alphabet[11]可以等于' b '。所以在我们以 61 为基数的数字系统中，11 等于 b。现在让我们用我们所学的知识来看看如何从 3GY 得到 10002:

```
const alphabet = "123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
const base = alphabet.lengthfunction decode(url){
  let result = 0;
  while (url){
    // see what each character equals in base 10
    let alphaIndex = alphabet.indexOf(url[0]);

    // find the number placment to act as base
    let power = url.length - 1;

    // increment the result
    result += alphaIndex * (Math.pow(base, power));

    // remove the left-most character in the url
    url = url.substring(1);
  }
  return result;
}decode('3GY')
// first loop: 2*61² 
// second loop: 41*61¹
// third loop: 59*61⁰
// decoded = 2*61² + 41*61¹ + 59*61⁰ => 10002 
```

现在我们可以用除法来计算 10002 如何等于 3GY:

```
const alphabet = "123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
const base = alphabet.lengthfunction encode(id){
  let encoded = '';
  while (id){

    // find out what the right most character is in base 10
    let toBaseTen = id % base;

    // put the charcter in the right-most position of the result
    encoded = alphabet[toBaseTen].toString() + encoded;

    // divide the number by the base to get ready to evaluate the next position
    id = Math.floor(id / base);
}
  return encoded;
}encode(10002)
// first loop: the remainder of 61 and 10002 is 59 => alphabet[59] is Y 
// second loop: the remainder of 61 and 163 is 41 => alphabet[41] is G
// third loop: the remainder of 61 and 2 is 2 => alphabet[2] is 3 
//result = 3GY
```

所以通过这个过程，我们实际上做了一个双射函数，这些函数允许一个集合中的每个元素恰好等于另一个集合中的另一个元素，反之亦然。在我们的例子中，3GY 只能解码为 10002，10002 只能编码为 3GY，这满足双射函数的要求。

就我们的数据库而言，我们的数据库可以有一个包含 id、shortURL 和 longURL 列的表

```
URLS table:
{id:10002, shortURL: 3GY, longURL: www.news.com/some-story/about/some-thing}
```

所以您可以使用您的 shortURL 编码到 id，然后通过 id 找到长 URL，这样您就可以重定向到长 URL。同样，当您输入一个要缩短的 URL，而该 URL 还没有包含在数据库中时，您可以创建一个新的条目，这会创建一个新的 id，然后您可以将该 id 编码到一个新的 shortURL 中。

我想当谈到网址，它的好处是有收缩。

我当然需要帮助来掌握这个概念。以下是一些帮助我摆脱困境的文章链接:

[](https://coligo.io/create-url-shortener-with-node-express-mongo/) [## 使用 NodeJs、Express 和 MongoDB 创建 URL Shortener

### 在本教程中，我们将使用 NodeJS，Express…创建一个类似于 bitly.com 网站的 URL 缩短服务

coligo.io](https://coligo.io/create-url-shortener-with-node-express-mongo/) [](https://code.tutsplus.com/tutorials/base-what-a-practical-introduction-to-base-encoding--net-27590) [## 基础什么？基本编码的实用介绍

### 在很小的时候，我们就学会用手指数数——从 1-5 开始，然后是 1-10，也许，如果你特别…

code.tutsplus.com](https://code.tutsplus.com/tutorials/base-what-a-practical-introduction-to-base-encoding--net-27590)