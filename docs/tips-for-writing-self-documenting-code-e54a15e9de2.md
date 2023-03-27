# 编写自我文档化代码的技巧

> 原文：<https://itnext.io/tips-for-writing-self-documenting-code-e54a15e9de2?source=collection_archive---------0----------------------->

## 如何编写其他开发人员实际上可以阅读的代码

![](img/de053dd5534a410808e0f02b0bfd8286.png)

***自文档化代码*** 只是“可读代码”的一种花哨说法，可读代码是让你在工作中不至于走神的东西。它不能代替真实的文档或恰当的注释，但是写更好的代码不会有坏处。下面说一些习惯，会让你的工作变得简单易懂。

# 不要使用神奇的数字

看看这个，告诉我这是什么意思:

```
if (**students**.length > **23**) {
```

那 23 是什么？我怎么会知道那是什么，我是金·凯利吗？那 23 是一个 [*幻数*](https://codeburst.io/software-anti-patterns-magic-numbers-7bc484f40544) ，这是一个很可怕的名字，因为听起来很好玩，但它是*不是*。这意味着有一个数字看起来很重要，但是没有上下文。总是给数字起一个名字:

```
const **maxClassSize** = 23;
if (**students**.length > **maxClassSize**) {
```

现在上面写着:如果我们的学生超过了允许的最大数量，做点什么。太棒了。这实际上引出了我的下一个观点:

# 使用清晰的变量名

我不知道为什么，但是我曾经对写长变量名非常敏感。这很愚蠢，因为`rStuNms`和`fStuNms`与`rawStudentNames`和`filteredStudentNames`相比简直糟糕透顶。它们看起来太长了，但是我要告诉你:两周不看代码之后，你将不会记得任何一个缩写。变量名非常重要，因为这是你告诉读者你的代码在做什么的机会:

```
const **fStuNms** = **stus**.map(**s** => **s**.n) 
*// vs*
const **filteredStudentNames** = **students**.map(**student** => {
  ***return*** **student**.name;
});
```

这是一个人为的例子，但你得到的想法。另一个有用的技巧是使用命名约定。如果你的值是布尔值，从`is`或`has`开始，比如`isEnrolled: true`。如果你的值存储一个数组，名字应该是复数，例如`students`。如有可能，编号应以`min`或`max`开头。对于函数，前面应该有一个有用的动词，比如`createSchedule`或者`updateNickname`。说到功能:

# 用微小的命名函数进行重构

变量不是添加有用解释的唯一地方，函数名也很棒！我曾经认为函数*仅仅是[干燥你的代码](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)的*，但是最近当我了解到[函数在用于可读性](https://www.amazon.com/Refactoring-Improving-Existing-Addison-Wesley-Signature/dp/0134757599/ref=asc_df_0134757599/?tag=hyprod-20&linkCode=df0&hvadid=312125971120&hvpos=1o1&hvnetw=g&hvrand=12058574021797762441&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9004398&hvtargid=pla-464425925893&psc=1&tag=&ref=&adgrpid=61316180839&hvpone=&hvptwo=&hvadid=312125971120&hvpos=1o1&hvnetw=g&hvrand=12058574021797762441&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9004398&hvtargid=pla-464425925893)的时候真的很棒的时候，我大吃一惊。看一下这段代码，告诉我发生了什么:

```
const handleSubmit = (event) => {
  event.preventDefault();
  NoteAdapter.update(currentNote)
    .then(() => {
      setCurrentAlert('Saved!')
      setIsAlertVisible(true);
      setTimeout(() => setIsAlertVisible(false), 2000);
     })
     .then(() => {
       if (hasTitleChanged) {
         context.setRefreshTitles(true); 
         setHasTitleChanged(false);
       }
     });
   };
```

有可能，但是怎么样:

```
const **showSaveAlertFor** = (**milliseconds**) => () => { 
  **setCurrentAlert**('*Saved!*')
  **setIsAlertVisible**(true);
  **setTimeout**(
    () => **setIsAlertVisible**(false), 
    **milliseconds,** );
};
const **updateTitleIfNew** = () => {
  if (**hasTitleChanged**) {
    **context**.setRefreshTitles(true); 
    **setHasTitleChanged**(false);
  }
};const **handleSubmit** = (**event**) => {
  **event**.preventDefault();
  **NoteAdapter**.update(**currentNote**)
    .then(**showSaveAlertFor**(2000))
    .then(**updateTitleIfNew**);
  };
```

嘣，干净多了。我们所做的只是将几行代码放入函数中，它得到了显著的改进。甚至是一个我们可以在其他地方使用的功能。代码行被快速移动到函数定义中，但是除非有问题，否则不需要直接阅读它们。在高层次上，我们有一个人类可读的事件链。函数定义也是标记变量的另一个好方法。`setTimeout`需要一个函数和毫秒数，但是现在我们已经清楚地自己添加了它，以便*额外的*有用。

# 添加有用的测试描述

可能最少谈论的将文档偷偷放入代码的方法是测试。假设我们有这个函数:

```
const **getDailySchedule** = (**student**, **dayOfWeek**) => { 
```

让我们假设这是一个由一堆其他函数组成的 runner 函数，所以它做了很多事情:它检索每日日程；如果星期几是周末，它返回一个空数组；如果学生被留堂，它会把它贴在课程表的末尾；如果学生没有在学校注册，它会打印一个链接到那个[刻薄女孩 Gif](https://giphy.com/explore/she-doesnt-even-go-here) 。

如果你试图把那一段作为评论，你会得到一些奇怪的表情。但是你知道那一段放在哪里会很好看吗？在测试中:

```
describe('getDailySchedule tests', () => {
  it("*retrieves the student's full schedule"*, () => { 
  it('*returns an empty array if given a weekend day'*, () => {  
  it('*adds detention if a student got one that day*', () => {
  it('*prints a gif link if student not enrolled yet*', () => {
```

这无疑是直接在代码中添加注释而不添加任何注释的最简单的方法。

# 底线:可读性高于灵活性

成为一名优秀的开发者并不意味着写出最聪明的代码，而是成为一名优秀的**队友**。高质量的软件很少是单独开发的，最终*其他人将需要阅读你的代码*。即使你是单独工作，当提到记忆代码时，现在的你和两周后的你几乎是不同的人。有很多方法可以写出可读的代码，比如[添加好的注释](https://medium.com/@mikecronin92/what-makes-a-good-code-comment-5267debd2c24)，但是你能做的最重要的事情就是开始思考它。

大家编码快乐，

麦克风