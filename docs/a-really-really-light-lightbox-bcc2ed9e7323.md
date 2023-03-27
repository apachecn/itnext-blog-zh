# 一个非常非常轻的灯箱

> 原文：<https://itnext.io/a-really-really-light-lightbox-bcc2ed9e7323?source=collection_archive---------10----------------------->

从我的 UX 写作中抽出一点时间来分享一些我刚刚创作的东西。我不断调整我的作品，在某个时候，我意识到是时候为我展示的图像实现一个灯箱了。buuuuuttttt…我不想要插件或任何太重的东西。只是一些简单的东西，使图像出现在一个模态，比他们在实际页面上稍微大一点。

所以…我自己用(喘息！)Jquery。但是，如果您正在寻找一些简单且非常容易实现的东西，我将向您介绍非常非常轻的 Lightbox:

# Jquery 代码:

*<脚本>*

$(文档)。ready(function() {
$('img ')。click(function(){
var modal = $('。模态’)；
modal.attr('src '，$(此)。attr(' src ')；
modal.css('display '，' block ')；
$('。背景)。css('display '，' block ')；
$(’。关闭)。css('display '，' block ')；
})；

$('.关闭)。单击(function() {
$('。模态)。css('显示'，'无')；
$(’。背景)。css('显示'，'无')；
$(本)。css('显示'，'无')；
})；
})；

*</脚本>*

# CSS:

*<风格>*

。背景{
位置:绝对；
宽度:100%；
身高:100%；
背景:# 000；
不透明度:0.5；
z 指数:1；
显示:无；}

。关闭{
位置:固定；
右:10%；
top:10%；
字体大小:80px
z 指数:3；
颜色:# fff
显示:无；
}

。modal {
显示:无；
位置:固定；
top:10px；
宽度:33%；
左:33%；
z 指数:2；
}
*</风格>*

# HTML:

(在你的标签附近的某个地方添加一个背景

，作为弹出窗口后面的黑色背景。)

![](””) *(你要在模态中展开的图像)*

 *(实际模态)*

X(关闭按钮)

你可以在我的网站[www.Mtraindesign.com](http://mtraindesign.com/)看到它的实现

# 编码快乐！！！