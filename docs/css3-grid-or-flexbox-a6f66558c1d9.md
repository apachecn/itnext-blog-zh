# Css3 网格或 Flexbox

> 原文：<https://itnext.io/css3-grid-or-flexbox-a6f66558c1d9?source=collection_archive---------3----------------------->

![](img/601981801cff580c7ece5ad0f2d1217c.png)

## 不共戴天的敌人还是不可能的朋友？跟我来一场 CSS 时代的冒险

我最初对 CSS 的新布局感到失望是因为我没有意识到这个系统与我试图定位的 HTML 是如何交织在一起的。公平地说，我已经习惯了过去的世界，在 CSS3 之前，在那里`float` ing、`position: relative`和`flexbox`风靡一时，很少强调特定`div`中`divs`的组织或数量，只要你在那里扔几个`float: clears`或`display: block`就行了。学习如何使用网格的第二个困难部分是认识到网格和 flexbox 可以很好地协同工作。是的，我说了。每篇让 grid 和 flexbox 对立的文章都是一个大谎言。

我导航到我的谷歌搜索结果的第二页，才发现你并没有被限制使用其中一个。背景:在最近的一次代码挑战中，我得到了一些设计规范和图片资源，并被要求用原始 CSS 复制一个网站的布局。让我们看看导航标签的 CSS 和它对应的 HTML:

```
#cssnav {  
   display: grid;
   grid-template-rows: 69px;  
   align-items:center;  
   grid-template-columns: 10px auto auto 10px;  

   border-bottom: 1px solid seashell;  
   position: fixed; width: 100%;  
   background: black;  
   opacity: 0.9;
}# html<!-- Navbar Section -->      
   <nav>          
      <img src="[http://assets.espn.go.com/i/golf/pga11/daly_288.png](http://assets.espn.go.com/i/golf/pga11/daly_288.png)"    id="logo" alt="">          
      <ul>            
          <li>
              <a href='#'>Fantastic Mullets in Golf</a>
          </li>            
          <li>
              <a href='#'>John Daly: The Legend</a>
          </li>            
          <li>
              <a href='#'>Login</a>
          </li>          
      </ul>      
   </nav>
```

为了创建一个网格，我首先将`nav`的`display`属性赋给`grid.`，规范要求我的高度正好是 69px，所以下一行是`grid-template-rows: 69px`。规范还要求我在 navbar 徽标和右边的 navbar 链接之间留有 10px 的空间。为了创建这个空间，我将空间定义为具有四列:`grid-template-columns: 10px auto auto 10px`，有效地创建了具有相应宽度的四列:第一列为 10px，第二列为 auto，第三列为第二个 auto，第四列即最后一列为第二个 10px。

现在，我需要告诉特定的 HTML 标签，它们应该位于这些列中的什么位置。我希望我的徽标从第二列的开始(或`start`)开始，并跨越到**(不包括)**第三列，同时我希望我的无序列表位于第三列的`end`位置，并从第三列跨越到**(不包括)**第四列:

```
#css #logo styling
nav #logo {  
   height: 50px;  
   justify-self: start;  
   grid-column-start: 2;  
   grid-column-end: 3;
}#unordered list styling
nav ul {  
   list-style-type: none;  
   justify-self: end;  
   grid-column-start: 3;  
   grid-column-end: 4;
}
```

通过指定我的 logo 具有属性`justify-self: start`,我将它定位在第二列的开始，紧靠我需要的左边的 10px 列。对于导航条链接，我做了相反的事情:`justify-self: end`。

在意识到一个网格如何处理它的列以及如何分配特定的 HTML 标签来跨越特定的列之后，我要解决的下一个挑战是尝试将图像放在网格的中心。**事实证明，将网格和** `**flex**` **ed dives 结合使用效果很好。我总共有 5 张图片，我想让这些图片水平居中，并确保它们周围有相同的边距。包含前 3 张图片的 div 的最大宽度必须是 960 像素。我可以通过一个简单的`display: flex`和一个伴随的`justify-content: center`来实现图像的居中，这有效地将所有项目打包在 flexbox div 的中心周围:**

```
#css .card-wrapper {
  display: grid;
  grid-template-columns: auto 960px auto;
  grid-auto-rows: auto auto;
}.card-wrapper-2 {
  grid-column-start: 2;
  grid-column-end: 3;
  display: flex;
  justify-content: center;
}.card {
  text-align: center;
  margin: 0 10px;
}.card h4{
  margin-top: 10px;
}.card img {
  height: 200px;
  width: 300px;
}#html <!-- Card section -->
      <div class="card-wrapper">
        <div class="card-wrapper-2">
          <div class="card">
            <img src="" alt="">
            <h4>John Daly</h4>
          </div>
          <div class="card">
            <img src="" alt="">
            <h4>Arnold Palmer</h4>
          </div>
          <div class="card">
            <img src="" alt="">
            <h4>Dustin Johnson</h4>
          </div>
        </div>
        <div class="card-wrapper-2">
          <div class="card">
            <img src="" alt="">
            <h4>Rory McElroy</h4>
          </div>
          <div class="card">
            <img src="" alt="">
            <h4>Bubba Watson</h4>
          </div>
        </div>
      </div>
```

直到你意识到 1)你可以定义一个网格，而不是像上面我定义的网格那样只限于一维(多列和一行),而是二维(多列和多行),以及 2)网格与 flexbox 配合得非常好，你才最终开始理解 CSS 网格的全部威力。下次见！**(资源:** [**CSS 网格**](https://css-tricks.com/snippets/css/complete-guide-grid/) **和**[**Flexbox**](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)**)上的大文档。**

*非常感谢阅读！我总是在我的博客和项目上寻找反馈。不要害怕留下评论或给我发消息。也在向软件开发职业过渡的旅程中？我很乐意连接上*[*Linkedin*](https://www.linkedin.com/in/bmcilhenny/)*。*

喜欢我的写作吗？查看我的其他文章👇在下面。

[](https://medium.com/@bmcilhen/sql-alchemy-pythons-activerecord-accb8645b500) [## SQL 炼金术:Python 的 ActiveRecord

### 巨人肩膀上的思考随笔

medium.com](https://medium.com/@bmcilhen/sql-alchemy-pythons-activerecord-accb8645b500) [](/so-you-want-to-host-your-single-age-react-app-on-github-pages-a826ab01e48) [## 所以你想在 GitHub 页面上托管你的单页 React 应用？

### 再想想！不…我只是在玩，这是可能的，但肯定需要一些配置来获得你的路线…

itnext.io](/so-you-want-to-host-your-single-age-react-app-on-github-pages-a826ab01e48) [](https://medium.com/@bmcilhen/connecting-amazons-s3-to-your-heroku-hosted-rails-app-or-d6290f622162) [## 将 AWS 的 S3 连接到 Heroku 托管的 Rails 应用程序，或者…

### 1)不要浪费 7 天时间让你的 AWS 反复被黑，2)不要重新格式化你的 production.rb 文件 100…

medium.com](https://medium.com/@bmcilhen/connecting-amazons-s3-to-your-heroku-hosted-rails-app-or-d6290f622162) [](https://medium.com/@bmcilhen/activerecord-michael-jackson-and-magic-1d211fa40efd) [## ActiveRecord、Dirty Attributes 和迈克尔杰克逊

### 作为一名视觉学习者，我总是在寻找更好的方法来消化我所学的东西。为了一些无法解释的…

medium.com](https://medium.com/@bmcilhen/activerecord-michael-jackson-and-magic-1d211fa40efd) [](https://medium.com/@bmcilhen/recreating-twitters-like-effect-with-with-mo-js-86af491c25a1) [## 用 mo.js 重现 Twitter 的“赞”效果

### 当我在 10 月份开始使用 Flatiron 时，我从来不认为自己是后端战士。如今，我倾向于…

medium.com](https://medium.com/@bmcilhen/recreating-twitters-like-effect-with-with-mo-js-86af491c25a1) [](https://medium.com/@bmcilhen/how-to-get-ruby-up-and-running-with-twilio-fast-twilio-trial-version-7c327632eef9) [## 如何快速启动并运行你的 Ruby 应用——Twilio 试用版

### Flatiron 为期四个月的强化网络开发项目就像一个道场:每两周有一次评估…

medium.com](https://medium.com/@bmcilhen/how-to-get-ruby-up-and-running-with-twilio-fast-twilio-trial-version-7c327632eef9)