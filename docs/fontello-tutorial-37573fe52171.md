# 丰泰罗教程

> 原文：<https://itnext.io/fontello-tutorial-37573fe52171?source=collection_archive---------0----------------------->

![](img/71b82602be87b439951f2708e6e03441.png)

## 如何在您的项目中使用流行的图标字体生成器 Fontello 和 Less 创建自己的图标字体库。

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Ffontello-tutorial-37573fe52171)

为了让我们达成共识，我将总结一下使用网页字体显示图标的利弊。

## 赞成的意见

*   字体是矢量，因此在高分辨率屏幕上不会出现像素化或模糊，而如果图形是光栅图形，则需要放大。
*   浏览器支持是你所需要的
*   一旦系统就位，使用它们就非常方便了。
*   可以说比发送图片更容易。
*   它们可以用 CSS 来控制，比如大小、颜色、阴影等等。

## 骗局

*   图标将是单色的。然而，对于多色来说，有一些花哨的技术，无论如何，现代趋势和 HIGs(人机界面指南)建议使用单色。

如果您需要有关图标字体及其使用的更多信息，以下是一些推荐的链接:

*   [图标字体用法的 HTML】](http://css-tricks.com/html-for-icon-font-usage/)
*   (为什么)[图标字体很牛逼](http://css-tricks.com/examples/IconFont/)

# 这些改进有多大意义？

使用 Fontello 可以显著降低页面重量。如果你使用的是 Font Awesome 版本，浏览器加载的字体文件大小约为 75kb。然而，用 Fontello 创建的字体文件只有 10kb 大小。当然，增益可能会根据需要加载的图标数量而有所不同。我加载了八个不同的图标，增益大约是 70kb。在某些情况下，网站可能会加载两种不同的字体文件来加载天气和社交媒体图标。如果他们不必使用所有的天气和社交图标，他们可以使用 Fontello 将字体合并到一个文件中。除了减轻页面重量，他们还将减少浏览器发出的请求数量。

除了性能的提高，这项服务还允许你给你的图标一个统一的名字。例如，字体牛逼前缀`fa-`到它所有的图标。其他图标字体可能也是如此。如果你使用多种图标字体，你将不得不跟踪不同的前缀。使用 Fontello，您可以为所有图标指定一个自己选择的前缀。

如果您只想使用 Fontello 上内置的字体图标，创建自定义图标字体的过程非常简单。这只需要 10 到 15 分钟，收获是值得的。

# 好吧，我被说服了，我该怎么做？

1.  导航到 fontello.com
2.  从列表中选择所需的字体图标，并将自定义 svg 图标或 svg 字体拖到自定义图标框中。
3.  命名你喜欢的字体“fontello”就行了。从右上角下载网页字体到你的本地机器。
4.  在你的“src/assets”目录而不是“build”目录下创建一个名为“font-icons/css”的文件夹
5.  将“fontello”文件复制到“font-icons/css”目录中，这是我的:

```
[@font](http://twitter.com/font)-face {
  font-family: 'fontello';
  src: url('/assets/font-icons/fonts/fontello.eot?65114073');
  src: url('/assets/font-icons/fonts/fontello.eot?65114073#iefix') format('embedded-opentype'),
       url('/assets/font-icons/fonts/fontello.woff2?65114073') format('woff2'),
       url('/assets/font-icons/fonts/fontello.woff?65114073') format('woff'),
       url('/assets/font-icons/fonts/fontello.ttf?65114073') format('truetype'),
       url('/assets/font-icons/fonts/fontello.svg?65114073#fontello') format('svg');
  font-weight: normal;
  font-style: normal;
}
/* Chrome hack: SVG is rendered more smooth in Windozze. 100% magic, uncomment if you need it. */
/* Note, that will break hinting! In other OS-es font will be not as sharp as it could be */
/*
[@media](http://twitter.com/media) screen and (-webkit-min-device-pixel-ratio:0) {
  [@font](http://twitter.com/font)-face {
    font-family: 'fontello';
    src: url('../font/fontello.svg?65114073#fontello') format('svg');
  }
}
*/
[class^="icon-"]:before, [class*=" icon-"]:before {
  font-family: "fontello";
  font-style: normal;
  font-weight: normal;
  speak: none;
  display: inline-block;
  text-decoration: inherit;
  text-align: center;
  font-variant: normal;
  text-transform: none;
  line-height: 1em;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.icon-car:before { content: '\e800'; } /* '' */
.icon-filter:before { content: '\e801'; } /* '' */
.icon-tick:before { content: '\e802'; } /* '' */
.icon-map:before { content: '\e803'; } /* '' */
.icon-map-o:before { content: '\e804'; } /* '' */
.icon-tickets:before { content: '\e805'; } /* '' */
.icon-date-time:before { content: '\e806'; } /* '' */
.icon-guests:before { content: '\e807'; } /* '' */
.icon-cart:before { content: '\e808'; } /* '' */
.icon-calendar:before { content: '\e809'; } /* '' */
.icon-close:before { content: '\e80a'; } /* '' */
.icon-cards:before { content: '\e80b'; } /* '' */
.icon-ok:before { content: '\e80c'; } /* '' */
.icon-arrow-right:before { content: '\e80d'; } /* '' */
.icon-my-booking:before { content: '\e80e'; } /* '' */
.icon-reference:before { content: '\e80f'; } /* '' */
.icon-thin-arrow-left:before { content: '\e810'; } /* '' */
.icon-thin-arrow-right:before { content: '\e811'; } /* '' */
.icon-arrow-left:before { content: '\e812'; } /* '' */
.icon-check-in:before { content: '\e813'; } /* '' */
.icon-faq:before { content: '\e814'; } /* '' */
.icon-flight-extras:before { content: '\e815'; } /* '' */
.icon-home:before { content: '\e816'; } /* '' */
.icon-hotel:before { content: '\e817'; } /* '' */
.icon-log-out:before { content: '\e818'; } /* '' */
.icon-thin-arrow-down:before { content: '\e819'; } /* '' */
.icon-parking:before { content: '\e81a'; } /* '' */
.icon-seats:before { content: '\e81b'; } /* '' */
.icon-attractions:before { content: '\e81c'; } /* '' */
.icon-mail:before { content: '\e81d'; } /* '' */
.icon-transfer:before { content: '\e81e'; } /* '' */
.icon-email:before { content: '\e81f'; } /* '' */
.icon-attractions-outlines:before { content: '\e820'; } /* '' */
.icon-star:before { content: '\e821'; } /* '' */
.icon-star-empty:before { content: '\e822'; } /* '' */
.icon-user:before { content: '\e823'; } /* '' */
.icon-baggage-outlines:before { content: '\e824'; } /* '' */
.icon-sport:before { content: '\e825'; } /* '' */
.icon-users:before { content: '\e826'; } /* '' */
.icon-continue:before { content: '\e827'; } /* '' */
.icon-guest-outlines:before { content: '\e828'; } /* '' */
.icon-meals:before { content: '\e829'; } /* '' */
.icon-email-outlines:before { content: '\e82a'; } /* '' */
.icon-car-outlines:before { content: '\e82b'; } /* '' */
.icon-transfer-outlines:before { content: '\e82c'; } /* '' */
.icon-th-large:before { content: '\e82d'; } /* '' */
.icon-th-list:before { content: '\e82e'; } /* '' */
.icon-print:before { content: '\e82f'; } /* '' */
.icon-return-plane:before { content: '\e830'; } /* '' */
.icon-outbound-plane:before { content: '\e831'; } /* '' */
.icon-download:before { content: '\e832'; } /* '' */
.icon-ok-1:before { content: '\e833'; } /* '' */
.icon-cancel:before { content: '\e834'; } /* '' */
.icon-plus:before { content: '\e835'; } /* '' */
.icon-minus:before { content: '\e836'; } /* '' */
.icon-minus-circled:before { content: '\e837'; } /* '' */
.icon-info-circled:before { content: '\e838'; } /* '' */
.icon-home-1:before { content: '\e839'; } /* '' */
.icon-lock:before { content: '\e83a'; } /* '' */
.icon-tag:before { content: '\e83b'; } /* '' */
.icon-tags:before { content: '\e83c'; } /* '' */
.icon-pencil:before { content: '\e83d'; } /* '' */
.icon-edit:before { content: '\e83e'; } /* '' */
.icon-print-1:before { content: '\e83f'; } /* '' */
.icon-doc:before { content: '\e840'; } /* '' */
.icon-cog:before { content: '\e841'; } /* '' */
.icon-phone:before { content: '\e842'; } /* '' */
.icon-calendar-1:before { content: '\e843'; } /* '' */
.icon-login:before { content: '\e844'; } /* '' */
.icon-logout:before { content: '\e845'; } /* '' */
.icon-resize-small:before { content: '\e846'; } /* '' */
.icon-resize-vertical:before { content: '\e847'; } /* '' */
.icon-resize-horizontal:before { content: '\e848'; } /* '' */
.icon-down-dir:before { content: '\e849'; } /* '' */
.icon-up-dir:before { content: '\e84a'; } /* '' */
.icon-left-dir:before { content: '\e84b'; } /* '' */
.icon-right-dir:before { content: '\e84c'; } /* '' */
.icon-down-open:before { content: '\e84d'; } /* '' */
.icon-left-open:before { content: '\e84e'; } /* '' */
.icon-right-open:before { content: '\e84f'; } /* '' */
.icon-up-open:before { content: '\e850'; } /* '' */
.icon-flight:before { content: '\e851'; } /* '' */
.icon-globe:before { content: '\e852'; } /* '' */
.icon-briefcase:before { content: '\e853'; } /* '' */
.icon-credit-card:before { content: '\e854'; } /* '' */
.icon-floppy:before { content: '\e855'; } /* '' */
.icon-arrows-cw:before { content: '\e856'; } /* '' */
.icon-cw:before { content: '\e857'; } /* '' */
.icon-ccw:before { content: '\e858'; } /* '' */
.icon-attention:before { content: '\e859'; } /* '' */
.icon-attention-circled:before { content: '\e85a'; } /* '' */
.icon-ok-circled:before { content: '\e85b'; } /* '' */
.icon-ok-circled2:before { content: '\e85c'; } /* '' */
.icon-cancel-circled:before { content: '\e85d'; } /* '' */
.icon-cancel-circled2:before { content: '\e85e'; } /* '' */
.icon-guest:before { content: '\e85f'; } /* '' */
.icon-baggage:before { content: '\e860'; } /* '' */
.icon-infant_white:before { content: '\e861'; } /* '' */
.icon-shop-front:before { content: '\e862'; } /* '' */
.icon-trash-empty:before { content: '\e863'; } /* '' */
.icon-location:before { content: '\e864'; } /* '' */
.icon-landing:before { content: '\e865'; } /* '' */
.icon-search-1:before { content: '\e866'; } /* '' */
.icon-takeoff:before { content: '\e867'; } /* '' */
.icon-wallet:before { content: '\e868'; } /* '' */
.icon-move:before { content: '\f047'; } /* '' */
.icon-twitter:before { content: '\f099'; } /* '' */
.icon-facebook:before { content: '\f09a'; } /* '' */
.icon-certificate:before { content: '\f0a3'; } /* '' */
.icon-tasks:before { content: '\f0ae'; } /* '' */
.icon-filter-1:before { content: '\f0b0'; } /* '' */
.icon-docs:before { content: '\f0c5'; } /* '' */
.icon-pinterest-circled:before { content: '\f0d2'; } /* '' */
.icon-pinterest-squared:before { content: '\f0d3'; } /* '' */
.icon-gplus-squared:before { content: '\f0d4'; } /* '' */
.icon-gplus:before { content: '\f0d5'; } /* '' */
.icon-money:before { content: '\f0d6'; } /* '' */
.icon-sort-down:before { content: '\f0dd'; } /* '' */
.icon-sort-up:before { content: '\f0de'; } /* '' */
.icon-mail-alt:before { content: '\f0e0'; } /* '' */
.icon-linkedin:before { content: '\f0e1'; } /* '' */
.icon-exchange:before { content: '\f0ec'; } /* '' */
.icon-suitcase:before { content: '\f0f2'; } /* '' */
.icon-coffee:before { content: '\f0f4'; } /* '' */
.icon-food:before { content: '\f0f5'; } /* '' */
.icon-doc-text:before { content: '\f0f6'; } /* '' */
.icon-building:before { content: '\f0f7'; } /* '' */
.icon-hospital:before { content: '\f0f8'; } /* '' */
.icon-beer:before { content: '\f0fc'; } /* '' */
.icon-plus-squared:before { content: '\f0fe'; } /* '' */
.icon-angle-left:before { content: '\f104'; } /* '' */
.icon-angle-right:before { content: '\f105'; } /* '' */
.icon-angle-up:before { content: '\f106'; } /* '' */
.icon-angle-down:before { content: '\f107'; } /* '' */
.icon-circle-empty:before { content: '\f10c'; } /* '' */
.icon-circle:before { content: '\f111'; } /* '' */
.icon-direction:before { content: '\f124'; } /* '' */
.icon-info:before { content: '\f129'; } /* '' */
.icon-attention-alt:before { content: '\f12a'; } /* '' */
.icon-calendar-empty:before { content: '\f133'; } /* '' */
.icon-ticket:before { content: '\f145'; } /* '' */
.icon-minus-squared:before { content: '\f146'; } /* '' */
.icon-minus-squared-alt:before { content: '\f147'; } /* '' */
.icon-ok-squared:before { content: '\f14a'; } /* '' */
.icon-euro:before { content: '\f153'; } /* '' */
.icon-pound:before { content: '\f154'; } /* '' */
.icon-dollar:before { content: '\f155'; } /* '' */
.icon-doc-inv:before { content: '\f15b'; } /* '' */
.icon-doc-text-inv:before { content: '\f15c'; } /* '' */
.icon-sort-name-up:before { content: '\f15d'; } /* '' */
.icon-sort-name-down:before { content: '\f15e'; } /* '' */
.icon-youtube-squared:before { content: '\f166'; } /* '' */
.icon-youtube:before { content: '\f167'; } /* '' */
.icon-youtube-play:before { content: '\f16a'; } /* '' */
.icon-flickr:before { content: '\f16e'; } /* '' */
.icon-tumblr:before { content: '\f173'; } /* '' */
.icon-tumblr-squared:before { content: '\f174'; } /* '' */
.icon-male:before { content: '\f183'; } /* '' */
.icon-sun:before { content: '\f185'; } /* '' */
.icon-bug:before { content: '\f188'; } /* '' */
.icon-wheelchair:before { content: '\f193'; } /* '' */
.icon-vimeo-squared:before { content: '\f194'; } /* '' */
.icon-plus-squared-alt:before { content: '\f196'; } /* '' */
.icon-bank:before { content: '\f19c'; } /* '' */
.icon-google:before { content: '\f1a0'; } /* '' */
.icon-building-filled:before { content: '\f1ad'; } /* '' */
.icon-recycle:before { content: '\f1b8'; } /* '' */
.icon-cab:before { content: '\f1b9'; } /* '' */
.icon-taxi:before { content: '\f1ba'; } /* '' */
.icon-tree:before { content: '\f1bb'; } /* '' */
.icon-database:before { content: '\f1c0'; } /* '' */
.icon-lifebuoy:before { content: '\f1cd'; } /* '' */
.icon-circle-thin:before { content: '\f1db'; } /* '' */
.icon-sliders:before { content: '\f1de'; } /* '' */
.icon-soccer-ball:before { content: '\f1e3'; } /* '' */
.icon-binoculars:before { content: '\f1e5'; } /* '' */
.icon-plug:before { content: '\f1e6'; } /* '' */
.icon-newspaper:before { content: '\f1ea'; } /* '' */
.icon-cc-visa:before { content: '\f1f0'; } /* '' */
.icon-cc-mastercard:before { content: '\f1f1'; } /* '' */
.icon-cc-discover:before { content: '\f1f2'; } /* '' */
.icon-cc-amex:before { content: '\f1f3'; } /* '' */
.icon-cc-paypal:before { content: '\f1f4'; } /* '' */
.icon-cc-stripe:before { content: '\f1f5'; } /* '' */
.icon-trash:before { content: '\f1f8'; } /* '' */
.icon-bus:before { content: '\f207'; } /* '' */
.icon-forumbee:before { content: '\f211'; } /* '' */
.icon-facebook-official:before { content: '\f230'; } /* '' */
.icon-pinterest:before { content: '\f231'; } /* '' */
.icon-server:before { content: '\f233'; } /* '' */
.icon-bed:before { content: '\f236'; } /* '' */
.icon-train:before { content: '\f238'; } /* '' */
.icon-subway:before { content: '\f239'; } /* '' */
.icon-cc-jcb:before { content: '\f24b'; } /* '' */
.icon-hand-paper-o:before { content: '\f256'; } /* '' */
.icon-hand-pointer-o:before { content: '\f25a'; } /* '' */
.icon-trademark:before { content: '\f25c'; } /* '' */
.icon-tripadvisor:before { content: '\f262'; } /* '' */
.icon-television:before { content: '\f26c'; } /* '' */
.icon-calendar-plus-o:before { content: '\f271'; } /* '' */
.icon-calendar-minus-o:before { content: '\f272'; } /* '' */
.icon-calendar-times-o:before { content: '\f273'; } /* '' */
.icon-calendar-check-o:before { content: '\f274'; } /* '' */
.icon-map-pin:before { content: '\f276'; } /* '' */
.icon-map-o-1:before { content: '\f278'; } /* '' */
.icon-map-1:before { content: '\f279'; } /* '' */
.icon-credit-card-alt:before { content: '\f283'; } /* '' */
.icon-shopping-bag:before { content: '\f290'; } /* '' */
.icon-shopping-basket:before { content: '\f291'; } /* '' */
.icon-wpforms:before { content: '\f298'; } /* '' */
.icon-blind:before { content: '\f29d'; } /* '' */
.icon-asl-interpreting:before { content: '\f2a4'; } /* '' */
.icon-handshake-o:before { content: '\f2b5'; } /* '' */
.icon-envelope-open:before { content: '\f2b6'; } /* '' */
.icon-envelope-open-o:before { content: '\f2b7'; } /* '' */
.icon-address-card:before { content: '\f2bb'; } /* '' */
.icon-address-card-o:before { content: '\f2bc'; } /* '' */
.icon-user-circle:before { content: '\f2bd'; } /* '' */
.icon-user-circle-o:before { content: '\f2be'; } /* '' */
.icon-user-o:before { content: '\f2c0'; } /* '' */
.icon-id-badge:before { content: '\f2c1'; } /* '' */
.icon-id-card:before { content: '\f2c2'; } /* '' */
.icon-id-card-o:before { content: '\f2c3'; } /* '' */
.icon-shower:before { content: '\f2cc'; } /* '' */
.icon-bath:before { content: '\f2cd'; } /* '' */
.icon-snowflake-o:before { content: '\f2dc'; } /* '' */
.icon-twitter-squared:before { content: '\f304'; } /* '' */
.icon-facebook-squared:before { content: '\f308'; } /* '' */
.icon-linkedin-squared:before { content: '\f30c'; } /* '' */
```

6.我用。LESS 是 CSS 的向后兼容语言扩展。开发人员可以使用预处理程序将更多的功能编译成 CSS。现在，将这一行包含到' _base.less '或' _imports.less '中，或者无论您如何构造。少一点等级。使用这一行:

```
[@import](http://twitter.com/import) “/assets/font-icons/css/fontello”;
```

7.现在，在您的 html 中，只需在类名中添加一个图标:

```
<span class="icon-facebook-squared"></span>
```