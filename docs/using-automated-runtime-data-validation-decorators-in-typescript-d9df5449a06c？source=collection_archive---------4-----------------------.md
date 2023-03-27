# 在 TypeScript 中使用自动化运行时数据验证装饰器

> 原文：<https://itnext.io/using-automated-runtime-data-validation-decorators-in-typescript-d9df5449a06c?source=collection_archive---------4----------------------->

![](img/6ca6cfd42c9e934a317a63988958b5bd.png)

作者图片

【TypeScript 的一个新包使我们能够自动检查运行时数据的有效性，而不必显式调用验证库。相反，*运行时数据验证*包让我们附加描述数据有效性的装饰器，并安排在给属性赋值或调用方法时自动执行验证。

在 JavaScript 中没有类型检查，也没有内置的数据验证。这是这门语言的核心特征，也是它如此受欢迎的部分原因。与此同时，我们在越来越大的项目中使用 JavaScript，理论上，这使得有必要使用软件工具来帮助我们编写更简洁的应用程序。在其他语言中，这意味着严格的类型检查和运行时数据检查，但是在 JavaScript 中，我们不想偏离核心命题太远。

TypeScript 是 JavaScript 试验高级语言特性的一种方法。例如，它将类型声明和编译时类型检查添加到 JavaScript 中。最近，在更新我的《在 Node.js 应用程序中使用 Typescript 和 TypeORM 的快速入门》一书(赞助商链接)时，我了解了如何使用 TypeScript decorators。从那以后，我创建了一个关于使用和实现 TypeScript 装饰器的系列文章[。有一篇文章是关于](https://techsparx.com/nodejs/typescript/decorators/introduction.html)[开发 TypeScript decorators，用于在 JavaScript/TypeScript](https://techsparx.com/nodejs/typescript/decorators/runtime-validation.html) 中执行运行时数据验证。从那篇文章中，我创建了`[runtime-data-validation](https://github.com/runtime-data-validation-js/runtime-data-validation-typescript)`包，它是数据验证的完整实现，在 TypeScript 程序中，它在两种情况下自动验证数据:

1.  带有附加验证装饰器的`set`访问器属性的赋值
2.  对参数附加了验证装饰符的类方法的调用

这篇文章是关于使用包的，而前一篇文章是关于实现执行运行时数据验证的装饰器。这种方法不同于像 Joi 或 AJV 这样的软件包，程序员必须显式地编写数据验证代码。使用`runtime-data-validation`，程序员将 decorators 附加到访问器或方法参数上，这样这些方法就能自动防止无效数据。

举个简单的例子，考虑这个类:

```
import {
    ValidateAccessor, IsFloatRange,
    conversions
} from 'runtime-data-validation';

const { ToFloat } = conversions;

export class SpeedExample {

    #speed: number;

    @ValidateAccessor<number | string>()
    @IsFloatRange(10, 70)
    set speed(nc: number | string) { 
        this.#speed = ToFloat(nc);
    }
    get speed() { return this.#speed; }

}
```

有一个私有属性`#speed`和一个处理器函数对`speed`，它处理私有属性的获取和设置。`@IsFloatRange`装饰器确保该值是在`10`和`70`之间的浮点数。它还处理一个碰巧是数字的`string`值。

```
const sp = new SpeedExample();
sp.speed = VALUE;
```

对于像`30`或`'30'`这样的值，这可以正确执行。对于像`300`或`'300'`或`'300 miles/hr'`这样的值，会抛出一个错误。该错误阻止了`set`访问器的执行，并阻止了属性被赋予不正确的值。

换句话说，`runtime-data-validation`包的核心价值主张是针对由验证装饰器保护的属性或方法，它们不会收到不正确的数据。任何不正确的数据都将被检测到，从而引发错误。

# 安装`runtime-data-validation`包

本教程是为 Node.js 项目编写的。在浏览器端项目中使用`runtime-data-validation`尚未经过测试。因此，设置一个 Node.js 项目，并对其进行配置以进行 TypeScript 开发。例如:

```
$ npm init -y 
$ npm install typescript @types/node --save-dev
```

接下来，要为支持 decorators 做两个设置。在您的`tsconfig.json`文件中进行这些设置:

```
{
    "compilerOptions": {
        ...
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true,
        ...
    }
}
```

完成这个设置后，按如下方式安装软件包:

```
$ npm install runtime-data-validation
```

上面显示的示例代码现在应该运行了。

有关该包的详细文档，请参见:[https://runtime-data-validation-js . github . io](https://runtime-data-validation-js.github.io)

# 类型检查与数据验证

数据验证与类型检查略有不同。例如，类型为`string`的变量`title`理论上可以包含任何字符串值。但是，应用程序可能要求这个变量具有给定地区/语言的文本，或者被约束为特定的格式。

类型检查确保变量`title`是一个`string`。数据验证不仅确保变量具有正确的类型，而且确保其当前值在所需的约束范围内。

# 执行装饰器和验证装饰器

从`runtime-data-validation`出口的装修工有两种。

```
import {
    IsIntRange, IsInt, IsFloatRange, IsFloat,
    ...
    ValidateParams, ValidateAccessor,
} from 'runtime-data-validation';
```

大多数的*验证装饰符*都被拼写成`@IsXYZZY`，这里应该理解为“ *value is type XYZZY* ”。它们执行数据验证，每个都包含一个验证函数。

两个*执行装饰器*分别是`@ValidateParams`和`@ValidateAccessor`。第一个用于方法，处理附加到方法参数的验证装饰器的执行，而第二个用于`set`访问器，处理附加到访问器的验证装饰器的执行。

为了了解两者的作用，我们来看一个更大的例子:

```
import {
    ValidateAccessor, ValidateParams,
    IsIntRange, IsInt, IsFloatRange,
    conversions
} from 'runtime-data-validation';

const { ToFloat, ToInt } = conversions;

export class ValidateExample {

    #year: number;

    @ValidateAccessor<number>()
    @IsIntRange(1990, 2050)
    @IsInt()
    set year(ny: number | string) { this.#year = ToInt(ny); }
    get year() { return this.#year; }

    @ValidateParams
    area(
        @IsFloatRange(0, 1000) width: number | string,
        @IsFloatRange(0, 1000) height: number | string
    ) {
        return ToFloat(width) * ToFloat(height);
    }

}
```

这里有两项受到验证装饰器的保护:

1.  私有属性`#year`和`set`访问器`year`，用`@IsInt`和`@IsIntRange`声明。这些装饰器确保它的值是一个介于`1990`和`2050`之间的整数。`@ValidateAccessor`装饰器处理验证装饰器的执行。
2.  方法`are`有两个参数，每个参数都附加了`@IsFloatRange`，确保值在`0`到`1000`的范围内。`@ValidateParams`装饰器处理验证装饰器的执行。

该类的测试可能如下所示:

```
const ve = new ValidateExample();

ve.year = 1999;
console.log({
    year: ve.year,
    area: ve.area(10, 200)
});

// Error: Value 2150 not an integer between 1990 and 2050
ve.year = 2150;

console.log(ve.area('10', '200'));

// Error: Value 'two hundred' not a float between 0 and 1000
console.log(ve.area('10', 'two hundred'));
```

对于值在可接受范围内的赋值或方法调用，语句执行时不会出错。对于具有超出范围或无效值的语句，将抛出此处显示的错误。比如写`'two hundred'`的程序员可能本意是好的，但是文本字符串不是数值。

# 验证功能

在 TypeScript 文档中，我们被敦促实现*类型保护*函数来帮助运行时类型检查。该功能是接收数据项，检查该数据项，并确定它是否具有正确的形状。

您可能声明了一个类型:

```
type CustomType = {
    flag1: boolean;
    speed: number;
    title?: string;
};
```

你的程序如何确定一个变量是否有这种形状？在这种情况下，不能使用`instanceof`。这个想法是检查字段并确保它们匹配。

```
const isCustomType = (value: any): value is CustomType => {
    if (typeof value !== 'object') {
        return false;
    }
    // Both of these fields are required in
    // the type definition
    if (typeof value?.flag1 !== 'boolean'
     || typeof value?.speed !== 'number') {
        return false;
    }
    // Speed must be a positive number
    if (value.speed < 0) {
        return false;
    }
    // This field is optional, and therefore can
    // be undefined.
    if (typeof value?.title !== 'undefined'
     && typeof value?.title !== 'string') {
        return false;
    }
    // Be certain that `title` is correct
    if (typeof value?.title !== 'undefined'
     && typeof value?.title === 'string') {
        if (!validators.isAscii(value.title)
         || !validators.matches(value.title, /^[a-zA-Z0-9 ]+$/)) {
            return false;
        }
    }
    return true;
};
```

参数的类型`any`允许测试任何变量。返回类型`value is CustomType`，就是 TypeScript 所说的*类型谓词*。在行为上，它与`boolean`没有区别，换句话说，它是一种`true`或`false`状态。在这个函数中有一些测试来确保`value`有正确的字段来匹配`Customtype`。

函数的最后一部分超出了类型检查。这将检查`valu.title`中字符串的格式。不管出于什么原因，这个应用程序希望它是包含字母、数字和空格的 ASCII 文本。

注意，为了验证文本格式，我们使用了`validators.isAscii`和`validators.matches`。这些函数中的每一个都有相应的验证装饰器。但是这个上下文不允许我们使用 decorators。

包含在`runtime-data-validation`中的是一组对应于验证装饰器的验证函数。每个验证装饰器使用一个验证函数。

要使用这些功能:

```
import { validators } from 'runtime-data-validation';
```

这个导入提供了对包含验证函数的`validators`对象的访问。

我们已经看到了如何使用这些函数。它们的行为与类型守卫函数相同，这意味着它们接受一个值，然后提供一个`boolean`通知您该值是否匹配。

# 自定义验证装饰器

虽然`runtime-data-validation`包包含了一长串验证装饰器，但它们并不是数据验证的最终目的。有接近无数的其他数据验证可以执行。

为此，`generateValidationDecorator`函数允许应用程序创建验证装饰器。例如， *CustomType* 类型的验证装饰器可能如下所示:

```
function IsCustom() {
  return generateValidationDecorator(
               isCustomType,
              `Value :value: is not a CustomType`); 
}
```

每个 TypeScript 装饰器都由一个函数实现。这意味着`IsCustom`函数可以像`@IsCustom()`一样用作装饰器。

`generateValidationDecorator`的第一个参数是用于验证的函数。该函数将接收一个值，并预期返回一个`boolean`，其中`true`表示该值是正确的。换句话说，这个函数需要一个类型保护函数。第二个参数是一个字符串，如果值不合适，就打印出来，`:value:`接收`util.inspect(value)`的结果。

假设您需要进一步定制验证。例如，`speed`字段可能有一个可接受的值范围。

```
function IsCustomRange(min: number, max: number) {
    return generateValidationDecorator(
        (value) => {
            if (!isCustomType(value)) return false;
            const ct = <CustomType>value;
            if (ct.speed < min || ct.speed > max) return false;
            return true;
        },
        `Value :value: is not a CustomType`);
}
```

这遵循装饰工厂模式。外部函数的参数配置如何验证值。在这种情况下，新验证函数的代码首先调用`isCustomType`，然后检查`speed`以验证它是否在要求的范围内。

要使用它，将`@IsCustomRange(0, 150)`附加到一个访问函数或方法参数。

# 摘要

`runtime-data-validation`包填补了 JavaScript/TypeScript 功能的空白。有了它，我们可以在值到达我们的对象实例之前自动验证它们。

它提供了一长串内置的验证装饰器和验证函数。此外，它让您可以轻松定义自己的验证装饰器。

由于 TypeScript 实现 decorator 的方式，decorator 只能附加到`set`访问器和方法参数上。若要验证其他内容，请使用验证函数，或者实现您自己的类型保护函数。

# 关于作者

[***David Herron***](https://davidherron.com)*:David Herron 是一名作家和软件工程师，专注于技术的明智使用。他对太阳能、风能和电动汽车等清洁能源技术特别感兴趣。David 在硅谷从事了近 30 年的软件工作，从电子邮件系统到视频流，再到 Java 编程语言，他已经出版了几本关于 Node.js 编程和电动汽车的书籍。*

*原载于*【https://techsparx.com】**。**