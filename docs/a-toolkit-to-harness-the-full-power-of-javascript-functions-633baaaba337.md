# 一个充分利用 JavaScript 函数的工具包

> 原文：<https://itnext.io/a-toolkit-to-harness-the-full-power-of-javascript-functions-633baaaba337?source=collection_archive---------2----------------------->

![](img/ed001befa0af3f5bf028b11b8f792655.png)

函数构成了 JavaScript 语言中不可或缺的角色——它们是执行逻辑的手段。但是 Javascript 中的函数不仅仅是“做事情”的代码块。充分利用它们的潜力，JavaScript 函数与[函数式编程](/the-power-of-functional-programming-in-javascript-cc9797a42b60)的范例完美契合，并为编码人员提供创建可扩展且高效的代码的能力。让我们看看如何！

# JavaScript 函数的正式定义

函数是被定义**一次**的代码块，但是可以被调用或执行**任意次**。一个函数由一系列被称为**函数体**的语句组成。可以将值传递给一个函数，函数将始终返回一个值(要么 *undefined* 如果没有显式设置，否则函数的 ***return*** 语句中规定的值)。

```
**function** helloWorld(){
   **return** 'hello world';
}helloWorld();   *//* returns *'hello world'***function** noReturnValue(){
   const t = 'hello world';
}noReturnValue();  // returns *undefined*
```

JavaScript 函数也是**一级对象**，因为它们可以像任何其他对象一样拥有**属性**和**方法**。它们实际上是 [**功能**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function) 对象。

JavaScript 函数因此可以被认为是可以被调用的 ***对象。***

在这种情况下，可以用有趣的方式操纵和利用 JavaScript 函数。例如，它们可以是

1.  ***赋给变量***
2.  ***存储在对象的属性或数组中的元素***
3.  ***作为参数传递给其他函数***
4.  ***作为其他功能的结果返回***
5.  ***通过*** 功能对象分配属性
6.  ***嵌套在其他功能*** 内

让我们看看这些令人敬畏的功能的一些例子！

# **1。给变量分配函数**

考虑这个函数:

```
**const** square = **function**(x) {
   **return** x * x;
};
```

该定义创建了一个新的函数对象，并将其赋给变量 ***square*** 。

注意，函数**没有被命名为**。当以这种方式定义一个函数时(即把它赋给一个变量)，函数的名字实际上是不重要的，因此是不需要的。可以将该函数赋给另一个变量，并且仍然以同样的方式工作。

以这种方式分配给变量的函数称为 ***函数字面量*** 。函数文字非常灵活，非常适合只使用一次且不需要命名的函数。

```
**const** a = square(4);   *// a contains the number 16*
**const** b = square;   *// Now b refers to the* ***same*** *function that a does*
**const** c = b(5);   *// c contains the number 25*
```

# **2。在对象属性中存储函数**

功能也可以分配给**对象属性**而不是全局属性。在这种情况下，它们被称为**方法**。

> 方法是 Javascript 函数，存储为对象的属性，并通过该对象调用。

```
**const** o = new **Object**();o.square = **function**(x){    *// 'square' is a method of object 'o'*
   **return** x * x;
}**const** y = o.square(16);    *// 256* **const** squareRoot = **function**(x){    *// a function literal*
  **return** Math.sqrt(x);
}o.root = squareRoot;   *// 'root' is now a method of object 'o'*
o.root(4);  *// 2*
```

将函数分配给数组元素:

```
**const** a = new Array(3);a[0] = **function**(x){return x * x;}
a[1] = 20;
a[2] = a[0](a[1]);  // the array element can now be invoked
```

# 旁白:JavaScript 方法和“this”关键字

正如我们刚刚看到的，方法是作为对象的属性存储并通过该对象调用的函数。

任何用作**方法**的**函数**都被有效地传递了一个隐式参数:**调用它的对象**。

一个方法通过其被调用的对象成为方法体中 ***这个*** 关键字的值。

你可以通过 ***这个*** 关键字来引用这个方法的对象。

**对象文字:**

```
**const** calculator = {
   operand1 : 1,
   operand2: 2,
   compute: function(){
       this.result = this.operand1 + this.operand2;
   }
}calculator.compute();
console.log(calculator.result);   // 3
```

**函数构造器:**

```
**function** Person(first, last, age, eyeColor) {
   this.firstName = first;
   this.lastName = last;
   this.age = age;
   this.eyeColor = eyeColor;
   this.name = function() {
      **return** this.firstName + “ “ + this.lastName;
   };
}**const** father = new Person(‘Patrick’, ‘Henry’, 56, ‘blue’);console.log(father.name());  // *'Patrick Henry'*
```

**ES6 类:**

```
**class** Individual{constructor(){
      this.status = 'indifferent';
   }smile(){
      this.status = 'happy';
   }frown(){
      this.status = 'sad';
   }};**const** Jack = new Individual();  *// new object created here*
Jack.smile();  *// invoking the object's method 'smile()'*
console.log(Jack.status);  // *'happy'*
```

# 旁白:将“this”关键字与 apply()、call()和 bind()一起使用

好了，那么 ***这个*** 关键字指的是调用方法的对象。但是你能在一个普通的 JavaScript 函数中利用这个关键字吗？

**是**！JavaScript 还提供了 ***apply()*** *，* ***call()*** 和 ***bind()*** 助手方法，允许我们调用函数**，就像调用对象**的方法一样。

*****call()，apply()*** 和 ***bind()*** 助手方法的第一个**参数是**对象，函数必须在该对象上被调用**；该参数成为函数体中 this 关键字的值。

任何剩余的 ***调用* ()** 的参数都是传递给被调用函数的**值**。

例如，向函数 f()传递两个数字，并作为对象 o 的方法调用它:

```
**const** o = {
   total: 23
}**const** f = function(a, b){
   **return** this.total + a + b;
}f.**call**(o, 1, 2);  *// 26*
```

***apply* ()** 方法类似于 ***call* ()** 方法，只是参数可以作为**数组**传递给函数:

```
**const** o = {
   total: 25
}**const** f = function(a, b){
   **return** this.total + a + b;
}f.**apply**(o, [1, 2]);  *// 28*
```

***bind*()**——作为 ECMAScript 5 的一部分添加——返回一个绑定函数，当稍后执行**时，该函数将具有 ***这个*** 关键字的正确上下文。**

**所以 ***绑定* ()** 可以在函数需要的时候使用**稍后**随时调用。**

```
**const** bound = f.**bind**(o, 1, 2); *// create a bound function with the   context which will be invoked later*console.dir(bound);  *// returns a function*console.log(bound());  *// invokes the bound function*
```

# ****3。将函数作为参数传递给另一个函数****

**因为 JavaScript 函数是一个对象，所以允许将它作为参数传递给另一个函数。事实上，像[**array . prototype . sort()**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)，[**array . prototype . reduce()**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)， [**setTimeout()**](https://www.w3schools.com/jsref/met_win_settimeout.asp) ，[**setInterval()**](https://www.w3schools.com/jsref/met_win_setinterval.asp)这样的 helper 方法都希望在它们的参数中有一个函数。**

```
**function** add(x, y) {
   **return** x + y;
}
**function** subtract(x, y) {
   **return** x - y;
}
**function** divide(x, y) {
   **return** x % y;
}
**function** multiply(x, y) {
   **return** x * y;
}*// this function has the 'operator' function as one of its arguments*
**function** operate(operator, operand1, operand2){
   **return** operator(operand1, operand2);
}**const** i = operate(add, operate(add, 2, 3), operate(multiply, 4, 5));
```

****Array.sort()** 方法:**

```
**const** myList = ['the', 'quick', 'brown', 'fox', 'jumped', 'over', 'the', 'lazy', 'dog'];**const** compareFunction = (a, b) => {
 **return** ( a < b ) ? -1 : ( a > b ) ? 1 : 0;
}*// sort my list alphabetically* myList.sort((a, b) => compareFunction(a, b));
```

# **4.从函数返回函数**

**作为函数的结果返回函数是完全可以接受的。**

```
**function** makeCake() {
   let status = 0;
   **return** {bakeInOven: **function**(){
            status += 1;
            return `status = ${status}`;
      },
      addIcing: **function**(){
            status += 1;
            return `status = ${status}`;
      },
      addCandles: **function**(){
            status += 1;
            return `status = ${status}`;
      }
    }}**const** cake = makeCake();cake.bakeInOven();  // "status = 1"
cake.addIcing();    // "status = 2"
cake.addCandles();  // "status = 3"
```

# **5.定义函数对象的属性**

**如前所述，每个 JavaScript 函数实际上都是一个`**Function**`对象。这可以从返回 true 的代码`(function(){}).constructor === Function`中看出。**

**在这种情况下，当一个函数需要一个值在调用中保持不变的变量时，使用 **Function** 对象的属性会很方便，而不是通过定义一个全局变量来混淆名称空间。**

**例如，假设您想编写一个函数，无论何时调用，它都会返回一个唯一的整数。该函数决不能两次返回相同的值。为了做到这一点，函数需要跟踪它已经返回的值，并且这些信息必须在函数调用中保持不变。**

**您可以将该信息存储在一个全局变量中，但这是不必要的，因为该信息仅由函数本身使用。最好将信息存储在函数对象的**属性中。****

```
*/* Create and initialise the “static” variable. Function declarations are processed before code is executed, so we can do this assignment before the function declaration. */*uniqueInteger.counter = 0;*/* Here’s the function. It returns a different value each time it is called and uses a “static” property of itself to keep track of the last value it returned. */***function** uniqueInteger(){
   *// increment and return our “static” variable* **return** uniqueInteger.counter ++; 
}const i = uniqueInteger(); *// 0*
const j = uniqueInteger(); *// 1*
const k = uniqueInteger(); *// 2*
const l = uniqueInteger(); *// 3*
```

# **6.函数内嵌套函数**

**在 Javascript 中，函数可以嵌套在其他函数中**

```
**function** hypothenuse (a, b) {
   **function** square(x) { return x * x};
 **return** Math.sqrt(square(a) + square(b));
}**const** h = hypothenuse(5,8);
*// 9.433981132056603*
```

**JavaScript 函数本质上是要执行的代码和执行它们的范围的组合。这种代码和范围的组合被称为**闭包**。所有的 JavaScript 函数都是闭包。闭包变得有趣的地方在于**嵌套函数**的情况。**

# **旁白:函数范围和闭包**

> **JavaScript 中的函数是词法范围的，而不是动态范围的。这意味着它们在定义**的**范围内运行，而不是在执行**的**范围内运行。**

**迷茫？好了，让我们后退一步，探索 JavaScript 中的作用域是如何工作的，然后再来看看这个相当神秘的语句。**

**当一个函数被**定义**时，**当前作用域链**被保存，并成为函数的**内部状态的一部分。****

**在顶层，作用域链仅仅由**全局对象**组成。**

**然而，当你定义一个**嵌套函数**时，词法链**也包括包含函数**。这意味着嵌套函数可以访问其包含函数的所有参数和局部变量。**

```
**function** foo(arg) {
   **function** bar() {
      console.log(`arg: ${arg}`);
   }
   bar();
}console.log(foo('hello'));  // arg: hello
```

**arg 的直接作用域是 ***foo()*** ，但是在嵌套作用域 ***bar()*** 中也是可以访问的。关于嵌套， **foo()是外部作用域**而 **bar()是内部作用域**。**

**通过**调用对象，可以以这种方式共享作用域链。****

# ****调用对象解释****

**当 javascript 解释器调用一个函数时，它执行以下操作:**

1.  ****它将作用域设置为定义函数时有效的作用域链。****
2.  **它将一个新对象(称为**调用对象**)添加到作用域链的前面。**
3.  **这个对象用一个名为 ***自变量*** 的属性初始化，该属性引用了函数的 [**自变量**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments) 对象。**
4.  **接下来，函数的**命名参数**被添加到调用对象中。**
5.  **函数中声明的任何**局部变量**也在该对象中定义。**

**由于这个调用对象在作用域链的**头**处，所以局部变量、函数参数和自变量对象**都在函数**的作用域内。**

**考虑在函数 **f** 中定义的函数 **g** 。当调用 **f** 时，作用域链由调用 f 的**调用对象、**后跟**全局对象组成。****

****g** 是在 **f** 内定义的**，因此范围链被保存为 **g** 定义的一部分。****

**当 **g** 然后**被调用**时，作用域链包括三个对象:**

*   ****自己的调用对象****
*   ****f 的调用对象****
*   ****全球对象****

```
**function** f(){ /* **my scope chain when invoked consists of** 
     1) my own call object 
     2) the global object */ **function** g(){ /* **my** **scope chain when invoked consists of** 
             1) my own call object 
             2) the call object of f when I was defined
             3) the global object */ }
}
```

**如果嵌套函数仅在外部函数中使用**，那么对嵌套函数的唯一引用仅在**调用对象**中。然后，当外部函数返回时，嵌套函数引用 call 对象，call 对象引用嵌套函数，但是由于没有对任何一个的其他引用，两个对象都可以用于**垃圾收集**。****

**然而，如果你在**全局作用域中引用了嵌套函数，事情就不同了。**在这种情况下，嵌套函数被**导出到其定义的**范围之外。**

**如何在全局范围内设置对嵌套函数的引用？通过使用嵌套函数作为外部函数的**返回值，或者通过将嵌套函数作为某个其他对象**的属性。**

**在这种情况下，嵌套函数有一个**外部引用**，嵌套函数**保留其对外部函数**的调用对象的引用。**

**在这种情况下，**调用那个**的对象**，外部函数**的一个特定调用**继续存在**，并且函数参数和局部变量的名称和值保存到这个对象中。**

**因此，*闭包*是函数和词法环境的组合，函数在词法环境中被**声明为**。**这个环境由闭包创建时在作用域内的所有局部变量组成** **(它的诞生作用域)**。**

**这个名字来源于这样一个事实，闭包“封闭”了函数的自由变量。如果变量没有在函数中声明，也就是说，如果它来自“外部”，那么它就是自由的**

> **如果一个函数离开了它的作用域，它会通过它的 call 对象保持与最初定义时存在的**作用域链的连接。****

**让我们来看一些实际的闭包例子。**

****关闭示例 1:****

```
**function** createIncrementer(startValue) {
   **return** **function** (step) {
      startValue += step;
      **return** startValue;
   };
}
```

**由 **createIncrementer()** 返回的函数不会失去与 **startValue** 的连接—变量**为函数提供跨函数调用**持续的状态:**

```
const counterA = createIncrementer(5);
const counterB = createIncrementer(10);counterA(2);    // 7
counterB(2);    // 12
counterA(7);    // 14
counterB(7);    // 19
```

**在这个例子中，有一个**对嵌套函数的外部引用**，嵌套函数**保留了对外部函数**的调用对象的引用。在这种情况下，外部函数**的那个**的一个特定调用**的调用对象继续存在**，并且函数参数和局部变量的名称和值保存到该对象中。**

****关闭示例 2:****

```
**function** makeBrowser() {
   const name = 'Mozilla';
   **function** displayName() {
      console.log(name); 
   }
   **return** displayName; 
}const myBrowser = makeBrowser();
myBrowser();  // *'Mozilla'*
```

**在本例中， **makeBrowser** 是对运行 **makeBrowser** 时创建的函数 **displayName** 的实例的引用。 **displayName** 的实例维护了对其词法环境的引用，变量 **name** 存在于该环境中。因此，当调用 **makeBrowser** 时，变量 **name** 仍然可用，并且“Mozilla”被记录到控制台。**

****关闭示例 3:****

```
**function** makeAdder(x) { 
   **return** **function**(y) { 
      **return** x + y; 
   }; 
} 
const add5 = makeAdder(5);
const add10 = makeAdder(10);
console.log(add5(2)); // *7*
console.log(add10(2)); // *12*
```

**在本例中，定义了函数 **makeAdder(x)** ，它接受一个参数 x，并返回一个新函数。它返回的函数只接受一个参数 y，并返回 x 和 y 的和。**

**本质上，makeAdder 是一个函数工厂——它创建可以向参数添加特定值的函数。这里我们使用我们的函数工厂创建两个新函数——一个给它的参数加 5，另一个加 10。**

****add5** 和 **add10** 都是闭包。它们共享相同的函数体定义，但是**存储不同的词法环境**。在 add5 的词法环境中，x 是 5，而在 add10 的词法环境中，x 是 10。**

****关闭示例 4:****

```
const uniqueId = (**function**() { 
   // the call object of this function holds our value let id = 0; // this is the private persistent value return **function**(){
      **return** id++; // return and increment
   }})(); // invoke the outer function after defining it
```

**也许您想编写一个能够在调用中记住某个值的函数，但是该值不能存储在局部变量中，因为 call 对象不会在调用中保持不变。**

**全局变量可以工作，但是会污染全局名称空间。**

**在上面的例子中，我们使用一个闭包来创建一个只有你的函数可以访问的持久私有变量。**

**外层函数返回一个可以访问持久值的嵌套函数(当嵌套函数被定义时，持久值被锁定在其作用域链中)。**

**然后，我们将这个嵌套函数存储在 uniqueId 变量中。**

```
let i = uniqueId();  // 0
let j = uniqueId();  // 1
let k = uniqueId();  // 2
let l = uniqueId();  // 3
```

**闭包是有用的，因为它允许您将一些数据(词法环境)与操作这些数据的函数相关联。**

**这与面向对象编程有明显的相似之处，在面向对象编程中，对象允许我们将一些数据(对象的属性)与一个或多个方法相关联。**

**因此，您可以在任何通常只使用一个方法的地方使用闭包。**

# **参考**

**[](https://books.google.co.za/books/about/JavaScript.html?id=2weL0iAfrEMC&printsec=frontcover&source=kp_read_button&redir_esc=y#v=onepage&q&f=false) [## JavaScript:权威指南

### 这第五版是完全修订和扩展，以涵盖 JavaScript，因为它是在今天的 Web 2.0 使用…

books.google.co.za](https://books.google.co.za/books/about/JavaScript.html?id=2weL0iAfrEMC&printsec=frontcover&source=kp_read_button&redir_esc=y#v=onepage&q&f=false) [](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions) [## 功能

### 一般来说，函数是一个“子程序”,可以被外部代码调用(或者在…

developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions) 

[**http://speakingjs.com/es5/ch16.html**](http://speakingjs.com/es5/ch16.html)

[](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) [## 关闭

### 闭包是函数和声明该函数的词法环境的组合。

developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) [](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function) [## 功能

### 函数构造器创建一个新的函数对象。直接调用构造函数可以创建函数…

developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)**