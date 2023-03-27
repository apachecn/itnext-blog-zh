# 了解角形管道

> 原文：<https://itnext.io/understanding-angular-pipes-5d1154f57d4f?source=collection_archive---------1----------------------->

## 配有用的例子！

![](img/df990ba937f83bd49c9b5ddd61ec3983.png)

> 管道是一种编写显示值转换的方法，可以在 HTML 中声明。它接受数据作为输入，并将其转换成所需的输出”。

角形管道的定义非常简单，并且意识到它们也非常有用。Angular 附带了一些有用的内置管道，可以随时使用，也允许我们创建自己的自定义管道；我们将在这里深入探讨。

# 内置管道

Angular 附带了一组管道，可以在任何模板中使用。我们现在要看一些最有用的以及如何使用它们。我们的期望不仅是让我们能够在角度应用中使用它们，还能帮助我们理解它们的工作原理。

## 数据管道

它根据提供的参数将日期值格式化为字符串。

```
const value = new Date('2019-03-01');<p>Birthdate: {{ value | date : 'dd/MM/YYYY' }}</p><!-- output '01/03/2019' -->
```

**输入:**一个日期对象，一个数字(从 UTC epoch 开始的毫秒)或一个 ISO 字符串；

**参数:**日期格式；

*您可以在这里查看所有格式选项:*[*https://angular.io/api/common/DatePipe*](https://angular.io/api/common/DatePipe)

## 百分比管道

它将一个数值转换成一个百分比字符串，根据提供的参数进行格式化。

```
const value = 23.1;<p>Percentage: {{ value | percent : '3.2-2' }}</p><!-- output '023,10%' -->
```

**输入:**一个数字；

**参数:**

*   `minIntegerDigits`:小数点前的最小整数位数。默认为`1`；
*   `minFractionDigits`:小数点后的最小位数。默认为`0`；
*   `maxFractionDigits`:小数点后最大位数。默认为`0`。

## SlicePipe

它将一个字符串转换成它的子串，或者将一个数组转换成它的子集。

```
const value = 'Leonardo';<p>Nickname: {{ value | slice : 0 : 2 }}</p><!-- output 'Leo' -->--------------------------------------------------const values = ['Red', 'Blue', 'Green', 'Purple'];<p>Colors:</p>
<ul>
  <li *[ngFor](https://angular.io/api/common/NgForOf)="let value of values| [slice](https://angular.io/api/common/SlicePipe) : 1 : 3" >
    {{ value }}
  </li>
</ul><!-- output 
  <ul>
    <li>Blue</li>
    <li>Green</li>
  </ul>
-->
```

**输入:**数组或字符串；

**参数:**

*   `startnumber`:要返回的子集的起始索引；
*   `endnumber`:要返回的子集的结束。

## 小写和大写

它将一个字符串转换为全部小写或全部大写。

```
const value = 'Leonardo';<p>In Lowercase: {{ value | lowercase }}</p><!-- output 'leonardo' --><p>In Uppercase: {{ value | uppercase }}</p><!-- output 'LEONARDO' -->
```

**输入:**一个字符串。

# 自定义管道

有时你可能需要**特定的转换**，在这种情况下默认的角管道是不够的。此时**自定义角度管道**发生，这允许您创建自己的管道，指定**输入**、**转换函数**和**输出**。

## 1.创建自定义管道

首先，我们将创建我们的管道。这将是一个非常简单的解释，其中我们将提供一个字符串参数，该参数将连接到一个 *Hello。*我们的做法如下:

```
import { Pipe, PipeTransform } from '[@angular/core](http://twitter.com/angular/core)';[@Pipe](http://twitter.com/Pipe)({
  name: 'sayHello'
})
export class GreetingsPipe implements PipeTransform { public transform(name: string): string {
    return 'Hello, ' + name;
  }}
```

在 **@Pipe decorator** 中指定的**名称**是将在 HTML 中使用的**管道的名称，如*日期*、*小写*、*大写*等。**

**转换函数**是魔法发生的地方，在这里你实际上对接收到的**输入**进行期望的转换。在上面的例子中，我们接收一个输入参数(**一个名称字符串**)，这是我们将要**应用转换函数的值。**转换函数中的第一个参数总是我们要处理的值。

然而，我们可以接收尽可能多的参数，以备不时之需。然后，我们将在 HTML 中提供它们，并能够在转换函数中使用它们，我们将进一步了解。最后，我们的转换函数**返回一个字符串**，这将是我们管道的**输出**。

## 2.注册自定义管道

然后，为了使用它，我们将注册它。要注册一个管道，你所要做的就是将它的类添加到一个模块声明中。在这里，我们还将为该管道创建一个模块，并将其导出以在其他模块中使用。

```
import { NgModule } from '@angular/core';
import { GreetingsPipe } from './greetings.pipe';@NgModule({
   declarations: [
     GreetingsPipe
   ],
   exports: [
     GreetingsPipe
   ]
})
export class GreetingsPipeModule { }
```

## 3.使用自定义管道

一旦您创建了自己的管道，在一个模块中注册了管道，并将该模块导入到您需要的地方，您的管道就可以使用了！

假设我们的组件中有一个属性为 *firstName、*的 *user* 对象，它是一个包含用户名字的字符串。然后，我们可以在 HTML 中添加一个段落来问候我们的用户，如下所示:

```
this.user = {
  firstName: 'Leonardo',
  ...
};--------------------------------------------------<div>
  <p>You are now logged in our application!</p>
  <p>{{ user.firstName | sayHello }}</p>
</div><!-- output
  <p>Hello, Leonardo</p>
-->
```

# 自定义管道示例

现在，我们将看到一些更真实的管道示例，它们可以在现实世界的应用中使用。

## 货币管道

首先，需要注意的是 Angular 确实有一个内置的[current ipe](https://angular.io/api/common/CurrencyPipe)供您使用。但是，我想展示如何构建一个，以防您需要一些自己的特定格式和出于学习目的。

假设我想要这个: *105.5 >雷亚尔 105.50*

```
import { Pipe, PipeTransform } from '[@angular/core](http://twitter.com/angular/core)';[@Pipe](http://twitter.com/Pipe)({
  name: 'currencyFormat'
})export class CurrencyFormat implements PipeTransform { public transform(
    value: number,
    currencySign: string = 'R$ ',
    chunkDelimiter: string = '.',
    decimalDelimiter: string = ','
  ): string { if (!value)
      return currencySign + '0' + decimalDelimiter + '00'; const changedValue = this.addCommas(value.toFixed(2));
    const formatted = changedValue.toString().replace(
      /[,.]/g, function (val) {
        return val === ',' ? chunkDelimiter : decimalDelimiter;
      }
    ); return currencySign + formatted;
  } private addCommas(nStr) {
    nStr += '';
    const x = nStr.split('.');
    let x1 = x[0];
    const x2 = x.length > 1 ? '.' + x[1] : '';
    const rgx = /(\d+)(\d{3})/;
    while (rgx.test(x1)) {
        x1 = x1.replace(rgx, '$1' + ',' + '$2');
    }
    return x1 + x2;
  }
}
```

正如我之前说过的，您可以根据需要接收尽可能多的输入参数，这就是我们的 *currencyFormat* 管道的情况。第一个参数是我们要格式化的值。然而，我们在这里也接收了 *currencySign (string)* 、 *chunkDelimiter (string)* 和 *decimalDelimiter (string)* ，所以我们可以在我们的*转换*函数中使用它们。这是我们使用烟斗的方式:

```
{{ 9999.9 | currencyFormat : 'R$' : '.' : ',' }}<!-- output: R$ 9.999.90 -->
```

## 文档格式管道

在巴西，每个人都有一份名为 *CPF* 的文件，这是一个具有特定格式的唯一号码。有时，我们需要通过接收包含其值的原始字符串来输出正确格式化的文档的值。

所以我们想要的是这个:*111111111111->111 . 111 . 111–11*

```
import { Pipe, PipeTransform } from '[@angular/core](http://twitter.com/angular/core)';[@Pipe](http://twitter.com/Pipe)({
  name: 'cpfFormat'
})
export class CpfFormat implements PipeTransform { public transform(cpf: string): string {
    if (!cpf || cpf.trim() === '') { return ''; }
    let formatted: string;
    formatted = cpf.substring(0, 3) + '.';
    formatted = formatted + cpf.substring(3, 6) + '.';
    formatted = formatted + cpf.substring(6, 9) + '-';
    formatted = formatted + cpf.substring(9, 11);
    return formatted;
  }}
```

这里的管道要简单得多。除了值本身，我们不需要任何其他参数，因为 CPF 的格式是一种定义好的模式。我们接收一个未格式化的 CPF 字符串，并返回格式化的 CPF 字符串。这是我们使用烟斗的方式:

```
{{ '99999999999' | cpfFormat }}<!-- output: 999.999.999-99 -->
```

# 额外:测试管道

为定制管道创建单元测试非常简单，因为它们基本上是纯函数；其接收输入并返回期望的输出。下面，我有一个为之前创建的 CPF 管道创建一个简单测试的例子。

```
import { CpfFormat } from './cpf.pipe';describe('[Pipe] CPF Format', () => {   let pipe: CpfFormat; beforeEach(() => {
    pipe = new CpfFormat();
  });  it('should return a valid \'cpf\' formatted correctly', () => {
    expect(pipe.transform('32726532950')).toBe('327.265.329-50');
  }); it('should return empty for a null \'cpf\'', () => {
    expect(pipe.transform(null)).toBe('');
    expect(pipe.transform('')).toBe('');
  });});
```

Angular Pipes 是 Angular 提供的一个非常简单但有用的功能，几乎可以肯定它会在整个应用程序中被大量使用。因此，重要的是了解你的方式，并能够有效地使用它们来满足你的需求。

希望有帮助！😉

# 参考资料:

 [## 有角的

### Angular 是一个构建移动和桌面 web 应用程序的平台。加入数百万开发者的社区…

angular.io](https://angular.io/guide/pipes)