# 咖啡师——愉快的 Espresso Android UI 测试

> 原文：<https://itnext.io/barista-enjoyable-espresso-android-ui-tests-59d1620bd99c?source=collection_archive---------2----------------------->

![](img/3a8a1294f15cde0d3e540154efebd140.png)

咖啡师——提供美味浓缩咖啡的人

咖啡师使开发 UI 测试更快、更容易、更可预测。它建立在 Espresso 之上，提供了一个简单明了的 API，消除了常见 Espresso 任务的大部分样板文件和冗长。你和你的 Android 团队将毫不费力地编写测试。

Espresso 对于 UI 测试来说是合理的，但是它有一些问题。代码冗长，API 几乎不提供开箱即用的功能。大部分都需要定制。

# 咖啡师如何把事情简单化？

首先，让我们看一个 Expresso 示例。

```
onView(withText("Button")).check(matches(isDisplayed()))
onView(withId(R.id.button)).perform(click())
```

这些例子非常**冗长**。做一些简单的事情需要很多代码。参数中有多个函数和参数。我们来观察一下咖啡师的替代方案。

```
clickOn(R.id.button);
assertDisplayed("Button");
```

代码直截了当，对于没有太多上下文的人来说，甚至对于非程序员来说，在更大的范围内更具可读性。

这些函数也可以与不同的参数一起使用，可以是字符串、视图 id 或字符串 id。

```
clickOn(R.id.button);
clickOn(R.string.button_text);
clickOn("Next");assertDisplayed("Hello world");
assertDisplayed(R.string.hello_world);
assertDisplayed(R.id.button);
assertDisplayed(R.id.button, "Hello world")
assertDisplayed(R.id.button, R.string.hello_world)
```

# 复杂示例

让我们来看一个完整的 Espresso 仪器测试。

现在咖啡师的选择是:

代码变得如此直观，以至于注释变得过时。**代码成为文档。**

# 列出测试

咖啡师很容易与列表互动。在 Espresso 上，这需要大量的代码，ListView 和 RecyclerView 是有区别的，在 Barista 上，这两种方法是可以互换的。

```
clickListItem(R.id.list, 4);
clickListItemChild(R.id.list, 3, R.id.row_button);
scrollListToPosition(R.id.list, 4);
assertListItemCount(R.id.list, 5)
assertListNotEmpty(R.id.list)
assertDisplayedAtPosition(R.id.list, 0, "text");
assertDisplayedAtPosition(R.id.list, 0, R.id.text_field, "text");
assertDisplayedAtPosition(R.id.list, 0, R.string.hello_world);
assertDisplayedAtPosition(R.id.list, 0, R.id.text_field, R.string.hello_world);
assertCustomAssertionAtPosition(R.id.list, 0, customViewAssertion);
clickSpinnerItem(R.id.spinner, 1);
```

在 ScrollView 中与 Espresso 交互需要滚动到每个视图，这不是很可靠。为了使测试更简单，咖啡师在与任何视图交互之前都会自动滚动，并且只在需要的时候才会这样做。

Expresso 也不能与 [NestedScrollView](https://stackoverflow.com/questions/35272953/espresso-scrolling-not-working-when-nestedscrollview-or-recyclerview-is-in-coor) 一起使用。

当与 ViewPager 交互时，相同的视图可能在多个片段上。咖啡师只与展示的东西互动，所以可以避免 AmbiguousViewMatcherException。

# 处理古怪的测试

古怪的测试是一场噩梦。从测试环境的不同到视图没有在正确的时间出现，测试变得不可靠的原因有很多。咖啡师提供了一些注释来处理这个问题。

测试需要确定性，但当一切都失败时，咖啡师会帮忙。注释允许定义测试运行的次数，如果通过了一次，就认为测试成功。

可以使用 **@Repeat** 注释，以便只有当所有执行都通过时，测试才通过。这对于测试的额外验证或者甚至是局部测试一个在持续集成中失败的不可靠的测试是有用的。

```
// Use a RuleChain to wrap ActivityTestRule with a FlakyTestRule
private ActivityTestRule<FlakyActivity> activityRule = new ActivityTestRule<>(FlakyActivity.class);
private FlakyTestRule flakyRule = new FlakyTestRule();

@Rule
public RuleChain chain = RuleChain.outerRule(flakyRule)
    .around(activityRule);

// Use @AllowFlaky to let flaky tests pass if they pass any time.
@Test
@AllowFlaky(attempts = 5)
public void someFlakyTest() throws Exception {
  // ...
}

// Use @Repeat to avoid flaky tests from passing if any repetition fails.
@Test
@Repeat(times = 5)
public void someImportantTest() throws Exception {
  // ...
}
```

总之，Barista 似乎是增加可读性和简化测试的一个很好的选择。因为它是建立在 Espresso 之上的，所以完全兼容，如果需要复杂的用例，Espresso 仍然可以使用。咖啡师没有增加复杂性，因为它只是由直接调用 Espresso 的静态函数组成。用起来真的没什么风险。

有关更多信息:

[](https://github.com/AdevintaSpain/Barista) [## GitHub-AdevintaSpain/咖啡师:提供优质浓缩咖啡的人

### Barista 使得开发 UI 测试更快、更容易、更可预测。建立在浓缩咖啡之上，它提供了一个简单的…

github.com](https://github.com/AdevintaSpain/Barista) [](https://dev.to/adevintaspain/making-android-ui-testing-enjoyable-3a1n) [## 让 Android UI 测试变得愉快

### Android 中的 UI 测试一直备受争议，原因有很多。测试很慢，因为它们必须在模拟器中运行…

开发到](https://dev.to/adevintaspain/making-android-ui-testing-enjoyable-3a1n)