# 面向对象编程是我们生活的世界

> 原文：<https://itnext.io/object-oriented-programming-is-the-world-we-live-in-cb8e758c99fe?source=collection_archive---------6----------------------->

![](img/97ecd8be9b10b0bc40387a433c78a974.png)

在我技术生涯的早期，我是一名 PL/SQL 程序员。我们为许多中小型公司开发了解决方案。PL/SQL 是一种基于过程的编程语言。一切都发生在程序中。有人邀请我去 [Java](https://www.java.com/) 开发，我欣然接受了这个机会。我以为用面向对象的编程语言开发会很容易。

在几次失误之后，我开始意识到这是多么的不同和困难。这就是为什么我想花点时间回顾一下面向对象开发中的一些基本原则。这将有助于在你继续前进时消除一些困惑。

# 面向对象编程

物品是我们生活的世界的一部分。我坐在沙发上，在电脑上写这篇文章。每一个都是一个物体。换句话说，用这些结构编程有助于我们思考和理解代码。对象更具一般性和理论性。

面向对象编程允许我们跟踪状态和行为。例如，我们的电脑可能开着也可能关着。代码的模块化也是一个好处。一起存储关于对象的信息。最重要的是，它还允许我们隐藏信息。我们的心智模式可以专注于其他事情。我们可以为类似的对象重用代码。然后我们可以替换类似的对象，轻松调试代码。

# 班级

类被称为 Java 的“蓝图”。一旦我们有了它们，我们就可以创建对象的许多副本。我们有来自 Java 的字符串类。使用类构造函数，我们可以创建许多副本。

```
String myString1 = new String("One Class");
	                String myString2 = new String("Two Class");
```

在这个例子中，我们分享了如何创建一些字符串对象。

```
public class Shoe {
	                int size = 1;
	                boolean smelly = false;
```

这里有一个简单的类叫做 Shoe。它有两个实例变量来跟踪它的大小和它是否有气味。

```
Shoe newShoe = new Shoe();
	                Shoe oldShoe = new Shoe();
```

我们可以很容易地创建一些鞋子对象。类或“蓝图”允许我们想做多少就做多少。

# 遗产

我们世界中的许多物体都是相关的。我们总是把东西放在一起。例如，我们已经开始了关于鞋子的对话。除了鞋子对象，我们还可以创建一个触发器对象。这将有助于我们为即将到来的海滩度假做准备。

```
class FlipFlop {
	                int size = 1;
	                boolean sandy = false;
                   }
```

这两个都是鞋，所以我们可以创建一个父对象。在我们的鞋类对象中，我们也可以有通用的可变尺寸。

```
class Footwear {
	                int size = 1;
                   }
```

然后我们可以更新我们的 Shoe 和 FlipFlop 类来反映父类。

```
class FlipFlop extends Footwear {
                   public class Shoe extends Footwear {
```

[继承](https://myitcareercoach.com/are-you-my-mother/)帮助我们将对象中相似的品质分组，以帮助开发。我们不需要从每一项开始。我们也可以重用功能。

# 连接

要驾驶你的车，你有几个接口点。例如，你可以用方向盘来控制方向。为了加速，我们使用加速器，用刹车减速。这些保护我们免受错综复杂的电机和制动系统。作为开发人员，我们也需要保护人们远离界面背后的东西。

```
public interface Speed {
	                void speedUp();
	                void slowDown();
                   }
```

这里我们有一个界面来控制速度。我们用两种方法定义接口来加速或减速。

```
public interface Steering {
	                void turnLeft();
	                void turnRight();
                   }
```

为了控制方向，我们也有一个接口。它允许我们向左转和向右转。

在幕后，我们可以有一辆车来实现这些，并有能力做每一件事。

```
public class Car implements Speed, Steering { @Override
	                public void turnLeft() {
	                   System.out.println("Turn left");
	                } @Override
	                public void turnRight() {
	                   System.out.println("Turn right");
	                } @Override
	                public void speedUp() {
	                   System.out.println("Speed up");
	                } @Override
	                public void slowDown() {
	                   System.out.println("Slow down");
	                } }
```

正如你在这个简单的类中看到的，我们正在实现接口并提供代码来完成这项工作。

# 包裹

乔治·福尔曼是著名的重量级拳击手。奇怪的是，他给他的五个儿子取名为小乔治。你可以想象，当你在一个满是乔治的房间里说话时，这可能很难。类似地，如果不是包，Java 代码也会有类似的问题。包就像文件夹一样，帮助我们给代码一个唯一的名字。

让我们看看 Java 中的[字符串](https://docs.oracle.com/javase/9/docs/api/java/lang/String.html)类。它在 java.lang 包中。Java 允许您访问 java.lang 包中的所有内容，而无需导入任何内容。换句话说，String 类的全名是 java.lang.String。

因此，如果我为一辆特斯拉汽车创建一个特殊的类，我们可能会把它放在一个名为 com.tesla 的包中。因此，如果我们使用 car 对象，它将被称为 com.tesla.Car。因此，我们将使用全名导入它。我们可以正确地引用它们，而不是为本田、丰田和特斯拉拥有多个不同的汽车对象。

**为这个鼓掌，跟着我上灵媒！**