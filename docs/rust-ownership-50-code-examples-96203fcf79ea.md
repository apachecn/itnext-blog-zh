# Rust 所有权:50 个代码示例

> 原文：<https://itnext.io/rust-ownership-50-code-examples-96203fcf79ea?source=collection_archive---------1----------------------->

![](img/13edb20e7be030bd18951dea2877892b.png)

图片来自[https://www.rust-lang.org/logos/error.png](https://www.rust-lang.org/logos/error.png)

## 在铁锈操场上试试吧

*变更日志:
2023 年 1 月 2 日—使用 Medium 的新代码块进行语法突出显示*

遇到所有权问题，比如借用一个已经被转移的价值？如果你是，那么我有 50 个代码片段(好吧，有 53 个)让我们在 Rust Playground 一起练习。

本文假设您大致了解基本类型，如`String`，整数类型，如`i32`、`Vec`，迭代器、片和结构。我假设你已经读过 Rust 编程语言。第四章:理解所有权)和[锈例](https://doc.rust-lang.org/rust-by-example/)书籍。

这些例子从简单的类型(从`String`开始)开始，到稍微复杂一点的类型。✅的意思是代码可以编译，❌的意思是不能。虽然一些例子为无法编译的代码提供了替代方案，但是请注意，这些选项并不详尽，只是针对初学者的。这些例子不包括可变性(因为我觉得它们更容易掌握)，也不涉及所有权异步编程。

# 内容

1.  `[String](#f4fc)`
2.  `[i32](#ed8d)`
3.  [Struct 带](#e17b) `[String](#e17b)` [字段](#e17b)
4.  [结构带](#6b61) `[u8](#6b61)` [字段](#6b61)
5.  [具有两个](#39d6) `[String](#39d6)` [字段的结构](#39d6)
6.  `[Vec<String>](#e657)`
7.  `[Iterator](#3c86)`
8.  `[&str](#e5f2)`

# 1.线

一个`[String](https://doc.rust-lang.org/std/string/struct.String.html)`是一个没有实现`Copy`特征的类型。

## ✅ **清单 1–1**

让我们创建一个`String`，然后用它调用`do_something`。

一切看起来正常…

```
fn main() {
    let name = String::from("Rust");
    do_something(name);
}

fn do_something(name: String) {
    println!("Hello, {}!", name);
}
```

```
Hello, Rust!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=faac85d612a7017d6b1193aff6df0dcb)

## ❌ **清单 1–2**

让我们在第 4 行添加一个`println!`语句…

```
fn main() {
    let name = String::from("Rust");
    do_something(name);
    println!("{}", name);
}

fn do_something(name: String) {
    println!("Hello, {}!", name);
}
```

```
error[E0382]: borrow of moved value: `name`
 --> src/main.rs:4:20
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=d2ce6f6d4f6f83e941636914c0f66f03)

这是违法的！这是因为我们试图使用(借用，在`println!`的情况下)已经移动到第 3 行`do_something`中的值`name`。

不能使用已移动的值。在`name`到达`println!`之前，它被移入`do_something`并在函数结束时*移出*范围。

我们能做什么？

## ✅ **清单 1–3**

我们可以做的是将 name 克隆为`name_clone`，这样我们将 name 用于第 5 行，将`name_clone`用于第 6 行。注意克隆是有成本的。

```
fn main() {
    let name = String::from("Rust");
    let name_clone = name.clone();

    do_something(name);
    println!("{}", name_clone);
}

fn do_something(name: String) {
    println!("Name: {}!", name);
}
```

```
Name: Rust!
Rust
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=1b034f361507f04bdf6cbeedc7e1caca)

## ✅ **清单 1–4**

或者我们可以通过第 3 行中的引用将`name`传递给`do_something`，并且仍然能够将其用于第 4 行中的`println!`。

```
fn main() {
    let name = String::from("Rust");
    do_something(&name);
    println!("{}", name);
}

fn do_something(name: &str) {
    println!("Hello, {}!", name);
}
```

```
Hello, Rust!
Rust
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=6e98c5b6a5164b1d2a046331e5794ebb)

## ✅清单 1–5

或者，如果你的程序允许的话，我们可以简单地改变一下代码。

这里，我们先借用`name`，然后再把它移入`do_something`。这个程序编译是因为在`do_something`行之后，在`main`范围内没有其他人会使用`name`。

```
fn main() {
    let name = String::from("Rust");
    println!("{}", name);
    do_something(name);
}

fn do_something(name: String) {
    println!("Hello, {}!", name);
}
```

```
Rust
Hello, Rust!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=1aadec2aaa62a7de28521b79a6e5c779)

# 2.i32

对于拥有`Copy`特质的人来说，生活要容易得多，比如`[i32](https://doc.rust-lang.org/std/primitive.i32.html)`。

## ✅ **清单 2–1**

还是那句话，先从简单的开始。我们创建`age`并用它来调用`do_something`。这里复制了`age`的值，因为`i32`类型实现了`Copy`特征。

```
fn main() {
    let age: i32 = 25;
    do_something(age);
}

fn do_something(age: i32) {
    println!("Hello, {}!", age);
}
```

```
Hello, 25!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=feb04c5816c7ddecb806881e39c0e18c)

## ✅ **清单 2–2**

第 4 行再次复制了`age`的值。

```
fn main() {
    let age: i32 = 25;
    do_something(age);
 println!("{}", age);
}

fn do_something(age: i32) {
    println!("Hello, {}!", age);
}
```

```
Hello, 25!
25
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=3cd8c1be29e0692e1a596178d9174379)

## ✅ **清单 2–3**

我们也可以通过引用来传递值，虽然*我觉得*不太习惯。

```
fn main() {
    let age: i32 = 25;
    do_something(&age);
    println!("{}", age);
}

fn do_something(age: &i32) {
    println!("Hello, {}!", age);
}
```

```
Hello, 25!
25
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=26eb7144eee0887ac8f9d04cde0cb3e9)

## ✅ ⚠️ **清单 2–4**

在第 3 行中，我们首先克隆了`age`，然后将其传递给`do_something`。程序编译，但是注意复制会进行两次——一次是在`.clone()`中，另一次是在`age`进入`do_something`范围时复制。

请注意，应该避免这种习语。参见**上的 Clippy lint clone _ on _ copy**这里的。

```
fn main() {
    let age: i32 = 25;
    do_something(age.clone());
    println!("{}", age);
}

fn do_something(age: i32) {
    println!("Hello, {}!", age);
}
```

```
Hello, 25!
25
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=e550a2a06f881d03d71e3178d800df28)

# 3.具有`String`字段的结构

如果一个类型的所有组件都实现了`Copy`，那么这个类型就可以实现`Copy`(参见[我的类型什么时候可以成为](https://doc.rust-lang.org/std/marker/trait.Copy.html#when-can-my-type-be-copy) `[Copy](https://doc.rust-lang.org/std/marker/trait.Copy.html#when-can-my-type-be-copy)` [？](https://doc.rust-lang.org/std/marker/trait.Copy.html#when-can-my-type-be-copy))。

在这些清单中，我们关注的是由一个`String`字段组成的`Movie`结构，其中*没有*实现`Copy`。因此，`Movie` *无法*实现`Copy`。

## ✅清单 3–1

目前看来还不错，不是吗？程序编译(忽略未使用的字段警告)，我们都很高兴。

```
#[derive(Debug)]
struct Movie {
    title: String,
}

fn main() {
    let movie = Movie { 
        title: String::from("Rust")
    };
    do_something(movie);
}

fn do_something(movie: Movie) {
    println!("Movie: {:?}!", movie);
}
```

```
Movie: Movie { title: "Rust" }!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=beb3e965a56641ad97632f7919e6473e)

## ❌ 清单 3–2

但是在添加了`println!`之后，编译器抱怨我们试图借用一个已经移入`do_something`的值`movie`。

在前面的清单中，我们已经看到过这种针对`String`的移动语义。

```
#[derive(Debug)]
struct Movie {
    title: String,
}

fn main() {
    let movie = Movie { 
        title: String::from("Rust")
    };
    do_something(movie);
    println!("Movie: {:?}", movie);
}

fn do_something(movie: Movie) {
    println!("Movie: {:?}!", movie);
}
```

```
error[E0382]: borrow of moved value: `movie`
  --> src/main.rs:10:29
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=57898774f167fad6dad057040e6c6bb4)

我们该怎么办？

## **✅清单 3–3**

我们可以借用`movie`而不是移动它。

```
#[derive(Debug)]
struct Movie {
    title: String,
}

fn main() {
    let movie = Movie { 
        title: String::from("Rust")
    };
    do_something(&movie);
    println!("Movie: {:?}", movie);
}

fn do_something(movie: &Movie) {
    println!("Movie: {:?}!", movie);
}
```

```
Movie: Movie { title: "Rust" }!
Movie: Movie { title: "Rust" }
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=e6e55488d4ce718c6ff7e20bcf8e7393)

## **✅清单 3–4**

或者我们可以为`do_something`克隆`movie`。这就需要`Movie`来实现`Clone`。

```
#[derive(Debug, Clone)]
struct Movie {
    title: String,
}

fn main() {
    let movie = Movie { 
        title: String::from("Rust")
    };
    do_something(movie.clone());
    println!("Movie: {:?}", movie);
}

fn do_something(movie: Movie) {
    println!("Movie: {:?}!", movie);
}
```

```
Movie: Movie { title: "Rust" }!
Movie: Movie { title: "Rust" }
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=8f4db43211110c0c1903f815c2584e97)

## **✅清单 3–5**

或者，如果你的程序允许的话，我们可以在不克隆的情况下稍微改变一下代码。在这里，将`println!`借用为`movie`，然后将`movie`移入`do_something`。

```
#[derive(Debug)]
struct Movie {
    title: String,
}

fn main() {
    let movie = Movie { 
        title: String::from("Rust")
    };
    println!("Movie: {:?}", movie);
    do_something(movie);
}

fn do_something(movie: Movie) {
    println!("Movie: {:?}!", movie);
}
```

```
Movie: Movie { title: "Rust" }
Movie: Movie { title: "Rust" }!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=52e772eafc2d0a33bb7a4e3cfd95ba7f)

# 4.带有 u8 字段的结构

如前所述，如果一个类型的所有组件都实现了`Copy`，那么这个类型就可以实现`Copy`。

在这些清单中，我们关注的是由一个实现`Copy`特征的`u8`字段组成的`Book`结构。这样一来，`Movie` *就可以*实现`Copy`。

## ✅ **清单 4–1**

在这个清单中，一切看起来都很正常，对吗？

```
#[derive(Debug)]
struct Book {
    id: u8,
}

fn main() {
    let book = Book { id: 1 };
    do_something(book);
}

fn do_something(book: Book) {
    println!("Book: {:?}!", book);
}
```

```
Book: Book { id: 1 }!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=8f03cf76c09fd56dbe37299b03eac55d)

## ❌ **清单 4–2**

直到我们在`do_something`下面加一个`println!`语句。发生了什么事？

因为`book`没有实现`Copy`特征(或者还没有实现)，Rust *将* `book`移入`do_something`。但是…因为`book`被移动，`println!`不能再使用该值。

```
#[derive(Debug)]
struct Book {
    id: u8,
}

fn main() {
    let book = Book { id: 1 };
    do_something(book);
    println!("Book: {:?}", book);
}

fn do_something(book: Book) {
    println!("Book: {:?}!", book);
}
```

```
error[E0382]: borrow of moved value: `book`
 --> src/main.rs:8:28
```

我们能做什么？

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=0d7204cf6da7b709ea3cdc6f573f9c5c)

## ❌ **清单 4–3**

一种方法是通过派生`Copy`来实现 Book 的`Copy`特征。

```
#[derive(Debug, Copy)]
struct Book {
    id: u8,
}

fn main() {
    let book = Book { id: 1 };
    do_something(book);
    println!("Book: {:?}", book);
}

fn do_something(book: Book) {
    println!("Book: {:?}!", book);
}
```

```
error[E0277]: the trait bound `Book: Clone` is not satisfied
 --> src/main.rs:1:17
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=ad90ab72c41ee22806c3a3ab67e3b375)

但是……我们仍然得到一个错误——特征边界`Book: Clone`不满足。

## ✅ **清单 4–4**

如果我们看一下[文档](https://doc.rust-lang.org/std/marker/trait.Copy.html#whats-the-difference-between-copy-and-clone)，它说所有为`Copy`的东西也必须实现`Clone`，因为`Clone`是一个父类。

因此，让我们在现有的`Copy`特征之上为`Book`实现`Clone`特征。

```
#[derive(Debug, Copy, Clone)]
struct Book {
    id: u8,
}

fn main() {
    let book = Book { id: 1 };
    do_something(book);
    println!("Book: {:?}", book);
}

fn do_something(book: Book) {
    println!("Book: {:?}!", book);
}
```

```
Book: Book { id: 1 }!
Book: Book { id: 1 }
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=5ad07c8da8842c55178df24c3ab7e1c7)

## ✅ **清单 4–5**

太棒了。我们也可以重新设计我们的程序，让`do_something`借用`book`来代替。

```
#[derive(Debug)]
struct Book {
    id: u8,
}

fn main() {
    let book = Book { id: 1 };
    do_something(&book);
    println!("Book: {:?}", book);
}

fn do_something(book: &Book) {
    println!("Book: {:?}!", book);
}
```

```
Book: Book { id: 1 }!
Book: Book { id: 1 }
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=a22b2118f4fb1529e9f942f913d8d378)

# **5。具有两个字符串字段的结构**

值可以从结构中移出吗？🤔

在本节中，我们将讨论*部分移动*。

## **✅清单 5–1**

目前看来一切正常…

```
#[derive(Debug)]
struct Person {
    name: String,
    alias: String,
}

fn main() {
    let person = Person { 
        name: "John".to_string(),
        alias: "Johan".to_string(),
    };
    print_alias(person.alias);
}

fn print_alias(alias: String) {
    println!("Person: {:?}!", alias);
}
```

```
Person: "Johan"!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=13b421d6a489dc624c36a89991007f11)

## **❌清单 5–2**

然而，在这个清单中，我们在`print_alias`之后添加了一个 print 语句。

程序没有编译，因为`person.alias`已经将*部分移动到`print_alias`中，但是我们尝试在`println!`行中整体借用`person`。*

```
#[derive(Debug)]
struct Person {
    name: String,
    alias: String,
}

fn main() {
    let person = Person { 
        name: "John".to_string(),
        alias: "Johan".to_string(),
    };
    print_alias(person.alias);
    println!("{:?}", person);
}

fn print_alias(alias: String) {
    println!("Person: {:?}!", alias);
}
```

```
error[E0382]: borrow of partially moved value: `person`
  --> src/main.rs:12:22
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=0611ce83e9af1663cd1027aa4f73ec38)

我们能做什么？有哪些合法的举动？

## **✅清单 5–3**

原来我们*可以*部分移动字段——只要确保我们以后不再尝试使用这个结构本身。

这里，我们将这两个字段移动到两个不同的函数中。

```
#[derive(Debug)]
struct Person {
    name: String,
    alias: String,
}

fn main() {
    let person = Person { 
        name: "John".to_string(),
        alias: "Johan".to_string(),
    };
    print_alias(person.alias);
    print_name(person.name);
}

fn print_alias(alias: String) {
    println!("Alias: {:?}!", alias);
}

fn print_name(name: String) {
    println!("Name: {:?}!", name);
}
```

```
Alias: "Johan"!
Name: "John"!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=fd51d900044e94a589332c6c8485dace)

## **✅清单 5–4**

我们可以通过引用来传递这些字段。

```
#[derive(Debug)]
struct Person {
    name: String,
    alias: String,
}

fn main() {
    let person = Person { 
        name: "John".to_string(),
        alias: "Johan".to_string(),
    };
    print_alias(&person.alias);
    print_name(&person.name);
}

fn print_alias(alias: &str) {
    println!("Alias: {:?}!", alias);
}

fn print_name(name: &str) {
    println!("Name: {:?}!", name);
}
```

```
Alias: "Johan"!
Name: "John"!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=d7c5c74fe3cea73da8ef411bafe5a8ed)

## **✅清单 5–5**

或者如果我们以后想再次使用`person`，我们可以克隆这些值。

```
#[derive(Debug)]
struct Person {
    name: String,
    alias: String,
}

fn main() {
    let person = Person { 
        name: "John".to_string(),
        alias: "Johan".to_string(),
    };
    print_alias(person.alias.clone());
    print_name(person.name.clone());
    println!("{:?}", person);
}

fn print_alias(alias: String) {
    println!("Alias: {:?}!", alias);
}

fn print_name(name: String) {
    println!("Name: {:?}!", name);
}
```

```
Alias: "Johan"!
Name: "John"!
Person { name: "John", alias: "Johan" }
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=43ff48bc0215e4a341a08a02aab68b78)

## **✅清单 5–6**

或者你可以移动东西而不需要克隆。这里我们先借用了`person`，然后部分移动了字段。

```
#[derive(Debug)]
struct Person {
    name: String,
    alias: String,
}

fn main() {
    let person = Person { 
        name: "John".to_string(),
        alias: "Johan".to_string(),
    };
    println!("{:?}", person);
    print_alias(person.alias);
    print_name(person.name);
}

fn print_alias(alias: String) {
    println!("Alias: {:?}!", alias);
}

fn print_name(name: String) {
    println!("Name: {:?}!", name);
}
```

```
Person { name: "John", alias: "Johan" }
Alias: "Johan"!
Name: "John"!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=f4dcc6f3684bad24294ef8d4196b1aa6)

# 6.Vec

像`String`，`[Vec](https://doc.rust-lang.org/std/vec/struct.Vec.html#)`一样被移动，因为它们没有实现`Copy`特性(见[此处](https://doc.rust-lang.org/std/vec/struct.Vec.html#trait-implementations))。同样的移动语义也适用。

向量(以及其他相关的集合)值得一谈，因为涉及到太多的语义——容器本身、元素和迭代器。

我们在这里使用`Vec`作为例子，因为它是集合中非常常见的数据结构。其他没有实现`Copy`特征的集合包括`HashMap`和`HashSet`。另一方面，数组的移动语义与结构的工作方式相似，因为它们依赖于项的类型——但那是另一天的事情了。

## **✅清单 6–1**

像往常一样，我们从开心的事情开始:

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    do_something(names);
}

fn do_something(names: Vec<String>) {
    println!("{:?}", names);
}
```

```
["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=307eea01cd9ed0e81f72547968c94700)

## **❌清单 6–2**

直到我们在`do_something`后面加上一个`println!`语句。

对于`String`类型，我们之前已经看到过这种编译器错误，在这种情况下，我们试图借用/使用一个已经被移动的值。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
   ];
    do_something(names);
    println!("{:?}", names);
}

fn do_something(names: Vec<String>) {
    println!("{:?}", names);
}
```

```
error[E0382]: borrow of moved value: `names`
 --> src/main.rs:7:22
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=949805ae8472f9fb4fe3edb57d7f9067)

我们能做什么？

## **✅清单 6–3**

我们可以先借用`names`，这样之后就可以把它移入`do_something`了。

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    println!("{:?}", names);
    do_something(names);
}

fn do_something(names: Vec<String>) {
    println!("{:?}", names);
}
```

```
["John", "Jane"]
["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=3fd48e7cd13a70efaa4faab673a7fd5a)

## **✅清单 6–4**

我们可以重新设计`do_something`来借用`names`，这样我们就可以在`println!`再次借用。

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    do_something(&names);
    println!("{:?}", names);
}

fn do_something(names: &[String]) {
    println!("{:?}", names);
}
```

```
["John", "Jane"]
["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=faaf4eb528f522351b79c5121c811217)

## **✅清单 6–5**

最后，我们可以克隆载体用于`do_something`。注意，如果一个向量的底层类型实现了`Clone`特征，那么这个向量就可以被克隆。

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
  ];
    do_something(names.clone());
    println!("{:?}", names);
}

fn do_something(names: Vec<String>) {
    println!("{:?}", names);
}
```

```
["John", "Jane"]
["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=eb0be758f7629102ba954cbb79aab1fc)

通过获取索引来读取向量中的元素是很复杂的。这和我们所知的编程语言不一样。

## **❌清单 6–6**

我是说你能看看这个吗！下面的代码在我所知道的编程语言中非常好，但是在这里，我们得到一个编译器错误，说我不能移出索引`Vec<String>`。

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    let name: String = names[0];
    println!("Hello, {}", name);
}
```

```
error[E0507]: cannot move out of index of `Vec<String>`
 --> src/main.rs:6:24
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=b0b6114f17c06ed0a42fcd6e1325e824)

但是为什么呢？

看，如果你只从一个`Vec`中移出一个元素，你会让向量处于一个**无效状态**——向量不再是同质元素的集合

不允许隐式移出`Vec`，因为这会**使向量处于无效状态**——一个元素被移出，其他元素不被移出(参见[这个](https://stackoverflow.com/questions/27904864/what-does-cannot-move-out-of-index-of-mean) StackOverflow post)。如果我要迭代这个向量，我可能会访问一个无效的内存(被移出的元素)😱。谢谢你保护我们，拉斯特。

那我们该怎么办？

## **✅清单 6–7**

我们可以通过使用索引操作符来借用我们想要的值。

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    let name: &str = &names[0];
    println!("Hello, {}", name);
}
```

```
Hello, John
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=3a2b3b74fa9fac331e805a86158e9c2b)

## **✅清单 6–8**

我们也可以使用`.get`方法来借用这个值。

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    let name: &str = 
    names.get(0).unwrap();
    println!("Hello, {}", name);
}
```

```
Hello, John
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=1dd433f1795e97c7131b165ef195f70f)

## **✅清单 6–9**

我们可以使用`.first()`或`.last()`方法来借用这个值(当然，如果你想要第一个和最后一个项目的话)。

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    let name: &str = 
    names.first().unwrap();
    println!("Hello, {}", name);
}
```

```
Hello, John
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=8dd83ef7fc141dcde84c39e881e90761)

但是……如果我想*拥有*一个元素呢？

## **✅清单 6–10**

我们使用`.into_iter()`方法来拥有单个元素。下一章将详细介绍迭代器。在这里，我们设法拥有调用`.next()`后的第一个元素:

```
fn main() {
    let names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    let mut names_iter = 
    names.into_iter();
    let name: String = 
    names_iter.next().unwrap();
    println!("Hello, {}", name);
}
```

```
Hello, John
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=166c5d7162d09dbf206b1161f4c9a8bd)

如果您想拥有第一个元素，这个例子就可以了。Afaik，如果你想拥有索引为 *n* 的元素(出于某种原因)，你应该调用`.skip(n)`然后调用`.next()`。

## **✅清单 6–11**

对于`Vec`，我们可以`.pop()`最后一个元素并拥有数据(当然，只有当你想要一个向量的最后一个元素时)。

```
fn main() {
    let mut names = vec![
       String::from("John"),
       String::from("Jane"),
   ];
    let name: String = names.pop()
    .unwrap();
    println!("Hello, {}", name);
}
```

```
Hello, Jane
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=b13290b5c7dd64754afe127e687dfb23)

# 7.迭代程序

当涉及到集合中元素的所有权时，迭代器扮演着极其重要的角色。

在这些例子中，我们将使用`Vec<String>`，有意地使用`String`作为元素(它没有实现`Copy`特征),这样我们可以在一个向量中演示它的移动语义。

## **for 循环**

## **✅清单 7–1**

让我们从遍历`names`的 for 循环开始。为什么是 for-loop？我们会谈到这一点。

但现在，生活是一床玫瑰:

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];
    for name in names {
        println!("{}", name);
    }
}
```

```
John
Jane
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=a3904f4e4f35218c5deb07f2a3f83405)

## **❌清单 7–2**

直到我们在 for 循环下面添加一个 print 语句…编译器开始抱怨。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    for name in names {
        println!("{}", name);
    }

    println!("Names: {:?}", names);
}
```

```
error[E0382]: borrow of moved value: `names`
 --> src/main.rs:11:29
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=61bffd8a9200a2f36531a59a1c797600)

显然，在循环之前有一个“隐式的`.into_iter`调用”

等等，*隐*？为什么？！但是好吧，让我们添加那个隐含的`.into_iter`:

## **❌清单 7–3**

添加`into_iter()`后，我们预计程序仍然无法编译。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    for name in names.into_iter() {
        println!("{}", name);
    }

    println!("Names: {:?}", names);
}
```

```
error[E0382]: borrow of moved value: `names`
 --> src/main.rs:11:29
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=81dd26cd3f2d47a9a163e39fa953d2dc)

## **❌清单 7–4**

Hmmmmm 我们给`names.into_iter()`赋个变量吧。所以我们可以想象这个动作。注意，我们仍然认为程序不会编译。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    let names_iter = names.into_iter();
    for name in names_iter {
        println!("{}", name);
    }
    println!("Names: {:?}", names);
}
```

```
error[E0382]: borrow of moved value: `names`
 --> src/main.rs:11:29
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=df4b0449605a42f4e902e4fa79ddd8b3)

正在发生的是，使用`into_iter`，我们正在*移动*元素从`names`到`names_iter`。因此，我们不能再使用`names`了！

我们该怎么办？

## **✅清单 7–5**

我们可以使用`.iter()`从`names`借用元素:

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    for name in names.iter() {
        println!("{}", name);
    }

    println!("Names: {:?}", names);
}
```

```
John
Jane
Names: ["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=915863eb52ba9dddb5c498cbb1ab8a25)

## **✅清单 7–6**

我们可以使用一个切片(它实现了`IntoIterator`)。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    for name in &names {
        println!("{}", name);
    }

    println!("Names: {:?}", names);
}
```

```
John
Jane
Names: ["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=0d332a54632258fbad2cca0dd2039240)

## **✅清单 7–7**

如果你的程序允许的话，我们可以稍微改变一下代码，先借用`names`，然后移动它:

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    println!("Names: {:?}", names);

    for name in names {
        println!("{}", name);
    }
}
```

```
Names: ["John", "Jane"]
John
Jane
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=36f20472e004e42b74c7db6463506d84)

## **✅清单 7–8**

我们也可以克隆载体。注意克隆是有成本的。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    for name in names.clone() {
        println!("{}", name);
    }

    println!("Names: {:?}", names);
}
```

```
John
Jane
Names: ["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=e748cbe0f4c52f20d8bf5839369179c4)

## **功能编程**

使用函数式编程习惯用法使得当您想要迭代元素时，迭代器的参与变得更加明显(特别是如果您来自不必直接处理迭代器的编程语言)。

与 for 循环不同，这里没有隐式的`.into_iter()`调用。同样，对于向量，我们总是需要调用`.iter()`、`.into_iter()`等。得到一个迭代器。

## **❌清单 7–9**

这里它没有编译，因为我们需要迭代元素，这意味着我们需要一个迭代器，比如说，`.iter()`。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    names.for_each(|name|
       println!("{}", name));

    println!("{:?}", names);
}
```

```
error[E0599]: `Vec<String>` is not an iterator
 --> src/main.rs:7:11
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=578112b4e95e4655baf89f35e87fe197)

## **✅清单 7–10**

这里，我们使用了`.iter()`和`.for_each()`的组合。

迭代器返回对元素的引用。之后我们仍然可以使用`names`对象:

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    names
        .iter()
        .for_each(|name|
           println!("{}", name));

    println!("{:?}", names);
}
```

```
John
Jane
["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=9aef2992265c97429f9881595ad86f41)

## **❌清单 7–11**

如果你想移动元素，使用`.into_iter()`。但是，请注意，我们不能在之后使用向量:

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    names
        .into_iter()
        .for_each(|name|
           println!("{}", name));

    println!("{:?}", names); 
}
```

```
error[E0382]: borrow of moved value: `names`
 --> src/main.rs:12:22
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=35ec96f6bb1cd1b658064098aeed5fb3)

## **✅清单 7–12**

你需要重新设计你的程序，这样你就不会使用被移动的元素。这里，我们将打印声明移到了前面:

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    println!("{:?}", names); 

    names
        .into_iter()
        .for_each(|name|
           println!("{}", name));
}
```

```
["John", "Jane"]
John
Jane
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=ffc3752256b543ad20c32f097d21aed0)

## **✅清单 7–13**

我们也可以在调用`.into_iter()`之前克隆这个向量。

```
fn main() {
    let names = vec![
        String::from("John"),
        String::from("Jane"),
    ];

    names.clone()
        .into_iter()
        .for_each(|name|
           println!("{}", name));

    println!("{:?}", names); 
}
```

```
John
Jane
["John", "Jane"]
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=f5ebced734aec3011fd60aa6d0a960da)

# 8.&str

共享引用(`&T`)也是`Copy`(此处见)。下面是一个常用类型的例子，字符串切片`&str`。

## ✅ **清单 8–1**:

我们可以在第 3 行和第 4 行传递`name`到`do_something`。请注意，我们正在复制*引用。*

```
fn main() {
    let name: &'static str = "Rust";
    do_something(name);
    do_something(name);
}

fn do_something(name: &str) {
    println!("Hello, {:?}!", name);
}
```

```
Hello, Rust!
Hello, Rust!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=d887f4a40dd3a6ccea5f7ae06610a1e7)

## ✅ ⚠️ **清单 8–2**:

类似于清单 2–4，在将`name`传递给`do_something`之前调用`.clone()`是多余的。

```
fn main() {
    let name: &'static str = "Rust";
    do_something(name.clone());
    do_something(name.clone());
}

fn do_something(name: &str) {
    println!("Hello, {:?}!", name);
}
```

```
Hello, Rust!
Hello, Rust!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=2fbea3e861cd47929ca233542d881bc3)

## ✅ ⚠️ **清单 8–3**:

在 Clippy 的建议下，`<&str>::clone(&name)`习语克隆了这个参考。程序可以编译，但我不确定这是否是惯用的。

```
fn main() {
    let name: &'static str = "Rust";
    do_something(<&str>::clone(&name));
    do_something(<&str>::clone(&name));
}

fn do_something(name: &str) {
    println!("Hello, {:?}!", name);
}
```

```
Hello, Rust!
Hello, Rust!
```

[游乐场](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=6b073e2f6039ef91fc640b0bb43168e7)

就这些了，伙计们！

我发表关于人工智能、机器学习、编程语言、Web 框架、生产力和学习的文章。

*如果你喜欢阅读更多关于 web 框架的内容，你可以通过我的推荐链接* [*订阅*](https://remykarem.medium.com/subscribe) *随时接收更新或者* [*注册*](https://remykarem.medium.com/membership) *！请注意，您的会员费的一部分将作为介绍费分摊给我。*