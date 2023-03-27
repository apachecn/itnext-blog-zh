# PHP 中如何检查变量？

> 原文：<https://itnext.io/how-to-check-variables-in-php-1366c39b0479?source=collection_archive---------1----------------------->

![](img/0404a35e1369802b283b8c159ab010a8.png)

PHP 是一种编程语言，由拉斯马斯·勒德尔夫于 1994 年创建，其主要目的是建立网站。现在，我们可以说它是一种通用的服务器端脚本语言，仍然受到大力支持。

在 PHP 中，我们并不总是从变量中得到期望值。因为在真实的软件开发场景中，开发人员可能

*   忘记声明变量或给声明的变量赋值。
*   调用函数时忘记传递所需的参数。
*   使用来自 API 请求或数据库查询的结果，该结果可能包含不需要的值。
*   以 Cookie 或会话值的形式直接访问全局变量，这些值尚未定义或具有意外值。

因为软件开发中的这些场景，作为开发人员，我们在编码时需要更加小心。我们需要采用防御性编程方法，这意味着代码要为处理意外情况做好准备，并始终保持警惕。

在我们的例子中，这意味着当处理变量时，我们应该总是期望变量值是意外的或不希望的，我们应该考虑到这一点来编码。这应该是最安全的编码方法之一。

PHP 社区已经用许多不同的解决方案涵盖了这些类型的场景，在本文中，我们将用以下主题来涵盖这些解决方案:

1.  检查变量的重要性
2.  如何使用内置函数检查 PHP 中的变量
3.  这些内置函数的行为
4.  决定正确的功能

# 1.检查变量的重要性

使用未检查变量的副作用有几个例子:

**场景 1:使用未声明的变量**

```
echo $undeclaredVariable;
```

**输出:**

```
PHP Notice:  Undefined variable: undeclaredVariable
```

**场景 2:使用不需要的变量值**

```
$emptyUserLastname = "";
echo "Hi Mr. " . $emptyUserLastname;
```

**输出:**

```
Hi Mr.
```

**场景 3:使用虚假的服务调用响应**

```
function getUserUnsuccessful() {
   return array();
}
$user = getUserUnsuccessful();
echo "Welcome Mr. " . $user["LASTNAME"];
```

**输出:**

```
PHP Notice:  Undefined index: LASTNAME
```

**场景 4:使用未检查的全局变量**

```
echo $_COOKIE["USER_ID"];
```

**输出:**

```
PHP Notice:  Undefined index: USER_ID
```

在这些场景中，您可以看到一些 PHP 通知和一些意外的行为。当然，在这种情况下，你的代码不会崩溃或终止，但由于这种未检查的变量使用，你可能会面临算法的意外行为，这也有另一个名称为 bug。

# 2.如何使用内置函数检查 PHP 中的变量

幸运的是 PHP 有很多内置函数来检查变量。但是我们不能在不知道它们确切行为的情况下使用它们。因为它会导致我们面对算法的意外行为。为了首先避免它，我们需要知道我们可以使用哪些函数，以及它们的确切行为是什么。

我最喜欢 PHP 的一点是，您不需要任何第三方库来解决这种情况，因为 PHP 社区已经涵盖了许多具有内置函数的不同主题。所以你不需要重新发明轮子。

PHP 第二个令人喜欢的部分是几乎每个内置函数都是可读的，这意味着你可以通过阅读函数名来理解它们的目的和行为。

> *警告:在 PHP 中任何函数的使用在如何传递参数或知道返回值方面都会有问题。这是对 PHP 抱怨最多的事情之一。PHP 的创始人拉斯马斯·勒德尔夫甚至提到，在过去的日子里他们没有任何好的例子。他们只有 C 实现作为例子。他们选择坚持这一点，以便在短时间内实现目标。*

在 PHP 中，变量检查有两个常用的函数 isset()和 empty()。现在让我们看看这些检查变量及其确切行为的内置函数。

# 3.这些内置函数的行为

# 伊塞特()

从函数名可以理解，它检查给定的参数是否设置为任何值。所以它接受一个变量作为参数，并检查给定的参数

*   已声明
*   已分配
*   具有除 null 之外的任何值。

如果变量满足所有这些条件，isset()函数将返回 TRUE。否则，它将返回 FALSE。

以下是使用 isset()函数的一些示例:

```
var_dump(isset($undeclaredVariable)); // prints bool(false)

$declaredVariable;
var_dump(isset($declaredVariable)); // prints bool(false)

$declaredAndAssignedVariable = NULL;
var_dump(isset($declaredAndAssignedVariable)); // prints bool(false)

$declaredAndAssignedVariableOtherThanNull = 1;
var_dump(isset($declaredAndAssignedVariableOtherThanNull)); // prints bool(true)

$boolean = FALSE;
var_dump(isset($boolean)); // prints bool(true)

$boolean = TRUE;
var_dump(isset($boolean)); // prints bool(true)

$string = "";
var_dump(isset($string)); // prints bool(true)

$string = "0";
var_dump(isset($string)); // prints bool(true)

$string = "A";
var_dump(isset($string)); // prints bool(true)

$integer = 0;
var_dump(isset($integer)); // prints bool(true)

$integer = 1;
var_dump(isset($integer)); // prints bool(true)

$float = 0.0;
var_dump(isset($float)); // prints bool(true)

$float = 0.1;
var_dump(isset($float)); // prints bool(true)

$array = array();
var_dump(isset($array)); // prints bool(true)

$array = array("A");
var_dump(isset($array)); // prints bool(true)
```

对于未声明和未赋值的变量，PHP 接受其值为 NULL。如您所见，除了未声明、未赋值和空值，isset()函数返回 TRUE。当我们希望确保变量是否被声明、赋值或不为空时，case isset()函数非常有用。

# 空()

我们可能会从函数名中误解函数的用途。看起来它使给定的参数为空，但实际上它只检查给定的参数是否为空。我可以听到反对的声音，比如 isEmpty()可能是这个函数更好的名字。

但这又是旧时代的问题，因为传统(不是神圣的传统，支持新 PHP 版本的向后兼容性，并鼓励旧版本用户更新他们的 PHP 版本)，它一直保持这种方式。

简而言之，empty()函数接受一个参数来检查给定变量是否为空。当我们观察它的行为时，它的行为类似于 isset()函数加上检查变量值空，这意味着如果给定的参数等于:

*   空
*   整数 0，双精度 0.00，浮点 0.0
*   空字符串(")
*   布尔假
*   空数组

此外，这个函数有一个边缘情况，它也假定一个字符串有“0”，“0.0”，“0.00 ..”字符也是空的。

在上述情况下，它返回 TRUE，因为 empty 函数接受这些值为空。

# 但是为什么呢？

因为 empty()函数变量的空性意味着在布尔等式方面检查变量值。所以 PHP 决定给定的变量值布尔等式是真还是假。

我们可以看到 boolval()函数的布尔值相等，该函数从[https://www.php.net/manual/en/function.boolval.php](https://www.php.net/manual/en/function.boolval.php)返回一个给定参数的布尔值。所以在 PHP 中:

*   空
*   整数 0
*   双 0.00
*   浮动 0.0…
*   空字符串(")
*   布尔假
*   空数组

布尔等式的值为假。基于此，如果我们总结一下 empty()函数:

```
**empty() = isset() + boolval()**
```

通过这个公式我们很容易理解 empty()函数的行为。我们可以说 empty()函数是比 isset()函数更宽的变量校验函数。所以我们不需要在同一种情况下一起使用 isset()和 empty()函数。这是关于决定变量的空值是否困扰我们。最终，我们应该至少使用其中一个来处理 PHP 中的变量。

下面是一些使用空函数()的例子

```
**var_dump(empty($undeclaredVariable)); // prints bool(true)**

**$declaredVariable;**
**var_dump(empty($declaredVariable)); // prints bool(true)**

**$declaredAndAssignedVariable = NULL;**
**var_dump(empty($declaredAndAssignedVariable)); // prints bool(true)**

**$declaredAndAssignedVariableOtherThanNull = 1;**
**var_dump(empty($declaredAndAssignedVariableOtherThanNull)); // prints bool(false)**

**$boolean = FALSE;**
**var_dump(empty($boolean)); // prints bool(true)**

**$boolean = TRUE;**
**var_dump(empty($boolean)); // prints bool(false)**

**$string = "";**
**var_dump(empty($string)); // prints bool(true)**

**// Edge case, empty() accepts "0", "0.0", "0.00", "0.000..." strings as empty strings**
**$string = "0";**
**var_dump(empty($string)); // prints bool(true)**

**$string = "A";**
**var_dump(empty($string)); // prints bool(false)**

**$integer = 0;**
**var_dump(empty($integer)); // prints bool(true)**

**$integer = 1;**
**var_dump(empty($integer)); // prints bool(false)**

**$float = 0.0;**
**var_dump(empty($float)); // prints bool(true)**

**$float = 0.1;**
**var_dump(empty($float)); // prints bool(false)**

**$array = array();**
**var_dump(empty($array)); // prints bool(true)**

**$array = array("A");**
**var_dump(empty($array)); // prints bool(false)**
```

所以现在我们知道可以使用 isset()和 empty()函数来检查变量。当我们将它们实现到我们的示例场景中时:

**场景 1:使用未声明的变量**

```
**if(isset($undeclaredVariable)) {**
 **echo $undeclaredVariable;**
**} else {**
 **echo "We found an undeclared variable.";**
**}**
```

**输出:**

```
We found an undeclared variable.
```

**场景 2:使用不需要的变量值**

```
**$emptyUserLastname = "";****if(!empty($emptyUserLastname)) {**
 **echo "Hi Mr. " . $emptyUserLastname;**
**} else {**
 **echo "Hi Mr. Anonymous";**
**}**
```

**输出:**

```
Hi Mr. Anonymous
```

**场景 3:使用虚假的服务调用响应**

```
**function getUserUnsuccessful() {**
 **return array();**
**}****$user = getUserUnsuccessful();****if(!empty($user["LASTNAME"])) {**
 **echo "Welcome Mr. " . $user["LASTNAME"] . "\n";**
**} else {**
 **echo "Hi Mr. Anonymous";**
**}**
```

**输出:**

```
Hi Mr. Anonymous
```

**场景 4:使用未检查的全局变量**

```
**if(isset($_COOKIE["USER_ID"])){**
 **echo $_COOKIE["USER_ID"];**
**} else {**
 **echo "User Id is not available.";**
**}**
```

**输出:**

```
User Id is not available.
```

对于所有更新的代码场景，我们去掉了 PHP 通知消息。对于场景 1 和 4，我们警告我们的用户，对于场景 2 和 3，我们避开不想要的算法行为。

empty()函数的作用是检查给定变量是否被声明、赋值并且没有空值。但是我们需要小心边缘情况，如字符串“0”，“0.0”，“0.00”，整数 0，双精度 0.00，浮点 0.0 等等。因为在这些边缘情况下，我们的算法可能会接受这些值为非空。因此，如果我们考虑这种情况，我们的算法可能会以一种意想不到的方式工作。

# 4.决定正确的功能

决定使用 isset()或 empty()函数的诀窍是:

*   如果关于变量的情况是我们应该考虑它是声明的、赋值的还是非空的，使用 isset()
*   如果情况不止如此，我们还关心变量是否为空，则使用 empty()。但是总是考虑 empty()函数的边缘情况。

不要同时使用 isset()和 empty()函数来检查同一个案例。这将是不必要的代码重复。

只有 isset()和 empty()函数不足以完全应用防御方法。我们需要用额外的 PHP 内置函数添加额外的检查，如下所示:

*   is_null
*   is_bool
*   is _ 数值
*   is_float
*   is_int
*   is_string
*   is _ 数组
*   是 _ 对象

本文并没有涵盖这些额外的 PHP 内置函数，但是您可以通过阅读它们来轻松理解它们的行为。此外，您可以通过在搜索框中键入函数名来检查它们在[https://www.php.net/manual/](https://www.php.net/manual/)中的确切行为。如果将它们与 isset()或 empty()函数一起使用，它们可以为您的算法增加额外的预防措施。

感谢阅读这篇文章。希望对你如何处理 PHP 中的变量有所帮助。在编程中，你应该总是使用防御性的方法。这是一个非常有益的习惯，也有助于你将来成为一名更好的程序员。