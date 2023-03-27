# 如何滚动至 ID 并将其置于屏幕中央

> 原文：<https://itnext.io/how-scroll-to-id-and-center-it-to-the-screen-553927c789c6?source=collection_archive---------0----------------------->

![](img/65df106b13c651d18dc064bed2753273.png)

费伦茨·阿尔马西在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

我正在开发网球应用程序 [Sliceer](http://sliceer.com) ，它具有球场预订功能，我想让管理员用户能够选择不同类型的预订，如预订、培训、固定预订、公司预订等。这些选项中的每一个都有一些细微的差别。

我们的想法是，当管理员点击现有预订时，我们应该打开面板，进入点击的预订类型(即固定)，让用户知道他们选择了正确的选项，并清楚地知道该选项已被选中。这个想法非常简单，使用普通 javascript 中的 scrollIntoView 来滚动到正确的选项。这样做效果很好，但是你必须手动滚动到正确的位置。让我在下面的例子中解释一下。

如你所见，你不希望用户有这样的体验。如果选项不在屏幕上，就像上面的例子一样，不清楚选择了哪个选项。所以为了解决这个问题，我使用了 ID 和 scrollIntoView()的小技巧。让我们看看如何解决这个问题。

```
<div id="Reservation">Rezervacija</div>
<div id="Training">Trening</div>
<div id="Rest">Ostalo</div>
<div id="Company">Kompanija</div>
<div id="Fix">Fiksno</div>
<div id="Tournament" >Turnir</div>
<div id="League" >Liga</div>
```

每个选项现在都有关联的 id，我们可以用它来滚动到所需的位置(在本例中是 div)。当用户点击特定的 div 时，这个 JS 代码就会被执行。如果你正在使用 Vue，你可以使用 click 事件，当它被触发时，应该执行上面的 JS。

```
*// id has to be one that is defined above: 'Reservation' or 
// 'Training' etc.**document.getElementById('id').scrollIntoView({inline: 'center'});*
```

神奇之处在于对象所提供的功能:

```
*scrollIntoView({inline: 'center'})*
```

简单吧？你只需要知道这些。顺便说一下，几乎没有其他选项，如“行为”、“内联”和“阻止”，这里的文章更深入一点。

这是我想要达到的最终结果:

我希望这有助于如果你需要类似的东西，也可以是垂直滚动！